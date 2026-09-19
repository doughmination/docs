# JavaScript & TypeScript

```bash
npm install jpcl     # or: bun add jpcl / pnpm add jpcl / yarn add jpcl
```

No runtime dependencies. Ships ESM with TypeScript types. **Node 20+**, Bun and Deno.

:icon-lockup-github: [*jpcl-lang/jpcl-npm*](https://github.com/jpcl-lang/jpcl-npm) · [*jpcl on npm*](https://www.npmjs.com/package/jpcl)

```ts
import { load } from "jpcl";

await load("data/servers.jp");
// {
//   SERVER_ID: { config: { disabled_channels: null, disabled_users: [9892, 82082, 8209] } },
//   SERVER_ID_2: { prefix: "!", modules: { moderation: true, fun: false } }
// }
```

A `.jp` document is a plain object. No wrapper class, nothing to unwrap.

---

## Values

| `.jp` type | JavaScript type |
|---|---|
| Object / section | plain object |
| Array | `Array` |
| String | `string` |
| Integer | `number`, or `bigint` beyond 2^53 |
| Float | `number` |
| Boolean | `boolean` |
| Null / empty value | `null` |

### Big numbers stay exact

JavaScript numbers lose precision past 2^53, which is exactly where Discord snowflakes and other 64-bit IDs live. By default an integer that fits safely parses to a `number`, and one that doesn't parses to a `bigint`, so no digit is ever silently changed:

```ts
jpcl.loads("[s]\nsmall: 42\nid: 111111111111111111\n");
// { s: { small: 42, id: 111111111111111111n } }
```

`bigint` values are written back as plain integers. Two overrides:

```ts
jpcl.loads(text, { integers: "bigint" });   // every integer becomes a bigint
jpcl.loads(text, { integers: "number" });   // every integer becomes a number
```

!!!warning `bigint` does not mix with `number`
`1n + 1` is a `TypeError`, and `JSON.stringify` refuses a `bigint` outright. If an ID only ever gets compared or printed, the default is what you want. If you do arithmetic on it, pass `integers: "number"` and accept the precision loss, or convert deliberately with `Number(id)`.
!!!

---

## Reading

```ts
import * as jpcl from "jpcl";

const data = await jpcl.load("data/servers.jp");   // -> object
const same = jpcl.loadSync("data/servers.jp");     // no await
const parsed = jpcl.loads(text);                   // from a string
```

Every file function has a synchronous twin — `loadSync`, `dumpSync`, `loadDirSync` — for startup code that would rather not `await`. Paths may be strings or `file:` URLs.

`parse` and `stringify` are aliases of `loads` and `dumps` if you prefer the `JSON` spelling.

### Options

```ts
jpcl.loads(text, { duplicateKeys: "last" });   // "error" (default), "first", "last"
jpcl.loads(text, { integers: "bigint" });      // "auto" (default), "bigint", "number"
jpcl.loads(text, { filename: "config.jp" });   // name shown in error messages
```

By default a repeated key is an error rather than a silent overwrite — see [*Duplicate keys*](/jpcl/syntax/#duplicate-keys).

---

## Writing

```ts
await jpcl.dump(data, "data/servers.jp");   // formatted, atomic write
jpcl.dumpSync(data, "data/servers.jp");
const text = jpcl.dumps(data);              // -> string
```

Writes are atomic by default: the file goes to a temporary neighbour and is renamed into place, so a crash or a concurrent reader never sees half a config. Missing parent directories are created.

### Options

```ts
jpcl.dumps(data, { indent: 4 });        // spaces per level (default 2)
jpcl.dumps(data, { width: 120 });       // column budget for inline arrays (default 88)
jpcl.dumps(data, { sortKeys: true });   // alphabetical instead of insertion order
jpcl.dumps(data, { ensureAscii: true }); // escape non-ASCII as \uXXXX
jpcl.dumps(data, {
  default: (v) => (v instanceof Date ? v.toISOString() : String(v)),
});
```

The `default` hook is how you write a `Date`, a `Set`, a class instance or anything else `.jp` has no type for. It must return a *different* type from the one it was handed.

### How JavaScript's own quirks show up

```ts
jpcl.dumps({ a: undefined, b: [undefined] });
// object properties whose value is undefined are skipped;
// undefined array elements are written as null -- same as JSON.stringify
```

A `Map` is written like an object, and may use integer keys — which is the workaround for the key-ordering issue below.

---

## Load a whole folder

```ts
const config = await jpcl.loadDir("data");            // { servers: {...}, roles: {...} }
const guilds = await jpcl.loadDir("data/guilds");     // { "1234567890": {...}, ... }
const all = await jpcl.loadDir("data", { recursive: true });
```

Each file becomes one key, named after the file without its `.jp` suffix. Nested files are keyed by their relative path (`"guilds/1234567890"`), and `pattern` picks which files count (default `"*.jp"`).

`loadDirSync` is the synchronous twin.

---

## Editing a config in place: `JPConfig`

`JPConfig` is a map-like object that remembers the file it came from.

```ts
import { JPConfig } from "jpcl";

const cfg = await JPConfig.load("data/servers.jp", { missingOk: true });

cfg.data.SERVER_ID.prefix;                               // plain object access
cfg.get("SERVER_ID");                                    // also: set, has, delete, keys, size
cfg.getPath("SERVER_ID.config.disabled_users", []);      // never throws
cfg.setPath("SERVER_ID.config.disabled_users", [9892]);  // creates missing sections
cfg.hasPath("SERVER_ID.prefix");
cfg.section("NEW_SERVER", { create: true }).prefix = "?";
cfg.merge({ SERVER_ID: { modules: { fun: true } } });    // deep merge
await cfg.save();                                        // atomic, back to its own path
await cfg.reload();                                      // discard in-memory changes
cfg.toObject();                                          // deep copy as a plain object
```

`loadSync`, `saveSync` and `reloadSync` do the same without promises.

### Constructors

```ts
await JPConfig.load("data/servers.jp");
await JPConfig.load("data/servers.jp", { missingOk: true });
JPConfig.loadSync("data/servers.jp");
JPConfig.loads(text, { path: "data/servers.jp" });
new JPConfig({ SERVER_ID: {} }, { path: "data/servers.jp" });
```

`missingOk: true` gives an empty config bound to the path, which is what a program that writes its config on first run wants:

```ts
const cfg = await JPConfig.load("data/settings.jp", { missingOk: true });
if (!cfg.hasPath("bot.token")) {
  cfg.setPath("bot.token", "");
  await cfg.save();
}
```

### Dotted paths

```ts
cfg.getPath("SERVER_ID.config.disabled_users", []);
cfg.setPath("SERVER_ID.config.disabled_users", [9892]);
cfg.hasPath("SERVER_ID.prefix");
```

`getPath` returns its fallback rather than throwing, at any depth. `setPath` creates every level it needs on the way in. Both take a third argument if `.` is an awkward separator for your keys.

Optional chaining gets you part of the way (`cfg.data?.SERVER_ID?.config?.disabled_users`), but there is no equivalent on the way *in* — `setPath` is the one that really saves you code.

### Remembered options

Formatting options given when loading or constructing are remembered by `save()`:

```ts
const cfg = await JPConfig.load("data/servers.jp", { indent: 4, sortKeys: true });
await cfg.save();                    // uses indent: 4, sortKeys: true
await cfg.save(null, { indent: 2 }); // overridden for this call only
await cfg.save("backup.jp");         // a different path; the config rebinds to it
```

---

## In the browser

`jpcl/core` is the same parser and writer without any filesystem access, for browsers, workers and edge runtimes:

```ts
import { loads, dumps, JPDecodeError } from "jpcl/core";
```

No `load`, `dump`, `loadDir` or `JPConfig` — those need a filesystem. Everything else is identical, so a config fetched over HTTP parses exactly as it would on disk:

```ts
const text = await fetch("/config.jp").then((r) => r.text());
const config = loads(text);
```

The package is marked `sideEffects: false`, so a bundler drops what you don't import.

---

## Errors

Every error derives from `JPError`.

```ts
import { JPError, JPDecodeError, JPEncodeError } from "jpcl";

try {
  await jpcl.load("data/servers.jp");
} catch (err) {
  if (err instanceof JPDecodeError) {
    console.log(err.message);     // the full message with the caret line
    console.log(err.line);        // 2
    console.log(err.col);         // 8
    console.log(err.pos);         // character offset
    console.log(err.filename);    // "data/servers.jp"
    console.log(err.rawMessage);  // the message without position or source
  }
}
```

Printed in full:

```
data/servers.jp:2:8: expected ':' after key 'prefix', found '"'
    prefix "!"
           ^
```

[*When it goes wrong*](/jpcl/errors) lists every message and what to do about each.

---

## TypeScript

Types ship with the package; nothing to install separately.

```ts
import type { JPObject, JPValue, ParseOptions, StringifyOptions } from "jpcl";
```

`load` and `loads` return `JPObject`, an index-signature object — a parsed config is unknown-shaped by definition, so you get `any` on property access rather than a lie about the shape.

If you know what your config should look like, assert it at the boundary and work with a real type from there:

```ts
interface Settings {
  bot: { name: string; prefix: string; enabled: boolean };
}

const settings = (await jpcl.load("data/settings.jp")) as unknown as Settings;
```

Better still, validate it — a config file is user input, and a type assertion does not check anything at runtime. Any schema library (Zod, Valibot, ArkType) works on the parsed object unchanged.

---

## Command line

```bash
npx jpcl check data/*.jp                      # validate; non-zero exit on failure
npx jpcl fmt -w data/servers.jp               # reformat in place
npx jpcl get data/servers.jp SERVER_ID.prefix # read one value
npx jpcl to-json data/servers.jp -o out.json
npx jpcl from-json out.json -o data/servers.jp
```

`-` reads stdin. Large integers survive `to-json` and `from-json` intact. Full reference: [*Command line*](/jpcl/cli).

---

## Differences from the other implementations

Four things, all of them JavaScript's doing rather than JPCL's:

| | |
|---|---|
| **Integers past 2^53 become `bigint`** | Python has unlimited integers; Go has `int64`. This is the one you are most likely to trip over. |
| **Whole floats become integers** | There is one number type, so `1.0` reads back as `1` and is written as `1`. |
| **Integer-like keys move to the front** | JavaScript objects always enumerate keys such as `"1234567890"` first, in ascending order, before every other key. A `[1234567890]` section therefore comes out ahead of `[SERVER_ID]` even if the file had it last. Pass a `Map` to `dumps` if that order matters. |
| **`undefined` is not a value** | Object properties holding `undefined` are skipped, and `undefined` array elements become `null` — the same rules as `JSON.stringify`. |

The first three are also listed under [*Round trips*](/jpcl/spec/#round-trips).

---

## Full API

```ts
// jpcl — everything
load(path, options?): Promise<JPObject>
loadSync(path, options?): JPObject
loads(text, options?): JPObject              // alias: parse
dump(value, path, options?): Promise<void>
dumpSync(value, path, options?): void
dumps(value, options?): string               // alias: stringify
loadDir(dir, options?): Promise<Record<string, JPObject>>
loadDirSync(dir, options?): Record<string, JPObject>

JPConfig                                     // load, loadSync, loads, new
JPError, JPDecodeError, JPEncodeError
SUFFIX, ENCODING, MAX_DEPTH, version

// jpcl/core — no filesystem
loads, dumps, parse, stringify
JPError, JPDecodeError, JPEncodeError
SUFFIX, ENCODING, MAX_DEPTH
```

`JPConfig` methods: `get`, `set`, `has`, `delete`, `keys`, `values`, `entries`, `size`, `data`, `getPath`, `setPath`, `hasPath`, `section`, `merge`, `toObject`, `dumps`, `save`, `saveSync`, `reload`, `reloadSync`.

---

## Next

>>> *[Worked examples](/jpcl/recipes)*
Complete projects using all of this.

>>> *[Command line](/jpcl/cli)*
The `jpcl` tool in full.

>>> *[When it goes wrong](/jpcl/errors)*
Every error message, explained.
>>>
