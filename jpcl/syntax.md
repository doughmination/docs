# Writing `.jp` files

Every rule of the format, one at a time, with an example each. This is the page to keep open while you write a config.

If you want the same information stated precisely — grammar, limits, exact resolution order — read [*Language reference*](/jpcl/spec) instead. If you have never done this before, [*Start here*](/jpcl/start) comes first.

---

## The shape of a file

A `.jp` file is a list of **entries**, optionally grouped under **section headers**, with **comments** anywhere you like.

```jp
# a comment

version: 2                 # a root-level entry

[SERVER_ID]                # a section header
prefix: "!"                # an entry inside that section
modules: {                 # an entry whose value is an object
  moderation: true,
  fun: false
}
```

That parses to:

```json
{
  "version": 2,
  "SERVER_ID": {
    "prefix": "!",
    "modules": { "moderation": true, "fun": false }
  }
}
```

---

## Sections

A `[NAME]` header opens a root key. Everything below it, until the next header, belongs to that section.

```jp
[SERVER_ID]
prefix: "!"
```

```json
{ "SERVER_ID": { "prefix": "!" } }
```

### Dotted headers nest

A dot inside a header creates a level of nesting, which saves you a pile of braces:

```jp
[guild.limits]
per_minute: 30
```

```json
{ "guild": { "limits": { "per_minute": 30 } } }
```

### Quote a header whose name contains a dot

If the name itself has a dot in it, quote it so JPCL doesn't split on it:

```jp
["weird.name"]
a: 1
```

```json
{ "weird.name": { "a": 1 } }
```

You can quote one segment of a dotted header and leave the rest bare — `[guild."my.limits"]` nests under `guild` with a single child called `my.limits`.

### Entries before the first header live at the root

Anything written above the first `[HEADER]` belongs to the document itself, not to a section:

```jp
version: 2

[SERVER_ID]
prefix: "!"
```

```json
{ "version": 2, "SERVER_ID": { "prefix": "!" } }
```

