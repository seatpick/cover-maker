# Cover Maker

Internal tool for the content team: generate on-brand 1200x500 cover images
with a headline/subtitle overlay, exported as PNG/JPG/WEBP under 120 KB.

**Live links:**
- Public, no login: https://seatpick.github.io/cover-maker/
- Private Claude Artifact (same tool, mirrored): https://claude.ai/code/artifact/63aa8949-e28b-48e0-9ec2-a21c3972a65b
- Source: https://github.com/seatpick/cover-maker

## How it works

Single self-contained HTML file, no build step, no external requests at
runtime (everything — including all 8 display fonts — is inlined).

- `template.html` — the actual source. Edit this one.
- `fonts/*.b64` — base64 of each embedded font (all Google Fonts, OFL
  license), used via inlined `@font-face` data URIs: `anton.b64`,
  `bebas.b64`, `archivo.b64`, `fredoka.b64`, `chewy.b64`, `bangers.b64`,
  `pacifico.b64`, `playfair.b64`.
- `artifact.html` — generated bare fragment (no doctype/head/body — the
  Claude Artifact tool supplies its own skeleton). Publish this one via
  the Artifact tool.
- `index.html` — generated full standalone document (proper doctype +
  charset meta + head/body). This is the repo root file GitHub Pages
  serves. Don't hand-edit either generated file — edit `template.html`
  and regenerate both (see below).

**Backgrounds** — 27 presets drawn live on `<canvas>` with gradients/shapes:
the original 20 abstract ones (Pitch Green, Flat Pitch, Maroon Fade, Royal
Blue, Pink Chevron, Crimson Arc, Sky Burst, Minimal Black, Purple Bloom,
Wave Blobs, Cyan Ring, Pill Grid, Zen Stack, Coral Corner, Retro Sunset,
Confetti Pop, Duotone Split, Halftone Fade, Checker Pop, Bubble Cluster),
plus realistic sports/venue textures (Soccer Pitch — crosshatch mow
pattern + touchline/center-circle/penalty-box markings; Baseball Field —
dirt diamond + mound + baselines; Stadium Crowd — tiered seating bands of
small deterministic dots with a floodlight glow and vignette; Basketball
Court — wood grain + key/free-throw arc), a Ticket Stub (notched ticket
shape + dashed perforation + barcode, inspired by a past cover), and two
SeatPick brand backgrounds (Brand Blue, with a redrawn ticket-notch mark
echoing the logo shape at low opacity; Brand Navy, flat). No image assets
to manage — everything is vector/gradient. Users can also upload their own
photo as a background (cover-fit cropped to 1200x500, **now draggable** —
click and drag directly on the canvas preview to reposition the crop;
touch drag works too). Confetti Pop and Stadium Crowd use a deterministic
sine-based pseudo-random hash (`pseudoRand`) for stable placement — never
`Math.random()`, which would reshuffle the pattern on every keystroke.

**Solid color backgrounds** — a "Solid color" row in the Background section
has one-click chips for the 8 exact SeatPick brand colors (White, Navy
#19213D, Blue #003BDE, Red #D31626, Amber #FFB60D, Green #0A9D58, Sky
#3E8DF7, Purple #A82FF4) plus a native color picker for any custom hex.
Picking a color auto-sets headline/subtitle text to black or white based
on the color's luminance (`autoContrastColor`), so it's always readable
without extra steps.

**Fonts** — a dropdown in the Text section switches the headline typeface:
Anton (sport bold), Bebas Neue (tall condensed), Archivo Black (grotesk),
Fredoka (rounded/friendly), Chewy (bubble), Bangers (comic shout), Pacifico
(script), Playfair Display (elegant serif). Each font has an `uppercase`
flag in the `FONTS` registry — true for the bold display faces, false for
Pacifico/Playfair Display, since forcing all-caps on a script face collapses
its natural connecting letterforms (this was a real bug caught in testing:
Pacifico headlines looked blocky and wrong until the per-font case flag was
added).

**Headline style** — Plain (default) or Pill badge, which draws a rounded
chip behind the headline with an underline rule, recreating the cream-pill
"LIGA DAS NAÇÕES" treatment from the source reference decks.

**Text effects** — None, Drop shadow, Outline, Glow, or 3D Extrude, with an
effect color and a strength slider. Shadow/Glow use canvas `shadow*`
properties; Outline does a `strokeText` pass before `fillText`; 3D Extrude
draws ~8 diagonally-offset copies of the fill color behind the main text
(retro poster look). All reset after the headline draws so they never
bleed into the badge underline or subtitle.

Note: an earlier version also had an icon picker/overlay (soccer ball,
trophy, etc.) — removed per feedback to keep the tool to backgrounds +
fonts + effects only.

**Templates** — a 22-entry "Quick start" gallery of one-click combos
(background + font + colors + alignment + badge + text effect): 8 general
styles (Big Question, Countdown Stat, Breaking Update, Ranking List,
Playful Badge, Bold Statement, Feature Spotlight, Calm Recap), 6
sport/category styles (Soccer Match and Basketball Recap now use the
realistic pitch/court backgrounds, plus Music Feature, Championship,
Ticket Drop, Travel Guide), 4 showcasing the newer fonts/effect (Retro
Poster, Bubble Party, Elegant Announce, Handwritten Note), and 4 more
sports/brand ones (Fan Zone, Diamond Classic, Ticket Alert, SeatPick
Blue). **All templates default to center/middle alignment with no drop
shadow** — per feedback that off-center text and drop shadow read as
dated/risky defaults; the effects that survived in templates (glow,
outline, 3D extrude) are used more sparingly and only where they suit the
specific look. Picking a template fills in every control at once; any
control can still be overridden manually afterward (this clears the
template's highlighted state but doesn't reset anything).

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
       .replace('__PLAYFAIR_BASE64__', load_b64('fonts/playfair.b64')))
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
