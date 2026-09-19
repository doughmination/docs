# When it goes wrong

Every message JPCL can produce, what it actually means, and how to fix it. The messages are the same in all three implementations, so this page applies whichever one you are using.

!!!secondary
One cosmetic difference: Go quotes names with double quotes (`expected ':' after key "prefix"`) where Python and JavaScript use single quotes (`'prefix'`). The messages are otherwise word-for-word identical, and this page uses the Python spelling.
!!!

---

## How to read an error

JPCL points at the exact character that upset it:

```
data/servers.jp:2:8: expected ':' after key 'prefix', found '"'
    prefix "!"
           ^
```

| Part | Meaning |
|---|---|
| `data/servers.jp` | The file. `<stdin>` if it came from a pipe. |
| `2:8` | Line 2, column 8. |
| `expected ':' after key 'prefix', found '"'` | What went wrong. |
| `prefix "!"` | The offending line, reproduced. |
| `^` | The exact character. |

**The caret is the whole diagnosis.** Look at the character it points at, then at the character before it, and the answer is almost always there. The line number is where JPCL *noticed*, which for an unclosed bracket is later than where you actually went wrong — but the message for those says where the bracket opened.

If you want to render the failure yourself, the error object carries `.line`, `.col`, `.pos`, `.filename` and the raw message. See your language's page for the field names.

---

## Parse errors

Raised when a file cannot be read. `JPDecodeError` in Python and JavaScript, `*jpcl.DecodeError` in Go.

### `expected ':' after key '...', found ...`

There is a key, but no colon after it.

```jp
# WRONG
prefix "!"

# WRONG -- JPCL uses ':', not '='
prefix = "!"
```

```jp
# RIGHT
prefix: "!"
```

### `expected a key, found ...`

Something appeared where a key was expected — usually a stray bracket or an opening brace with no key in front of it.

```jp
# WRONG -- no key
[SERVER_ID]
{x: 1}
```

If the message says `found end of file`, the document ended mid-entry — usually an unclosed `{` much further up that swallowed the rest of the file.

### `expected ',', a line break or ']' after value, found ...`

A value was read, then something that can't follow it. The `'}'` and "or a line break" variants are the same problem inside an object, or at the top level of a section.

```jp
# WRONG -- no comma between the two
roles: ["moderator" "admin"]
```

```jp
# RIGHT
roles: ["moderator", "admin"]
```

!!!warning A missing comma does not always error
Between two *unquoted* values, the missing comma is absorbed instead. `{rate: 30 burst: 5}` is valid, and gives you one key whose value is the string `"30 burst: 5"` — because an unquoted value runs until a comma, a bracket, a `#` or a line break. If a value comes back as unexpected text, look for a missing comma before you look anywhere else.
!!!

### `unterminated object: missing '}'`

A `{` was opened and never closed. The position points at **where the object started**, not at the end of the file.

```jp
# WRONG
limits: {
  rate: 30
```

### `unterminated array: missing ']'`

Same, for `[`.

```jp
# WRONG
roles: [111, 222
```

### `unterminated array: found '}' before ']'`

Your brackets are crossed — an array was closed with a brace. The position points at where the array **opened**.

```jp
# WRONG
roles: [111, 222,}
```

It usually means a bracket further up is the wrong *kind*, not that this one is. Count your `{` and `[` from the top of the section.

### `unterminated section header`

A `[` opened a header and no `]` closed it before the line ended.

```jp
# WRONG
[SERVER_ID
```

### `section header cannot be empty` / `section header segment cannot be empty`

`[]`, or a dotted header with a gap in it.

```jp
# WRONG
[]
[guild.]
[.limits]
[guild..limits]
```

```jp
# RIGHT
[guild.limits]
```

### `unexpected ... after section header`

Something other than a line break followed the `]`.

```jp
# WRONG
[SERVER_ID] prefix: "!"
```

```jp
# RIGHT
[SERVER_ID]
prefix: "!"
```

### `expected a line break before a section header`

A `[HEADER]` appeared on the same line as the entry before it. A header always starts its own line.

```jp
# WRONG
[SERVER_ID]
prefix: "!" [SERVER_ID_2]
```

### `duplicate key '...'`

The same key appears twice in the same object.

```jp
# WRONG
[SERVER_ID]
prefix: "!"
prefix: "?"
```

