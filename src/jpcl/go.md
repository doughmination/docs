---
order: 840
label: Go
icon: ../media/go.svg
---

# Go

```bash
go get github.com/jpcl-lang/jpcl-go
```

No runtime dependencies — standard library only. **Go 1.27+**.

:icon-lockup-github: [*jpcl-lang/jpcl-go*](https://github.com/jpcl-lang/jpcl-go)

```go
package main

import (
	"fmt"

	"github.com/jpcl-lang/jpcl-go"
)

func main() {
	data, err := jpcl.Load("data/servers.jp", jpcl.ParseOptions{})
	if err != nil {
		panic(err)
	}
	fmt.Println(data.Keys()) // [SERVER_ID SERVER_ID_2]
}
```

The format and the error messages are identical to the Python and TypeScript implementations. The *API* leans on Go idioms rather than mirroring their syntax, and this page is mostly about where those idioms show through.

---

## Values

| `.jp` type | Go type |
|---|---|
| Object / section | `*jpcl.Object` |
| Array | `[]any` |
| String | `string` |
| Integer | `int64` |
| Float | `float64` |
| Boolean | `bool` |
| Null / empty value | `nil` |

Values are carried around as `any`, so reading a nested value means a type assertion at each step. That is Go, not JPCL — see [*Getting at values*](#getting-at-values) for the tidy way to do it.

### `Object` is an ordered map

Go's built-in `map` does not preserve insertion order, and the writer needs to reproduce keys in the order they were written. `jpcl.Object` is a small ordered-map type the package defines for that reason:

```go
obj := jpcl.NewObject()
obj.Set("prefix", "!")
obj.Set("id", int64(1234567890))

value, ok := obj.Get("prefix")   // "!", true
obj.Len()                        // 2
for _, key := range obj.Keys() { // "prefix", then "id" -- insertion order
	fmt.Println(key)
}
```

There is no `obj["key"]` syntax, because Go has no operator overloading. There is also no `map[string]any` support in the writer — building a document means calling `Set`, not writing a map literal, so the ordering guarantee always holds.

### A note on integer size

Integers are `int64`, not arbitrary precision like Python's `int` and not `bigint`-backed like JavaScript's. A number larger than roughly 9.2×10^18 loses precision by falling through to `float64`.

That is fine for Discord snowflakes, which top out well below it, but it is a real limit to know about if you need bigger integers. [*Integer width*](/jpcl/spec/#integer-width) compares all three.

---

## Reading

```go
data, err := jpcl.Load("data/servers.jp", jpcl.ParseOptions{})   // from a path
data, err := jpcl.Parse(text, jpcl.ParseOptions{})               // from a string
```

### `ParseOptions`

```go
type ParseOptions struct {
	Filename      string        // used in error messages
	DuplicateKeys DuplicateKeys // DuplicateError (default), DuplicateFirst, DuplicateLast
}
```

`DuplicateError` is the zero value, so a bare `jpcl.ParseOptions{}` gives you the strict default — a repeated key is an error rather than a silent overwrite. See [*Duplicate keys*](/jpcl/syntax/#duplicate-keys).

### Getting at values

Reaching into a parsed document by hand is verbose:

```go
section, _ := data.Get("SERVER_ID")
config, _ := section.(*jpcl.Object).Get("config")
users, _ := config.(*jpcl.Object).Get("disabled_users")
list := users.([]any)
```

Each of those assertions panics if the shape isn't what you assumed. For anything beyond one level, use `JPConfig.GetPath` instead — it walks the path safely and returns your default at any depth:

```go
cfg, _ := jpcl.LoadJPConfig("data/servers.jp", false, jpcl.ConfigOptions{})
users := cfg.GetPath("SERVER_ID.config.disabled_users", []any{}).([]any)
```

One assertion instead of four, and no panic on a missing section.

### Load a whole folder

```go
config, err := jpcl.LoadDir("data", "", false, jpcl.ParseOptions{})
// map[string]*jpcl.Object{"servers": ..., "roles": ...}

guilds, err := jpcl.LoadDir("data/guilds", "", false, jpcl.ParseOptions{})
everything, err := jpcl.LoadDir("data", "", true, jpcl.ParseOptions{}) // recursive
```

The arguments are directory, glob pattern (`""` means `*.jp`), recursive, and parse options.

Each file becomes one entry, named after the file without its `.jp` suffix. The returned value is a plain Go `map`, since it is a lookup table rather than a document you write back out — unlike `*jpcl.Object`, its iteration order is not meaningful.

---

## Writing

```go
text, err := jpcl.Dumps(obj, jpcl.DefaultStringifyOptions())
err = jpcl.Dump(obj, "data/servers.jp", true, jpcl.DefaultStringifyOptions()) // atomic
```

### `StringifyOptions`

```go
type StringifyOptions struct {
	Indent      int                 // spaces per level
	Width       int                 // column budget for inline arrays
	SortKeys    bool                // sort keys alphabetically instead of insertion order
	EnsureASCII bool                // escape non-ASCII characters
	Default     func(value any) any // convert otherwise-unsupported types
}
```

!!!danger Always start from `DefaultStringifyOptions()`
Go structs have no default field values, so a bare `StringifyOptions{}` means `Indent: 0, Width: 0` — indistinguishable from an explicit choice, and it produces unindented output with every array broken across lines.

```go
opts := jpcl.DefaultStringifyOptions()  // {Indent: 2, Width: 88}
opts.SortKeys = true
text, err := jpcl.Dumps(obj, opts)
```

This is the single most common mistake with the Go package.
!!!

### The `Default` hook

For types `.jp` has no representation for — `time.Time`, a custom struct, anything else:

```go
opts := jpcl.DefaultStringifyOptions()
opts.Default = func(v any) any {
	if t, ok := v.(time.Time); ok {
		return t.Format(time.RFC3339)
	}
	return fmt.Sprint(v)
}
```

It must return a *different* type from the one it was handed, or you get `default() returned an unconverted ...`.

### Atomic writes

`Dump`'s third argument is `atomic`. With `true`, the file goes to a temporary neighbour and is renamed into place, so a crash or a concurrent reader never sees half a config. Missing parent directories are created.

---

## Editing a config in place: `JPConfig`

`JPConfig` wraps an `*Object` and remembers the file it came from, adding dotted-path access and atomic saving. Its backing store is a public field, `Data`, since Go has no subscript operator to stand in for Python's `cfg["key"]`:

```go
cfg, err := jpcl.LoadJPConfig("data/servers.jp", true, jpcl.ConfigOptions{
	Write: jpcl.DefaultStringifyOptions(),
})
// missingOK: true -- an empty config bound to the path if it doesn't exist yet

prefix, _ := cfg.Data.Get("SERVER_ID")                            // plain Object access
cfg.GetPath("SERVER_ID.config.disabled_users", []any{})           // never fails on a missing section
cfg.SetPath("SERVER_ID.config.disabled_users", []any{int64(9892)}) // creates missing sections
cfg.HasPath("SERVER_ID.prefix")
sec, _ := cfg.Section("NEW_SERVER", true)                          // create: true
sec.Set("prefix", "?")
cfg.Merge(other, true)  // deep merge
cfg.Save("")            // atomic, back to its own bound path
cfg.Reload()            // discard in-memory changes
cfg.ToDict()            // deep copy as a plain *Object
```

### Constructors

```go
jpcl.LoadJPConfig(path, missingOK, opts)   // read a file
jpcl.LoadsJPConfig(text, path, opts)       // from a string, optionally bound to a path
jpcl.NewJPConfig(obj, opts)                // from data you already have
```

`missingOK: true` gives an empty config bound to the path, which is what a program that writes its config on first run wants:

```go
cfg, err := jpcl.LoadJPConfig("data/settings.jp", true, jpcl.ConfigOptions{
	Write: jpcl.DefaultStringifyOptions(),
})
if err != nil {
	return err
}
if !cfg.HasPath("bot.token") {
	cfg.SetPath("bot.token", "")
	if _, err := cfg.Save(""); err != nil {
		return err
	}
}
```

### Paths and sections

```go
cfg.GetPath("SERVER_ID.config.disabled_users", []any{})  // returns the default at any depth
err := cfg.SetPath("SERVER_ID.prefix", "!")              // creates missing levels
cfg.HasPath("SERVER_ID.prefix")
sec, err := cfg.Section("SERVER_ID", false)              // error if missing
sec, err := cfg.Section("NEW_SERVER", true)              // created empty if missing
```

`SetPath` returns an error only when a path segment holds something that isn't a section — `cannot descend into "a.b": it holds string, not a section`. `Section` returns an error if the name is missing without `create`, or holds a non-section.

### Saving

```go
path, err := cfg.Save("")             // back to the bound path
path, err := cfg.Save("backup.jp")    // somewhere else; the config rebinds to it
text, err := cfg.Dumps()              // just the text
cfg.FilePath()                        // (path, bound bool)
cfg.SetFilePath("data/other.jp")
```

`Save("")` on a config with no bound path is an error.

!!!info Formatting is fixed at construction
Unlike the Python and TypeScript versions, formatting options live in `ConfigOptions.Write` and are fixed when the config is built — `Save` and `Dumps` always use them, with no per-call override. Again, this avoids the zero-value ambiguity. For a one-off different format, call the package-level function directly:

```go
text, err := jpcl.Dumps(cfg.Data, customOpts)
```
!!!

---

## Errors

Every parse failure is a `*jpcl.DecodeError`, which points at the exact offending character:

```
data/servers.jp:2:8: expected ':' after key "prefix", found '"'
    prefix "!"
           ^
```

It carries `Message`, `Doc`, `Pos`, `Line`, `Col` and `Filename`, and implements `error`:

```go
data, err := jpcl.Load(path, jpcl.ParseOptions{})
var decErr *jpcl.DecodeError
if errors.As(err, &decErr) {
	fmt.Println(decErr.Line, decErr.Col)
}
```

Every write failure is a `*jpcl.EncodeError` — an unsupported type (nothing matches `nil`/`bool`/`string`/an integer type/`float32`/`float64`/`*jpcl.Object`/`[]any`), a circular reference, or binary data (`[]byte`, which has no `.jp` representation; convert it to a string first).

[*When it goes wrong*](/jpcl/errors) lists every message and what to do about each.

---

## Command line

```bash
go install github.com/jpcl-lang/jpcl-go/cmd/jpcl@latest

jpcl check data/*.jp                      # validate; non-zero exit on failure
jpcl fmt -w data/servers.jp               # reformat in place
jpcl get data/servers.jp SERVER_ID.prefix # read one value
jpcl to-json -o out.json data/servers.jp
jpcl from-json -o data/servers.jp out.json
```

`-` reads stdin in place of a filename.

!!!warning Flags before filenames
Flags must come **before** positional arguments — `jpcl fmt -w file.jp`, not `jpcl fmt file.jp -w`. Go's standard `flag` package stops looking for flags at the first non-flag argument.

This is the one place the Go CLI is a step behind the Python (`argparse`) and TypeScript (`util.parseArgs`) versions, which accept either order.
!!!

Full reference: [*Command line*](/jpcl/cli).

---

## Differences from the other implementations

| | |
|---|---|
| **`*Object`, not a map** | An ordered map type, because the writer's key ordering has to be reproducible and Go's `map` isn't ordered. Build documents with `Set`. |
| **`int64`, not arbitrary precision** | Anything past roughly 9.2×10^18 falls through to `float64`. |
| **Options structs, not keyword arguments** | And no default field values — always start from `DefaultStringifyOptions()`. |
| **`JPConfig` formatting is fixed at construction** | No per-call override on `Save`. |
| **CLI flags must precede filenames** | A `flag` package limitation. |

---

## Full API

```go
// Reading
jpcl.Load(path string, opts ParseOptions) (*Object, error)
jpcl.Parse(text string, opts ParseOptions) (*Object, error)
jpcl.LoadDir(dir, pattern string, recursive bool, opts ParseOptions) (map[string]*Object, error)

// Writing
jpcl.Dumps(obj *Object, opts StringifyOptions) (string, error)
jpcl.Dump(obj *Object, path string, atomic bool, opts StringifyOptions) error
jpcl.DefaultStringifyOptions() StringifyOptions

// Objects
jpcl.NewObject() *Object
(*Object) Get(key string) (any, bool)
(*Object) Set(key string, value any)
(*Object) Keys() []string
(*Object) Len() int

// Config
jpcl.NewJPConfig(data *Object, opts ConfigOptions) *JPConfig
jpcl.LoadJPConfig(path string, missingOK bool, opts ConfigOptions) (*JPConfig, error)
jpcl.LoadsJPConfig(text, path string, opts ConfigOptions) (*JPConfig, error)
(*JPConfig) GetPath, SetPath, HasPath, Section, Merge, ToDict, Dumps, Save, Reload
(*JPConfig) FilePath, SetFilePath
(*JPConfig) Data *Object

// Errors and constants
jpcl.DecodeError, jpcl.EncodeError
jpcl.DuplicateError, jpcl.DuplicateFirst, jpcl.DuplicateLast
jpcl.Suffix   // ".jp"
jpcl.MaxDepth // 200
jpcl.Version
```

---

## Next

>>> *[Worked examples](/jpcl/recipes)*
Complete projects using all of this.

>>> *[Command line](/jpcl/cli)*
The `jpcl` tool in full.

>>> *[When it goes wrong](/jpcl/errors)*
Every error message, explained.
>>>
