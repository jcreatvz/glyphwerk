# Glyphwerk°

**Live: [jcreatvz.github.io/glyphwerk](https://jcreatvz.github.io/glyphwerk/)**


A single-file, in-browser bench for transforming every glyph of a font — shear, rotate, scale, warp, roughen, rename, export. No server, no build step, no upload. The font never leaves your machine.

Drop the HTML on any static host (GitHub Pages, Netlify, a local file) and it runs.

---

## Why

Every time I wanted a fake-oblique, a stretched wide, or a wave-warped display cut of a font I already owned, the workflow was: open FontLab / Glyphs, remember which menu the transform lived under, run it, export, install, curse. Glyphwerk collapses that into a slider you drag while the specimen updates in real time. It's a scratchpad for glyph transforms — not a replacement for a real font editor, but faster than one when all you need is "the same font, but leaning 18° with a taper."

---

## Features

### Load
- **TTF, OTF, WOFF** (WOFF2 is not supported — the parser doesn't read it)
- Drag-and-drop onto the drop zone or anywhere on the page
- Shows family, style, glyph count, UPM, and outline format

### Transform (affine)
- **Slant / shear X** — ±45°. The classic "make it italic" move.
- **Shear Y** — ±30°. Vertical skew for stranger effects.
- **Rotate each glyph** — ±180°. Not usually what you want globally, but useful on symbol or icon fonts.
- **Scale X / Y** — 25%–250% independently. Condensed, extended, tall, squat.
- **Transform origin** — `baseline` / `x-mid` / `center`. Determines the pivot point for rotation and shear. Baseline is the traditional italic behavior; x-mid keeps lowercase optically centered; center is best for symbol fonts.

### Warp (nonlinear)
- **Wave ripple** + **frequency** — sinusoidal vertical displacement along each glyph. Bézier segments are automatically subdivided so curves stay accurate under the warp.
- **Bulge / pinch** — ±100. Fisheye-style expansion or contraction at the glyph midline.
- **Taper (perspective)** — ±100. Widens or narrows the top vs. bottom of each glyph, imitating a perspective lean.
- **Roughen** — 0–100. Deterministic per-node jitter, seeded from node coordinates so the preview and the export are pixel-identical (no random reshuffle on save).

### Shape (boolean geometry)
- **Offset (grow / shrink)** — ±inflate or deflate every glyph's outline. Positive = chunkier letterforms; negative = thinner (push far enough and thin strokes erode away entirely, which is its own look). Bakes into the exported font.
- **Stroke** — `OFF / OUTLINE / INLINE / CENTER` with a weight slider. Converts each glyph to a hollow ring: outline hugs the outside edge, inline sits inside the original footprint, center straddles the contour. All three are real geometry — the exported font is genuinely hollow, not a preview trick.
- Powered by a vendored copy of Clipper (polygon boolean/offset engine) — no CDN dependency; the file stays fully self-contained.

### Wordmark · logo mode (knockout)
- **Knockout overlaps** — where two letters overlap, the priority letter carves a gap channel out of its neighbor (a classic lettering "knockout" / notch), with an adjustable gap width and a `LATER CUTS / EARLIER CUTS` priority toggle.
- Knockout is *relational* — each letter's final shape depends on its neighbors — so it can't be baked into an installable font. Instead it exports the specimen text as flattened **SVG artwork**, ready for logo and wordmark use.
- Pull tracking negative to overlap the letters, then knock out.

### Metadata
- Customizable **Designer, Foundry, Copyright, Version, URL** fields (designer defaults to JC Lutao)
- Written into the exported font's name table — visible in font managers, inspectors, and OS font panels

### Specimen thumbnail
- One-click **PNG download (1200×630)** — a styled specimen card showing the wordmark/sample in the transformed font, the family + style name, glyph count, UPM, designer credit, and year. Sized for GitHub social previews and product tiles.

### Spacing
- **Tracking** — −1000 … +400 font units. Negative tracking pulls letters together, into, and past each other for overlap compositions
- **Widen advances to fit slant** — automatically pads the advance width so steep obliques don't clip at word edges

### Rename
- Family and style name fields
- Live PostScript name preview (auto-sanitized: strips spaces and punctuation the way font tables require)

### Preview
- Live specimen with labeled `ASC / CAP / X-HEIGHT / BASELINE / DESC` guides
- Editable sample text
- Size slider (40–220px)
- **Scrub-drag**: hold and drag horizontally on the specimen to slew the slant angle in real time
- Proof-sheet grid — every drawn glyph in the font rendered live at the current transform settings

### Export
- Downloads a fresh `.otf` (CFF outlines)
- All 100% of glyphs from the source carried through, unicode mappings preserved
- New family / style names applied
- Original hinting is dropped (any nonzero transform invalidates it anyway)

---

## Getting started

### Option A: open locally
Save `glyphwerk.html` anywhere, double-click it. That's it.

### Option B: host on GitHub Pages
```
git init
git add glyphwerk.html
git commit -m "glyphwerk"
git branch -M main
git remote add origin git@github.com:jcreatvz/glyphwerk.git
git push -u origin main
# then: Settings → Pages → Deploy from main / root
```
The app file is `index.html`, served at the repo root by Pages.

### Option C: drop into any static site
It's a single self-contained file. Copy it into any static folder and link to it.

---

## Controls reference

| Section | Control | Range | Notes |
|--------|---------|-------|-------|
| Transform | Slant / shear X | −45° … +45° | Positive = top slants right (standard italic direction) |
| Transform | Shear Y | −30° … +30° | Vertical shear |
| Transform | Rotate each glyph | −180° … +180° | Per-glyph rotation around origin |
| Transform | Scale X / Y | 25% … 250% | Independent axes |
| Transform | Origin | baseline / x-mid / center | Pivot for shear + rotate |
| Warp | Wave ripple | 0 … 100 | Amplitude as % of UPM × 0.12 |
| Warp | Wave frequency | 0.25 … 4 | Cycles per glyph width |
| Warp | Bulge / pinch | −100 … +100 | Y-scale falloff, peak at glyph center |
| Warp | Taper | −100 … +100 | X-scale as function of Y |
| Warp | Roughen | 0 … 100 | Deterministic jitter |
| Shape | Offset | −100 … +100 | Grow/shrink outlines, ±5% of UPM |
| Shape | Stroke | off / outline / inline / center | Hollow ring conversion |
| Shape | Stroke weight | 2 … 100 | Ring thickness, up to 4.5% of UPM |
| Wordmark | Knockout gap | 0 … 120 | Cut channel width, up to 6% of UPM |
| Wordmark | Priority | later / earlier | Which overlapping letter wins |
| Spacing | Tracking | −1000 … +400 | Font-unit offset; negative overlaps |
| Spacing | Slant compensation | on / off | Widen advances to fit steep obliques |

Every numeric readout is click-editable — tap the value, type a number, hit enter.

---

## Technical notes

**Parser / builder**: [opentype.js](https://github.com/opentypejs/opentype.js) v1.3.4 via CDN. All font parsing, path decomposition, and CFF assembly runs in the browser.

**Bézier subdivision under warp**: linear transforms (slant, rotate, scale) preserve straight lines and curves — a single transform of endpoints and control points is correct. Nonlinear warps (wave, bulge, taper, roughen) don't — a straight bézier segment through a curved warp is no longer a bézier of the same order. Glyphwerk detects nonlinear mode and subdivides each Q and C segment 2–3 levels via de Casteljau before applying the warp, so curves visibly hold their shape instead of flattening.

**Determinism**: the roughen effect uses a simple hash function (`sin(x·12.9898 + y·78.233 + seed·37.719) × 43758.5453`, floor-fractional) seeded per node. Same input coordinates → same displacement, every render. What you see in preview is byte-identical to what exports.

**Rendering**: HiDPI canvas via `devicePixelRatio` scaling. Both the specimen and proof-sheet redraws are wrapped in a single `requestAnimationFrame`, so dragging a slider fires at most one render per frame.

**Boolean geometry**: offset, stroke rings, and knockouts run on [Clipper](http://www.angusj.com/delphi/clipper.php) (Angus Johnson's polygon clipping library, v6.4.2), vendored minified into the file. Glyph beziers are flattened to fine polygons (adaptive chord tolerance) before boolean ops, so offset/stroke output is line-segment geometry — visually smooth at display sizes, and valid CFF.

**Knockout algorithm**: each glyph in the run is inflated by the gap width, then subtracted from every lower-priority neighbor. Later-cuts priority reads like cards fanned rightward; earlier-cuts is the reverse.

**Metrics**: reads `sxHeight` and `sCapHeight` from OS/2 when present; falls back to measuring the `x` and `H` glyph bounding boxes if the table is missing or empty.

**Output format**: OTF (CFF) only. opentype.js writes CFF cleanly; its glyf output is less reliable. OTF installs and renders anywhere TTF does — this is a rendering-format choice, not a compatibility loss.

---

## Limitations

- **WOFF2 input** — not supported by the parser. Convert to OTF or TTF first.
- **Italic metadata** — the exported font gets a new name but doesn't set the `post.italicAngle`, `hhea` caret slope, `OS/2.fsSelection` italic bit, or `head.macStyle` italic bit. Some apps use these to decide whether to display a font as "italic" in style menus. If you're making a production oblique you plan to distribute, apply that metadata separately.
- **Hinting is dropped** — any nonzero transform invalidates the original hints (they were calculated for the un-transformed outlines). If you need hinted output, that's a job for a real font editor after the transform is baked.
- **Shape/stroke output is flattened** — when offset or stroke is active, exported outlines are fine line segments rather than beziers. Invisible at display sizes; if you need curve-fitted output, run the result through a font editor's "simplify" pass.
- **Knockout doesn't export to font** — it's relational (depends on letter neighbors), so it ships as SVG wordmark artwork instead. This is a property of font formats, not a missing feature.
- **Kerning tables aren't rewritten** — the source kerning survives, but the transform can shift optimal kerns. For display use this is usually fine; for text sizes, re-kern.
- **No undo history** — sliders reset per-section (Transform reset / Warp reset / Reset all), but there's no timeline. Save presets by URL-hashing the state if you want persistence (not currently implemented; PR-worthy).

---

## Roadmap ideas

- URL-hash persistence of the current transform (share a link that reproduces a warp)
- Preset library (Oblique 12°, Extended 130%, etc.)
- Per-unicode-range selective transforms (e.g., only uppercase, only Latin-1)
- Proper italic metadata write on export (slant angle, fsSelection bits)
- WOFF export via a compression layer
- Curve re-fitting after boolean geometry (bezier output instead of segments)
- Kerning-table re-scaling with tracking / scale-X
- Batch export at multiple settings ("Regular, Oblique 10°, Oblique 15°" in one download)

---

## File structure

Everything lives in one HTML file. Structure inside:

```
glyphwerk.html
├── <style>       — design tokens, layout, control styling
├── <body>        — header, rail (controls), main (preview + grid)
└── <script>      — state, transform math, path subdivision, export
```

CDN dependency: opentype.js (single script tag, versioned).

---

## Credits

**Author & designer** — JC Lutao
**Built with** — [opentype.js](https://opentype.js.org) by Frederik De Bleser · [Clipper](http://www.angusj.com/delphi/clipper.php) by Angus Johnson (vendored)
**Type used in the UI** — Instrument Sans + Azeret Mono (Google Fonts)

---

## License

Pick one that fits your use. If you don't have a preference, MIT is a safe default.

```
MIT License

Copyright (c) 2026 JC Lutao

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