Decide which one you meant and delete the other. If you genuinely want the later value to win, load the file with the duplicate policy set to `last` — see [*Duplicate keys*](/jpcl/syntax/#duplicate-keys).

### `section '[...]' is defined twice`

The same section header appears twice. Merge the two by hand, or load with the `last` policy, which merges them for you.

### `cannot open section '[...]': ...`

A header tried to nest inside something that isn't an object.

```jp
# WRONG -- guild is a number, so [guild.limits] has nowhere to go
guild: 1

[guild.limits]
rate: 30
```

### `unterminated string literal`

A quote was opened and the file ended before it closed.

```jp
# WRONG -- and the file stops right there
name: "Clanker
```

Watch for a stray `"` inside a string — the second quote closes it early and the third opens a new one that never ends. If the string had a line break in it rather than running to the end of the file, you get the longer message below instead.

### `unterminated string literal (use \n for a line break)`

A quoted string ran into an actual line break.

```jp
# WRONG
message: "first line
second line"
```

```jp
# RIGHT
message: "first line\nsecond line"
```

```jp
# ALSO RIGHT -- a trailing backslash continues the literal
message: "first line \
second line"
```

### `invalid escape sequence '\x'`

A `\` followed by something JPCL doesn't recognise.

```jp
# WRONG
k: "a\qb"
```

To mean a literal backslash, double it: `"a\\qb"`. [*Escape sequences*](/jpcl/spec/#escape-sequences) lists everything that is valid after a backslash.

### `'\u...' escape needs 4 hex digits` / `'\U...' escape needs 8 hex digits`

A Unicode escape was cut short. `\u` takes exactly four hex digits, `\U` exactly eight — pad with leading zeros: `\U0001F600`.

The usual cause is not a Unicode escape at all, but a **Windows path**:

```jp
# WRONG -- \U is read as the start of an 8-digit escape
path: "C:\Users\notes"
```

```jp
# RIGHT -- double the backslashes
path: "C:\\Users\\notes"
```

```jp
# ALSO RIGHT -- forward slashes work on Windows too
path: "C:/Users/notes"
```

### `'\u...' is an unpaired surrogate` / `'\U...' is outside the Unicode range`

The escape names something that isn't a valid character. Half of a surrogate pair, or a code point above `U+10FFFF`. Write the character itself instead, or use the correct code point.

### `unterminated escape sequence`

The file ended immediately after a `\`.

### `nesting deeper than 200 levels`

`{` and `[` may nest 200 deep. If you have hit this legitimately, the structure wants flattening — dotted section headers are usually the answer. If you have hit it accidentally, you probably have a missing `}` somewhere and JPCL has been descending ever since.

---

## Write errors

Raised when your *program* hands JPCL something it cannot put in a file. `JPEncodeError` in Python and JavaScript, `*jpcl.EncodeError` in Go. These are bugs in the calling code rather than problems with a file.

### `object of type '...' is not serialisable to .jp; pass default= to convert it`

You tried to write a value JPCL has no representation for — a `datetime`, a `set`, a class instance, a function.

Supply a `default` hook that converts it:

+++ Python
```python
jpcl.dumps(data, default=str)

# or something more deliberate
def convert(value):
    if isinstance(value, datetime):
        return value.isoformat()
    raise TypeError(value)

jpcl.dumps(data, default=convert)
```
+++ JavaScript
```js
jpcl.dumps(data, {
  default: (v) => (v instanceof Date ? v.toISOString() : String(v)),
});
```
+++ Go
```go
opts := jpcl.DefaultStringifyOptions()
opts.Default = func(v any) any {
	if t, ok := v.(time.Time); ok {
		return t.Format(time.RFC3339)
	}
	return fmt.Sprint(v)
}
text, err := jpcl.Dumps(obj, opts)
```
+++

### `default() returned an unconverted '...'`

Your `default` hook handed back the same type it was given, which would loop forever. Return something JPCL can write — a string is nearly always the right answer.

### `binary data has no .jp representation; decode or encode it to a string first`

You passed `bytes` (Python), a `Buffer`/`Uint8Array` (JavaScript) or `[]byte` (Go). `.jp` is a text format. Decode it if it's text, or base64-encode it if it isn't.

### `keys must be strings (or ints), got '...'`

An object key was something else — a tuple, an object, `None`. Integer keys are fine and become strings on the way out, which is why `[1234567890]` sections round-trip.

### `circular reference detected`

Something in the structure contains itself, directly or through a chain. A shared reference that isn't a cycle is fine — the same list appearing under two keys writes out twice, without complaint.

### `the top level of a .jp document must be a mapping, got '...'`

You passed a list, a string or a number to `dumps`. The top level of a `.jp` document is always an object, because top-level values become `[SECTION]` headers. Wrap it: `{"items": [...]}`.

---

## Other errors

### `no such config directory: ...`

`load_dir` was pointed at a path that doesn't exist or isn't a directory. Check the path is relative to where the program *runs*, not to where the source file lives — this catches everyone at least once.

### `this config has no path to reload from` / `this config has no path; call save(path) or set .path first`

You called `reload()` or `save()` on a `JPConfig` that was built from a string rather than loaded from a file, so it has no path to work with. Pass a path to `save()`, set `.path`, or construct the config with one.

### `indent must be >= 0`

A negative `indent` was passed to the writer.

### `unknown option(s): ...`

A typo'd option name. The message lists what it didn't recognise — check the spelling and the case convention for your language (`sort_keys` in Python, `sortKeys` in JavaScript).

---

## Problems that produce no error at all

The hard ones. Nothing crashes; the file is just not what you meant.

### My value came out as a string

An unquoted value is read as a keyword, then as a number, then as text. If it isn't one of the first two, it's text.

```jp
version: 1.2.3      # string "1.2.3" -- not a number, so it stays text
enabled: yes        # string "yes" -- 'yes' is not a keyword; use true
count: 1_000        # integer 1000 -- underscores are separators
id: "42"            # string "42" -- the quotes win
```

`true`, `false`, `null`, `none`, `nil`, `inf`, `infinity` and `nan` are the keywords. `yes`, `no`, `on` and `off` are not.

### My value came out as null

You wrote the value on the line after the colon:

```jp
# WRONG -- config is null, and { retries: 3 } is a syntax error just after
config:
  { retries: 3 }
```

The opening bracket must sit on the colon's line. See [*Empty values*](/jpcl/syntax/#empty-values) for why.

### Half of my value vanished

An unquoted value stops at a `#`, a comma, or a closing bracket. With a `#`, the rest is read as a comment and silently discarded:

```jp
label: issue #1 reported
# -> label is "issue"
```

With a comma it usually errors instead, because what follows is read as the next key:

```jp
csv: a, b, c
# -> expected ':' after key 'b', found ','
```

Quote it either way:

```jp
label: "issue #1 reported"
csv: "a, b, c"
```

### My comments disappeared

Something rewrote the file. `jpcl fmt -w`, or a program calling `save()` or `dump()`. The writer reproduces data, not annotations.

Keep hand-annotated files separate from the files your program writes to. A common split is a committed `servers.example.jp` full of comments, and a live `servers.jp` that the program owns.

### My sections came back in a different order

Two possibilities.

**In JavaScript**, keys that look like integers are always enumerated first, in ascending order, before everything else. That is the language, not JPCL. A `[1234567890]` section will come out ahead of `[SERVER_ID]` however you wrote it. Pass a `Map` to `dumps` if that order matters.

**In any implementation**, `sort_keys` was switched on somewhere.

### A root-level key moved to the top of the file

That is deliberate. A scalar written after a header would read back as part of that section, so the writer hoists all root-level scalars above the first header. The data is unchanged. See [*Round trips*](/jpcl/spec/#round-trips).

### `1.0` came back as `1`

JavaScript only. It has a single number type, so a whole float is indistinguishable from an integer. Python and Go keep the distinction.

### A big ID lost its last few digits

You are in Go, where integers are `int64`, and the value exceeded it — or in JavaScript with `integers: "number"` forced on. In JavaScript's default mode, anything past 2^53 arrives as a `bigint` and stays exact. See [*Integer width*](/jpcl/spec/#integer-width).

### My edits to the file do nothing

Most programs read their config once, at startup. Restart it. If it still ignores you, check you are editing the file the program actually loads — print the resolved path at startup and you will usually find the answer immediately.

---

## Still stuck?

Three things worth trying, in order:

>>> *Run `jpcl check` on the file*
It gives you the first error with an exact position, which is more than most programs will.

>>> *Run `jpcl to-json` on the file*
This shows you exactly what your file parses to. When the file is valid but the values are wrong, this finds it in seconds.

>>> *Reduce it*
Delete half the file and check again. Repeat. Three or four rounds narrows almost anything to a single line.
>>>

If you have found a real bug, open an issue on the relevant repository under [*jpcl-lang*](https://github.com/jpcl-lang), and include the smallest file that reproduces it.
