# Command line

All three implementations ship the same `jpcl` command, with the same subcommands, flags and output. Use whichever you have installed; the results are identical.

```bash
jpcl check data/*.jp                      # validate; non-zero exit on failure
jpcl fmt -w data/servers.jp               # reformat in place
jpcl get data/servers.jp SERVER_ID.prefix # read one value
jpcl to-json data/servers.jp -o out.json
jpcl from-json out.json -o data/servers.jp
```

---

## Getting the command

+++ Python
Installing the package installs the command:

```bash
pip install jpcl
```

`python -m jpcl ...` works identically if the bare `jpcl` command isn't on your `PATH`.
+++ JavaScript
Installing the package into a project makes the command available through `npx`:

```bash
npm install jpcl
npx jpcl check data/servers.jp
```

For a copy you can use anywhere:

```bash
npm install -g jpcl
```
+++ Go
`go get` installs the library only. The command is a separate install:

```bash
go install github.com/jpcl-lang/jpcl-go/cmd/jpcl@latest
```

!!!warning Flags before filenames
The Go build requires flags to come **before** positional arguments — `jpcl fmt -w file.jp`, never `jpcl fmt file.jp -w`. Go's standard `flag` package stops looking for flags at the first non-flag argument. The Python and JavaScript builds accept either order.
!!!
+++

---

## Reading from stdin

Every command accepts `-` in place of a filename, so JPCL drops into a pipeline:

```bash
cat data/servers.jp | jpcl check -
curl -s https://example.com/config.jp | jpcl to-json -
jpcl to-json data/servers.jp | jq '.SERVER_ID.prefix'
```

`jpcl fmt -w -` is the one combination that does nothing useful: there is no file to write back to, so the result goes to stdout regardless.

---

## `check`

Validate files. Prints one line per file and exits non-zero if any failed.

```bash
jpcl check data/servers.jp
jpcl check data/*.jp
jpcl check -q data/*.jp
```

| Flag | Meaning |
|---|---|
| `-q`, `--quiet` | Print only failures. Successes are silent. |

Output:

```
ok  data/servers.jp
ok  data/roles.jp
```

A failure goes to stderr with the exact position:

```
data/servers.jp:2:8: expected ':' after key 'prefix', found '"'
    prefix "!"
           ^
```

Checking stops at the first error **in each file**, but carries on to the next file, so one run tells you which files are broken even if it doesn't list every fault within them.

!!!info Use this in CI
`jpcl check data/*.jp` is a one-line CI step that catches a broken config before it reaches production. It exits `1` on any failure, which is all most CI systems need.

```yaml
- run: pip install jpcl && jpcl check data/*.jp
```
!!!

---

## `fmt`

Reformat files to canonical style. Without `-w`, the result goes to stdout and the file is untouched.

```bash
jpcl fmt data/servers.jp          # print the formatted version
jpcl fmt -w data/servers.jp       # rewrite it in place
jpcl fmt -w --indent 4 data/*.jp
```

| Flag | Default | Meaning |
|---|---|---|
| `-w`, `--write` | off | Rewrite each file in place instead of printing. |
| `--indent N` | `2` | Spaces per nesting level. |
| `--width N` | `88` | Column budget before an inline array breaks onto several lines. |
| `--sort-keys` | off | Sort keys alphabetically instead of keeping the order they were written in. |

With `-w`, a file that is already correctly formatted is left completely alone — not rewritten with identical content — so timestamps and file watchers stay quiet. Files that did change are reported on stderr:

```
reformatted data/servers.jp
```

!!!warning `fmt -w` drops your comments
Reformatting is a parse followed by a write, and the writer does not reproduce comments. Running `jpcl fmt -w` over a hand-annotated file will silently delete every `#` note in it.

Run `jpcl fmt` without `-w` first and read the output if you are not sure what a file will lose.
!!!

---

## `get`

Print one value, addressed by a dotted path.

```bash
jpcl get data/servers.jp SERVER_ID.prefix
jpcl get data/servers.jp SERVER_ID.config.disabled_users
jpcl get -r data/servers.jp SERVER_ID.prefix
```

| Flag | Meaning |
|---|---|
| `-r`, `--raw` | Print strings unquoted. Has no effect on other types. |

The value is printed as JSON, which means strings come out quoted:

```console
$ jpcl get data/servers.jp SERVER_ID.prefix
"!"

$ jpcl get -r data/servers.jp SERVER_ID.prefix
!

$ jpcl get data/servers.jp SERVER_ID.config.disabled_users
[
  9892,
  82082,
  8209
]
```

`-r` is what you want in a shell script:

```bash
PREFIX=$(jpcl get -r data/servers.jp SERVER_ID.prefix)
```

A path that doesn't exist is an error, not an empty result — it prints `no such path: ...` to stderr and exits `1`. That distinction matters in scripts: a missing setting fails loudly instead of quietly becoming an empty string.

---

## `to-json`

Convert a `.jp` file to JSON.

```bash
jpcl to-json data/servers.jp
jpcl to-json data/servers.jp -o out.json
jpcl to-json --indent 0 data/servers.jp | jq .
```

| Flag | Default | Meaning |
|---|---|---|
| `-o`, `--output FILE` | stdout | Write to a file instead of printing. |
| `--indent N` | `2` | Spaces per level in the JSON output. |

Useful for feeding a `.jp` config to anything that only speaks JSON, and for seeing exactly what your file parses to when the answer is surprising.

---

## `from-json`

Convert JSON to a `.jp` file.

```bash
jpcl from-json out.json
jpcl from-json out.json -o data/servers.jp
echo '{"a": {"b": 1}}' | jpcl from-json -
```

| Flag | Default | Meaning |
|---|---|---|
| `-o`, `--output FILE` | stdout | Write to a file instead of printing. |
| `--indent N` | `2` | Spaces per nesting level. |
| `--width N` | `88` | Column budget before an inline array breaks. |
| `--sort-keys` | off | Sort keys alphabetically. |

Every top-level object in the JSON becomes a `[SECTION]`. Top-level scalars are hoisted above the first section — see [*Round trips*](/jpcl/spec/#round-trips).

!!!info Migrating an existing config
`from-json` is the quickest way to move an existing JSON config over:

```bash
jpcl from-json config.json -o config.jp
jpcl check config.jp
```

Then open the result and add the comments JSON never let you write.
!!!

---

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Everything succeeded. |
| `1` | A file failed to parse, a path didn't exist, or a file couldn't be read or written. |
| `2` | The command line itself was wrong — unknown subcommand, missing argument, unrecognised flag. |

---

## Other flags

| Flag | Meaning |
|---|---|
| `--version` | Print the version and exit. |
| `-h`, `--help` | Print usage. Works on the top level and on each subcommand: `jpcl fmt --help`. |
