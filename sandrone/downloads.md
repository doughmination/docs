# Hosted downloads

`/yt-dlp` is the one command that leaves something on disk. This page explains exactly what, where, for how long, and who can reach it — because "the bot hosts a file for you" deserves more than a footnote.

## What happens when you run it

1. You give it a YouTube link and pick **Video** or **Audio**.
2. Sandrone checks whether it already has that exact video, in that exact format, still on disk. If it does, you get the existing link straight back and its clock resets. Nothing is downloaded again.
3. Otherwise it downloads into a fresh directory with a random name.
4. If the finished file fits your server's upload limit, it is **attached to the reply** and the directory is deleted immediately. Nothing is kept.
5. If it doesn't fit, the file **stays on the host** and you get a link, along with the resolution, the size, and when it expires.
6. Either way, anything over the size cap is refused before it can fill the disk.

The upshot: small downloads never touch storage at all. Only the ones Discord won't take get hosted.

## The link

Hosted files are served from **`sandrone.doughmination.gay`**, and only from there. A "Sandrone download" on any other domain is not one — see [Official links](/sandrone/official-links/).

The URL looks like this:

```
https://sandrone.doughmination.gay/<random-slot>/<filename>
```

The slot is a random, unguessable identifier generated per download. There is no index page, no directory listing, and no way to browse what exists — the only route to a file is the exact link.

That cuts both ways: **anyone holding the link can download the file** while it exists. Treat a hosted link the way you'd treat any unlisted URL. It is not private storage, and it isn't meant to be.

## How long it lasts

**24 hours** from the last time it was touched, and "touched" means either of:

- someone fetching the link, or
- someone running `/yt-dlp` on the same video and format again.

Both reset the clock, so a file that is actively being used doesn't get swept out from under whoever is downloading it. A file nobody touches is deleted on the next sweep after its 24 hours are up. The sweep runs hourly, so in practice a file lives for its retention window plus up to an hour.

There is no way to extend that indefinitely, and no way to pin a file. If you need it permanently, download it and keep your own copy.

## Limits

| | |
|---|---|
| **Hard cap** | 2 GiB. Anything larger is refused, with the estimated size in the message and a nudge towards the Audio option. |
| **Video quality** | Up to 1080p. MP4, preferring H.264 video and AAC audio, so it plays on everything. |
| **Audio quality** | 192 kbps MP3, with the video's metadata and its thumbnail embedded as cover art. |
| **Attachment threshold** | Your server's own upload limit, which depends on its boost tier. In DMs, 10 MB. |
| **Playlists** | Ignored. A playlist link gets you the one video. |
| **Sources** | `youtube.com`, `youtu.be`, `music.youtube.com`. Nothing else. |

## What's in the file, and what isn't

The stored file is the YouTube media and nothing else. It is **not** tagged with:

- your username or user ID,
- the server or channel you ran the command in,
- the time you ran it, or
- anything else about who asked.

Alongside it sits a small record used to make the cache work — the video's ID, the format, the title, the resolution and the size. Nothing in it identifies a person, and there is no log anywhere of who requested what. If you deleted the whole downloads folder, there would be nothing left to connect any file to any user.

## If you'd rather nothing was hosted

Two options:

- **Keep it under the attachment limit.** Pick **Audio** instead of Video, or choose a shorter video. Anything that fits gets attached and immediately deleted.
- **Don't use the command.** There's no hidden path — `/yt-dlp` is the only thing that writes media to disk.

## Copyright

Sandrone hands you a file from YouTube. Whether you have the right to that file is between you, YouTube, and whoever owns the content — not something the bot can judge.

Don't use it to redistribute things you have no right to redistribute; that's in the [Terms of Service](/sandrone/tos/), and it's the kind of thing that gets a hobby bot taken down.

## Errors you might see

>>> *"That one's ~*n* MiB, over the 2048 MiB download cap. Try the Audio option?"*
The video is bigger than the cap. Audio-only is dramatically smaller and is usually what you wanted anyway.

>>> *"That link has no video stream. Try the Audio option?"*
Some YouTube URLs are audio-only — a lot of `music.youtube.com` links are. Switch the type to Audio.

>>> *"Only YouTube links are supported."*
Exactly what it says. Sandrone does not download from other sites.

>>> *"yt-dlp couldn't fetch that: …"*
Something upstream refused: the video is private, age-gated, region-blocked, deleted, or YouTube changed something and yt-dlp needs updating. The message carries the underlying reason.
>>>
