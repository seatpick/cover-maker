# Cover Maker

Internal tool for the content team: generate on-brand 1200x500 cover images
with a headline/subtitle overlay, exported as PNG/JPG/WEBP under 120 KB.

**Live links:**
- Public, no login: https://seatpick.github.io/cover-maker/
- Private Claude Artifact (same tool, mirrored): https://claude.ai/code/artifact/63aa8949-e28b-48e0-9ec2-a21c3972a65b
- Source: https://github.com/seatpick/cover-maker

## How it works

Single self-contained HTML file, no build step, no external requests at
runtime (everything — including all 13 display fonts — is inlined).

- `template.html` — the actual source. Edit this one.
- `fonts/*.b64` — base64 of each embedded font (all Google Fonts, OFL
  license): `anton`, `bebas`, `archivo`, `fredoka`, `chewy`, `bangers`,
  `pacifico`, `playfair`, `marker` (Permanent Marker), `caveat`, `graffiti`
  (Rubik Wet Paint), `heebo-latin`/`heebo-hebrew`, `cairo-latin`/`cairo-arabic`.
- `artifact.html` — generated bare fragment (no doctype/head/body — the
  Claude Artifact tool supplies its own skeleton). Publish this one via
  the Artifact tool.
- `index.html` — generated full standalone document (proper doctype +
  charset meta + head/body). This is the repo root file GitHub Pages
  serves. Don't hand-edit either generated file — edit `template.html`
  and regenerate both (see below).

## Layout / sections (rail, top to bottom)

