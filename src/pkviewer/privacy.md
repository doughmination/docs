---
order: 10
label: Privacy
icon: ../media/legal.png
---

pkviewer is a presentation layer over PluralKit. Most of what appears on a pkviewer page is not pkviewer's to begin with — it belongs to PluralKit, and pkviewer only shows what PluralKit already makes public.

### What pkviewer shows

Only public PluralKit information. If you keep a member private in PluralKit, that member does not appear on pkviewer, and pkviewer offers no way to tell that a private member exists — a private member and a member that never existed are indistinguishable.

Signing in as the system's owner does not change this. pkviewer reads the same public information a visitor would.

### What pkviewer stores

Its own presentation data, and nothing more: your account and the Discord account linked to it, which systems you manage, your chosen addresses, your appearance and layout settings, any social links you add, and any [badges](/pkviewer/badges) pkviewer has offered you along with your answer.

pkviewer keeps a short-lived copy of PluralKit's public responses so pages load quickly and keep working if PluralKit is briefly unavailable. When a page is showing saved information, it says so.

### Your Discord account

pkviewer asks Discord only for your account ID and username, and only to confirm who you are. It does not read your servers, your messages, or your email, and it does not display your Discord account on any public page unless you add it yourself as a social link.

### PluralKit tokens

pkviewer does not require a PluralKit token, and no feature depends on one. If you choose to use one to confirm a claim, it is used within a single request and then discarded — never saved, never written to a log, never sent to a browser.

### Links you add

Social links are shown as links and nothing else. pkviewer does not visit them, does not fetch previews, and does not load anything from them on your visitors' behalf.

### Public pages

Public pages need no account and set no login cookie, whether or not you happen to be signed in. A public page is built from public information only — it never contains your account, your Discord link, or anything about your session — so what a visitor sees is exactly what you see.

pkviewer serves public pages, sign-in and management from one address. Nothing on a public page can execute: there is no custom JavaScript, no HTML you can write, and no free-form CSS. Appearance settings are a fixed list of validated choices, and social links are shown as links and never loaded.

Public pages can be found by search engines. Your management pages cannot — they are marked so search engines skip them.

### Badges and credits

A badge only appears on your page after you accept it, and you can hide or decline it at any time. A hidden badge is indistinguishable from one that was never given.

Credits are different: an entry on the credits page is a name someone chose to be credited under, added by hand, and connected to no account. If you would rather not be listed, or want the wording changed, ask and it will be changed.

### Deleting things

Releasing a system removes management access and hides its settings. The configuration is kept for thirty days so a change of mind is recoverable, and the address is held for seven days before anyone else can take it. After that the configuration is gone.
