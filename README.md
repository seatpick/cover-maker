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
- `backgrounds/*.b64` — base64 of each licensed "Team background" photo
  (see below). The matching raw image files (`backgrounds/*.webp`) are
  gitignored — only the base64 text is committed, same pattern as fonts.
- `artifact.html` — generated bare fragment (no doctype/head/body — the
  Claude Artifact tool supplies its own skeleton). Publish this one via
  the Artifact tool.
- `index.html` — generated full standalone document (proper doctype +
  charset meta + head/body). This is the repo root file GitHub Pages
  serves. Don't hand-edit either generated file — edit `template.html`
  and regenerate both (see below).

## The "Build my cover" wizard

A green **"✨ Build my cover"** button in the masthead opens a modal:
headline + subtitle text fields, a Topic picker (**Sports** — Soccer,
Football, Basketball, Tennis, Hockey, Baseball, Boxing, Motorsport —
and **Music** — Pop, Rock, Metal, K-Pop, Indie, Hip-Hop, EDM, Latin,
Country — 17 topics total), and a Vibe picker (Fun/Serious/
Professional). "Build it" applies a cover and closes the modal; a small
🎲 button next to it re-rolls a fresh variation for the same topic/vibe
without closing, so you can shuffle through options before committing.

This does **not** reuse the template gallery — it's a small generative
system of its own. `TOPIC_COLOR_VARIANTS` gives each topic **two**
background+accent color-pair options chosen to actually look like the
topic (Tennis is court-green `#2E7D32` + tennis-ball yellow-green,
Soccer is pitch-green, Basketball is orange+black, Boxing is deep
red/black, Hip-Hop is black+gold, EDM is midnight-blue+cyan, etc.), and
`VIBE_STYLE_VARIANTS` gives each vibe **two** font+effect+tilt options
(Fun → Chewy+highlighter or Bangers+outline, Serious → Anton+shadow or
Bebas+glow, Professional → Archivo Black or Playfair Display, both
clean/no effect). Build/Shuffle pick a random variant from each side —
2 colors × 2 styles = 4 possible looks per topic/vibe pair, not just 1 —
then combine into `bg`/`headlineColor`/`subtitleColor`/`font`/`effect`/
`effectColor`/`tilt`, applied the same way a template click is. Headline/
subtitle text is set from the modal's own fields — deliberately not
AI-generated copy, just carrying over what you typed in the wizard.

An earlier version mapped topic+vibe to existing template *cards*
instead of generating colors directly — replaced after feedback that it
didn't give accurate-enough colors per topic (e.g. Tennis should read
green, and reusing an unrelated template didn't guarantee that). The
topic list and per-topic/vibe variation count both grew after a direct
follow-up ask to add more options and make the results feel less
repetitive/more "smart."

## Layout / sections (rail, top to bottom)

