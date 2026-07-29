# Cover Maker

Internal tool for the content team: generate on-brand 1200x500 cover images
with a headline/subtitle overlay, exported as PNG/JPG/WEBP under 120 KB.

**Live links:**
- Public, no login: https://seatpick.github.io/cover-maker/
- Private Claude Artifact (same tool, mirrored): https://claude.ai/code/artifact/63aa8949-e28b-48e0-9ec2-a21c3972a65b
- Source: https://github.com/seatpick/cover-maker

## How it works

Single self-contained HTML file, no build step, no external requests at
runtime (everything — including all 10 display fonts — is inlined).

- `template.html` — the actual source. Edit this one.
- `fonts/*.b64` — base64 of each embedded font (all Google Fonts, OFL
  license): `anton`, `bebas`, `archivo`, `fredoka`, `chewy`, `bangers`,
  `pacifico`, `playfair`, `marker` (Permanent Marker), `caveat`.
- `artifact.html` — generated bare fragment (no doctype/head/body — the
  Claude Artifact tool supplies its own skeleton). Publish this one via
  the Artifact tool.
- `index.html` — generated full standalone document (proper doctype +
  charset meta + head/body). This is the repo root file GitHub Pages
  serves. Don't hand-edit either generated file — edit `template.html`
  and regenerate both (see below).

**Backgrounds** — 32 presets drawn live on `<canvas>`, organized into a few
families:

- *Sports/venue realism*: Soccer Pitch (crosshatch mow pattern + touchline/
  center-circle/penalty-box), Baseball Field (dirt diamond + mound),
  Stadium Crowd (tiered seating bands of small deterministic dots, a
  floodlight glow, and a vignette), Basketball Court (wood grain + key/
  free-throw arc).
- *Premium/editorial* (mesh gradients + film grain, added after the first
  background pass read as "clip art" rather than stylish): Mesh Aurora,
  Grain Paper, Spotlight Glow, Quiet Arc, Ink Duotone, Deep Glow, Soft
  Wave. Built from two primitives — `softBlob()` (a blurred radial-gradient
  color blob, for the mesh-gradient look) and `addGrain()` (a small noise
  tile drawn as a repeating pattern with `globalCompositeOperation =
  'overlay'` — proper film grain; an earlier per-pixel-dot version aliased
  into ugly static at display scale and was replaced).
- *Pastel minimalism* (flat muted color + grain only, no shapes — Danish/
  Japanese minimalist poster style): Sage Minimal, Dusty Blue, Blush
  Minimal, Warm Sand, Olive Note, Stone Gray.
- *Ticket Stub*: a big notched ticket shape + dashed perforation + barcode,
  inspired by a past cover.
- *SeatPick brand*: Brand Blue (with a redrawn ticket-notch mark echoing
  the logo shape, low-opacity watermark) and Brand Navy.
- *Original abstract set* (gradients + ribbons/rings/arcs): Pitch Green,
  Flat Pitch, Maroon Fade, Royal Blue, Crimson Arc, Minimal Black, Cyan
  Ring, Coral Corner, Retro Sunset, Duotone Split (now a soft blurred
  diagonal blend, not a hard-edge cut), Halftone Fade, Bubble Cluster.

(Removed in a later pass, after feedback that they read as clip-art rather
than stylish: Pink Chevron, Sky Burst, Purple Bloom, Wave Blobs, Pill Grid,
Zen Stack, Confetti Pop, Checker Pop.)

Users can also **upload their own photo** as a background (cover-fit
cropped to 1200x500, **draggable** — click/touch-drag directly on the
canvas to reposition the crop, backed by a `focusX`/`focusY` state pair), or
**pick a solid color** — one-click chips for the 8 exact SeatPick brand
colors (White, Navy #19213D, Blue #003BDE, Red #D31626, Amber #FFB60D,
Green #0A9D58, Sky #3E8DF7, Purple #A82FF4) plus a native color picker for
any custom hex, with headline/subtitle text auto-set to black or white
based on the color's luminance (`autoContrastColor`).

**My Backgrounds** — once a photo is uploaded, "Save to My Backgrounds"
stores a compressed 480×200 JPEG thumbnail in `localStorage`
(`coverMakerCustomBackgrounds`) so it can be reused later without
re-uploading. Each saved swatch has a small × to delete it. This is
**personal/browser-local, not shared with the team** — the tool is fully
static with no backend, so there's no way to make an uploaded image visible
to other people using the tool without adding real server infrastructure.
"Remove photo" reverts the background to the default preset without
touching anything saved.

**Logo overlay** — a separate upload (independent of background choice) is
centered on the canvas, drawn after the background but before the headline
text (so text always stays legible on top of it), with a size slider and a
Remove button. Persists across background/template switches since it's a
constant brand element, not a style choice — only clicking Remove clears it.

**Fonts** — a dropdown in the Text section switches the headline typeface:
Anton (sport bold), Bebas Neue (tall condensed), Archivo Black (grotesk),
Fredoka (rounded/friendly), Chewy (bubble), Bangers (comic shout), Pacifico
(script), Playfair Display (elegant serif), Permanent Marker (brush/
sharpie), Caveat (handwritten). Each font has an `uppercase` flag in the
`FONTS` registry — true for the bold display faces, false for the
script/serif/handwritten ones, since forcing all-caps on those collapses
their natural connecting letterforms (a real bug caught in testing:
Pacifico headlines looked blocky and wrong until the per-font case flag
was added — worth checking again whenever a new font is added).

**Headline style** — Plain (default) or Pill badge, which draws a rounded
chip behind the headline with an underline rule, recreating the cream-pill
"LIGA DAS NAÇÕES" treatment from the source reference decks.

**Text effects** — None, Drop shadow, Outline, Glow, or 3D Extrude, with an
effect color and a strength slider. Shadow/Glow use canvas `shadow*`
properties; Outline does a `strokeText` pass before `fillText`; 3D Extrude
draws ~8 diagonally-offset copies of the fill color behind the main text
(retro poster look). All reset after the headline draws so they never
bleed into the badge underline or subtitle.

**Subtitle spacing** — a slider (`state.subtitleGap`, default 18px, range
0–60) controls the gap between the headline block and the subtitle; it was
previously a hardcoded constant.

Note: an earlier version also had an icon picker/overlay (soccer ball,
trophy, etc.) — removed per feedback to keep the tool to backgrounds +
fonts + effects only.

**Templates** — a 28-entry "Quick start" gallery of one-click combos
(background + font + colors + alignment + badge + text effect + subtitle
spacing). **All templates default to center/middle alignment with no drop
shadow** — per feedback that off-center text and drop shadow read as
dated/risky defaults. Picking a template fills in every control at once;
any control can still be overridden manually afterward (this clears the
template's highlighted state but doesn't reset anything). Logo overlay and
uploaded/saved photos are untouched by template clicks, since those are
user content, not style.

**Safe-zone margin** — top/bottom margin is 42px (was 56px), matching the
same proportion as the 100px left/right margin on the 1200×500 canvas
(100/1200 ≈ 42/500), so the safe-zone guide reads as an even border on all
sides instead of looking disproportionately thick top/bottom.

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
       .replace('__CAVEAT_BASE64__', load_b64('fonts/caveat.b64')))
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
when served without an explicit charset — not worth re-introducing that
risk, especially now that the raw file is also served directly (not just
through the Artifact tool's own wrapper). Run this check after editing,
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
