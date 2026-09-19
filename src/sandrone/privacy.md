---
order: 50
label: Privacy Policy
icon: ../media/legal.png
---

> Effective 17 September 2026

Sandrone is a personal, hobby Discord bot. It has no analytics and no user accounts. It keeps three small local databases and writes a couple of kinds of file to its host, and this policy explains what's in each of them.

### What Sandrone stores about you

Sandrone keeps three small local databases on its host, alongside the files described below.

- **mods**, per server: the user IDs and role IDs a server's own admins have granted permission to change Sandrone's settings there, via `/mod add`. Nothing is added unless someone with Manage Server puts it there, and `/mod reset` clears a server's list at any time.
- **incidents**, per server: the IDs of channels a server's admins have told the puppet-incident feature to skip, via `/incidents ignore`. Just channel IDs — no user data.
- **errors**, one log shared across every server: up to the 50 most recent distinct faults, kept until the bot owner clears them with `/debug errors clear`. Each entry records the command that failed, the error text, a traceback, and a label — name and ID — for the user, channel and server involved, so the fault can be found and fixed. It is visible only to the bot owner, through owner-only `/debug` commands, and is never posted publicly.

Beyond those, the bot writes `cog_state.json`, which records which command modules the owner has switched off, and it caches the game artwork `/genshin` draws its cards from — Genshin Impact character art, not anything of yours. Neither of those two contains user data of any kind.

### Hosted downloads (`/yt-dlp`)

`/yt-dlp` fetches a YouTube link as a file. When the result is small enough it is attached to the reply and nothing is kept. When it is too big to attach, the file is saved on the bot's host and the reply links to it instead.

- What is stored is the media file itself, the YouTube video or audio. It is not tagged with your name, your user ID, the server, or the time, and there is no index of who asked for what. Alongside it sits a small record used for caching — the video's ID, the format, the title, the resolution and the size — which likewise identifies no one.
- Each file lives in its own directory with a random, unguessable name. Anyone who has the link can download it while it exists; it is not otherwise listed or searchable.
- Files are served only from `sandrone.doughmination.gay`. See [Official links](/sandrone/official-links/).
- A sweep deletes each file once it is older than the retention window, 24 hours by default. Asking for the same video again while its copy still exists hands back the same link and restarts that 24-hour timer, so a file in active use is not swept out from under it; fetching the link directly does the same.
- The bot refuses to keep any single file larger than a configured cap (2 GiB by default), so one large pull cannot fill the host's disk.

If you would rather a download not sit on the host at all, keep it under your server's attachment limit, or do not use the command. [Hosted downloads](/sandrone/downloads/) covers the mechanics in full.

### What Discord sends when you run a command

When you use a slash command, Discord delivers your user ID, your username, and whatever text you typed into the command's options. Sandrone uses these in memory to build its reply and then discards them. They are not written down, not retained after the reply is sent, and not shared with anyone beyond the third parties listed below.

Two things can outlast the reply. One is a hosted `/yt-dlp` file, covered above — the media only, never your ID or the text you typed. The other is an entry in the errors database, covered above, if the command itself failed: that entry does record your name, your ID, and whatever you typed if that's what caused the fault.

Sandrone does not have the message content intent. It cannot read your conversations — only what you type into a slash command's options.

For the puppet-incident feature ([Sandrone's moods](/sandrone/commands/#sandrones-moods)), the bot does notice when a message is sent, so it knows which channel is currently active. All it keeps is a single "most recently active channel" pointer, held in memory and wiped on restart. It records no message content — it cannot read any — no author, and no history; the previous channel is simply overwritten by the next.

### Third-party services

Some commands work by asking another service on your behalf. When you use one, the search term you typed is sent to that service, which has its own privacy policy and its own logs. Sandrone has no control over what they keep.

- **GitHub**, which receives the username or repository you name in `/github` and `/repo`.
- **Codeberg**, which receives the username you name in `/codeberg`.
- **Wikipedia**, which receives your search term from `/wikipedia`.
- **Urban Dictionary**, which receives your search term from `/urban-dictionary`.
- **PluralKit**, which is linked by `/snippet`'s plurality explainer, and which `/pksystem` and `/pkfront` query directly. Those two send the Discord ID of whoever they're run on (you, or the user you name) to PluralKit's API, to look up that account's registered system.
- **Pluralpedia** and **morethanone.info**, which are linked by `/snippet`'s plurality explainer.
- **girlcockx.com**, which receives the post URL you give `/tweet`, to build the embed.
- **Bluesky's public API**, which receives the handle and post ID you give `/bluesky`. The resulting embed links to **xsky.app**.
- **WHOIS servers**, which receive the domain or IP you give `/whois`. The lookup starts at IANA and follows referrals to the registry and registrar for that name, so more than one operator sees the query. It never carries anything about you — WHOIS has no facility for it.
- **The Doughmination API** (`doughmination.uk`), which `/profile` queries with the Discord ID of whoever the command is run on, and which `/genshin` queries with the Genshin UID you type. That UID is a game account number you supply, not something Sandrone knows about you, and it is not recorded anywhere.
- **Enka.Network**, which is where the Doughmination API gets Genshin account data from. Your UID reaches it through that API when you run `/genshin`; nothing about your Discord account does.
- **The Argos Translate package index**, which `/translate` contacts to list and download language models. The translation itself runs locally on the bot's host — the text you type is **not** sent to a translation service.
- **cataas**, which serves `/kitty` a random image and receives nothing about you.
- **m.doughmination.gay**, the CDN that serves images and GIFs, which your Discord client fetches the way it fetches any embedded image.

One thing worth being explicit about: `/stats` calls **ip-api.com** to report where the bot's own server is hosted. It never sends your IP address, and it cannot, because the bot does not have it. The result is cached until the next restart, so it is asked at most once per run of the bot.

### Logs

When a command fails, the bot prints the same details to its own console on the host machine and records them in the errors database described above: the command's name, the error text, which can include a search term you passed if that's what caused it, and your name and ID. Console output is not stored beyond the terminal's own scrollback and is not published; the database entry persists until it ages out past 50 entries or the owner clears it.

The console also records module loads and unloads, the count of expired downloads removed by each sweep, and the channel a puppet incident was posted into (its name and server, never a user). None of it mentions a user. Nothing else is logged.

### Age-restricted commands

A small number of commands are marked age-restricted. Discord itself decides who may run them and where, based on your account age setting and the channel's rating. Sandrone does not record who used them.

### Children

Sandrone is used through Discord, which requires its users to be at least 13, or older where local law says so. It is not directed at children, and since it collects nothing, it holds no children's data to speak of.

### Your rights

Most of what Sandrone touches about you passes through in memory and is gone. What can persist is a moderator entry in the **mods** database, if a server's admin put you there, or your name and ID in an **errors** entry, if a command you ran happened to fail. Email `admin@doughmination.win` to ask what's held about you or to have it removed; a moderator entry can also just be revoked with `/mod remove` by anyone with Manage Server in that server. The bot's entire source is public and linked at the bottom of this page, if you would like to check any of this for yourself.

Your data on Discord itself is a separate matter, governed by [Discord's Privacy Policy](https://discord.com/privacy).

### Changes

If this policy changes, the effective date above changes with it, and the edit will be visible in the site's commit history.

### Contact

Questions about this policy, or anything else privacy-related: email `admin@doughmination.win`, or ask in the [support server](https://discord.gg/N8gCjS294R).

The source is at [github.com/doughmination/sandrone](https://github.com/doughmination/sandrone), and every claim on this page can be checked against it.
