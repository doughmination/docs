pkviewer separates two things that are easy to confuse:

- **Appearance** — how things look. Colours, typefaces, corners, spacing.
- **Layout** — what appears and how it is arranged. Columns, card style, ordering, which details to show.

They are edited separately because they answer different questions.

### Presets

Five presets are a one-click starting point, and they differ in more than colour — typeface, corner treatment, card style, density and light or dark ground all change:

| | |
|---|---|
| **Notebook** | The default. Warm off-white, serif headings, soft outlined cards. |
| **Broadsheet** | Editorial and dense. Display serif, square corners, hairline rules. |
| **Bloom** | Soft and friendly. Rounded typeface, large corners, filled cards. |
| **Terminal** | Deliberately technical. Monospaced, near-black, no card decoration. |
| **Midnight** | Modern dark. Geometric headings, filled cards, soft corners. |

A preset fills in the settings and then gets out of the way — everything stays editable afterwards.

### Member pages

Every setting on a member's page starts out following the system's appearance. You override only what you want different.

Each setting is in one of three states:

- **Using system appearance** — follows whatever the system uses, and keeps following it if the system changes.
- **Custom** — set for this member only.
- **Platform default** — ignores the system entirely and uses pkviewer's own default.

That third state exists so a member can step out of a strong system theme without having to restate every default by hand.

Colour scheme is the one setting a member cannot override — a member page flipping to dark while the system page is light reads as broken navigation rather than personal expression.

### What you cannot do yet

Custom fonts beyond the built-in set, custom domains and per-member logins are not available. Typefaces come from a fixed list so that no page can pull a font from somewhere unexpected.

Custom CSS **is** available, under **Advanced CSS** — see the [CSS reference](/pkviewer/css). It is a restricted dialect rather than a free hand: no `url()`, no absolute positioning, and badges and the site notice cannot be restyled. The reference explains each restriction and why it exists.
