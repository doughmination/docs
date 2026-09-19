---
order: 900
label: Worked examples
icon: beaker
---

# Worked examples

Complete, working setups you can copy. Every example shows the `.jp` file *and* the code that uses it, in all three languages.

If a line of code here is unfamiliar, your language's page explains it: [Python](/jpcl/py), [JavaScript](/jpcl/npm), [Go](/jpcl/go).

---

## 1. A per-server bot config

The canonical case: one section per Discord server, each with its own settings.

**`data/servers.jp`**

```jp
# One section per guild ID. Section names stay strings when parsed.

[1234567890]
prefix: "!"
log_channel: 9876543210
modules: {
  moderation: true,
  fun: true
}

[2345678901]
prefix: "?"
# Not chosen yet -- an empty value says so out loud.
log_channel:
modules: {
  moderation: true,
  fun: false
}
```

**Reading it**

+++ Python
```python
import jpcl

servers = jpcl.load("data/servers.jp")

for guild_id, settings in servers.items():
    print(guild_id, settings["prefix"], settings["modules"]["fun"])
```
+++ JavaScript
```js
import { load } from "jpcl";

const servers = await load("data/servers.jp");

for (const [guildId, settings] of Object.entries(servers)) {
  console.log(guildId, settings.prefix, settings.modules.fun);
}
```
+++ Go
```go
servers, err := jpcl.Load("data/servers.jp", jpcl.ParseOptions{})
if err != nil {
	return err
}

for _, guildID := range servers.Keys() {
	v, _ := servers.Get(guildID)
	settings := v.(*jpcl.Object)
	prefix, _ := settings.Get("prefix")
	fmt.Println(guildID, prefix)
}
```
+++

**Looking one server up, safely**

A server that isn't in the file yet should fall back to a default rather than crash. This is what `getPath` is for:

+++ Python
```python
from jpcl import JPConfig

cfg = JPConfig.load("data/servers.jp")

prefix = cfg.get_path(f"{guild_id}.prefix", "!")
fun_enabled = cfg.get_path(f"{guild_id}.modules.fun", False)
```
+++ JavaScript
```js
import { JPConfig } from "jpcl";

const cfg = await JPConfig.load("data/servers.jp");

const prefix = cfg.getPath(`${guildId}.prefix`, "!");
const funEnabled = cfg.getPath(`${guildId}.modules.fun`, false);
```
+++ Go
```go
cfg, err := jpcl.LoadJPConfig("data/servers.jp", false, jpcl.ConfigOptions{})
if err != nil {
	return err
}

prefix := cfg.GetPath(guildID+".prefix", "!").(string)
funEnabled := cfg.GetPath(guildID+".modules.fun", false).(bool)
```
+++

Missing at *any* level returns the default — a missing guild is as safe as a missing setting inside one. That is the whole point: no nested `if` chains, no `try`/`except`, no optional chaining.

---

## 2. A config that writes itself on first run

A program with no config file yet should create one rather than refuse to start. `missing_ok` gives you an empty config already bound to the path, so the first `save()` puts a real file there.

+++ Python
```python
from jpcl import JPConfig

DEFAULTS = {
    "bot": {"name": "Clanker", "prefix": "!", "enabled": True},
    "limits": {"per_minute": 30, "burst": 5},
}

cfg = JPConfig.load("data/settings.jp", missing_ok=True)

if not cfg:                 # empty means the file didn't exist
    cfg.merge(DEFAULTS)
    cfg.save()
    print("Wrote a starter config to data/settings.jp -- go and fill it in.")

print(cfg.get_path("bot.name"))
```
+++ JavaScript
```js
import { JPConfig } from "jpcl";

const DEFAULTS = {
  bot: { name: "Clanker", prefix: "!", enabled: true },
  limits: { per_minute: 30, burst: 5 },
};

const cfg = await JPConfig.load("data/settings.jp", { missingOk: true });

if (cfg.size === 0) {
  cfg.merge(DEFAULTS);
  await cfg.save();
  console.log("Wrote a starter config to data/settings.jp -- go and fill it in.");
}

console.log(cfg.getPath("bot.name"));
```
+++ Go
```go
cfg, err := jpcl.LoadJPConfig("data/settings.jp", true, jpcl.ConfigOptions{
	Write: jpcl.DefaultStringifyOptions(),
})
if err != nil {
	return err
}

if cfg.Data.Len() == 0 {
	cfg.SetPath("bot.name", "Clanker")
	cfg.SetPath("bot.prefix", "!")
	cfg.SetPath("bot.enabled", true)
	cfg.SetPath("limits.per_minute", int64(30))
	cfg.SetPath("limits.burst", int64(5))
	if _, err := cfg.Save(""); err != nil {
		return err
	}
	fmt.Println("Wrote a starter config to data/settings.jp -- go and fill it in.")
}

fmt.Println(cfg.GetPath("bot.name", ""))
```
+++

