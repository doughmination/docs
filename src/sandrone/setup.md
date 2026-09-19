---
order: 950
label: Adding Sandrone
icon: plus-circle
---

# Adding Sandrone

Sandrone can be added two different ways, and they are not the same thing.

## Server install

Adding the bot **to a server** puts it in that server's member list, and everyone there gets its commands. You need **Manage Server** on the server to do this.

[Invite Sandrone](https://discord.com/oauth2/authorize?client_id=1539185764904996974)

Pick the server from the dropdown, review the permissions, and authorise.

## User install

Adding the bot **to your account** carries its commands with you. They then work in DMs, in group DMs, and in any server you are in — even servers that have never added Sandrone. Nobody needs to give you permission for this; it is your account.

Use the same invite link and choose **Add to My Apps** instead of a server.

Two things to know about a user install:

- Only you see the replies in a server that hasn't added the bot. Discord marks them as only visible to you.
- Commands that need the bot to actually be in the server — anything reading a member's server-specific profile — fall back to the global version.

!!!secondary
A server can turn user-installed apps off entirely, under **Server Settings → Integrations**. If your commands vanish in one particular server, that's usually why.
!!!

## Permissions Sandrone asks for

Sandrone asks for very little, and it checks before it acts rather than failing halfway through. If it is missing something it needs, it says so in a message only you can see, naming the permission.

| Permission | Used for |
|---|---|
| **Embed Links** | Nearly every command. Replies are embeds. |
| **Attach Files** | `/qr`, `/pride`, `/yt-dlp`, and `/json` when the output is too long to show inline. |
| **Send Messages** | Replying at all. |
| **Use Application Commands** | Registering the slash commands. |

It does not ask for Manage Messages, Manage Roles, Kick, Ban, Administrator, or the message content intent. It cannot read your conversations — it only ever receives what you type into a slash command's options.

If you would rather lock commands down, Discord's own **Server Settings → Integrations → Sandrone** panel lets you restrict individual commands to particular roles or channels. Sandrone respects that entirely; it has no permission system of its own beyond the owner-only `/cog` commands.

## Age-restricted commands

`/nsfwgif` is flagged age-restricted at the Discord level. That means Discord — not Sandrone — decides who may run it and where, based on your account's age setting and whether the channel is marked as age-restricted. Sandrone does not record who ran it. See the [*Command reference*](/sandrone/commands#age-restricted-18) for the detail.

## Where commands work

| Context | Server install | User install |
|---|---|---|
| A server that added the bot | Yes | Yes |
| A server that didn't | No | Yes, replies visible only to you |
| DM with Sandrone | Yes | Yes |
| DM or group DM with someone else | No | Yes |

## Removing it

- **From a server** — **Server Settings → Integrations → Sandrone → Remove App**, or just kick the bot.
- **From your account** — right-click the bot in your DM list, or find it under **Settings → Authorised Apps**.

Nothing is left behind either way. There is no account to delete and no stored data to erase, because there was never any to begin with — see [*Privacy Policy*](/sandrone/privacy).

## If something isn't working

>>> *The commands don't appear when I type `/`*
Refresh your client first: **Ctrl+R** on Windows and Linux, **⌘+R** on Mac, or swipe the app away and reopen it on mobile. Discord caches the command list aggressively and a refresh fixes it most of the time. (`/snippet refresh` posts these instructions, for pasting at someone else.)

If they still don't appear, check the bot is actually in the server, and that **Server Settings → Integrations** hasn't restricted the commands to a role or channel you're not in.

>>> *A command replies "I am missing `Embed Links` to run this command"*
Exactly what it says: the bot's role in that channel doesn't have that permission. Check the channel's permission overwrites as well as the role itself — a channel-level deny beats a server-level allow.

>>> *A command that used to exist has gone*
Commands are grouped into modules, and the owner can switch a module off with `/cog unload` — for maintenance, or because something upstream broke. It'll be back. The [repository](https://github.com/doughmination/sandrone) is the source of truth for what currently exists.

>>> *Everything is failing at once*
The bot is probably restarting, or the host is down. It usually comes back within about half an hour; the [support server](https://discord.gg/N8gCjS294R) is the place to ask. There is no uptime guarantee — see [*Terms of Service*](/sandrone/tos).
>>>
