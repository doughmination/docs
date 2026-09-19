---
order: 700
label: Language reference
icon: book
---

# Language reference

The precise version of the format: what the parser accepts, what it produces, and what the writer guarantees. This is the page to read if you are implementing JPCL yourself, or if you need to know exactly where a boundary lies.

For the same rules explained conversationally, read [*Writing .jp files*](/jpcl/syntax).

---

## Document model

A `.jp` document is an ordered mapping of string keys to values.

| `.jp` type | Python | JavaScript | Go |
|---|---|---|---|
| Object / section | `dict` | plain object | `*jpcl.Object` |
| Array | `list` | `Array` | `[]any` |
| String | `str` | `string` | `string` |
| Integer | `int` | `number` or `bigint` | `int64` |
| Float | `float` | `number` | `float64` |
| Boolean | `bool` | `boolean` | `bool` |
| Null / empty | `None` | `null` | `nil` |

Key order is insertion order, and is preserved through a round trip. Go's built-in `map` does not preserve order, which is why the Go implementation defines its own ordered `Object` type.

### Encoding

Documents are UTF-8. A leading UTF-8 byte-order mark is stripped if present. Both `\n` and `\r\n` line endings are accepted.

---

## Lexical structure

### Whitespace

Spaces and tabs separate tokens and are otherwise insignificant. Line breaks are significant: they terminate an entry and terminate a bare token.

### Comments

```
comment = "#" *( any character except line break )
```

A comment may appear anywhere a token may appear, including inside `{}` and `[]`. It runs to the end of the line and is discarded.

### Bare tokens

Three contexts read an unquoted run of characters, each stopping at a different set:

| Context | Terminated by |
|---|---|
| Header segment | `.` `]` `#` line break |
| Key | `:` `,` `{` `}` `[` `]` `"` `'` `#` line break |
| Value | `,` `}` `]` `#` line break |

The captured run is then trimmed of surrounding whitespace. A bare key may therefore contain spaces and dots; a bare value may contain spaces, dots, colons and brackets' worth of other punctuation, but not a comma, a closing brace or bracket, or a `#`.

### Quoted tokens

Either `"` or `'` opens a string; the same character closes it. The two are interchangeable and carry no difference in meaning or escaping.