!!!warning Root entries have to go at the top
This only works *above* the first header. There is no way to return to the root once a section has opened — anything after a header is read as part of that section. This is also why a rewritten file moves root scalars to the top; see [*Round trips*](/jpcl/spec/#round-trips).
!!!

### A section name can't be used twice

```jp
# WRONG
[SERVER_ID]
a: 1

[SERVER_ID]
b: 2
```

That is an error by default, on the grounds that you almost certainly meant two different sections. If you genuinely want the two merged, load the file with the duplicate-key policy set to `last` — see [*Duplicate keys*](#duplicate-keys) below.

### Sections don't have to hold anything

An empty section is fine, and gives you an empty object:

```jp
[placeholder]
```

---

## Entries

An entry is `key: value`.

```jp
prefix: "!"
```

### Keys usually need no quotes

A bare key may contain spaces, but not brackets, commas or quotes. Quote it if it needs any of those:

```jp
prefix: "!"
display name: "Clanker"        # spaces are fine unquoted
"weird, key": 1                # a comma means you must quote
"[bracketed]": 2
```

### Separators are flexible

Entries are separated by a line break, a comma, or both. Trailing and repeated commas are accepted and ignored:

```jp
[SERVER_ID]
a: 1
b: {x: 1, y: 2,}
c: [1, 2, 3,]
d: 4, e: 5
```

All five keys parse, and the stray comma after `2` and `3` is harmless. This means you can paste JSON-ish fragments in without tidying the commas first.

### A key can't be used twice in one place

```jp
# WRONG
[SERVER_ID]
prefix: "!"
prefix: "?"
```

By default that is an error rather than a silent overwrite, because a duplicated key is nearly always a mistake — usually a half-finished edit. See [*Duplicate keys*](#duplicate-keys).

---

## Empty values

A key with nothing after the colon is legal, and means "this exists and has no value":

```jp
config: {
  disabled_channels:,      # empty
  timeout:                 # also empty
}
```

It parses to `None` in Python, `null` in JavaScript, and `nil` in Go — the same as writing `null` explicitly.

This is the one feature JPCL has that JSON doesn't, and it is there because half-configured settings are a real state in a real config file. `log_channel:` says *"I know about this setting, I haven't picked a channel yet"* in a way that a missing key can't.

!!!danger The rule this costs you
Because an empty value is legal, **a value must start on the same line as its `:`**. JPCL cannot look at the next line to decide — by the time it gets there, the entry is already over.

```jp
# WRONG -- this is an empty value, then a stray object
config:
  { retries: 3 }
```

```jp
# RIGHT -- the { is on the colon's line; the contents wrap freely
config: {
  retries: 3
}
```
!!!

---

## Values

| Type | Examples |
| --- | --- |
| String | `"hello"`, `'hello'`, `hello world` (unquoted) |
| Integer | `42`, `-7`, `1_000`, `0xff`, `0o755`, `0b1010` |
| Float | `3.5`, `1e3`, `inf`, `-inf`, `nan` |
| Boolean | `true`, `false` (case-insensitive, so `True` works too) |
| Null | `null`, `none`, `nil`, or nothing at all |
| Object | `{a: 1, b: 2}` |
| Array | `[1, 2, 3]` |

### How an unquoted value is read

An unquoted value is tried as a **keyword** first (`true`, `false`, `null`, `none`, `nil`, `inf`, `nan`), then as a **number**, and if it is neither, it is kept as a **plain string**.

```jp
a: true          # boolean
b: 42            # integer
c: 3.5           # float
d: hello world   # string
e: "true"        # string -- the quotes win
f: "42"          # string
```

So `enabled: true` gives you a boolean you can test directly, and `name: Clanker` gives you the text `Clanker` with no fuss. If you need the literal text `"true"`, quote it.

### When to quote

Quote a value when it contains a `#`, a comma, a bracket, or leading/trailing whitespace you want to keep. Single and double quotes both work and mean the same thing.

```jp
label: "issue #1"            # unquoted, the # would start a comment
csv: "a, b, c"               # unquoted, 'b' would be read as the next key
padded: "  keep my spaces  "
apostrophe: "it's fine"      # double quotes, so the ' is ordinary
quote_mark: 'say "hi"'       # single quotes, so the " is ordinary
```

### Numbers

Underscores are allowed as digit separators, and the usual bases are understood:

```jp
population: 1_000_000
colour: 0xff8800
permissions: 0o755
flags: 0b1010
ratio: 3.5
big: 1e9
```

Non-finite floats are spelled `inf`, `-inf` and `nan`.

!!!info Very large integers
Discord snowflakes and other 64-bit IDs sit right at the edge of what some languages can hold exactly. Each implementation handles that differently — Python has unlimited integers, JavaScript switches to `bigint` past 2^53, Go uses `int64`. If you work with IDs, read the note on your language's page: [Python](/jpcl/py), [JavaScript](/jpcl/npm/#big-numbers-stay-exact), [Go](/jpcl/go/#a-note-on-integer-size).
!!!

### Strings and escapes

Strings honour the usual escapes:

| Escape | Means |
|---|---|
| `\n` | line break |
| `\t` | tab |
| `\\` | a literal backslash |
| `\"` `\'` | a quote mark |
| `\uXXXX` | a character by 4-digit hex code |
| `\U0001F600` | a character by 8-digit hex code |
| `\` at end of line | continue this string onto the next line, ignoring that line's indentation |

```jp
message: "first line\nsecond line"
emoji: "\U0001F600"
long: "this string carries on \
onto the next line"
```

A plain line break inside a quoted string is an error — the message even tells you to use `\n` instead. Use the trailing backslash if you want to wrap a long string in the source without putting a break in the value.

### Objects and arrays

Objects use `{}`, arrays use `[]`, and they nest as deep as you like:

```jp
[1234567890]
limits: {
  rate: {per_minute: 30, burst: 5},
  warn_threshold: 3
}
staff_roles: [111111111111111111, 222222222222222222]
mixed: [1, "two", true, null, {a: 1}, [2, 3]]
```

Both may be empty: `{}` and `[]`.

An array element cannot be *empty* the way an object value can — `[1, , 2]` is not two elements and a gap. Write `null` if you want a hole:

```jp
slots: [1, null, 3]
```

---

## Comments

`#` runs to the end of the line, and is allowed anywhere — including inside objects and arrays:

```jp
# a whole-line comment

[SERVER_ID]
prefix: "!"        # a trailing comment
limits: {
  # a comment inside an object
  rate: 30,        # and another
}
roles: [
  111,             # inside an array, too
  222
]
```

!!!warning Comments do not survive a rewrite
If a program loads the file and saves it again, the comments are gone. The data is reproduced exactly; the notes are not. Keep files you annotate by hand separate from files your program writes to.
!!!

---

## Duplicate keys

By default, a repeated key or a repeated section header is an **error**. That is deliberate: silently keeping one of the two is how a config ends up quietly not doing what the file says.

You can change that when loading, if you would rather it didn't stop you:

| Policy | Behaviour |
|---|---|
| `error` | **Default.** Refuse the file and point at the second occurrence. |
| `first` | Keep the first value, ignore later ones. |
| `last` | Keep the last value. Repeated *sections* are merged into one. |

+++ Python
```python
jpcl.load("data/servers.jp", duplicate_keys="last")
```
+++ JavaScript
```js
await jpcl.load("data/servers.jp", { duplicateKeys: "last" });
```
+++ Go
```go
jpcl.Load("data/servers.jp", jpcl.ParseOptions{DuplicateKeys: jpcl.DuplicateLast})
```
+++

---

## Things that are *not* allowed

A short list of things people try, so you can rule them out quickly:

| Not allowed | Instead |
|---|---|
| `key = value` | `key: value` |
| A value starting on the line after its `:` | Put the opening `{` or `[` on the colon's line |
| A raw line break inside `"..."` | Use `\n`, or `\` at the end of the line |
| `[1, , 2]` for a gap in an array | `[1, null, 2]` |
| `//` or `/* */` comments | `#` |
| A section header inside `{}` | Nest with `{}`, or use a dotted header |
| Returning to the root after a header | Put root entries above the first header |
| Nesting deeper than 200 levels | Restructure; this cap is a guard against hostile input |

---

## A complete annotated file

Everything above, in one file:

```jp
# Template config -- copy to servers.jp and fill in.
# Section names are Discord guild IDs; they stay strings when parsed.

# Keys before the first [header] belong to the document root.
version: 2

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

[1234567890]
prefix: "?"
# An empty value means "set this later".
log_channel:
# Arrays wrap automatically once they outgrow the line.
staff_roles: [111111111111111111, 222222222222222222, 333333333333333333]
limits: {
  # Nested objects may go as deep as you need.
  rate: {per_minute: 30, burst: 5},
  warn_threshold: 3
}
```

Run `jpcl check` against it and it passes. Run `jpcl to-json` and you can see exactly what it becomes.

---

## Next

>>> *[Worked examples](/jpcl/recipes)*
Complete projects using all of this.

>>> *[When it goes wrong](/jpcl/errors)*
Every error message, and what to do about it.

>>> *[Language reference](/jpcl/spec)*
The same rules, stated precisely.
>>>
