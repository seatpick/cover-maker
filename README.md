# Cover Maker

Internal tool for the content team: generate on-brand 1200x500 cover images
with a headline/subtitle overlay, exported as PNG/JPG/WEBP under 120 KB.

**Live link (shared with the content team):**
https://claude.ai/code/artifact/63aa8949-e28b-48e0-9ec2-a21c3972a65b

## How it works

Single self-contained HTML file, no build step, no external requests
(everything — including all 4 display fonts — is inlined so it works
inside the Claude Artifact sandbox).

- `template.html` — the actual source. Edit this one.
- `fonts/*.b64` — base64 of each embedded font (all Google Fonts, OFL
  license), used via inlined `@font-face` data URIs: `anton.b64` (Anton),
  `bebas.b64` (Bebas Neue), `archivo.b64` (Archivo Black), `fredoka.b64`
  (Fredoka).
- `index.html` — generated file (`template.html` with the four
  `__..._BASE64__` placeholders replaced by the contents of the matching
  `fonts/*.b64` file). This is what gets published. Don't hand-edit it —
  edit `template.html` and regenerate.

**Backgrounds** — 14 presets drawn live on `<canvas>` with gradients/shapes
(Pitch Green, Flat Pitch, Maroon Fade, Royal Blue, Pink Chevron, Crimson
Arc, Sky Burst, Minimal Black, Purple Bloom, Wave Blobs, Cyan Ring, Pill
Grid, Zen Stack, Coral Corner) — no image assets to manage. Users can also
upload their own photo as a background (cover-fit cropped to 1200x500).

**Fonts** — a dropdown in the Text section switches the headline typeface
between Anton (sport bold), Bebas Neue (tall condensed), Archivo Black
(grotesk), and Fredoka (rounded/friendly).

**Headline style** — Plain (default) or Pill badge, which draws a rounded
chip behind the headline with an underline rule, recreating the cream-pill
"LIGA DAS NAÇÕES" treatment from the source reference decks.

**Text effects** — None, Drop shadow, Outline, or Glow, with an effect
color and a strength slider. Implemented via canvas `shadow*` properties
(shadow/glow) and a `strokeText` pass before `fillText` (outline); reset
after the headline draws so it never bleeds into the badge underline or
subtitle.

**Icons** — 8 hand-drawn vector icons (soccer ball, basketball, music
note, trophy, ticket, sparkle, map pin, clock) with a color picker, size
slider, and position (4 corners, or "next to headline" which recreates
the small pink accent mark from the original reference deck). Drawn on
canvas rather than emoji so they stay visually consistent with the rest
of the flat/bold design and render identically for every viewer. The
emoji shown on the icon-picker buttons themselves are UI-only labels
(not exported) and are written as `\uXXXX` JS escapes in the source, not
literal characters — see the encoding note below.

**Templates** — a "Quick start" gallery of 14 curated one-click combos
(background + font + colors + alignment + badge + icon + text effect),
including 8 general styles (Big Question, Countdown Stat, Breaking
Update, Ranking List, Playful Badge, Bold Statement, Feature Spotlight,
Calm Recap) and 6 sport/category templates (Soccer Match, Basketball
Recap, Music Feature, Championship, Ticket Drop, Travel Guide). Picking
a template fills in every control at once; any control can still be
overridden manually afterward (this clears the template's highlighted
state but doesn't reset anything).

Export uses a binary search over JPEG/WEBP quality to land just under the
120,000-byte budget; PNG has no quality lever, so if a PNG export comes out
over budget the tool shows a warning with a one-click "switch to WEBP".

## To update

1. Edit `template.html`.
2. Regenerate `index.html`:
   ```
   cd /Users/maya/SeatPick-AI/cover-maker
   python3 - <<'PY'
   def load_b64(path):
       return open(path, encoding='ascii').read().strip().replace('\n','')
   tpl = open('template.html', encoding='utf-8').read()
   out = (tpl
       .replace('__FONT_BASE64__', load_b64('fonts/anton.b64'))
       .replace('__BEBAS_BASE64__', load_b64('fonts/bebas.b64'))
       .replace('__ARCHIVO_BASE64__', load_b64('fonts/archivo.b64'))
       .replace('__FREDOKA_BASE64__', load_b64('fonts/fredoka.b64')))
   open('index.html', 'w', encoding='utf-8').write(out)
   PY
   ```
3. Republish via the Artifact tool with `url` set to the live link above
   (redeploys in place, doesn't mint a new URL).

**Encoding note:** keep all UI copy/strings in `template.html` plain ASCII
(use `...` not `…`, `-` not em dash, HTML entities like `&times;`/`&mdash;`
for visible text, `\uXXXX`/`\u{XXXXX}` JS escapes for any emoji in string
literals like the icon-picker labels). Earlier versions had literal
Unicode characters inside JS string literals which rendered as mojibake
when served without an explicit charset — not worth re-introducing that
risk. Run this check after editing before regenerating `index.html`:
```
python3 -c "print(len([c for c in open('template.html', encoding='utf-8').read() if ord(c) > 127]))"
```
should print `0`.
