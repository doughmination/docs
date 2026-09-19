---
order: 1000
label: Start here
icon: rocket
---

# Start here

This page assumes you have never used a configuration format before, and that "the terminal" is a place you visit reluctantly. Nothing here is skippable — read it top to bottom once and you will have a working config file you can actually use.

If you already know what JSON is and just want the rules, go to [*Writing .jp files*](/jpcl/syntax) instead.

---

## What a config file is for

A program often needs to be told things: which server to run on, which features to switch off, who is allowed to use it. You *could* write those facts into the code itself, but then changing one means editing the program.

A **config file** is a separate file that holds those settings. The program reads it when it starts. You change the file, restart the program, and the behaviour changes — no code touched.

JPCL is one way to write that file. A JPCL file ends in `.jp`.

---

## What a `.jp` file looks like

Here is a complete, valid one. Read it before reading the explanation — most of it explains itself.

```jp
# Settings for my Discord bot.

[SERVER_ID]
prefix: "!"
log_channel:
disabled_users: [9892, 82082, 8209]
```

Line by line:

| Line | What it is |
|---|---|
| `# Settings for my Discord bot.` | A **comment**. Everything after a `#` is ignored. Notes to yourself go here. |
| `[SERVER_ID]` | A **section header**. It opens a named group. Everything below it belongs to that group, until the next header. |
| `prefix: "!"` | An **entry**: a *key* (`prefix`), a colon, and a *value* (`"!"`). |
| `log_channel:` | An entry with **no value**. Legal, and it means "this exists but isn't set yet". |
| `disabled_users: [...]` | An entry whose value is a **list**. Square brackets, commas between items. |

That is genuinely most of the format.

---

## Step 1 — pick a language

JPCL has three implementations. They are the same format — a file written by one is read identically by the others — so pick whichever language you are writing your program in.

If you have no idea, pick **Python**. It is the one most people learn first and the examples are shortest.

+++ Python
You need **Python 3.14 or newer**. To check, open a terminal and run:

```bash
python --version
```

