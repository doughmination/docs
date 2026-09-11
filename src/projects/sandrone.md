---
order: 1000
label: Sandrone
icon: https://raw.githubusercontent.com/doughmination/sandrone/refs/heads/main/assets/avatar.png
---

Sandrone is an all-in-one Discord bot: lookups, text and developer tools, media embeds, pride flag avatars, YouTube downloads, and a handful of things that exist purely for fun.

You can invite the bot [*here*](https://discord.com/oauth2/authorize?client_id=1539185764904996974)

GitHub: :icon-lockup-github: [*doughmination/sandrone*](https://github.com/doughmination/sandrone)

Other mirrors are available on [*Codeberg*](https://codeberg.org/clove/sandrone) and [*dough-git*](https://backup.doughmination.gay/clove/sandrone).

### Is this the right bot?

Sandrone only ever uses three sets of links of its own:

| | |
|---|---|
| **Discord's own domains** | `discord.com` and `discord.gg` — the invite link and the support server. |
| `sandrone.is-a.bot` | The short link for the bot. It redirects here, to these docs. |
| `sandrone.doughmination.gay` | Where `/yt-dlp` serves a download too big to attach. |

That's the whole list. If something claiming to be Sandrone sends you to a different site, asks you to log in anywhere, or hands you a "download" on some other domain, it isn't this bot. [*Official links*](/projects/sandrone/official-links) has the full detail, including how to check the account itself.

### What it does

Everything is a slash command, so Discord fills in the options as you type. The [*Command reference*](/projects/sandrone/commands) lists all of them; the short version:

- **Lookups** — GitHub, Codeberg, Wikipedia, Urban Dictionary, WHOIS, Discord profiles, PluralKit systems.
- **Tools** — unit conversion, case conversion, JSON formatting, regex checking, translation, QR codes, encoding and decoding.
- **Media** — YouTube downloads, X/Twitter and Bluesky embeds, pride flag profile pictures.
- **Fun** — cats, GIFs, an 8-ball with an attitude, The Game, and `/judge`. Sandrone also has moods: she refuses roughly one command in ten, and occasionally says something stupid in whatever channel is busy.

It works in servers, in DMs, and as a [*user install*](/projects/sandrone/setup) you can carry into any server.

### Privacy, briefly

Sandrone does not have a database. It does not store anything about you, it does not track you, and there is nothing stored about you to sell. Commands are handled in memory and the result is thrown away.

The one exception is `/yt-dlp`: a download that's too large to attach to a Discord message is saved on the host and served from a link for a day, then deleted automatically. The file is the YouTube content itself, never anything about who asked for it. The [*Privacy Policy*](/projects/sandrone/privacy) covers this in full, and [*Hosted downloads*](/projects/sandrone/downloads) explains the mechanics.

!!!warning
“Sandrone” is a Genshin Impact character. This is an unofficial fan project, not affiliated with or endorsed by HoYoverse. See the [*Terms of Service*](/projects/sandrone/tos) for licensing.
!!!
