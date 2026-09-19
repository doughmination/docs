Custom CSS lets you style your pages by hand, from **Advanced CSS** in your system's management pages.

There are two places to write it:

- **Your system's** stylesheet, under Advanced CSS, applies to every page in your system.
- **A member's own** stylesheet, from that member's management page, applies only to their page — layered *on top of* the system one, so a member starts from your look and changes what they want different.

It is not quite the CSS you are used to. pkviewer does not serve what you type — it reads your stylesheet, keeps the parts it recognises, and writes out its own version. Anything it does not recognise is skipped and listed back to you with the line number, so a rule that was dropped never looks like a rule that did nothing.

!!!secondary
Nothing here can break your page permanently. Clear the box, save, and you are back to your theme.
!!!

### Selectors are scoped for you

Write `.card`, not `#pkv-user .card`. Every selector is rewritten to apply only inside your page's content, so you cannot accidentally restyle the rest of the site.

```css
.card { border-radius: 18px; }
.identity h1 { letter-spacing: -0.02em; }
```

`html`, `body` and `:root` are refused rather than silently doing nothing — they are where you would reach for a page background, and the [variables below](#variables) are how you set that instead.

### What you can target

These are the parts of a public page:

| Class | What it is |
|---|---|
| `.page` | The whole content column |
| `.identity` | Avatar, name and the details beside them |
| `.avatar`, `.banner` | The images at the top |
| `.meta-list` | ID, pronouns, tag, member count |
| `.prose` | The description |
| `.socials` | Your links |
| `.directory` | The member grid |
| `.member-card` | One member in the grid |
| `.member-card .name` | A member's name |
| `.card-blurb` | A member's short description |
| `.muted` | Secondary text anywhere |

Ordinary element selectors work too: `h1`, `h2`, `p`, `a`, `ul`, `li`, `img`.

### Variables { #variables }

Your theme's settings are available as CSS variables, so you can build on your theme rather than fighting it. Change the colour in **Appearance** and CSS that reads these follows along.

| Variable | What it holds |
|---|---|
| `--pkv-color-page` | Page background |
| `--pkv-color-surface` | Card background |
| `--pkv-color-text` | Text |
| `--pkv-color-muted` | Secondary text |
| `--pkv-color-accent` | Accent |
| `--pkv-color-border` | Lines and borders |
| `--pkv-font-body` | Body typeface |
| `--pkv-font-heading` | Heading typeface |
| `--pkv-font-size` | Base text size |
| `--pkv-radius` | Corner rounding |
| `--pkv-density` | Spacing multiplier |
| `--pkv-avatar-size` | Avatar size |
| `--pkv-avatar-radius` | Avatar corner rounding |
| `--pkv-surface-bg` | Card background, after card style |
| `--pkv-surface-border` | Card border, after card style |

```css
.member-card {
  border: 2px solid var(--pkv-color-accent);
  border-radius: calc(var(--pkv-radius) * 2);
}
```

Spacing steps `--sp-1` through `--sp-12` are also available, and scale with your density setting.

### What is not available

Four things are restricted. Each one answers a specific problem rather than being a general caution.

==- `url()` — only three hosts
A stylesheet can link to these and nowhere else:

| Host | For |
|---|---|
| `fonts.googleapis.com` | Google Fonts stylesheets, via `@import` |
| `fonts.gstatic.com` | the font files those stylesheets use |
| `m.doughmination.gay` | the pkviewer CDN — images, fonts, anything static |

Anywhere else is refused. A request tells whoever hosts it the address of everyone who looked at your page, and it is also how CSS is used to *read* a page: a selector matching only certain content, paired with a request, reports back one character at a time. Restricting where requests can go means that technique has nowhere to send anything — the three hosts above are ones pkviewer already asks your visitors to contact, and nobody can read their logs.

The host has to match exactly and be `https`. `https://fonts.googleapis.com.somewhere-else.com/` is a different host, and refused.
==- `position: absolute` and `fixed`
Only `static` and `relative` are available.

Taking an element out of the flow lets it be laid on top of the page, and a convincing fake sign-in box on a real pkviewer address is the thing this prevents. Use flexbox and grid for layout; both are fully available.
==- `!important`
It is reserved for the rules that keep badges and the site notice in place.

If author CSS could use it, those would become a specificity race that the page eventually loses.
==- Badges and the site notice
Selectors naming `.pkvb` or `.site-footer` are refused.

A [badge](/pkviewer/badges) is pkviewer's statement about a system, and the notice is what stops a third-party site reading as an official one. Neither is part of your page's appearance. See [why badges cannot be faked](/pkviewer/badges).
===

`@media`, `@supports` and `@font-face` all work. `@keyframes` does not yet.

`@import` works for `fonts.googleapis.com` only. It is the one thing pkviewer serves without reading first — the stylesheet that arrives is whatever Google returns — so it is locked to the single host that exists to return font stylesheets. Importing from the CDN is refused for the same reason: it serves arbitrary files, and an arbitrary file imported as a stylesheet would be rules that never met the compiler.

Anything else not on the supported property list is skipped and reported, rather than applied.

### Fonts

The typefaces in [Appearance](/pkviewer/appearance) are loaded for you; use `var(--pkv-font-body)` and `var(--pkv-font-heading)` to reach them.

For anything else, load it yourself. The usual Google Fonts line works:

```css
@import url("https://fonts.googleapis.com/css2?family=Lora:wght@400;700&display=swap");

.identity h1 { font-family: "Lora", Georgia, serif; }
```

Or point `@font-face` at a file on the CDN:

```css
@font-face {
  font-family: "My Font";
  src: url("https://m.doughmination.gay/fonts/myfont.woff2") format("woff2");
  font-display: swap;
}
```

A `@font-face` whose `src` is refused is dropped whole, rather than left naming a family it cannot supply. `font-family` on its own accepts any name, but a font nobody has installed simply falls back.

### A worked example

```css
/* Softer cards with an accent edge */
.member-card {
  background-color: var(--pkv-color-surface);
  border: 1px solid var(--pkv-color-border);
  border-left: 3px solid var(--pkv-color-accent);
  border-radius: 4px;
  transition: border-color 150ms ease;
}

.member-card:hover {
  border-color: var(--pkv-color-accent);
}

/* Tighter headings */
.identity h1 {
  font-size: 2.2rem;
  letter-spacing: -0.02em;
  line-height: 1.1;
}

/* Two columns on wide screens */
@media (min-width: 60em) {
  .directory {
    grid-template-columns: repeat(2, 1fr);
  }
}
```

### If something does not apply

The editor lists every skipped line underneath your stylesheet, with the reason. The commonest causes:

- a property that is not on the supported list
- a `url()` pointing somewhere other than the three hosts above
- `!important`
- a selector naming a badge or the site notice
- `html`, `body` or `:root`

A stylesheet with problems still saves, and the rules that are fine still apply. You lose the line, not the page.
