# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Static HTML/CSS/JS personal portfolio website for Nonpawit Saisuwan. No build system, no package manager, no framework — open the files directly in a browser.

## Deployed files (repo root)

| File | Purpose |
|---|---|
| `index.html` | Main portfolio website |
| `Resume - Nonpawit Saisuwan v2.html` | 1-page A4 print resume |
| `headshot-ns-white.png` | Shared profile photo (referenced by both HTML files) |

`index.html` links to `Resume - Nonpawit Saisuwan v2.html` via a download button — both must stay at the same directory level.

## Source / design files (`project/`)

`project/` is a Claude Design handoff bundle — source prototypes, not deployed files. Changes here do not go live; copy to root manually. `project/support.js` is the Claude Design dc-runtime compiled bundle — do not edit it.

## Architecture: index.html

Single self-contained file (~1,700 lines). No external JS libraries.

**Theme** — dark terminal aesthetic via CSS custom properties:
- `--bg: #0A0B0D`, `--red: #FF2D46`, `--live: #36E07F`, `--cyan: #4DE0E0`

**Sections** (in order): Profile → WAMOER → Built (6 case studies) → Capabilities → Track Record → Vision → Credentials → Connect

**Bilingual toggle (EN/TH)**
- `data-i18n="key"` attributes mark translatable elements
- JS dict `TH{}` maps keys to Thai strings
- `setLang(lang)` swaps innerHTML, updates `<html lang>`, persists to `localStorage`
- Default language is English

**Scroll animations** — IntersectionObserver adds `.visible` class to `.reveal` elements on entry

**Stat counters** — `data-target` + `data-suffix` on counter elements; JS animates from 0 on first viewport entry

**WAMOER framework** — User's 6-phase problem-solving system (Why → Assumption → Measure → Options → Execute → Result); appears as a section with expandable cards

## Architecture: Resume - Nonpawit Saisuwan v2.html

Self-contained A4 print resume. `@page { size: A4 portrait; margin: 0; }` — open in browser and Print → Save as PDF.

**Theme** — `--red: #C8102E`, Inter font, white background  
**Layout** — 2-column grid: 58mm left sidebar / 1fr right main content  
**Photo** — `headshot-ns-white.png` with `transform: scale(1.8) translateY(8%)` for face crop

## Fonts (CDN, both files)

`index.html`: Space Grotesk, JetBrains Mono, Sora, Noto Sans Thai  
Resume: Inter
