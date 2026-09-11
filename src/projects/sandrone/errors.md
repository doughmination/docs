---
order: 600
label: Error messages
icon: alert
---

# Error messages

What Sandrone says when something goes wrong, and what it actually means. Most of these are the bot telling you something useful rather than falling over.

Anything in this section that starts *"I am missing"* or *"You are missing"* is only ever shown to you, not the channel.

## Permission errors

>>> *"I am missing `Embed Links` to run this command."*
The **bot's** role doesn't have that permission in this channel. Nearly every reply is an embed, so this one stops almost everything.

Check the channel's permission overwrites as well as the role itself — a channel-level deny beats a server-level allow. The permission table in [Adding Sandrone](/projects/sandrone/setup/#permissions-sandrone-asks-for) lists what each command needs.

>>> *"I am missing `Attach Files` to run this command."*
Same thing, for a command that sends an image or a file: `/qr`, `/pride`, `/json` with long output.

>>> *"You are missing … to run this command."*
**You** don't have the permission, not the bot. Ask a moderator.

>>> *"You do not have permission to execute this command"*
An owner-only command — the `/cog` group. These are listed in the docs for completeness; they aren't for general use.

>>> *"This command can only be used in a server."*
Some things genuinely need server context and can't work in a DM.
>>>

Sandrone checks permissions **before** it starts work, not halfway through, so a permission error never leaves you with a half-finished reply.

## Lookups that found nothing

These are answers, not failures — the thing you asked about doesn't exist or isn't visible.

| Message | What it means |
|---|---|
| *"That GitHub account does not exist."* | No such user. Check the spelling; GitHub usernames are not case-sensitive but they are exact. |
| *"That repository does not exist."* | No such repo, or it's private. |
| *"Give the repository as `username/repo`."* | The input wasn't in `owner/name` shape. A full GitHub URL works too. |
| *"That Codeberg account does not exist."* | No such user on Codeberg. |
| *"That Wikipedia page does not exist…"* | Try different capitalisation — Wikipedia titles can be case-sensitive past the first letter. |
| *"Could not find that definition in Urban Dictionary"* | No entry for that term. |
| *"No WHOIS server published a record for `…`"* | The referral chain didn't reach a server with a record. Usually a TLD that doesn't run public WHOIS. |
| *"No WHOIS record found for `…`"* | The domain is genuinely unregistered, or the registry says it's available. |
| *"No Discord profile found for @…"* | The profile API hasn't seen that user. A coverage gap, not a fault on your side. |
| *"@… doesn't have a registered PluralKit system."* | Exactly that. |
| *"…'s current front is private."* | The system has front privacy switched on. Working as intended. |
| *"No Enka.Network record for UID `…`"* | `/genshin` found no such account. The profile may be private, unindexed, or the UID is wrong. |
| *"This account has no visible characters."* | The account exists but shows nobody. Enable *Display all your characters* on the in-game Character Showcase, or pin a few, then retry. |
| *"Enka.Network is unavailable right now"* | The upstream source is down. Sandrone says so rather than showing old figures as current; try again shortly. |

## Bad input

| Message | What it means |
|---|---|
| *"That doesn't look like a URL."* | `/yt-dlp` needs an `http://` or `https://` link. |
| *"Only YouTube links are supported."* | `/yt-dlp` accepts `youtube.com`, `youtu.be` and `music.youtube.com`, and nothing else. |
| *"That doesn't look like a `twitter.com` or `x.com` post link."* | `/tweet` needs a link to a specific post, with `/status/<id>` in it — not a profile. |
| *"That doesn't look like a `bsky.app` or `xsky.app` post link."* | `/bluesky` needs a post link, with `/profile/…/post/…` in it. |
| *"That doesn't look like a domain or IP address. Try `example.com`."* | `/whois` couldn't make sense of the input. |
| *"That doesn't look like a Genshin UID — it should be 9–10 digits."* | `/genshin` checks the shape before looking anything up. The UID is on your in-game profile. |
| *"There are no letters or digits in that to re-case."* | `/case` was given punctuation only. |
| *"That isn't valid JSON"* | With the parser's complaint, the line and column, and a caret under the exact character. |
| *"Invalid pattern"* | `/regex` couldn't compile it, with the column and a caret. |
| *"I have no unit called `…`"* | `/convert` didn't recognise the unit — pick from the suggestions, or check the [unit reference](/projects/sandrone/units/). |
| *"… is length and … is mass — those don't convert."* | Cross-category conversion. |
| *"I have no flag called `…`"* | `/pride` didn't recognise the flag name; the [flag list](/projects/sandrone/flags/) has all thirty and their aliases. |
| *"I have no model for `…`"* | `/translate` doesn't have that language code. |
| *"Those are the same language."* | `/translate` source and target match. |
| *"Could not decode that text…"* | `/decrypt` was given the wrong method for the text. |

## Upstream problems

These mean a service Sandrone depends on is having a bad day. Waiting and retrying is usually the whole fix.

| Message | Service |
|---|---|
| *"Couldn't reach Codeberg — try again in a moment."* | Codeberg |
| *"Couldn't reach Urban Dictionary — try again in a moment."* | Urban Dictionary |
| *"Couldn't reach the WHOIS servers — try again in a moment."* | The WHOIS server for that TLD |
| *"Couldn't reach girlcockx.com — try again in a moment."* | The X/Twitter embed proxy |
| *"Couldn't reach Bluesky — try again in a moment."* | Bluesky's public API |
| *"Couldn't reach the Argos package index — try again in a moment."* | The translation model index |
| *"Could not fetch profile"* / *"Account Not Found"* | The Doughmination API |

## `/yt-dlp` in particular

>>> *"That one's ~*n* MiB, over the 2048 MiB download cap. Try the Audio option?"*
Past the hard 2 GiB cap. Audio-only is much smaller.

>>> *"That link has no video stream. Try the Audio option?"*
Some YouTube URLs — many `music.youtube.com` ones — have no video track at all.

>>> *"yt-dlp couldn't fetch that: `…`"*
Something upstream refused: private, age-gated, region-blocked, deleted, or YouTube changed something and yt-dlp needs a version bump. The underlying reason is in the message.
>>>

More on all of this in [Hosted downloads](/projects/sandrone/downloads/).

## `/pride` in particular

>>> *"Couldn't download that profile picture."*
Discord's CDN didn't hand over the avatar. Usually transient.

>>> *"The animation wouldn't fit under the upload limit here, so here's a still instead."*
Not an error — the GIF was too big even at the smallest fallback size, so you got a PNG. Servers with a higher boost tier have more room.
>>>

## Anything else

>>> *"Something went wrong running `/…`: …"*
The catch-all: something failed that wasn't anticipated. The message carries the underlying error, and the same thing is printed to the bot's own console on the host.

If you can reproduce it, that message is exactly what to put in a [bug report](https://github.com/doughmination/sandrone/issues) — with the command and the options you used.
>>>

## Sandrone just refused

>>> *"No."* / *"Absolutely not."* / *"The puppet has reviewed your request. It has declined."*
Not an error. Roughly one command in ten, Sandrone turns it down on purpose with a short gold panel instead of running it. There's no cooldown — just run it again. See [Sandrone's moods](/projects/sandrone/commands/#sandrones-moods).
>>>

## Nothing at all happened

If the command didn't even appear, or nothing came back, that's an install or availability issue rather than an error — see [If something isn't working](/projects/sandrone/setup/#if-something-isnt-working).
