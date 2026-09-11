---
order: 1000
label: All Commands
icon: ../../media/commands.png
---

# Commands

Everything Sandrone answers to. All of them are slash commands, so Discord will fill in the options for you as you type.

Angle brackets mean an option is required, square brackets mean it is optional. Where an option offers suggestions as you type, that's noted — Discord calls it autocomplete, and it is filtered live against what's actually available.

Nothing here is stored about you. The only command that leaves anything on disk is `/yt-dlp`, which hosts oversized downloads for a day; the [Privacy Policy](/projects/sandrone/privacy/) and [Hosted downloads](/projects/sandrone/downloads/) explain both.

---

## Bot

### `/help`

Everything Sandrone can do, in one panel. It opens on an overview; the dropdown switches between categories — the same groupings this page uses. Takes no options.

### `/stats`

Ping, uptime, hosting region, and the running version.

The region is looked up once, the first time anyone runs the command after a restart, and then reused — so it costs nothing on subsequent runs. It reports where *the bot's server* is, never anything about you. If the lookup fails it just says `Unknown`.

### `/links`

Every related link: inviting the bot, the support server, the website, and the source code. The footer carries the running version.

Every link in that reply is on one of the domains listed in [Official links](/projects/sandrone/official-links/). If you ever see a `/links` reply pointing somewhere else, you're not talking to this bot.

---

## People

### `/profile [user]`

A user's Discord profile. Defaults to you.

Shows whatever that profile actually has: user ID and status always, then pronouns, bio, banner, Nitro tier, clan tag, timezone, up to ten badges, and up to ten connected accounts. The embed takes the user's own accent colour where they have one.

This reads from the Doughmination profile API rather than from Discord directly, which is how it can see pronouns, badges and connections that a bot normally cannot. It means a user the API has never seen returns *"No Discord profile found"* — that's a gap in coverage, not an error on your side.

### `/pfp [user] [server]`

A user's profile picture, full size. Defaults to you, and to their **Global** picture.

Set `server` to **Server** for their per-server picture instead. Someone who hasn't set a different one for the server simply gets their global one back.

### `/uid [user]`

A user's Discord ID. Defaults to you.

