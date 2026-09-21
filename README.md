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

### Glyph browser
- **Every glyph in the font** — the grid is virtualized (only visible cells render), so a 6,000-glyph CJK or symbol font scrolls as smoothly as a 100-glyph display face. Empty slots (space, unmapped) show as dashed advance boxes.
- **Search / filter** — by character (`A`), glyph-name substring (`quote`), or unicode hex (`0041`, `U+0042`)
- **Per-cell metadata** — codepoint and live advance width under each glyph
- **Glyph inspector** — click any cell for a zoomed view with metric guides, a **ghost overlay** of the untouched source outline behind the transformed result, and a full readout: name, unicode, advance (source → transformed), left/right sidebearings, bounding box, contour count. Navigate with `←` `→`, close with `Esc`.

### Per-glyph editing
- **Override fields in the inspector** — set an individual glyph's advance width, nudge it (ΔX / ΔY), scale it (SX / SY %), or rotate it, independently of the global transforms. Overrides stack *under* the global pipeline (local edit first, global transforms on top) and bake into the export. Edited cells show a blue dot in the browser; `RESET` clears one glyph.
- **Transform scope** — apply the global Transform / Warp / Shape sections to `ALL`, `A–Z`, `a–z`, `0–9`, or a hand-picked `SEL`ection (shift-click cells in the browser to build it). Out-of-scope glyphs pass through completely untouched — slant just the caps, outline just the digits.

### SVG import (custom glyph artwork)
- **Paste or drop an SVG into any glyph** — open a glyph in the inspector, then `PASTE SVG` (or `Ctrl+V`, or drop an `.svg` file onto the panel). The artwork becomes that glyph's outline: y-flipped into font space, scaled to a **fit height** (`CAP / X-H / EM`), sitting on the baseline with an automatic sidebearing.
- **REPLACE or ADD** — swap the glyph's outline entirely, or layer the SVG's contours on top of what's there.
- **Parser coverage** — full path data (`M L H V C S Q T A Z`, absolute + relative, implicit repeats), `rect`, `circle`, `ellipse`, `polygon`, `polyline`, nested groups, and `transform` attributes (translate/scale/rotate/matrix/skew). Arcs are converted to cubic béziers. Stroke-only elements and `defs`/masks are skipped — fills are what a font can hold.
- **New glyph slots** — `+ GLYPH` in the browser header adds a slot for any character the font doesn't have (type the character or `U+hex`). Paste artwork into it and the export maps it in the cmap — type `★` and get your logo. Imports auto-set the slot's advance width.
- Custom outlines become the glyph's *source*: global transforms, warps, scope, and per-glyph overrides all stack on top, and everything persists through project files, autosave, and undo.

### Browser & inspector view
- Oversized glyphs (warps, offsets, big scales, wild edits) **shrink proportionally to fit their cell** in the glyph browser instead of overflowing into neighbours; normal glyphs keep the shared scale for honest size comparison.
- The inspector's view canvas and the Edit Points canvas are the **same size** (responsive: 480px, 360px on narrow layouts) and share **one view**: pan by dragging, zoom with the wheel or the toolbar `＋/－`, `FIT` re-frames — in *both* panels, with your zoom carried across entering/exiting edit mode. Switching glyphs re-fits. **Double-click the glyph in the view panel to jump straight into Edit Points.**

### Alignment in the advance
- **L / C / R** buttons in the inspector set a glyph flush-left (LSB = 0), centered, or flush-right (RSB = 0) within its advance — written to the ΔX override, so it's undoable, saved in projects, and cleared by RESET.
- **Align all in scope** (Spacing section) applies the same to every glyph the current Transform scope covers, in one undo step — center all caps, flush all digits.

### Unicode block view
- **GLYPHS / UNICODE** toggle in the browser header. Unicode mode shows **every codepoint slot in a block** — pick from the block list (Basic Latin through Dingbats and PUA) or type a custom hex range (`2600-27BF`, capped at 4,096 slots per view).
- Mapped codepoints render normally; your virtual slots show as theirs; **blank means blank** — unassigned slots are empty cells with just their hex label.
- **CLICK-CREATE toggle** (off by default): when on, clicking a blank slot creates a glyph slot right there and opens the inspector ready for an SVG paste. When off, blanks just report themselves — no accidental slots.

### Spacing by hand
- In Edit Points, the two dashed advance lines are **draggable spacing handles**: drag the right line to set the advance width live, drag the left line to grow or shrink the left sidebearing (the advance follows and the glyph holds still on screen). Both snap to the ink edges — one gesture for LSB = 0 or RSB = 0 — show live LSB/RSB in the HUD, sync the ADV/ΔX fields, and commit as one undo step. Advance can't go below 0 (font format limit); overlap still comes from tracking.

