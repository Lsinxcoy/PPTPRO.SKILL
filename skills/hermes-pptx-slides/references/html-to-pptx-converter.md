# HTML → PPTX Offline Converter (python-pptx, no Playwright)

Pattern for converting HTML slide drafts into PPTX **without** a browser engine.
Use when Playwright/Chromium is unavailable or you want a pure-Python, offline path.
Mirrors the capability of Anthropic's `html2pptx.js` (which uses a real browser for layout) but
with a **coordinate + container-offset** model instead of rendered-layout measurement.

## Model

- Fixed canvas in CSS pt: 16:9 → 720×405, 4:3 → 720×540, 16:10 → 720×450. pt→inch = ÷72.
- Elements positioned by absolute `left/top/width/height` (pt) when present; otherwise **flow** top-down.
- **Container offset stack**: each `div`/`body` accumulates `margin + padding` into a cumulative
  (offx, offy). Absolute children add the current offset. Without this, nested boxes (e.g. a card
  inside a padded container) overlap or land off-canvas — this was the #1 bug.
- `<style>` `.class` rules MUST be parsed and merged with inline `style` (inline wins). A stdlib
  `HTMLParser` does NOT resolve CSS classes; only `style=` attributes are visible. Parsing the
  `<style>` block is required or class-styled boxes silently produce no shape.
- Text tags: `<p>/<h1>..<h6>/<ul>/<ol>/<li>`. Bare text in `<div>`/`<span>` is ignored (same rule
  as the upstream JS). `<li>` must be added to BOTH the start-tag (enter text mode) AND end-tag
  (flush) handlers — missing the end-tag silently drops all list items.
- `<div>` with background/border/radius/shadow → AUTO_SHAPE. `border-radius ≥ min(w,h)/2` → OVAL,
  `>0` → ROUNDED_RECTANGLE, else RECTANGLE.
- `<div class="placeholder">` → emit a dashed outline shape AND record its box so the caller can
  `slide.shapes.add_chart(...)` into that region.

## python-pptx 1.x gotchas (verified, cost real debugging time)

1. **Dashed line**: there is NO `MSO_LINE_DASH_STYLE` enum in python-pptx 1.x. `from pptx.enum.line
   import MSO_LINE_DASH_STYLE` → ModuleNotFoundError. Write the OOXML directly:
   ```python
   from pptx.oxml import parse_xml
   from pptx.oxml.ns import qn, nsdecls
   ln = shape.line._get_or_add_ln()
   for ex in ln.findall(qn('a:prstDash')):
       ln.remove(ex)
   ln.append(parse_xml(f'<a:prstDash {nsdecls("a")} val="dash"/>'))
   ```
2. **Visible bullets**: setting `paragraph.level = 0` alone does NOT render a bullet glyph in
   LibreOffice render. Add a real char bullet:
   ```python
   pPr = para.get_or_add_pPr()
   for ex in pPr.findall(qn('a:buChar')):   # clear any existing
       pPr.remove(ex)
   pPr.append(parse_xml(f'<a:buChar {nsdecls("a")} char="•"/>'))
   ```
3. **Gradient fills**: CSS `linear-gradient`/`radial-gradient` cannot be expressed in PPTX. Detect
   and ERROR OUT with a clear message telling the user to rasterize the gradient/SVG to PNG (via
   Pillow/Sharp) and reference it with `<img>` — do not silently drop it.
4. **None heights**: flow-positioned shapes/text often have `h=None`. Guard `h = box.get("h") or 40`
   before `Inches(...)` or you get `TypeError: '<' not supported between NoneType and float`.

## Validation loop (the skill's iron rule)

Never trust blind generation. After building the pptx: `soffice --headless --convert-to pdf`
then `pymupdf` → PNG, then `vision_analyze` per page. In this session a green card was found to be
**missing entirely** (class styles unparsed) only because the OBJECTIVE shape-list check caught it —
vision alone reported "no green box" but the root cause was invisible without reading `shapes`.
Always assert on structure (enumerate shapes + fills) AND appearance.

## Confirmed-working interpreter (this Windows host)

`C:/Users/41228/AppData/Roaming/uv/python/cpython-3.11.15-windows-x86_64-none/python.exe`
has python-pptx + pymupdf + lxml + defusedxml. soffice at
`C:/Program Files/LibreOffice/program/soffice.exe`. Pass NATIVE Windows paths (not MSYS `/c/...`)
to this uv python, or use `cygpath -w`.