Handy for checking a bot is the bot it claims to be — see [Checking the account itself](/projects/sandrone/official-links/#checking-the-account-itself).

---

## PluralKit

Both of these query [PluralKit](https://pluralkit.me) directly, using the Discord ID of whoever they're pointed at. If you run one on someone else, the reply is only shown to you — looking up a stranger's system shouldn't announce itself to the channel.

### `/pksystem [user]`

A user's PluralKit system: name, description, system ID, avatar, banner, tag, and pronouns, in the system's own colour. Defaults to you.

### `/pkfront [user]`

Who's currently fronting in a user's PluralKit system, with the first fronter's avatar, banner, colour and pronouns. Defaults to you.

A system with front privacy switched on returns *"that system's current front is private"* rather than leaking it, and a system with nobody fronting says so.

---

## Lookups

### `/github <username>`

Look up a GitHub user: bio, website, email, location, company, followers and following, join date, public repositories and gists, and whether they're open to being hired.

A `@` on the front is stripped, so `@octocat` and `octocat` both work.

### `/repo <repository>`

Look up a GitHub repository, given as `username/repo`. Shows the description, and stars, forks and open issues as links straight to those pages.

It'll also take a full URL — `https://github.com/user/repo`, with or without `www.`, a trailing slash, or `.git` — and reduce it down for you.

### `/codeberg <username>`

Look up a Codeberg user: description, website, email, location, followers, following, public repository count, and join date.

### `/wikipedia <query>`

Summarise a Wikipedia article — the first 500 characters of the lead section, with a thumbnail from the page and a link to the full article.

Results can be case-sensitive; if a page doesn't turn up, try adjusting your capitalisation. Landing on a disambiguation page gets you the list of possible articles instead of a summary.

### `/urban-dictionary <query>`

Look up a definition on Urban Dictionary. Returns the top definition, an example if there is one, the author, and the up and down vote counts. Cross-references in the definition — the `[bracketed]` words — become links to their own entries.

### `/whois <query>`

Look up the WHOIS record for a domain or an IP address.

Sandrone speaks the WHOIS protocol itself, on port 43. It starts at IANA, follows the referral to the registry for that TLD, then on to the registrar, up to four hops, and merges what comes back — so you get the registrar's detailed record rather than the registry's thin one.

For a **domain** it reports the registrar, registration, update and expiry dates, the registrant and country, DNSSEC status, domain status codes, and up to eight name servers.

For an **IP address** it reports the allocated range and CIDR, the network name, the organisation it's allocated to, country, allocation and update dates, and the abuse contact.

You can paste almost anything domain-shaped at it. A full URL, a `www.` prefix, an email address, a port on the end, a trailing dot — all get trimmed down to the name before it asks.

Some registries deliberately return almost nothing (GDPR redaction is the usual reason), so a sparse reply is often the registry being sparse rather than a failure. Where nothing structured comes back at all, you get the raw record.

---

## Text and developer tools

### `/qr <text>`

Turn text or a URI into a QR code image.

### `/encrypt <input> [method]`

Encode text with **Base64**, **Base32**, **ROT13**, or a **Caesar cipher** (a shift of 3). Defaults to Base64. The reply is only shown to you.

### `/decrypt <input> [method]`

Reverse one of those: decode Base64 or Base32, or undo ROT13 or a Caesar cipher. Defaults to Base64. The reply is only shown to you.

Picking the wrong method for the text gives you an error rather than nonsense.

!!!warning
These are *encodings*, not security. Base64 and Base32 hide nothing at all, and ROT13 and Caesar are puzzles from before computers existed. Do not use any of them for anything that matters.
!!!

### `/case <text> <to>`

Convert text between naming conventions. Up to 500 characters.

The reply shows the case it detected on the way in as well as the result, so a conversion that goes strangely usually explains itself.

| Style | `parse XML http request` becomes |
|---|---|
| lowercase | `parse xml http request` |
| UPPERCASE | `PARSE XML HTTP REQUEST` |
| Title Case | `Parse Xml Http Request` |
| camelCase | `parseXmlHttpRequest` |
| PascalCase | `ParseXmlHttpRequest` |
| snake_case | `parse_xml_http_request` |
| kebab-case | `parse-xml-http-request` |
| SCREAMING_SNAKE_CASE | `PARSE_XML_HTTP_REQUEST` |

Splitting is smarter than a plain space split: runs of capitals stay together unless a lowercase letter follows, so `parseXMLHttpRequest` breaks up as `parse` / `XML` / `Http` / `Request` rather than one letter at a time.

### `/json <data> [indent]`

Pretty-print compact JSON. Up to 4000 characters in, indented with **2 spaces**, **4 spaces** or a **Tab** — 2 spaces by default.

A valid document comes back with what it contains (an object with *n* keys, an array with *n* items) and the before-and-after character count. Output that fits goes inline in the embed; anything longer is attached as `formatted.json` instead.

Invalid JSON gets you the parser's own complaint, the line and column, and the offending line with a caret under the exact character — which is usually enough to spot the missing comma without reading the whole thing.

### `/regex <pattern>`

Check a regular expression and flag its problems. Up to 500 characters. Python's regex flavour.

A pattern that doesn't compile gets the error, the column, and a caret pointing at it. One that does compile gets its capturing group count, any named groups with their numbers, and its inline flags — then a check for four things that are usually mistakes:

| Flagged | Why |
|---|---|
| Leading or trailing whitespace | It has to match literally, and it's almost always a paste artefact. |
| A quantified group inside a quantifier, like `(a+)+` | On a near-miss this backtracks catastrophically. That's how a ReDoS hang happens. |
| An empty alternation branch, like `(a\|)` | Matches nothing at all; nearly always meant to be `(a)?`. |
| A pattern that matches the empty string | It will accept empty input. Check your `*` quantifiers. |

There's also a note, not a warning, if the pattern is unanchored — fine for searching, wrong if you meant to validate a whole value.

### `/convert <value> <source> <to>`

Convert a value between units, across 12 categories and around 100 units.

Both unit options suggest as you type, and they narrow each other: once one side is set, only units that actually convert into it are offered. Symbols are matched exactly before names, so `b` stays bits and `B` stays bytes. Common shorthand works too — `kph`, `lbs`, `floz`, `mph`, `kcal`.

Asking for something impossible tells you why rather than returning a number: converting metres into kilograms says so in as many words.

The full unit list is on the [Unit reference](/projects/sandrone/units/) page.

### `/translate <text> <source> [to]`

Translate text between languages. Up to 1000 characters, into English unless you say otherwise.

Translation runs locally through Argos Translate — an offline neural model, not a cloud API. Your text is not sent to a translation service.

Both language options suggest as you type. The first use of a given language pair downloads its model, so it can take a moment; after that it's instant.

Where no direct model exists, Sandrone pivots through English — Japanese to German becomes Japanese to English to German — which works, at the usual cost of translating twice. A pair that can't be reached even that way says so.

Titles in Japanese-style brackets — `『…』` and `《…》` — are lifted out before translation and put back afterwards, so a song or book title inside a sentence survives intact instead of being mangled into English.

---

## Media

### `/yt-dlp <url> [type]`

Fetch a YouTube link as a video or audio file. Defaults to video.

Video comes back as MP4, up to 1080p, preferring H.264 and AAC so it plays everywhere. Audio comes back as a 192 kbps MP3, with the video's metadata and its thumbnail embedded as cover art.

A result that fits your server's upload limit is attached directly; a bigger one is hosted and linked instead, for 24 hours. Asking for the same video again within that window returns the same link and resets its timer. Anything over the 2 GiB cap is refused.

Only YouTube links work — `youtube.com`, `youtu.be` and `music.youtube.com`. Playlists are ignored; you get the one video.

[Hosted downloads](/projects/sandrone/downloads/) covers the hosting side in full: what is kept, for how long, who can reach it, and how to avoid it.

### `/tweet <url>`

Embed an X/Twitter post, video included, via a third-party proxy.

Takes any `twitter.com` or `x.com` post link, including `www.` and `mobile.` variants. The embed carries the text, the first image (or the video's thumbnail), likes, retweets, replies and views, and a count of any further attachments. `@mentions` in the text become links.

Where the post has a video, the direct video link is posted as a second message, so Discord plays it inline.

### `/bluesky <url>`

Embed a Bluesky post, video included, via a third-party proxy.

Takes `bsky.app` links and the usual embed-fixer mirrors — `xsky.app`, `fxbsky.app`, `bskyx.app`, `bskx.app`, `psky.app`, `cbsky.app`. Handles images, video, and quote posts. Deleted, blocked and private posts each say which.

### `/pride [options]`

Put a pride flag around someone's profile picture — 30 flags, two at once if you want, as a still PNG or a spinning GIF.

Every option is documented on the [Pride flags](/projects/sandrone/flags/) page, which also lists all 30 flags and the names they answer to.

### `/snippet <snip>`

Repost a commonly used explainer, so you don't type it out for the hundredth time.

| Snippet | Covers |
|---|---|
| `adb` | The active developer badge, and the fact it was removed on 5 December 2025. |
| `refresh` | How to refresh your Discord client on each platform. |
| `pluralkit` | What PluralKit is. |
| `plural` | What plural.gg is. |
| `plurality` | What plurality is, with links to further reading. |
| `userproxies` | A guide to setting up a userproxy. |

---

## Fun

### `/kitty`

A random cat.

### `/fungif <gif>`

One of a small set of cat GIFs, picked by name: `angry-cat`, `bitey-cat`, `meow-cat`, `sus-cat`, `want-pats`.

### `/explain-the-game`

You just lost the game. Posts all eight rules, so everyone else loses it too.

### `/8ball <question>`

Ask Sandrone a yes-or-no question. Don't expect her to be nice about it.

### `/pi`

The first 100 digits of pi, in ten-digit groups.

### `/judge [user]`

Sandrone delivers a verdict on someone. Defaults to you.

It's a personality feature, not a moderation tool — the verdicts are nonsense, and a given person always gets the same one, so she appears to hold a grudge she does not actually remember anywhere. A few people have hand-written verdicts; everyone else gets one picked from a pool. Nobody is pinged by being judged.

---

## Sandrone's moods

Sandrone has a personality layer that runs without you asking for it. None of it is stored anywhere and none of it survives a restart.

**She refuses commands.** Roughly one command in ten gets turned down with a short, rude panel instead of running. Run it again. There is no cooldown and no grudge; it's a dice roll every time.

**She has incidents.** Every so often, in whatever channel has been active, Sandrone posts something stupid and unprompted — the puppet has found the vents, a tool has gone missing, that sort of thing. It only fires where the conversation already is, never in a dead channel or a DM.

---

## Genshin Impact

### `/genshin <uid>`

Look up any Genshin Impact account by UID and browse its characters as rendered cards. Also answers to `/gi` and `/uidlookup`.

The UID is the 9–10 digit number on your in-game profile. Sandrone checks the shape before asking anything, so a typo comes back straight away rather than as a failed lookup.

**Reading the card.** Each character is drawn as an image — build, weapon and artifacts — with the accent colour following their element. The line underneath gives the name, level, where the data came from, and your position in the roster:

- 🟢 **Live showcase** — pinned to the in-game Character Showcase, so the build is current.
- ⚪ **Last known** — owned, but not currently pinned, so the build is the last one seen.

**Moving around.** ◀ and ▶ step through the roster, highest level first. The dropdown jumps straight to a character; on accounts with more than 25, **◀ names** and **names ▶** page the dropdown itself.

Anyone can press the buttons, not just whoever ran the command, and they keep working after a restart rather than going dead.

**If nothing comes back.** An account with no visible characters needs *Display all your characters* enabled on the in-game Character Showcase, or a few pinned. A UID with no record at all is usually a private or unindexed profile — or a wrong number.

Data comes from the Doughmination API, which sources it from Enka.Network. When Enka is unreachable Sandrone says so rather than showing stale figures as current.

---

## Age-restricted 18+

Discord decides who may run these, and where, from your account's age setting and the channel's rating. Sandrone does not record who used them.

### `/nsfwgif <gif>`

An age-restricted GIF, picked by name from a fixed set.

---

## Owner only — restricted

These manage the bot itself and only answer to the owner. They are listed for completeness.

Sandrone's commands are grouped into modules — cogs — that can be switched on and off without restarting the bot. That's how a broken or noisy command gets pulled at short notice.

Modules are named by category, like `fun.eightball` or `tools.regex`; a couple sit at the top level and are just `genshin` and `snippets`. Both `load` and `unload` suggest the valid names as you type.

### `/cog list`

Show which command modules are loaded, and whether that survives a restart.

### `/cog load <name>`

Switch a command module back on. Stays on across restarts.

### `/cog unload <name>`

Switch a command module off. Stays off across restarts.

### `/incident`

Force a puppet incident into the last active channel, instead of waiting for one to happen on its own. Does nothing if there's no channel that currently qualifies.

---

## Something missing?

Commands come and go as the bot gets worked on. The [bot's repository](https://github.com/doughmination/sandrone) is the source of truth.

If a command you were using has disappeared, it's most likely been unloaded temporarily — see [If something isn't working](/projects/sandrone/setup/#if-something-isnt-working).