### Metadata that behaves
- Loading a font now **prefills the Metadata fields from its own name table** (designer, foundry, copyright, version, URL) — JC Lutao defaults only fill true blanks. What the fields show is what exports.
- Exports carry the version in **both** places macOS looks: the name table *and* `head.fontRevision` (opentype.js pins the latter at 1.0; Glyphwerk byte-patches it and repairs the font checksums). Bump the version before re-installing to beat the macOS font cache.

### Node editor (point-level vector editing)
- **EDIT POINTS** in the inspector opens any glyph in a full vector editor: on-curve anchors as squares, control handles as circles with handle lines, metric guides, and advance-width markers.
- **Drag** any point. Anchors carry their attached control handles with them, and coincident contour start/end points move together so closed contours never tear open.
- **Multi-select** — Shift+click toggles points in and out of the selection, **Shift+drag** on empty space draws a marquee, **Ctrl/Cmd+A** selects everything. Dragging any selected point moves the whole selection (overlapping anchor/handle selections are deduplicated so nothing moves twice); arrows nudge the group; **Del** removes them all. **Esc** clears the selection first, exits the editor on the second press. Plain drag still pans.
- **Transform box** — any selection of 2+ points grows an Illustrator-style bounding box with 8 handles floating just outside it: corners scale both axes (**Shift = proportional**), edge handles scale one axis, and the opposite handle stays pinned as the origin. The HUD reads out live W×H in font units. Clicking a selected point itself always *moves* — handles never steal the click.
- **Shift-drag a point = axis lock** — the drag clamps to horizontal or vertical by whichever direction dominates, re-evaluated live so you can pivot mid-drag.
- **Undo/redo live in the editor** — `↩ ↪` on the editor toolbar (with `Cmd+Z / Cmd+Shift+Z` still working); every drag, scale, insert, and delete is one step, and a burst of arrow-nudges groups into a single step.
- **Construction SVG export** — `SVG ⤓` on the editor toolbar exports the construction view itself as vectors: metric guides with labels, advance markers, faint fill, contour outline, handle lines, and every on/off-curve point — the type-anatomy visual foundries use in specimens and brand kits. Layered groups (`guides / fill / outline / handle-lines / points`) so each is separately restylable in Illustrator; honors your ΔX/ΔY placement. **WYSIWYG: the export matches the canvas exactly — your current zoom sets the exported size, and strokes, points, and labels come out at the same pixel weights you see on screen.** Zoom in for a large export with fine strokes, out for a compact one with chunky construction marks. Labels are live text (Azeret Mono with mono fallback).
- The editor canvas is **placement-true**: your ΔX/ΔY alignment shows against the real advance guides, and a faint blue ghost of the final transformed result sits beneath the edit outline.
- **Double-click a segment** to insert a point (curves are split exactly with de Casteljau — the shape doesn't change until you move something). **Del** removes: an anchor deletes its segment; a control handle demotes its curve to a straight line. Contour start points are protected.
- **Toolbar** — `FIT / ＋ / － / ALL / NONE` buttons for view fitting, zooming, and selection without keyboard shortcuts; hovering a point shows a grab cursor, and anchors take priority over overlapping handles when clicking.
- **Arrows** nudge the selected point (Shift = ×10), with **snapping** to baseline / x-height / cap / ascender / descender and the advance edges while dragging. **Wheel** zooms at the cursor; drag empty space to pan.
- Edits happen in **source space** — global transforms, warps, and scope still apply on top — and ride the same override system as SVG imports: undo/redo covers every drag, and projects/autosave persist the edited outlines. Opening and closing the editor without touching anything leaves the glyph unmarked.

### Projects, undo, autosave
- **Undo / redo** — every committed change (slider release, toggle, override edit, selection) is a history step. `Ctrl/Cmd+Z`, `Ctrl+Shift+Z` / `Ctrl+Y`, or the header buttons. 60 steps.
- **Project files** — save all settings, per-glyph overrides, selection, names, and metadata as a `.gwproj.json`; load it back any time. The file references the font rather than embedding it — reload the font file yourself.
- **Autosave** — sessions persist in the browser per font signature; reload the same font and your edits restore automatically (`Reset all` discards).

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
- **Project files don't embed the font** — a `.gwproj.json` stores your edits, not the font binary, so keep the font file alongside it. (Deliberate: avoids baking licensed font data into shareable files.)

---

## Roadmap ideas


- URL-hash persistence of the current transform (share a link that reproduces a warp)
- Preset library (Oblique 12°, Extended 130%, etc.)
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