If that prints `Python 3.14.0` or higher, you are fine. If it says the command is not found, or prints an older number, install Python from [*python.org/downloads*](https://www.python.org/downloads/).

!!!warning
On Windows, tick **"Add python.exe to PATH"** in the installer. If you don't, the terminal won't find it and nothing on this page will work.
!!!
+++ JavaScript
You need **Node 20 or newer** (Bun and Deno also work). To check:

```bash
node --version
```

If that prints `v20.0.0` or higher, you are fine. Otherwise install Node from [*nodejs.org*](https://nodejs.org/).
+++ Go
You need **Go 1.27 or newer**. To check:

```bash
go version
```

If it prints `go version go1.27.1` or higher, you are fine. Otherwise install Go from [*go.dev/dl*](https://go.dev/dl/).
+++

---

## Step 2 — install JPCL

Make a folder for your project first, and open a terminal *inside it*. Then:

+++ Python
```bash
pip install jpcl
```

If `pip` isn't found, try `python -m pip install jpcl` instead.
+++ JavaScript
```bash
npm install jpcl
```

If your project has no `package.json` yet, run `npm init -y` first.
+++ Go
```bash
go get github.com/jpcl-lang/jpcl-go
```

If your project has no `go.mod` yet, run `go mod init example.com/myproject` first.
+++

To check it worked, run:

```bash
jpcl --version
```

It should print a version number. If the command isn't found, that's normal in some setups — use `python -m jpcl` (Python) or `npx jpcl` (JavaScript) everywhere this guide says `jpcl`.

!!!info Go and the command line
`go get` installs the *library*, not the command. If you want the `jpcl` command in Go, install it separately with `go install github.com/jpcl-lang/jpcl-go/cmd/jpcl@latest`.
!!!

---

## Step 3 — write your first file

Make a folder called `data`, and inside it a file called `settings.jp`. Put this in it:

```jp
# My first JPCL file.

[bot]
name: "Clanker"
prefix: "!"
enabled: true
max_warnings: 3
```

Save it.

!!!info Why a `data` folder?
Nothing forces it. It is just a tidy habit — configs in one place, code in another. [*Worked examples*](/jpcl/recipes) has a layout that scales.
!!!

---

## Step 4 — check it before you trust it

Before wiring it into any program, ask JPCL whether the file is actually valid:

```bash
jpcl check data/settings.jp
```

Success looks like this:

```
ok  data/settings.jp
```

If instead you get something with a `^` pointing at a character, the file has a typo. Don't panic — that arrow is pointing straight at the problem. [*When it goes wrong*](/jpcl/errors) explains every message JPCL can produce.

Get into the habit of running `jpcl check` after every hand edit. It takes half a second, and it turns "my program crashes on startup and I don't know why" into "line 4 has a missing brace".

---

## Step 5 — read it from a program

Now the part that matters. Make a file next to your `data` folder and paste in the whole thing:

+++ Python
Save as `main.py`:

```python
import jpcl

data = jpcl.load("data/settings.jp")

print(data)
print(data["bot"]["name"])
print(data["bot"]["max_warnings"] + 1)
```

Run it:

```bash
python main.py
```
+++ JavaScript
Save as `main.js`:

```js
import { load } from "jpcl";

const data = await load("data/settings.jp");

console.log(data);
console.log(data.bot.name);
console.log(data.bot.max_warnings + 1);
```

Add `"type": "module"` to your `package.json`, then run:

```bash
node main.js
```
+++ Go
Save as `main.go`:

```go
package main

import (
	"fmt"

	"github.com/jpcl-lang/jpcl-go"
)

func main() {
	data, err := jpcl.Load("data/settings.jp", jpcl.ParseOptions{})
	if err != nil {
		panic(err)
	}

	bot, _ := data.Get("bot")
	name, _ := bot.(*jpcl.Object).Get("name")
	fmt.Println(name)
}
```

Run it:

```bash
go run main.go
```
+++

You should see `Clanker` printed back at you. That is the whole loop: a file on disk becomes ordinary data in your program.

Notice what happened without you asking. `enabled: true` came through as a real boolean, not the text `"true"`. `max_warnings: 3` came through as a number you can do arithmetic on. JPCL works out the type from how the value is written — [*Values*](/jpcl/syntax/#values) has the full list.

---

## Step 6 — change it from a program

Reading is half of it. Programs often need to *write* settings back — remembering a choice, recording a first run.

Use `JPConfig`. It is a wrapper that remembers which file it came from, so you never have to repeat the path:

+++ Python
```python
from jpcl import JPConfig

cfg = JPConfig.load("data/settings.jp")

cfg.set_path("bot.prefix", "?")
cfg.save()
```
+++ JavaScript
```js
import { JPConfig } from "jpcl";

const cfg = await JPConfig.load("data/settings.jp");

cfg.setPath("bot.prefix", "?");
await cfg.save();
```
+++ Go
```go
cfg, err := jpcl.LoadJPConfig("data/settings.jp", false, jpcl.ConfigOptions{
	Write: jpcl.DefaultStringifyOptions(),
})
if err != nil {
	panic(err)
}

cfg.SetPath("bot.prefix", "?")
cfg.Save("")
```
+++

Open `data/settings.jp` again and `prefix` will now be `"?"`.

`bot.prefix` is a **dotted path** — "inside `bot`, the key `prefix`". It saves you reaching through each level by hand, and on the way *in* it creates any missing levels for you.

!!!warning Comments are lost on save
Look closely at the file you just saved: the `# My first JPCL file.` comment is gone. A program rewriting a file reproduces the *data*, and JPCL does not keep comments through that round trip. Keep hand-annotated files separate from the ones your program writes to. [*Round trips*](/jpcl/spec/#round-trips) covers this, and the one other thing that shifts.
!!!

---

## The five mistakes everyone makes

These account for almost every error you will hit.

### 1. Starting a value on the next line

```jp
# WRONG
config:
  { retries: 3 }
```

```jp
# RIGHT
config: {
  retries: 3
}
```

A value must **begin on the same line as its colon**. It can then wrap onto as many following lines as it likes, but the opening `{` or `[` has to be up there with the `:`. This is the price of `timeout:` meaning "empty" — otherwise JPCL could not tell the two apart.

### 2. Forgetting a closing bracket

```jp
# WRONG
users: [1, 2, 3
```

Every `[` needs a `]` and every `{` needs a `}`. The error message says `unterminated array: missing ']'` and points at where the array started.

### 3. Using `=` instead of `:`

```jp
# WRONG
prefix = "!"
```

```jp
# RIGHT
prefix: "!"
```

JPCL uses colons, like JSON. TOML and `.env` files use `=`; JPCL does not.

### 4. Not quoting a value that needs it

```jp
# WRONG -- this parses fine, but the value is just "issue".
# The # started a comment and ate the rest of the line.
label: issue #1 reported
```

```jp
# RIGHT
label: "issue #1 reported"
```

Unquoted values are fine for ordinary text, but quote anything containing a `#`, a comma, a bracket, or spaces at either end that you want to keep.

### 5. Editing the file while the program is running

Most programs read their config once, at startup. Changing the file afterwards does nothing until you restart. If your edit seems to have been ignored, restart before you go looking for a bug.

---

## Where to go next

>>> *[Writing .jp files](/jpcl/syntax)*
Every rule of the format, in plain English, with an example for each.

>>> *[Worked examples](/jpcl/recipes)*
Complete projects you can copy — a bot config, a settings file that writes itself, a folder of per-server files.

>>> *Your language's page*
[Python](/jpcl/py), [JavaScript](/jpcl/npm) or [Go](/jpcl/go) — the full API for each.
>>>
