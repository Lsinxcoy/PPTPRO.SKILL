---
name: hermes-pptx-slides
description: Create and edit PowerPoint slides with python-pptx on Windows. Covers non-ASCII path workarounds, slide insertion at specific positions, and PPTX integrity verification.
license: Proprietary
---

# Hermes PPTX Slides

Guidance for editing .pptx files with python-pptx on this Windows system. Complements the PptxGenJS-oriented documentation/pptx skill.

## When to Use This vs Other Skills

| Need | Use This Skill | Use Instead |
|------|---------------|-------------|
| python-pptx programmatic editing (add/move slides, shapes) | **hermes-pptx-slides** (this skill) | — |
| PPTX ↔ HTML bidirectional conversion with fidelity | — | `pptx-bridge` (NEW - extractor + generator) |
| Professional HTML presentation from scratch | — | `frontend-slides` (NEW) |
| Quick browser-native deck (no toolchain) | — | `browser-native-deck` (NEW) |

## Python environment

Always use the venv python, never bare `python` (which launches the Hermes Gateway stub):

```
/c/Users/41228/AppData/Local/hermes/hermes-agent/venv/Scripts/python
```

## Non-ASCII path workaround (CRITICAL)

python-pptx Presentation() raises PackageNotFoundError for paths with non-ASCII characters (Chinese, Japanese, etc.) on some Windows Python installs. This is intermittent and depends on terminal encoding context.

Detection: if you get PackageNotFoundError for a file that ls/find can see:
1. Verify the file is a valid ZIP: `python -c "from zipfile import ZipFile; ZipFile('file.pptx').close(); print('OK')"`
2. If zip opens fine but Presentation() fails, it is a path encoding issue
3. Fix: copy the file to an ASCII-only path, or cd into the parent directory before running Python (bash handles the encoding)

## Slide insertion at a specific position

add_slide() only appends at the end. To insert at a specific index:

Workflow: add slides at end, then reorder sldIdLst in presentation.xml:
1. Note the last two sldId entries (they are your new slides)
2. Remove them from sldIdLst
3. Insert them at the desired position
4. Rezip from inside the unpacked directory

## Integrity checks

After editing, verify:
- File size is reasonable (not 10x or 0.1x original)
- Count slides matches expected
- Spot-check key text is present on target slides

## HTML → PPTX conversion (offline, no Playwright)

Full pattern + python-pptx 1.x OOXML workarounds in
[references/html-to-pptx-converter.md](references/html-to-pptx-converter.md).
Headlines:
- Use a **container-offset stack** (cumulative margin+padding) so nested boxes land correctly —
  omitting it is the #1 cause of overlapping/off-canvas shapes.
- Parse `<style>` `.class` rules yourself (stdlib `HTMLParser` does NOT); otherwise class-styled
  boxes silently produce no shape.
- python-pptx 1.x has **no `MSO_LINE_DASH_STYLE`** (write `a:prstDash` via `parse_xml`) and
  `level=0` alone does NOT draw a bullet glyph (add `a:buChar`). Both verified by debugging.
- Always verify structure (enumerate shapes + fills) AND appearance (soffice→pdf→png→vision).

## Terminal context

The terminal tool runs bash (git-bash/MSYS), not PowerShell or cmd.exe. Use POSIX syntax in all commands.

## References
- `pptx-bridge` skill: complete PPTX ↔ HTML pipeline (extractor + generator + verifier)
- `frontend-slides` skill: HTML presentation builder with PPTX conversion guidance