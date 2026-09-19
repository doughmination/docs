---
order: 900
label: Official links
icon: verified
---

# Official links

Discord bots get impersonated. A copied avatar, a copied name and a link to somewhere that wants your password is a two-minute job, so this page is the reference for what the real Sandrone actually looks like and where it will actually send you.

## The only domains Sandrone uses

Sandrone has three sets of links of its own, and nothing else:

| Domain | What it is |
|---|---|
| `discord.com`, `discord.gg` | Discord's own. The invite link is a `discord.com/oauth2/authorize` URL; the support server is a `discord.gg` invite. |
| `sandrone.is-a.bot` | The bot's short link. It redirects to these docs. |
| `sandrone.doughmination.gay` | The download host. `/yt-dlp` links point here when a file is too big to attach to a message. |

Nothing else on that list. In particular, Sandrone will **never**:

- ask you for your password, your token, or a 2FA code — for any service, ever;
- ask you to "verify", "re-authorise" or "re-link" your Discord account;
- send you a login page, a QR code to scan with the Discord app, or a browser extension;
- DM you first, unless you ran a command in a DM;
- hand you a download link on any domain other than `sandrone.doughmination.gay`.

!!!danger
Discord's own QR login is the one to watch for. If *anything* — a bot, a "giveaway", a "Nitro offer" — asks you to scan a QR code with your Discord app, that is someone taking over your account. Sandrone has no QR login and never will. `/qr` only turns text you typed into an image.
!!!

## Checking the account itself

The genuine bot's application ID is **`1539185764904996974`**. That number is the one thing an impersonator cannot copy — the display name, avatar, banner and "about me" can all be duplicated exactly, the ID cannot.

Two ways to check it:

- Run `/uid` on the bot — `/uid user: @Sandrone` — and compare the number.
- Or turn on Developer Mode in Discord (**Settings → Advanced → Developer Mode**), right-click the bot and choose **Copy User ID**.

The invite link for the real bot has the same ID in it:

```
https://discord.com/oauth2/authorize?client_id=1539185764904996974
```

Don't rely on the **App** tag, a badge, or a screenshot — any of those can be faked or staged. The ID is the proof.

## Where the source lives

The code is public and mirrored in three places. All three are the same repository:

| | |
|---|---|
| GitHub | [github.com/doughmination/sandrone](https://github.com/doughmination/sandrone) |
| Codeberg | [codeberg.org/clove/sandrone](https://codeberg.org/clove/sandrone) |
| dough-git | [backup.doughmination.gay/clove/sandrone](https://backup.doughmination.gay/clove/sandrone) |

A fork of that code on someone else's account is not Sandrone, and running one as a public bot is not permitted — see [*Terms of Service*](/sandrone/tos) and [*Development*](/sandrone/development).

## Services Sandrone talks to

Separate question, and worth not confusing with the list above: several commands work by fetching something from a third party, so their links show up in the replies. Those are *their* domains, not Sandrone's, and they are only ever the result of a command you ran.

| Command | Reaches out to |
|---|---|
| `/github`, `/repo` | `github.com` |
| `/codeberg` | `codeberg.org` |
| `/wikipedia` | `wikipedia.org` |
| `/urban-dictionary` | `urbandictionary.com` |
| `/pksystem`, `/pkfront` | `pluralkit.me` |
| `/tweet` | `girlcockx.com`, linking back to `x.com` |
| `/bluesky` | `public.api.bsky.app`, linking to `xsky.app` |
| `/kitty` | `cataas.com` |
| `/whois` | The WHOIS servers for the domain you asked about, over port 43 |
| `/profile`, `/genshin` | `doughmination.uk` |
| `/translate` | The Argos Translate package index |
| `/stats` | `ip-api.com`, for the bot's own hosting region |
| Images and GIFs in embeds | `m.doughmination.gay`, the CDN |

The [*Privacy Policy*](/sandrone/privacy) says what each of those receives.

## Reporting an impersonator

If you find a bot pretending to be this one:

- Report it to Discord (**right-click → Report**), which is the only route that actually gets it removed.
- Tell the server's moderators, so it can be kicked in the meantime.
- Let us know in the [support server](https://discord.gg/N8gCjS294R) so other servers can be warned.

If you think you have already entered credentials somewhere: change your Discord password immediately, which invalidates every existing session, then turn on 2FA and check **Settings → Authorised Apps** for anything you don't recognise.