The file it writes:

```jp
[bot]
name: "Clanker"
prefix: "!"
enabled: true

[limits]
per_minute: 30
burst: 5
```

`merge` is a deep merge, so it is also how you top up an *existing* config with settings added in a later version, without stamping over what the user has already changed:

+++ Python
```python
cfg = JPConfig.load("data/settings.jp", missing_ok=True)
merged = {**DEFAULTS}
merged.update(cfg.to_dict())   # the user's values win
cfg.merge(merged)
cfg.save()
```
+++ JavaScript
```js
const cfg = await JPConfig.load("data/settings.jp", { missingOk: true });
const existing = cfg.toObject();
cfg.merge(DEFAULTS);           // fill the gaps
cfg.merge(existing);           // the user's values win
await cfg.save();
```
+++ Go
```go
defaults := jpcl.NewObject()
// ... populate defaults ...

existing := cfg.ToDict()
cfg.Merge(defaults, true)   // fill the gaps
cfg.Merge(existing, true)   // the user's values win
cfg.Save("")
```
+++

---

## 3. Changing a setting while the program runs

A command like `!prefix ?` should write the change back to disk so it survives a restart.

+++ Python
```python
from jpcl import JPConfig

cfg = JPConfig.load("data/servers.jp", missing_ok=True)

def set_prefix(guild_id: int, prefix: str) -> None:
    cfg.set_path(f"{guild_id}.prefix", prefix)
    cfg.save()
```
+++ JavaScript
```js
import { JPConfig } from "jpcl";

const cfg = await JPConfig.load("data/servers.jp", { missingOk: true });

async function setPrefix(guildId, prefix) {
  cfg.setPath(`${guildId}.prefix`, prefix);
  await cfg.save();
}
```
+++ Go
```go
cfg, err := jpcl.LoadJPConfig("data/servers.jp", true, jpcl.ConfigOptions{
	Write: jpcl.DefaultStringifyOptions(),
})

func setPrefix(guildID, prefix string) error {
	if err := cfg.SetPath(guildID+".prefix", prefix); err != nil {
		return err
	}
	_, err := cfg.Save("")
	return err
}
```
+++

`set_path` creates the section if the server has never been configured, so there is no "does this server exist yet" branch to write.

!!!info Why the save is safe
`save()` writes to a temporary file next to the real one and renames it into place. A crash mid-write leaves the old config intact rather than a truncated one, and a reader never catches the file half-written. You get that without asking for it.
!!!

!!!warning Two things to watch
**Comments are lost.** The moment a program saves a file, every `#` note in it is gone. Keep hand-annotated files separate from the ones your program owns — recipe 6 shows the usual split.

**Saving is not a queue.** If several parts of your program save the same config at once, the last write wins. For anything concurrent, funnel saves through one place.
!!!

---

## 4. One file per server

Once you have more than a handful of servers, one file each beats one big file. A parse error then takes out one server instead of all of them.

```
data/
└─ guilds/
   ├─ 1234567890.jp
   └─ 2345678901.jp
```

**`data/guilds/1234567890.jp`**

```jp
[settings]
prefix: "!"
log_channel: 9876543210

[permissions]
staff_roles: [111111111111111111, 222222222222222222]
banned_users: []
```

+++ Python
```python
import jpcl

guilds = jpcl.load_dir("data/guilds")
# {'1234567890': {'settings': {...}, 'permissions': {...}}, ...}

for guild_id, data in guilds.items():
    print(guild_id, data["settings"]["prefix"])
```
+++ JavaScript
```js
import { loadDir } from "jpcl";

const guilds = await loadDir("data/guilds");
// { "1234567890": { settings: {...}, permissions: {...} }, ... }

for (const [guildId, data] of Object.entries(guilds)) {
  console.log(guildId, data.settings.prefix);
}
```
+++ Go
```go
guilds, err := jpcl.LoadDir("data/guilds", "", false, jpcl.ParseOptions{})
if err != nil {
	return err
}

for guildID, data := range guilds {
	settings, _ := data.Get("settings")
	prefix, _ := settings.(*jpcl.Object).Get("prefix")
	fmt.Println(guildID, prefix)
}
```
+++