Every section can be collapsed via its header chevron (only "Preview in
a blog" starts collapsed by default). Background, Your Images, Effects,
and both Headline/Subtitle text groups each have their own **Clear**
button, separate from the global "Reset to defaults" in Export.

1. **Quick start — templates**: one merged gallery, 4 cards per row
   (same grid as Backgrounds), 26 entries total, capped at 12 with a
   "Show more" toggle; a filter box searches by name. Cards show just
   the template name now — the "font + effect" subtitle on each card
   was removed per feedback that names alone are enough. This used to
   be two separate galleries — a curated "Quick start" template list
   and a "Template Style" genre gallery (3 colors + font + effect
   bundles named by genre, e.g. K-Pop, Soccer) — merged into one after
   direct feedback that they served the same purpose; four of the
   original genre cards (Festival, Comic Hype, Motivation, Concert Glow)
   were removed again in a later cleanup pass, and K-Pop/New Drop were
   redesigned (K-Pop: pastel pink + purple instead of hot pink + white;
   New Drop: black background + neon-green glow instead of tan/brown).
   Picking a card fills every text/effect control; any control can
   still be overridden after. Logo overlay and uploaded/saved photos
   are untouched by card clicks — those are user content, not style.
2. **Background**: a "Team backgrounds" gallery (real licensed photos,
   baked into the tool — see below), a "Backgrounds" gallery (18
   procedural/canvas-drawn presets, capped at 12 + "Show more", plus a
   filter box), and — moved here from "Your images" — the SeatPick
   brand-color chips and custom color picker (grouping is "this section
   picks the background," full stop).
3. **Your images**: upload a background photo (+ Remove photo, + Save to
   My Backgrounds), the My Backgrounds gallery, and the logo overlay
   controls. No longer has a color picker (see above).
4. **Text**: split into two clearly separated groups so it's obvious
   which control affects what — **Headline** (text, font, color, size,
   align, vertical position, pill badge + color) and **Subtitle** (text,
   font, color, size, spacing above), each with its own Clear button.
   Subtitle also has a **"Match headline"** button that copies the
   headline's font and color over (not size — subtitle is deliberately
   a different scale).
5. **Effects**: text effect (shadow/outline/glow/3D extrude/chromatic
   split/two-tone lines/highlighter marker/neon tube) + color + strength
   + tilt.
6. **Export**: format picker, download, reset.

The stage column (left) has the canvas, safe-zone toggle, size estimate,
and (below all of that, collapsed by default) the "Preview in a blog"
mockups.

## Team backgrounds (real licensed photos)

A small, separate gallery above the procedural presets, for actual
photos the team has the rights to use — currently two: **Match Night**
(aerial night stadium) and **Goal Net** (ball hitting the net, close
crop). Unlike everything else in this tool, these are real raster
images, not canvas-drawn recreations.

**How it works:** each one is baked directly into `template.html` as a
base64 data URI (`registerTeamBackground(key, label, dataUri, ...)`,
merged into the same `PRESETS` object everything else uses, so templates/
swatch-active-state/reset all treat it identically to a procedural
preset — it just draws a cover-fit photo instead of running a draw
function). This is a deliberate architecture choice: the tool is a
single static file with no backend, and the Claude Artifact mirror
blocks loading images from external URLs, so "shared with the whole
team" can only mean "compiled into the file everyone loads" — there's
no self-serve shared-upload option without building real backend
infrastructure (a different, bigger project). Practical implication:
**adding or swapping a team background requires a code change +
redeploy** (ask for it same as any other tool change) — it is not
something anyone can add themselves the way "My Backgrounds" works.

**To add another one:** crop/save the source photo to exactly 1200x500
(cover-fit centered, or pick a deliberate focus crop), save it as
`backgrounds/<name>.webp`, base64-encode it to `backgrounds/<name>.b64`,
add a `registerTeamBackground(...)` call, and add the matching
`.replace('__<NAME>_BASE64__', ...)` line to the regen script below.
WebP was chosen over JPEG for these — same visual quality at roughly
half the file size for this kind of soft/gradient-heavy photography.

## Backgrounds

18 presets (down to 19 from 32 after removing Brand Blue, Maroon Fade,
Royal Blue, Crimson Arc, Cyan Ring, Coral Corner, Retro Sunset, Duotone
Split, Halftone Fade, Soccer Pitch, Baseball Field, Basketball Court,
Ticket Stub per direct feedback — their now-orphaned draw functions were
deleted too, not left as dead code — then 3 stadium-diagram styles added,
then five removed again in follow-up cleanups: Night Stadium once the
real "Match Night" team photo covered the same mood, Pitch Green, Seat
Map, and Arena Bowl all as direct cleanup asks):

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
- *Original set*: Flat Pitch, Minimal Black, Bubble Cluster.

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
- **Text effects**: None, Drop shadow, Outline, Glow, 3D Extrude, plus 5
  added later from reference style images:
  - **Chromatic Split** — red + cyan offset copies behind the main fill,
    a glitch/RGB-split look (see the "Planet Arcadia" template).
  - **Two-Tone Lines** — alternates fill color between the headline color
    and effect color per line (odd/even), with tighter line spacing; use
    2-line headlines (see "Bulk Deal").
  - **Highlighter Marker** — a rotated colored rectangle behind each line,
    cycling a small palette (yellow/pink/blue/green), like a highlighter
    pen mark (see "Highlight My Words").
  - **Neon Tube** — stroke-only text (no fill) with a strong glow, reads
    as a hollow neon-tube outline.
  - **Hard Shadow** — a single crisp, unblurred offset duplicate behind
    the main fill (unlike Drop Shadow, which is blurred, or 3D Extrude,
    which stacks 8 copies) — the flat "bubble-letter" shadow look from
    packaging/poster design (see "Velvet Vibes", "Color Shadow",
    "Streaming Now").
  All effects use the same color + strength slider (where applicable),
  and reset after the headline draws so they never bleed into the
  subtitle. (The "Feature Spotlight", "After Dark", "Marker Tag", and
  "Glow Up" templates that used to showcase some of these were removed
  in a later cleanup pass — the effects themselves are still available
  directly from the dropdown.)
- **Tilt**: slider (`state.tilt`, -12 to 12 degrees, default 0) rotates
  the whole headline+subtitle block around its own center — added so
  effects like Drop Shadow / Glow can also read as a tilted, energetic
  sticker-style headline (see "New Drop"). Flair originally used a tilt
  too but had it removed after feedback that it made the headline read
  as misaligned rather than stylized.
- **Subtitle spacing**: slider (`state.subtitleGap`, default 28px, range
  0–60), was a hardcoded constant. Default raised from 18px after
  feedback that headline and subtitle read as too cramped together.
- **Vertical centering** (top/middle/bottom) uses real font metrics —
  `ctx.measureText(...).actualBoundingBoxAscent` / `actualBoundingBoxDescent`
  on the actual first/last rendered line — rather than a fixed guess.
  An earlier version used a flat `lineHeight*0.82` heuristic for where a
  line's visible ink starts above its baseline; that guess didn't match
  real display fonts well and a real published cover showed the headline
  sitting visibly below true center. Verified the fix by rendering to an
  off-screen canvas and scanning pixel rows for the actual ink bounding
  box — center now lands within 1-2px of true center across fonts and
  single-line/headline+subtitle cases.

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
       .replace('__CAIRO_ARABIC_BASE64__', load_b64('fonts/cairo-arabic.b64'))
       .replace('__MATCH_NIGHT_BASE64__', load_b64('backgrounds/match-night.b64'))
       .replace('__GOAL_NET_BASE64__', load_b64('backgrounds/goal-net.b64')))
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
