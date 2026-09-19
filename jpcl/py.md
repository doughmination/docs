# Python

```bash
pip install jpcl       # or: uv add jpcl
```

No runtime dependencies. **Python 3.14+.**

:icon-lockup-github: [*jpcl-lang/jpcl-py*](https://github.com/jpcl-lang/jpcl-py) · [*jpcl on PyPI*](https://pypi.org/project/jpcl/)

```python
>>> import jpcl
>>> jpcl.load("data/servers.jp")
{'SERVER_ID': {'config': {'disabled_channels': None,
                          'disabled_users': [9892, 82082, 8209]}},
 'SERVER_ID_2': {'prefix': '!', 'modules': {'moderation': True, 'fun': False}}}
```

A `.jp` document is a plain `dict`. There is no custom type to learn, nothing to convert, and nothing to unwrap — whatever you already know about dicts applies.

---

## Values

| `.jp` type | Python type |
|---|---|
| Object / section | `dict` |
| Array | `list` |
| String | `str` |
| Integer | `int` |
| Float | `float` |
| Boolean | `bool` |
| Null / empty value | `None` |

Python's `int` is arbitrary precision, so Discord snowflakes and other large IDs are exact with no special handling. Dicts have preserved insertion order since 3.7, which is what lets the writer reproduce your key order.

---

## Reading

```python
import jpcl

data = jpcl.load("data/servers.jp")     # from a path
data = jpcl.loads(text)                 # from a string
```

`load` also accepts an open text file, or anything with a `read()` method:

```python
with open("data/servers.jp", encoding="utf-8") as fh:
    data = jpcl.load(fh)
```

### Options

```python
jpcl.load("servers.jp", duplicate_keys="last")           # "error" (default), "first", "last"
jpcl.loads(text, filename="config.jp")                   # name shown in error messages
jpcl.loads(text, dict_factory=collections.OrderedDict)   # build a different mapping type
```

By default a repeated key is an error rather than a silent overwrite. Pass `duplicate_keys="first"` or `"last"` if you would rather it not be — see [*Duplicate keys*](/jpcl/syntax/#duplicate-keys).

---

## Writing

```python
jpcl.dump(data, "data/servers.jp")      # formatted, atomic write
text = jpcl.dumps(data)                 # -> str
```

Writes are atomic by default: the file goes to a temporary neighbour and is renamed into place, so a crash or a concurrent reader never sees half a config. Missing parent directories are created. Pass `atomic=False` to write straight to the path, and `dump` also accepts an open file object.

### Options

```python
jpcl.dumps(data, indent=4)          # spaces per level (default 2)
jpcl.dumps(data, width=120)         # column budget for inline arrays (default 88)
jpcl.dumps(data, sort_keys=True)    # alphabetical instead of insertion order
jpcl.dumps(data, ensure_ascii=True) # escape non-ASCII as \uXXXX
jpcl.dumps(data, default=str)       # convert otherwise-unsupported types
```

The `default` hook is how you write a `datetime`, a `Decimal`, a `Path` or anything else `.jp` has no type for:

```python
from datetime import datetime

def convert(value):
    if isinstance(value, datetime):
        return value.isoformat()
    return str(value)

jpcl.dumps({"run": {"started": datetime.now()}}, default=convert)
```

It must return a *different* type from the one it was handed, or you get `default() returned an unconverted ...`.

---

## Load a whole folder

```python
config = jpcl.load_dir("data")                     # {'servers': {...}, 'roles': {...}}
guilds = jpcl.load_dir("data/guilds")              # {'1234567890': {...}, ...}
everything = jpcl.load_dir("data", recursive=True)
```

Each file becomes one key, named after the file without its `.jp` suffix. With `recursive=True`, nested files are keyed by their relative path with `/` separators — `data/guilds/1234.jp` becomes `"guilds/1234"`.

`pattern` picks which files count (default `"*.jp"`), and any other keyword is forwarded to `load`:

```python
jpcl.load_dir("data", pattern="*.example.jp", duplicate_keys="last")
```

A missing directory raises `NotADirectoryError`.

---

## Editing a config in place: `JPConfig`

`JPConfig` is a `MutableMapping` that remembers the file it came from. Everything you can do to a dict, you can do to it — plus dotted paths and saving.

```python
from jpcl import JPConfig

cfg = JPConfig.load("data/servers.jp", missing_ok=True)

cfg["SERVER_ID"]["prefix"]                              # plain dict access
cfg.get_path("SERVER_ID.config.disabled_users", [])     # never raises
cfg.set_path("SERVER_ID.config.disabled_users", [9892]) # creates missing sections
cfg.has_path("SERVER_ID.prefix")
cfg.section("NEW_SERVER", create=True)["prefix"] = "?"
cfg.merge({"SERVER_ID": {"modules": {"fun": True}}})    # deep merge
cfg.save()                                              # atomic, back to its own path
cfg.reload()                                            # discard in-memory changes
cfg.to_dict()                                           # deep copy as a plain dict
```

### Constructors

```python
JPConfig.load("data/servers.jp")                    # read a file
JPConfig.load("data/servers.jp", missing_ok=True)   # or start empty if absent
JPConfig.loads(text, path="data/servers.jp")        # from a string, bound to a path
JPConfig({"SERVER_ID": {}}, path="data/servers.jp") # from data you already have
```

`missing_ok=True` gives an empty config bound to the path, which is exactly what a program that writes its config on first run wants:

```python
cfg = JPConfig.load("data/settings.jp", missing_ok=True)
if not cfg.has_path("bot.token"):
    cfg.set_path("bot.token", "")
    cfg.save()
```

### Dotted paths

```python
cfg.get_path("SERVER_ID.config.disabled_users", [])
cfg.set_path("SERVER_ID.config.disabled_users", [9892])
cfg.has_path("SERVER_ID.prefix")
```

`get_path` returns its default rather than raising, at any depth — a missing `SERVER_ID` is as safe as a missing `prefix`. `set_path` creates every level it needs on the way in. Both take `sep=` if `.` is awkward for your key names.

This is the whole reason `JPConfig` exists. The dict equivalent of `set_path` is four lines of `setdefault`, and the equivalent of `get_path` is a `try`/`except` chain that is easy to get subtly wrong.

### Sections

```python
section = cfg.section("SERVER_ID")                  # KeyError if missing
section = cfg.section("NEW_SERVER", create=True)    # created empty if missing
```

Raises `TypeError` if the name holds something that isn't a mapping.

### Remembered options

Formatting options given to the constructor are remembered by `save()`:

```python
cfg = JPConfig.load("data/servers.jp", indent=4, sort_keys=True)
cfg.save()                     # uses indent=4, sort_keys=True
cfg.save(indent=2)             # overridden for this call only
cfg.save("backup.jp")          # a different path; the config rebinds to it
```

Read options (`duplicate_keys`) are remembered by `reload()` in the same way.

---

## Errors

Every error derives from `jpcl.JPError`.

```python
from jpcl import JPError, JPDecodeError, JPEncodeError
```

`JPDecodeError` is also a `ValueError`, and `JPEncodeError` is also a `TypeError`, so existing `except ValueError` handlers keep working.

```python
try:
    data = jpcl.load("data/servers.jp")
except JPDecodeError as exc:
    print(exc)          # the full message with the caret line
    print(exc.line)     # 2
    print(exc.col)      # 8
    print(exc.pos)      # character offset
    print(exc.filename) # "data/servers.jp"
    print(exc.raw_message)  # the message without position or source
```

Printed in full, it looks like this:

```
data/servers.jp:2:8: expected ':' after key 'prefix', found '"'
    prefix "!"
           ^
```

[*When it goes wrong*](/jpcl/errors) lists every message and what to do about each.

---

## Command line

```bash
jpcl check data/*.jp                      # validate; non-zero exit on failure
jpcl fmt -w data/servers.jp               # reformat in place
jpcl get data/servers.jp SERVER_ID.prefix # read one value
jpcl to-json data/servers.jp -o out.json
jpcl from-json out.json -o data/servers.jp
```

`python -m jpcl ...` works identically, which is useful when the bare command isn't on your `PATH`. `-` reads stdin. Full reference: [*Command line*](/jpcl/cli).

---

## Differences from the other implementations

The Python implementation is the reference one, so mostly the question is what the *others* do differently. For completeness:

| | |
|---|---|
| **Integers are exact** | Arbitrary precision, unlike Go's `int64`. No `bigint` distinction to think about, unlike JavaScript. |
| **Floats keep their type** | `1.0` stays a float, unlike JavaScript. |
| **Keyword arguments** | Options are keyword-only, so `jpcl.dumps(data, indent=4)` rather than an options object. |
| **`dict_factory`** | Python alone lets you choose the mapping type the parser builds. |

---

## Full API

```python
import jpcl

jpcl.load(source, *, duplicate_keys="error", dict_factory=None) -> dict
jpcl.loads(text, *, filename=None, duplicate_keys="error", dict_factory=None) -> dict
jpcl.dump(obj, target, *, atomic=True, **options) -> None
jpcl.dumps(obj, **options) -> str
jpcl.load_dir(directory, *, pattern="*.jp", recursive=False, **options) -> dict[str, dict]

jpcl.JPConfig(data=None, *, path=None, **options)
jpcl.JPConfig.load(path, *, missing_ok=False, **options)
jpcl.JPConfig.loads(text, *, path=None, **options)

jpcl.JPError
jpcl.JPDecodeError    # also a ValueError
jpcl.JPEncodeError    # also a TypeError

jpcl.SUFFIX           # ".jp"
jpcl.ENCODING         # "utf-8"
jpcl.__version__
```

`JPConfig` methods: `get_path`, `set_path`, `has_path`, `section`, `merge`, `to_dict`, `dumps`, `save`, `reload`, plus the whole `MutableMapping` protocol.

---

## Next

>>> *[Worked examples](/jpcl/recipes)*
Complete Python projects using all of this.

>>> *[Command line](/jpcl/cli)*
The `jpcl` tool in full.

>>> *[When it goes wrong](/jpcl/errors)*
Every error message, explained.
>>>