Each file becomes one key, named after the file without its `.jp` suffix. Go recursive with `recursive=True` / `{ recursive: true }` / the third argument, and nested files are keyed by their relative path (`"guilds/1234567890"`).

Writing one back is an ordinary `JPConfig` bound to that file:

+++ Python
```python
from jpcl import JPConfig

def guild_config(guild_id: int) -> JPConfig:
    return JPConfig.load(f"data/guilds/{guild_id}.jp", missing_ok=True)

cfg = guild_config(1234567890)
cfg.set_path("settings.prefix", "?")
cfg.save()
```
+++ JavaScript
```js
import { JPConfig } from "jpcl";

const guildConfig = (guildId) =>
  JPConfig.load(`data/guilds/${guildId}.jp`, { missingOk: true });

const cfg = await guildConfig(1234567890);
cfg.setPath("settings.prefix", "?");
await cfg.save();
```
+++ Go
```go
func guildConfig(guildID string) (*jpcl.JPConfig, error) {
	return jpcl.LoadJPConfig("data/guilds/"+guildID+".jp", true, jpcl.ConfigOptions{
		Write: jpcl.DefaultStringifyOptions(),
	})
}

cfg, err := guildConfig("1234567890")
cfg.SetPath("settings.prefix", "?")
cfg.Save("")
```
+++

---

## 5. Failing helpfully when the config is broken

If a config is malformed, the useful thing is to say so clearly and stop — not to crash with a stack trace the person running it cannot read.

+++ Python
```python
import sys
import jpcl

try:
    config = jpcl.load("data/settings.jp")
except FileNotFoundError:
    sys.exit("No data/settings.jp found. Copy settings.example.jp and fill it in.")
except jpcl.JPDecodeError as exc:
    sys.exit(f"Your config has a syntax error:\n\n{exc}")
```
+++ JavaScript
```js
import { load, JPDecodeError } from "jpcl";

let config;
try {
  config = await load("data/settings.jp");
} catch (err) {
  if (err.code === "ENOENT") {
    console.error("No data/settings.jp found. Copy settings.example.jp and fill it in.");
  } else if (err instanceof JPDecodeError) {
    console.error(`Your config has a syntax error:\n\n${err.message}`);
  } else {
    throw err;
  }
  process.exit(1);
}
```
+++ Go
```go
config, err := jpcl.Load("data/settings.jp", jpcl.ParseOptions{})
if err != nil {
	var decErr *jpcl.DecodeError
	if errors.Is(err, os.ErrNotExist) {
		log.Fatal("No data/settings.jp found. Copy settings.example.jp and fill it in.")
	} else if errors.As(err, &decErr) {
		log.Fatalf("Your config has a syntax error:\n\n%v", decErr)
	}
	log.Fatal(err)
}
```
+++

Printing the error is enough. It already contains the file, the line, the column, the offending source line and a caret under the exact character:

```
data/settings.jp:4:12: expected ':' after key 'prefix', found '"'
    prefix "!"
           ^
```

If you would rather build your own message, the error carries `.line`, `.col`, `.pos`, `.filename` and the raw message separately.

---

## 6. Organising your configs

Nothing is enforced, but this layout is what `load_dir` is built for:

```
your-project/
├─ data/
│  ├─ servers.jp            # one file per concern
│  ├─ roles.jp
│  ├─ servers.example.jp    # committed template, safe to publish
│  └─ guilds/               # optional: one file per entity
│     ├─ 1234567890.jp
│     └─ 9876543210.jp
└─ src/
```

A few habits that save pain later:

>>> *One file per concern*
A parse error then takes out one feature, not everything.

>>> *Keep live data out of git, and commit a template instead*
```gitignore
data/*.jp
!data/*.example.jp
```
The template is where your comments live — it is the one file no program will ever rewrite, so the notes survive.

>>> *Use IDs as section names*
`[1234567890]` parses to the string key `"1234567890"`, and integer keys are stringified on write, so `{1234567890: {...}}` round-trips.

>>> *Write through `save()` rather than by hand*
So an interrupted write cannot truncate a live config.

