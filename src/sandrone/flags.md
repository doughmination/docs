---
order: 700
label: Pride flags
icon: heart
---

# Pride flags

`/pride` wraps a pride flag around a profile picture. Thirty flags, two at once if you like, still or spinning, with enough knobs to get something that actually looks good rather than something that merely exists.

Run with no options at all and you get your own avatar, rainbow flag, circular ring, 90% size. Everything below is optional.

## Options

| Option | What it does | Default |
|---|---|---|
| `user` | Whose profile picture to use. | You |
| `flag` | The flag to use. Suggests as you type. | Pride (rainbow) |
| `flag2` | A second flag, drawn as a column beside the first. | None |
| `style` | **Circle**, **Square**, or **Overlay**. | Circle |
| `size` | How much of the picture stays visible, as a percent (10–100). | 90 |
| `opacity` | How solid the flag is, as a percent (0–100). | 100 |
| `rotation` | Turn the flag by this many degrees (0–360). | 0 |
| `gradient` | Blend the stripes into each other instead of hard edges. | Off |
| `animated` | Spin the flag, as a GIF. | Off |

### Styles

- **Circle** — the flag is a ring, the picture sits in a circle in the middle. The classic look.
- **Square** — the same, with a square hole instead of a round one. Better for avatars that are already square.
- **Overlay** — no hole at all. The flag is painted over the whole picture, which only really works with `opacity` turned down.

`size` and `opacity` interact more than you'd expect. With Circle or Square, `size` is how big the hole is — lower it to get a thicker band of flag. With Overlay, `size` does nothing and `opacity` does all the work; somewhere around 40–60 is usually where it stops looking like a mistake.

### Two flags

Set both `flag` and `flag2` and they're drawn side by side as two vertical halves, then the whole thing is rotated together if you asked for rotation. The embed title names both.

### Animation

`animated: True` renders the flag spinning through a full turn, as a GIF, taking three seconds per rotation.

GIFs are large, and Discord's upload limit varies by server. Sandrone renders at the largest size that fits — trying 256px, then 224, 192, 160, and 128 — and if even the smallest is too big for the channel, it falls back to a still PNG and tells you it did. The footer always reports what you actually got: size, frame count, and file size.

## The flags

Thirty flags. The **Also answers to** column lists extra search terms the picker recognises, so you can type the word you'd normally reach for.

| Flag | Also answers to |
|---|---|
| Abrosexual | |
| Agender | |
| Aroace | `aro ace`, `aroaceflux` |
| Aromantic | `aro` |
| Asexual | `ace` |
| Bigender | |
| Bisexual | `bi` |
| Demiboy | |
| Demigender | |
| Demigirl | |
| Genderfluid | `fluid` |
| Genderflux | |
| Genderqueer | |
| Grayromantic | |
| Graysexual | |
| Lesbian | `wlw`, `sapphic` |
| MLM | `gay`, `achillean`, `gay men`, `vincian` |
| MLM (older) | `gay` |
| Nonbinary | `enby`, `nb` |
| Omnisexual | |
| Pansexual | `pan` |
| Plural | `system`, `did`, `osdd` |
| POC | `progress`, `inclusive`, `black`, `brown` |
| Polyamorous | `poly` |
| Polysexual | |
| Pride | `rainbow`, `lgbt`, `lgbtq`, `gay pride` |
| Queer | |
| Transfeminine | |
| Transgender | `trans` |
| Transmasculine | |

Ask for a flag that doesn't exist and Sandrone names the one it couldn't find and tells you to pick from the suggestions, rather than quietly giving you a rainbow.

!!!secondary
Missing a flag? Open an issue on the [repository](https://github.com/doughmination/sandrone/issues) with the flag's name and its stripe colours in hex, and it can be added.
!!!

## Some combinations worth trying

| | |
|---|---|
| A thicker band | `size: 75` |
| Soft, blended stripes | `gradient: True` |
| A tinted picture rather than a ring | `style: Overlay`, `opacity: 45` |
| Two identities at once | `flag: trans`, `flag2: lesbian` |
| Diagonal stripes | `rotation: 45` |
| The full works | `animated: True`, `gradient: True`, `size: 80` |

## Credit

The rendering follows the [LGBTQ+ Profile Picture Overlay Generator](https://pride-pfp.xyz), which is MIT licensed. Sandrone reimplements the same approach in Pillow — painting the flag onto an oversized layer, rotating the whole thing, then cropping the middle back out — so the results line up with what that site produces.
