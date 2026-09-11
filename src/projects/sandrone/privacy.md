---
order: 50
label: Privacy Policy
icon: ../../media/legal.png
---

> Effective 11 September 2026

Sandrone is a personal, hobby Discord bot. It has no database, no analytics, and no user accounts. It writes three kinds of file to its host, none of them about you, and this policy explains the little that does happen.

### What Sandrone stores about you

Nothing. There is no database, no log of who ran what, and no profile kept on any user. The bot writes `cog_state.json`, which records which command modules the owner has switched off; it writes the temporary media files described below; and it caches the game artwork `/genshin` draws its cards from, which is Genshin Impact character art, not anything of yours. None of the three contains user data of any kind: nothing links a stored file to the person who requested it.

### Hosted downloads (`/yt-dlp`)

`/yt-dlp` fetches a YouTube link as a file. When the result is small enough it is attached to the reply and nothing is kept. When it is too big to attach, the file is saved on the bot's host and the reply links to it instead.

- What is stored is the media file itself, the YouTube video or audio. It is not tagged with your name, your user ID, the server, or the time, and there is no index of who asked for what. Alongside it sits a small record used for caching — the video's ID, the format, the title, the resolution and the size — which likewise identifies no one.
- Each file lives in its own directory with a random, unguessable name. Anyone who has the link can download it while it exists; it is not otherwise listed or searchable.
- Files are served only from `sandrone.doughmination.gay`. See [Official links](/projects/sandrone/official-links/).
- A sweep deletes each file once it is older than the retention window, 24 hours by default. Asking for the same video again while its copy still exists hands back the same link and restarts that 24-hour timer, so a file in active use is not swept out from under it; fetching the link directly does the same.
- The bot refuses to keep any single file larger than a configured cap (2 GiB by default), so one large pull cannot fill the host's disk.

If you would rather a download not sit on the host at all, keep it under your server's attachment limit, or do not use the command. [Hosted downloads](/projects/sandrone/downloads/) covers the mechanics in full.

### What Discord sends when you run a command

When you use a slash command, Discord delivers your user ID, your username, and whatever text you typed into the command's options. Sandrone uses these in memory to build its reply and then discards them. They are not written down, not retained after the reply is sent, and not shared with anyone beyond the third parties listed below.

The one thing that can outlast the reply is a hosted `/yt-dlp` file, covered above, and that is the media only, never your ID or the text you typed.

Sandrone does not have the message content intent. It cannot read your conversations — only what you type into a slash command's options.

For the puppet-incident feature ([Sandrone's moods](/projects/sandrone/commands/#sandrones-moods)), the bot does notice when a message is sent, so it knows which channel is currently active. All it keeps is a single "most recently active channel" pointer, held in memory and wiped on restart. It records no message content — it cannot read any — no author, and no history; the previous channel is simply overwritten by the next.

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

When a command fails, the bot prints the command's name and the error text to its own console on the host machine. That output can include a search term you passed if the term is what caused the error. It is not stored long-term, not published, and not linked to your account.

The console also records module loads and unloads, the count of expired downloads removed by each sweep, and the channel a puppet incident was posted into (its name and server, never a user). None of it mentions a user. Nothing else is logged.

### Age-restricted commands

A small number of commands are marked age-restricted. Discord itself decides who may run them and where, based on your account age setting and the channel's rating. Sandrone does not record who used them.

### Children

Sandrone is used through Discord, which requires its users to be at least 13, or older where local law says so. It is not directed at children, and since it collects nothing, it holds no children's data to speak of.

### Your rights

Because nothing about you is stored, there is nothing to export, correct, or delete, so a deletion request would have no data to act on. If you would like to check that for yourself, the bot's entire source is public and linked at the bottom of this page.

Your data on Discord itself is a separate matter, governed by [Discord's Privacy Policy](https://discord.com/privacy).

### Changes

If this policy changes, the effective date above changes with it, and the edit will be visible in the site's commit history.

### Contact

Questions about this policy, or anything else privacy-related: email `admin@doughmination.win`, or ask in the [support server](https://discord.gg/N8gCjS294R).

The source is at [github.com/doughmination/sandrone](https://github.com/doughmination/sandrone), and every claim on this page can be checked against it.