1. **Quick start — templates**: 11 curated one-click combos (background +
   font + colors + alignment + badge + text effect + subtitle spacing).
   Cut down from 28 after direct feedback ("remove X, Y, Z...") — see the
   list below for what survived. All default to center/middle alignment
   with no drop shadow effect. Picking a template fills every control;
   any control can still be overridden after (clears the template's
   highlighted state, doesn't reset anything). Logo overlay and uploaded/
   saved photos are untouched by template clicks — those are user content,
   not style.
2. **Background**: just the preset swatch gallery (19 presets, cut down
   from 32 — see below).
3. **Your images**: upload a background photo (+ Remove photo, + Save to
   My Backgrounds), the My Backgrounds gallery, solid color picker
   (including SeatPick brand-color chips), and the logo overlay controls.
   Deliberately separated from "2. Background" so user-supplied content
   isn't mixed with curated presets.
4. **Text**: headline (textarea — supports Enter for manual line breaks),
   subtitle, colors, font picker, size, align, vertical position, subtitle
   spacing, pill badge toggle.
5. **Effects**: text effect (shadow/outline/glow/3D extrude) + color +
   strength.
6. **Export**: format picker, download, reset.

The stage column (left) has the canvas, safe-zone toggle, size estimate,
and (below all of that) the "Preview in a blog" mockups.

## Backgrounds

19 presets (down from 32 — removed Brand Blue, Maroon Fade, Royal Blue,
Crimson Arc, Cyan Ring, Coral Corner, Retro Sunset, Duotone Split,
Halftone Fade, Soccer Pitch, Baseball Field, Basketball Court, Ticket
Stub per direct feedback; their now-orphaned draw functions were deleted
too, not left as dead code):

- *Premium/editorial* (mesh gradients + film grain): Mesh Aurora, Grain
  Paper, Spotlight Glow, Quiet Arc, Ink Duotone, Deep Glow, Soft Wave.
  Built from `softBlob()` (blurred radial-gradient color blob) and
  `addGrain()` (a small noise tile drawn as a repeating pattern with
  `globalCompositeOperation = 'overlay'` — proper film grain; an earlier
  per-pixel-dot version aliased into ugly static at display scale).
- *Pastel minimalism* (flat muted color + grain only): Sage Minimal,
  Dusty Blue, Blush Minimal, Warm Sand, Olive Note, Stone Gray.
- *SeatPick brand*: Brand Navy (flat).
- *Sports*: Stadium Crowd (tiered seating bands of small deterministic
  dots, floodlight glow, vignette).
- *Original set*: Pitch Green, Flat Pitch, Minimal Black, Bubble Cluster.

Users can also **upload their own photo** (cover-fit cropped to 1200x500,
**draggable** — click/touch-drag directly on the canvas to reposition the
crop), or **pick a solid color** — one-click chips for the 8 exact
SeatPick brand colors (White, Navy #19213D, Blue #003BDE, Red #D31626,
Amber #FFB60D, Green #0A9D58, Sky #3E8DF7, Purple #A82FF4) plus a custom
color input, with headline/subtitle text auto-set to black or white based
on luminance (`autoContrastColor`).

**My Backgrounds** — "Save to My Backgrounds" stores a compressed 480×200
JPEG thumbnail in `localStorage` (`coverMakerCustomBackgrounds`) for reuse
without re-uploading. **Personal/browser-local, not shared with the
team** — no backend to store shared files. "Remove photo" reverts to the
default preset without touching anything saved.

## Logo overlay

A separate upload, independent of background choice. Has:
- **Size** slider.
- **Drag to reposition** — click/touch-drag the logo directly on the
  canvas (hit-tests the logo's own bounding box first, falls through to
  background-photo dragging if the click misses it).
- **Layer order**: Behind text (default) or In front of text.
- Persists across background/template switches (only Remove clears it),
  since it's a constant brand element, not a per-cover style choice.

## Fonts

13 options via the Text section dropdown: Anton, Bebas Neue, Archivo
Black, Fredoka, Chewy, Bangers, Pacifico, Playfair Display, Permanent
Marker, Caveat, **Rubik Wet Paint** (graffiti/spray-paint style), **Heebo**
(Hebrew + Latin), **Cairo** (Arabic + Latin).

Each font has an `uppercase` flag in the `FONTS` registry — true for bold
display faces, false for script/serif/handwritten/Hebrew/Arabic ones,
since forcing all-caps on those collapses their natural letterforms (a
real bug caught in testing with Pacifico).

**RTL support**: Heebo and Cairo have `rtl:true` in `FONTS`, which sets
`ctx.direction = 'rtl'` before drawing the headline/subtitle (reset to
`'ltr'` afterward). Each is embedded as two `@font-face` blocks sharing
the same family name with different `unicode-range` (Latin subset +
script-specific subset), matching Google's own subsetting technique —
this is necessary because a single font file/src can't easily cover two
disjoint Unicode blocks with different glyph sets otherwise. Verified
both Hebrew and Arabic render with correct shaping and direction.

**Headline text**: the textarea supports Enter for manual line breaks —
`wrapLines()` splits on `\n` into paragraphs first, then word-wraps each
paragraph independently, instead of collapsing all whitespace like a
single greedy word-wrap would.

## Headline style & effects

- **Pill badge**: rounded chip behind the headline (no underline — that
  was removed per feedback).
- **Text effects**: None, Drop shadow, Outline, Glow, 3D Extrude, with a
  color + strength slider. All reset after the headline draws so they
  never bleed into the subtitle.
- **Subtitle spacing**: slider (`state.subtitleGap`, default 18px, range
  0–60), was a hardcoded constant.

## Safe zone

Two guide layers when "Show safe zone" is on:
1. The main safe-zone rectangle — margin 140px sides / 60px top-bottom
   (tightened from 100/42 after a real published cover showed headline
   text getting clipped in a blog's card-thumbnail crop).
2. A second, tighter dashed rectangle (`MOBILE_CROP_RATIO`, currently a
   **4:3 placeholder** — the real ratio from the CMS hasn't been
   confirmed yet) approximating a mobile card-thumbnail crop.

**"Preview in a blog"** (in the stage column, under the canvas): two live
mockups built from the actual canvas via `stage.toDataURL()` — a
4:3-cropped card thumbnail and a native-aspect full article hero, styled
to resemble the real blog ("Ticket Insider" wordmark, byline, pill
category tag). Confirmed this reproduces the real clipping bug from the
reference screenshots. **Update `MOBILE_CROP_RATIO` and the `.mock-card-media`
CSS `aspect-ratio` once the actual crop ratio is known** — both currently
assume 4:3.

Export uses a binary search over JPEG/WEBP quality to land just under the
120,000-byte budget; PNG has no quality lever, so if a PNG export comes out
over budget the tool shows a warning with a one-click "switch to WEBP".

## To update

1. Edit `template.html`.
2. Regenerate `artifact.html` and `index.html`:
   ```
   cd /Users/maya/SeatPick-AI/cover-maker
   python3 - <<'PY'
   def load_b64(path):
       return open(path, encoding='ascii').read().strip().replace('\n','')

   tpl = open('template.html', encoding='utf-8').read()
   fragment = (tpl
       .replace('__FONT_BASE64__', load_b64('fonts/anton.b64'))
       .replace('__BEBAS_BASE64__', load_b64('fonts/bebas.b64'))
       .replace('__ARCHIVO_BASE64__', load_b64('fonts/archivo.b64'))
       .replace('__FREDOKA_BASE64__', load_b64('fonts/fredoka.b64'))
       .replace('__CHEWY_BASE64__', load_b64('fonts/chewy.b64'))
       .replace('__BANGERS_BASE64__', load_b64('fonts/bangers.b64'))
       .replace('__PACIFICO_BASE64__', load_b64('fonts/pacifico.b64'))
       .replace('__PLAYFAIR_BASE64__', load_b64('fonts/playfair.b64'))
       .replace('__MARKER_BASE64__', load_b64('fonts/marker.b64'))
       .replace('__CAVEAT_BASE64__', load_b64('fonts/caveat.b64'))
       .replace('__GRAFFITI_BASE64__', load_b64('fonts/graffiti.b64'))
       .replace('__HEEBO_LATIN_BASE64__', load_b64('fonts/heebo-latin.b64'))
       .replace('__HEEBO_HEBREW_BASE64__', load_b64('fonts/heebo-hebrew.b64'))
       .replace('__CAIRO_LATIN_BASE64__', load_b64('fonts/cairo-latin.b64'))
       .replace('__CAIRO_ARABIC_BASE64__', load_b64('fonts/cairo-arabic.b64')))
   open('artifact.html', 'w', encoding='utf-8').write(fragment)

   marker = '<div class="wrap">'
   idx = fragment.index(marker)
   head_bits, body_bits = fragment[:idx], fragment[idx:]
   standalone = (
       '<!doctype html>\n<html lang="en">\n<head>\n'
       '<meta charset="utf-8">\n'
       '<meta name="viewport" content="width=device-width, initial-scale=1">\n'
       + head_bits + '</head>\n<body>\n' + body_bits + '\n</body>\n</html>\n'
   )
   open('index.html', 'w', encoding='utf-8').write(standalone)
   PY
   ```
3. Republish the Claude Artifact from `artifact.html` with `url` set to
   the private link above (redeploys in place, doesn't mint a new URL).
4. Push to GitHub to update the public site:
   ```
   git add -A && git commit -m "..." && git push
   ```
   GitHub Pages rebuilds automatically (~30-90s). Auth is via `gh` (already
   set up under account `mayag-seatpick`; `gh auth setup-git` wires it into
   plain `git push`).

**Encoding note:** keep all UI copy/strings in `template.html` plain ASCII
(use `...` not `…`, `-` not em dash, HTML entities like `&times;`/`&mdash;`
for visible text, `\uXXXX`/`\u{XXXXX}` JS escapes for any literal special
character in a JS string literal). An earlier version had literal Unicode
characters inside `<script>` string literals which rendered as mojibake
when served without an explicit charset. Run this check after editing,
before regenerating the outputs:
```
python3 -c "print(len([c for c in open('template.html', encoding='utf-8').read() if ord(c) > 127]))"
```
should print `0`.

**Why two output files:** the Claude Artifact tool wraps published content
in its own `<!doctype>`/`<head>`/`<body>` skeleton and explicitly asks for
a bare fragment — no doctype/html/head/body tags. But GitHub Pages serves
whatever file is there byte-for-byte, so a bare fragment there has no
doctype (quirks-mode rendering) and no charset meta. Hence `artifact.html`
(fragment, for the Artifact tool) and `index.html` (full standalone
document, for GitHub Pages / any direct hosting) are both generated from
the same `template.html`.
