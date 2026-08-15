# blyzr.design

Portfolio site. Static, no build step, no framework.

## Structure

```
blyzr-portfolio/
├── index.html                  both layouts, CSS picks one at 860px
├── favicon.svg                 vector, preferred by modern browsers
├── favicon.ico                 16/32/48/64 fallback
├── favicon-16x16.png
├── favicon-32x32.png
├── apple-touch-icon.png        180×180
├── site.webmanifest
├── robots.txt
└── assets/
    ├── css/main.css
    ├── js/main.js
    └── img/                    project imagery goes here
        ├── icon-192.png
        └── icon-512.png
```

## Running it locally

Paths are absolute (`/assets/...`), so it needs a server rather than
opening the file directly.

```bash
cd blyzr-portfolio
python3 -m http.server 8080
```

Then http://localhost:8080

## Putting it on the homelab

It is a static folder, so anything will serve it. Drop it in a web root and
point an NPM proxy host at it, or run it from a container:

```bash
docker run -d --name blyzr -p 8080:80 \
  -v /path/to/blyzr-portfolio:/usr/share/nginx/html:ro nginx:alpine
```

## Two layouts, one document

Both live in `index.html`. `@media (max-width: 860px)` and the matching
`matchMedia` in `main.js` decide which is active, and `applyMode()` keeps
the JS in step with the CSS.

- **Index** (wide): typographic project list, sticky preview panel, light
  table proximity lighting.
- **Specimen** (narrow): large `br.` mark, project bands, loupe reveal tied
  to a fixed reading zone at 42% viewport height.

## How the header dock works

Scrolling through the dead zone (`deadZone()`, the height of the hero title
plus its subtitle) does nothing. Past that threshold GSAP plays the whole
transition as one 0.78s tween, and it reverses at 72% of the threshold. The
hysteresis gap stops it flapping, and because the tween owns the value it can
never rest half-finished.

Every run rolls new Recursive axis values (`rollAxes`), so the weight, slant,
casual axis and tracking land somewhere slightly different each time.

## Replacing the placeholder artwork

Placeholder gradients are `.a1` to `.a9` at the end of the artwork block in
`main.css`. Swap each for the real thing:

```css
.a1{background:url("/assets/img/meridian.jpg") center/cover}
```

Rect positions are cached and only re-read when `stale` is set, on scroll and
resize. If you add an element that moves independently, set `stale = true` when
it does, or it will light from the wrong place.

The lighting filters sit on `.art`, so real imagery inherits the effect with
no other changes. Note that the desaturated resting state will read very
differently on photography than it does on these gradients, so the falloff
numbers will likely want retuning once real images are in.

## Palette: Ink and lead

Printer's blacks with vermilion as the only voice. Set in `:root`.

| Token | Value | Role |
|---|---|---|
| `--base` | `#0f0f0f` | page |
| `--surface` | `#1a1a1a` | raised |
| `--text` | `#e8e6e1` | body |
| `--subtle` | `#b5b0a8` | secondary |
| `--muted` | `#8a8681` | labels |
| `--accent` | `#d4472f` | vermilion, used sparingly |

Each project carries its own ink (`ink` in the `projects` array), drawn from a
press-ink range rather than a rainbow: verdigris, ochre, burnt sienna, payne's
grey, olive drab, rust, slate, amber. The live accent drifts toward whichever
project has focus, which is why the dot changes colour.

## Spacing

One 4px scale, `--s1` (4px) through `--s9` (96px). No off-scale value exists in
the stylesheet; if you need one, the scale is wrong, not the exception.
Radius is `--r1` (2px) for frames and `--r2` (6px) for panels. Two values.

## Motion budget

Three ambient effects, deliberately. Earlier revisions ran ten at once, which is
why none of them registered.

1. **Light table** — proximity lighting on rows and the preview
2. **Live mark** — cursor or gyroscope tilt on the monogram
3. **Header dock** — the hero title docking on scroll

Grain is texture, not motion. Cut along the way: the lagging trail glow,
background parallax, the dot's drop-shadow glow, and the pointer-entry colour
bloom. If you add one back, take one out.

## Tuning constants

Top of `main.js`:

| Constant | Current | What it does |
|---|---|---|
| `TILT` | `1.0` | Live mark rotation multiplier |
| `MOBILE_TILT` | `1.45` | Same, on touch |
| `SPREAD` | `0.99` | Light radius as a fraction of the smaller viewport dimension |
| `MARK_HERO_PX` | `32` | Header mark size before docking |
| `MARK_DOCK_PX` | `17` | Header mark size after docking |
| `DOCK_DURATION` | `0.78` | Dock tween length in seconds |
| `READING_ZONE` | `0.42` | Where the mobile loupe sits, as a fraction of viewport height |

## Still to do

- Real project imagery and copy
- Case study pages behind "See full case study"
- About and contact pages
- Self-host Recursive instead of the Google Fonts CDN
- Decide whether the `.row-blurb` single-line clip survives your real project names
- `sitemap.xml`
- Open Graph and Twitter card images
