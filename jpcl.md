# JPCL - Just Parsable Configuration Language

**A configuration language that borrows TOML's sections and JSON's nesting.**

JPCL files end in `.jp`. They use `[SECTION]` headers at the top level and `{...}` / `[...]` structures inside them. Keys need no quotes, `#` starts a comment, trailing commas are fine, and a value is allowed to be *empty*.

```jp
[SERVER_ID]
config: {
  disabled_channels:,
  disabled_users: [9892, 82082, 8209]
}

[SERVER_ID_2]
prefix: "!"
modules: {
  moderation: true,
  fun: false
}
```

GitHub: :icon-lockup-github: [*jpcl-lang*](https://github.com/jpcl-lang)

### Why another format

JSON has no comments, demands quotes on every key, and rejects a trailing comma. TOML has comments and headers, but nesting anything non-trivial means either deeply dotted keys or a table per level.

`.jp` takes the half of each that suits configuration files people edit by hand:

| | |
|---|---|
| **Sections for the top level** | `[SERVER_ID]` reads better than another brace. |
| **JSON for everything below it** | Nest objects and arrays as deep as you like. |
| **No ceremony** | Unquoted keys, comments anywhere, trailing commas ignored. |
| **Empty values are legal** | `disabled_channels:,` means the key exists and has no value yet — a real state in configs that JSON can only spell as `null`. |

It is a small, fully specified format with a strict parser, precise error messages, and a deterministic writer, so files stay stable when a program rewrites them.

### Three implementations, one format

A file written by any of these reads back unchanged in the other two. Same syntax, same error messages, same deterministic writer — the Python and Go versions are byte-for-byte identical, and JavaScript differs only in where it puts numeric section names, which is the language's doing rather than JPCL's.

| Language | Install | Docs |
|---|---|---|
| Python | `pip install jpcl` | [*Python*](/jpcl/py) |
| JavaScript / TypeScript | `npm install jpcl` | [*JavaScript & TypeScript*](/jpcl/npm) |
| Go | `go get github.com/jpcl-lang/jpcl-go` | [*Go*](/jpcl/go) |

All three ship the same `jpcl` command-line tool — see [*Command line*](/jpcl/cli).

### Where to start

>>> *Never used a config format before?*
[*Start here*](/jpcl/start) walks you from nothing to a working config, assuming no prior knowledge.

>>> *Just want the syntax?*
[*Writing .jp files*](/jpcl/syntax) explains every rule in plain English, with examples.

>>> *Want to copy a working setup?*
[*Worked examples*](/jpcl/recipes) has complete, runnable projects in all three languages.

>>> *Something broke?*
[*When it goes wrong*](/jpcl/errors) lists every error message and what to do about it.

>>> *Writing your own parser?*
[*Language reference*](/jpcl/spec) is the precise version — grammar, limits, round-trip guarantees.
>>>

!!!secondary
JPCL is MIT licensed. All three implementations live under the [*jpcl-lang*](https://github.com/jpcl-lang) organisation on GitHub.
!!!