An unescaped line break inside a string is an error (`unterminated string literal (use \n for a line break)`). Use `\n`, or a trailing `\` to continue the literal onto the next source line.

#### Escape sequences

| Sequence | Produces |
|---|---|
| `\n` `\r` `\t` `\v` `\f` `\b` `\a` | the corresponding control character |
| `\0` | NUL |
| `\e` | escape (`U+001B`) |
| `\\` `\"` `\'` `\/` | the literal character |
| `\uXXXX` | the code point given by 4 hex digits |
| `\UXXXXXXXX` | the code point given by 8 hex digits |
| `\` + line break | nothing; the string continues on the next line, and the indentation at the start of that line is eaten |

Anything else after a `\` is `invalid escape sequence`. A `\uXXXX` that names an unpaired surrogate, or a `\UXXXXXXXX` outside the Unicode range, is rejected rather than replaced.

---

## Grammar

```
document    = *( entry-sep ) [ root-entries ] *( section )
root-entries= entry *( entry-sep entry )
section     = header *( entry-sep ) [ entries ]
header      = "[" segment *( "." segment ) "]"
segment     = bare-segment / quoted
entry       = key ":" [ value ]
key         = bare-key / quoted
value       = object / array / quoted / bare-value
object      = "{" [ members ] "}"
members     = entry *( entry-sep entry ) [ entry-sep ]
array       = "[" [ elements ] "]"
elements    = value *( element-sep value ) [ element-sep ]
entry-sep   = 1*( "," / line-break )
element-sep = 1*( "," / line-break )
```

Separators are greedy: any run of commas and line breaks counts as one separator, so leading, repeated and trailing commas are all accepted and carry no meaning.

### The same-line rule

A value must **begin** on the same line as its `:`. Once begun, an object or array may span as many lines as it likes.

This follows directly from empty values being legal. `timeout:` followed by a line break is a complete entry with the value null; there is no lookahead that could distinguish it from a value that was going to appear on the next line.

---

## Value resolution

An unquoted value is resolved in this order:

1. **Empty** — an empty or whitespace-only token yields null.
2. **Keyword** — matched case-insensitively against the table below.
3. **Number** — see [*Numbers*](#numbers).
4. **String** — the trimmed token, verbatim.

### Keywords

| Spelling (case-insensitive) | Value |
|---|---|
| `true` | true |
| `false` | false |
| `null`, `none`, `nil` | null |
| `nan` | NaN |
| `inf`, `infinity`, `+inf`, `+infinity` | positive infinity |
| `-inf`, `-infinity` | negative infinity |

Case-insensitivity is deliberate: `True`, `None` and `NULL` all work, so a file written by someone thinking in Python or in JSON parses either way.

### Numbers

An optional `+` or `-` sign, then:

| Form | Example | Type |
|---|---|---|
| Decimal integer | `42`, `-7`, `1_000` | integer |
| Hexadecimal | `0xff` | integer |
| Octal | `0o755` | integer |
| Binary | `0b1010` | integer |
| Decimal float | `3.5`, `-0.25` | float |
| Exponential | `1e3`, `2.5e-8` | float |

Underscores are permitted as digit separators in any of these. A token that starts like a number but does not parse as one falls through to being a string — `1.2.3` is the string `"1.2.3"`, not an error.

### Integer width

The format itself places no limit on integer size; the implementations do, differently:

| | |
|---|---|
| **Python** | Arbitrary precision. No limit. |
| **JavaScript** | An integer that fits in a `number` exactly (up to 2^53) becomes one; beyond that it becomes a `bigint`, so no digit is ever silently changed. Configurable with the `integers` option. |
| **Go** | `int64`. A value beyond roughly 9.2×10^18 falls through to `float64` and loses precision. |

Discord snowflakes and other 64-bit IDs are the usual reason this matters. They are safe in all three, but in JavaScript they arrive as `bigint`.

---

## Structural rules

### Sections

A header's segments are resolved left to right, creating objects as needed.

* A segment that already names an object descends into it.
* A segment that already names a non-object is an error: `cannot open section '[...]': ...`.
* Re-opening a section that already exists is `section '[...]' is defined twice`, unless the duplicate-key policy is `last`, in which case the two are merged.
* A header segment may not be empty: `[]`, `[a.]` and `[.a]` are all errors.

### Root entries

Entries appearing before the first header attach to the document root. There is no syntax for returning to the root after a header has opened.

### Duplicate keys

Within one object, a repeated key is governed by the parse-time policy:

| Policy | Repeated key | Repeated section |
|---|---|---|
| `error` *(default)* | `duplicate key '...'` | `section '[...]' is defined twice` |
| `first` | first value kept | first section kept, later entries ignored |
| `last` | last value kept | sections merged, later entries winning |

### Nesting depth

`{` and `[` may nest to **200** levels. Beyond that the parser refuses with `nesting deeper than 200 levels`. The cap exists so that a hostile or accidentally generated document cannot exhaust the stack; no legitimate config comes close.

### `__proto__`

In the JavaScript implementation, a key named `__proto__` is defined as an ordinary own property rather than being allowed to replace an object's prototype. A `.jp` file cannot cause prototype pollution.

---

## Writer

The writer is deterministic: the same input always produces byte-identical output, so a file written twice does not churn in version control.

### Layout rules

* Every top-level object value becomes a `[SECTION]`, separated by a blank line.
* Section entries sit one per line, with no separating commas.
* Nested objects always expand across lines. `{}` is the only inline object form.
* Arrays stay inline while they fit within `width` (default 88 columns), then break one element per line.
* Null is written as an empty value inside objects (`key:`) and as `null` inside arrays, since an array element cannot be empty.
* Keys are quoted only when they must be.
* Insertion order is preserved unless `sortKeys` / `sort_keys` / `SortKeys` is set.

### Options

| Option | Default | Effect |
|---|---|---|
| `indent` | `2` | Spaces per nesting level. |
| `width` | `88` | Column budget before an inline array breaks. |
| `sort_keys` | `false` | Sort keys alphabetically instead of keeping insertion order. |
| `ensure_ascii` | `false` | Escape non-ASCII characters as `\uXXXX`. |
| `default` | none | A function called to convert a value the writer does not otherwise support. |

Spelled `snake_case` in Python, `camelCase` in JavaScript and `PascalCase` fields in Go.

### Atomic writes

File-writing functions write to a temporary neighbour and rename it into place. A crash or a concurrent reader therefore never sees a half-written config. Missing parent directories are created.

---

## Round trips

Parse → write → parse is lossless for **data**. Two things do not survive a rewrite in any implementation:

| | |
|---|---|
| **Comments are dropped** | The writer reproduces values, not annotations. A hand-annotated file loses its notes when a program saves it. |
| **Root scalars move to the top** | They are hoisted above the first section, because anything after a header would read back as part of that section. The data is unchanged; its position in the file is not. |

JavaScript adds two of its own, both inherited from the language:

| | |
|---|---|
| **Integer-like keys move to the front** | JavaScript objects always enumerate keys such as `"1234567890"` first, in ascending order, before every other key. Pass a `Map` to `dumps` if the order of such keys matters. |
| **Whole floats become integers** | There is one number type, so `1.0` reads back as `1` and is written as `1`. |

Everything else — key order, types, structure, string content — comes back identical, and a file written by any implementation is read identically by the other two.

---

## Errors

Two error families, both carrying enough structure to render your own message.

### Decode errors

Raised for any malformed document. They carry the message, the document, the character offset, the line, the column and the filename, and render as:

```
data/servers.jp:2:8: expected ':' after key 'prefix', found '"'
    prefix "!"
           ^
```

`JPDecodeError` in Python (a `ValueError`) and JavaScript; `*jpcl.DecodeError` in Go.

### Encode errors

Raised when a value cannot be serialised — an unsupported type, a non-string key, a circular reference, or binary data, which has no `.jp` representation.

`JPEncodeError` in Python (a `TypeError`) and JavaScript; `*jpcl.EncodeError` in Go.

[*When it goes wrong*](/jpcl/errors) lists every message either family can produce.

---

## Reserved for the future

The format is small on purpose and the parser is strict, so the following are errors today rather than silently ignored — which keeps the door open for them to mean something later:

* `//` and `/* */` comments.
* Multi-line string delimiters.
* Dates and times as a distinct type. Use a quoted ISO 8601 string, and the `default` writer hook to convert on the way out.
* Section headers inside `{}`.
