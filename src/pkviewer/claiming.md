---
order: 100
label: Claiming a system
icon: key
---

Claiming a system on pkviewer lets you manage how it appears on the web. It is not ownership of the PluralKit system itself — PluralKit remains in charge of the identity and the data.

### Usually automatic

If the Discord account you sign in with is linked to a PluralKit system, pkviewer can confirm that on its own. Discord tells us who you are, PluralKit tells us which system that account is linked to, and nothing else changes hands.

This is the ordinary path and needs nothing from you beyond signing in.

### If that does not work

If your Discord account is not linked to the system, or you are claiming from a different account, pkviewer gives you a short code to put in your system description for a moment. It checks for the code, confirms the claim, and then you can take the code back out.

### The token option

There is a third option that uses a PluralKit token, and it is the last resort rather than the normal route.

!!!warning
A PluralKit token is not read-only. It can change and delete members. pkviewer uses it inside a single request to check which system it belongs to, and then discards it — it is never saved, never logged, and never sent to your browser. If you use this option, running `pk;token refresh` afterwards is a reasonable habit.
!!!

**No feature of pkviewer requires a token.** The two options above both work without one.

### One system, one claim

A system that is already claimed will not be taken over automatically, even by someone who can prove they are linked to it. If a system needs to move between accounts, that is handled by hand — the legitimate case and an account takeover look identical from the outside, so there is no safe automatic answer.

### Who can claim

Anyone with a Discord account can sign in and claim a system, as long as they can show the system is theirs. There is no waiting list. Viewing public pages needs no account at all.
