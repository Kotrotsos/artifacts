# DesignPro.md — launch summary

**Date:** 2026-04-25
**Stack:** Next.js 15 + Tailwind v4 + Drizzle/Postgres + Claude Agent SDK + Vitest
**Deploy:** Railway, project `designpro-md`
**Domain:** https://designpro-md-production.up.railway.app
**Test suite:** 29/29 passing across parser, validator (9 Google rules), generator, contrast (WCAG 2.1), exporter (Tailwind/CSS/SCSS/DTCG/JSON)

## Surface area shipped

| Page | Purpose |
|---|---|
| `/` | Landing with feature cards |
| `/editor` | Visual GUI: color pickers, Google Fonts dropdowns, scale inputs, live hero, generated DESIGN.md |
| `/playground` | CodeMirror DESIGN.md editor + 9 lint rules + WCAG contrast pairs |
| `/examples` | 13 ready-made systems (3 Google + 10 mine) with detail pages |
| `/components` | 10 built-in components + custom-component creator |
| `/render` | Claude Agent SDK streams full HTML mockups using the user's tokens (6 targets) |
| `/docs/*` | 13 documentation pages with a sidebar select-box jumper |
| `/admin` | Overview, users, designs, raw events (admin-only) |

## Why it matters

DESIGN.md is Google Labs' open format for shared brand context across AI agents. DesignPro.md is the most complete editor and playground for it: deterministic linting + WCAG checks + exports for Tailwind/DTCG/CSS, plus AI-driven UI rendering that obeys the tokens.

## What's next

- Import flows (URL extraction, Tailwind config import, Figma)
- Public design sharing with permalinks
- Community gallery with submissions
- Per-component variants editor
