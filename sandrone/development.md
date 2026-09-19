# Development

How Sandrone is put together, and how to run it locally if you're contributing.

!!!warning
Read this first. Sandrone is under the **Doughmination Authorised Source Licence (DASL-1.2)**, which is source-available, not open source. Anyone may fork it and run it locally *for the purpose of preparing a contribution*. Running it as your own bot — public or private, for yourself or anyone else — needs written authorisation. See [Licensing](#licensing) at the bottom.
!!!

## Requirements

| | |
|---|---|
| Python | 3.14 |
| Package manager | [uv](https://docs.astral.sh/uv/) |
| ffmpeg | Optional. Used by `/yt-dlp`; falls back to the bundled `imageio-ffmpeg` binary if not on `PATH`. |

## Getting it running

```bash
git clone https://github.com/doughmination/sandrone.git
cd sandrone
uv sync
cp .env.example .env
# fill in BOT_TOKEN
uv run python -m sandrone
```

`BOT_TOKEN` is the only variable the bot won't start without. Everything else has a working default.

## Configuration

All configuration is environment variables, read from `.env` at startup.

| Variable | Default | What it does |
|---|---|---|
| `BOT_TOKEN` | — | **Required.** Your bot's token from the [Discord developer portal](https://discord.com/developers/applications). |
| `BOT_PREFIX` | `!` | Prefix for legacy non-slash commands. Effectively vestigial; everything is a slash command. |
| `DEV_MODE` | `false` | Set `true` to hot-reload cogs on save. See [below](#dev-mode). |
| `GITHUB_TOKEN` | — | A personal access token. Without it the `github` cog is skipped entirely at startup, and `/github` and `/repo` don't exist. |
| `DOWNLOADS_DIR` | `./downloads` | Where `/yt-dlp` puts files too big to attach. |
| `DOWNLOADS_HOST` | `0.0.0.0` | Interface the download server binds to. |
| `DOWNLOADS_PORT` | `2020` | Port it listens on. |
| `DOWNLOADS_URL` | `http://localhost:<port>` | The public base URL links point at — whatever your reverse proxy exposes the port as. |
| `DOWNLOADS_RETENTION_HOURS` | `24` | How long a hosted file survives without being touched. |
| `DOWNLOADS_MAX_SIZE_MIB` | `2048` | Refuse any single download larger than this. |

The bot's version comes from `pyproject.toml` rather than an environment variable, and is what `/stats` and `/links` report.

## Project layout

```
sandrone/          The bot itself
  __main__.py      Entry point; refuses to start without a token
  checks.py        Permission checks, and the handlers for when one refuses
  client.py        Bot subclass, prefix rules, extension discovery, dev reload, shutdown
  config.py        Environment, paths, and which cogs are disabled
  mood.py          Sandrone's personality — sassy refusals, /judge verdicts, incident lines
  website.py       Serves the landing page and sitemap
commands/          Always-on cogs (help, stats, links, admin)
  cogs/            Everything that can be unloaded, in category folders
    fun/           animals, gifs, the_game, eightball, judge, incidents
    lookups/       github, codeberg, wikipedia, ubdict, whois
    media/         bluesky, twitter, yt_dlp, pride
    nerd/          pi
    nsfw/          horny_gifs
    people/        profile, pfp, userid, pluralkit
    tools/         case, convert, decrypt, encrypt, json_format, qr, regex, translate
    genshin.py     single-file cogs that don't warrant a folder sit at the top level
    snippets.py
utils/             Shared helpers
  __init__.py      Terminal colour
  choices.py       Choice sets that accept the same spellings on slash and prefix
  components.py    Components V2 panels, plus markdown escaping and code blocks
  doughmination.py Client for the Doughmination API
  downloads.py     The download server, slot management, and expiry sweep
  genshin_card.py  Renders the Genshin character card image
  pride.py         Flag rendering, ported from pride-pfp.xyz
  usage.py         Turns a mistyped command into a panel showing its real shape
tests/             pytest
```

### The cog system

Extensions are discovered by scanning `commands/` and `commands/cogs/` for `.py` files — there is no registry to update. Add a file with a `setup(bot)` function and it loads on the next start.

The split matters:

- **`commands/`** — always loaded. `stats`, `links`, `admin`. Scanned one level deep.
- **`commands/cogs/`** — can be switched off at runtime with `/cog unload`, which writes the name into `cog_state.json` so it stays off across restarts. Scanned recursively.

Cogs in `commands/cogs/` are grouped into category folders — `fun/`, `tools/`, `lookups/`, `media/`, `people/`, `nsfw/`. A cog's **handle** — what `/cog load` and `/cog unload` take, and what's stored in `cog_state.json` — is its path under `commands/cogs/` with the slashes written as dots: `commands/cogs/fun/eightball.py` is `fun.eightball`, and a file left at the top level like `genshin.py` is just `genshin`. Adding a category is only a new folder; there is nothing to register.

A cog can also decline to load itself. The `github` cog does exactly that when `GITHUB_TOKEN` is unset: it prints a note and returns without registering, rather than failing loudly on every command.

After any load or unload the command tree is re-synced with Discord.

### Dev mode

With `DEV_MODE=true`, the bot watches `commands/cogs/` — category folders included — and reloads any file you save, then re-syncs the tree. Cogs you've disabled through `/cog unload` are skipped rather than quietly coming back.

You still need a restart for changes to `sandrone/`, `utils/`, or the always-on cogs in `commands/`.

### Permission checks

`checks.has_permissions` is a thin wrapper over discord.py's own check, with two deliberate differences:

- It reads `interaction.app_permissions`, so it's checking what the **bot** can do in this channel, not what you can.
- It passes in DMs unless `guildOnly=True`, because permissions don't apply there.

Failures raise `BotMissingPermissions`, which the global handler turns into the *"I am missing `Embed Links`…"* message, only visible to the person who ran the command. Checking up front is why a permission problem never leaves a half-sent reply.

That handler lives in the same module. `sandrone/checks.py` holds both halves of the lifecycle — the checks that decide whether a command runs, and the handlers that turn any failure, a refused check or a mistyped argument alike, into a reply — so adding a guard and giving it a voice is one edit rather than two.

### The download server

`utils/downloads.py` runs a small aiohttp server alongside the bot, serving exactly one route: `/{slot}/{name}`.

- Each download gets a **slot** — a directory named with a random URL-safe token, marked with a dotfile so the sweep can tell it apart from anything else in the folder.
- A slot's metadata records the video ID, format, title, resolution and size — enough for the cache, and nothing about who asked.
- Path traversal is refused at several points: the slot name must match a strict pattern, the resolved directory must be a direct child of the downloads root, filenames can't start with a dot or contain separators, and the resolved file's parent must be the slot itself.
- Both serving a file and re-requesting the same video touch its mtime, so active files don't expire underneath a download in progress.
- A background task sweeps hourly and deletes any slot past the retention window.

## Tests and checks

```bash
uv run pytest -q          # tests
uv run ruff check .       # lint
uv run ruff format --check .
uv run ty check           # type check
```

CI runs all four on every push and pull request, on Python 3.14. Pull requests are also mirrored to Codeberg and to the backup forge, and the contributor and activity images are regenerated on push to `main`.

The test suite deliberately covers the parts where being wrong is expensive rather than embarrassing: download slot handling and path traversal, the permission decorator, config parsing, flag name resolution, and `/yt-dlp` URL parsing.

## Contributing

Pull requests come with conditions under the licence — worth knowing before you spend an evening on one:

- **Only listed people may open pull requests.** The README names who; ask first if you're not on it.
- **You must meaningfully author your contribution.** AI is explicitly allowed as a tool — for explanation, research, spotting bugs, suggesting approaches, debugging, tests, documentation — but delegating the substantive work to it and submitting the result with a cursory glance is a licence breach, not a style preference.
- **Forking is allowed for everyone**, without asking, but only to prepare and submit contributions. A fork is not permission to run your own copy.

For **bugs**, open an issue on the [repository](https://github.com/doughmination/sandrone/issues) — anyone can do that.

For **security vulnerabilities**, email `admin@doughmination.win` rather than opening an issue. There's no bounty, but reporters get credit.

## Licensing

The full text is in [`licence.md`](https://github.com/doughmination/sandrone/blob/main/licence.md) in the repository. The short version:

| You may | Without asking |
|---|---|
| Read and study the source | Yes |
| Fork it to prepare a contribution | Yes |
| Test your changes locally | Yes |
| Submit a pull request | If you're listed as authorised |
| **Run your own instance** | **No — needs written authorisation** |
| Deploy, distribute, publish, or commercialise it | **No — needs written authorisation** |

"Sandrone" and the Doughmination branding are not covered by any of it. Authorisation to use the code is not authorisation to use the name.

And the perennial one: "Sandrone" is a Genshin Impact character. This is an unofficial fan project, not affiliated with or endorsed by HoYoverse or COGNOSPHERE.
