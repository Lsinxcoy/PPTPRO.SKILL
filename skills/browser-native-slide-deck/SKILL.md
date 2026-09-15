---
name: browser-native-slide-deck
description: "Use when the user wants a PowerPoint-style deliverable but a real .pptx toolchain is unavailable or too brittle. Build one self-contained HTML file with fixed 1280x720 slides, browser-native navigation, and no runtime dependencies beyond a browser."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [productivity, documents, browser, html, slides, presentation]
---

# Browser-Native Slide Deck

Generate a single HTML file that behaves like a slide deck. Prefer this over `.pptx` when:
- the user asked for "PPT" but only needs training/internal delivery,
- `.pptx` toolchains are unavailable or unreliable on Windows,
- iteration speed matters more than print-ready packaging.

## When to Use This vs Other Skills

| Need | Use This Skill | Use Instead |
|------|---------------|-------------|
| Quick internal deck, fixed 1280×720, no toolchain | **browser-native-slide-deck** (this skill) | — |
| Professional presentation with visual style exploration | — | `frontend-slides` (NEW) |
| PPTX ↔ HTML conversion with fidelity | — | `pptx-bridge` (NEW) |
| Design token system for consistent theming | — | `theme-factory` (NEW) |

## Slide size
- Width `1280px`, height `720px`.
- Add `page-break-after: always` so print-to-PDF preserves slides.

## Required slide types
- `slide.cover` — gradient background, centered title/subtitle, meta line
- `slide.section` — full-bleed chapter divider with section number
- `slide.standard` — tag + title + subtitle + content grid

## Navigation pattern
- Bottom-right prev/next buttons.
- Bottom-left `current / total` counter.
- Top fixed progress bar.
- JS: `go(n)` changes a `.selected` class, calls `scrollIntoView({behavior:'smooth'})`, updates counter/progress text.

## Reusable content classes
- `.grid-2`, `.grid-3`, `.grid-4`
- `.card.red|.blue|.green|.gold|.dark` with left border accent
- `.step-row` + `.step-num` for numbered action sequences
- `.check-item` + `.check-box` for checklist slides
- `.highlight`, `.warn-box`, `.danger-box` for callouts
- `table` with dark header row and alternating row backgrounds

## Speaker notes
- Keep speaker notes hidden inside each slide, not at the bottom.
- Example: `<div class="speaker-notes" style="display:none">...</div>`

## Pitfalls
- Do not embed slide content in JS. Keep it in static HTML so printing and SEO work.
- Avoid fixed fonts that depend on system availability; use `"Microsoft YaHei","PingFang SC","Helvetica Neue",sans-serif`.
- Do not use browser automation to screenshots; deliver the HTML directly and let the user open it.

## References
- `frontend-slides` skill: visual exploration (3 previews), viewport-fit guarantees, PPTX conversion guidance
- `theme-factory` skill: 10 preset themes, CSS design tokens, WCAG AA validation
- `pptx-bridge` skill: python-pptx extractor/generator with Windows non-ASCII path workaround