>>> *Validate in CI*
`jpcl check data/*.jp` — see recipe 8.
>>>

A template worth copying:

**`data/servers.example.jp`**

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

---

## 7. Migrating an existing JSON config

One command:

```bash
jpcl from-json config.json -o config.jp
jpcl check config.jp
```

Every top-level object in the JSON becomes a `[SECTION]`. Top-level scalars are hoisted above the first header.

Open the result and add the comments JSON never let you write. If the JSON was hand-maintained, this is usually where you discover which settings nobody remembers the purpose of.

To go the other way — feeding a `.jp` config to something that only speaks JSON:

```bash
jpcl to-json config.jp -o config.json
jpcl to-json config.jp | jq '.bot.prefix'
```

Round-tripping `to-json` then `from-json` is lossless for data, and loses comments in the same way any rewrite does.

---

## 8. Validating in CI

A broken config should fail the build, not production. One step:

+++ GitHub Actions
```yaml
name: Check configs

on: [push, pull_request]

jobs:
  configs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.14"
      - run: pip install jpcl
      - run: jpcl check data/*.jp
```
+++ Pre-commit hook
`.git/hooks/pre-commit`:

```bash
#!/bin/sh
set -e
jpcl check data/*.jp
```

Then `chmod +x .git/hooks/pre-commit`.
+++ Makefile
```makefile
.PHONY: check
check:
	jpcl check data/*.jp
	jpcl fmt data/servers.jp | diff -q - data/servers.jp
```

The second line fails if the file isn't in canonical format, which keeps diffs clean.
+++

`jpcl check` exits non-zero if any file fails, which is all most CI systems need. `-q` suppresses the successes if you only want to see problems.

---

## 9. Handling large IDs

Discord snowflakes and other 64-bit IDs sit at the edge of what some languages hold exactly.

```jp
[1234567890123456789]
owner: 987654321098765432
staff: [111111111111111111, 222222222222222222]
```

+++ Python
Nothing to do. Python integers are arbitrary precision:

```python
data = jpcl.load("data/servers.jp")
owner = data["1234567890123456789"]["owner"]   # int, exact
```
+++ JavaScript
Integers past 2^53 arrive as `bigint`, so every digit survives:

```js
const data = await load("data/servers.jp");
const owner = data["1234567890123456789"].owner;  // 987654321098765432n

String(owner);              // "987654321098765432" -- for display or an API call
owner === 987654321098765432n;  // true -- compare against a bigint literal
```

`bigint` does not mix with `number` (`owner + 1` throws) and `JSON.stringify` refuses it outright. Convert deliberately when you need to:

```js
String(owner)   // for display, URLs, API calls
Number(owner)   // only if you accept precision loss
```

Force one or the other at parse time if you prefer:

```js
await load("data/servers.jp", { integers: "bigint" });  // everything is bigint
await load("data/servers.jp", { integers: "number" });  // everything is number
```
+++ Go
Integers are `int64`, which holds every snowflake comfortably:

```go
data, _ := jpcl.Load("data/servers.jp", jpcl.ParseOptions{})
section, _ := data.Get("1234567890123456789")
owner, _ := section.(*jpcl.Object).Get("owner")
id := owner.(int64)   // exact
```

Beyond roughly 9.2×10^18 a value falls through to `float64` and loses precision, but nothing Discord issues comes close.
+++

**Section names are always strings.** `[1234567890]` gives you the key `"1234567890"`, in every implementation, so look it up with a string.

---

## 10. Reading a config in the browser

`jpcl/core` is the parser and writer without any filesystem access, for browsers, workers and edge runtimes:

```ts
import { loads, JPDecodeError } from "jpcl/core";

const text = await fetch("/config.jp").then((r) => r.text());

try {
  const config = loads(text);
  console.log(config.theme.colour);
} catch (err) {
  if (err instanceof JPDecodeError) {
    console.error(`Line ${err.line}, column ${err.col}: ${err.rawMessage}`);
  }
}
```

This also makes a decent in-browser config editor: parse what the user typed, show the error with its line and column if it fails, and `dumps` it back out when it doesn't.

JavaScript only — Python and Go have no browser equivalent.

---

## Next

>>> *[Writing .jp files](/jpcl/syntax)*
Every rule of the format.

>>> *[When it goes wrong](/jpcl/errors)*
Every error message, explained.

>>> *[Command line](/jpcl/cli)*
The `jpcl` tool in full.
>>>
