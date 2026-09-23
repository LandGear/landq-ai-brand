# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex and others) when working with code in this repository. `CLAUDE.md` beside it is exactly the one line `@AGENTS.md`; keep all content here.

> **This is a PUBLIC repository** (served on GitHub Pages). Never add internal, client or commercially sensitive material — no client names, contracts, pricing negotiations, NDA material, credentials, or private details of other LandQ.AI repositories. When in doubt, leave it out and ask.

## Project Overview

LandQ.AI is an AI-powered property development platform for Australian real estate developers. It automates land feasibility analysis by querying government open data APIs, generates feasibility reports via LLM, and provides a unified system for the entire development lifecycle.

**Live:** https://landgear.github.io/landq-ai-brand/pitch

## Commands

```bash
npm run dev          # Dev server on port 3456 (pitch deck at http://localhost:3456/landq-ai-brand/pitch)
npm run build        # Static export to out/ (with basePath /landq-ai-brand)
npm run lint         # next lint
node scripts/generate-pdf.mjs  # Generate pitch deck PDF → public/LandQ-Pitch-Deck.pdf (requires dev server running)
```

Other PDF generators in `scripts/` follow the same pattern (Puppeteer against the running dev server on port 3456, output into `public/`): `generate-brochure-pdf.mjs`, `generate-proposal-pdf.mjs`, `generate-sow-pdf.mjs`, `generate-tech-solution-pdf.mjs`. `generate-mca-pdf.mjs` is the exception: it renders an HTML file from outside this repository (`../../`).

**Deployment:** every push to `main` runs `.github/workflows/deploy.yml` (`npm ci` → `npm run build` → GitHub Pages from `out/`). `npm run deploy` (`gh-pages -d out`) also exists as a manual path.

## Architecture

This repository contains brand and pitch material only — no product code.

Routes under `src/app/`:

- `pitch/` — online pitch deck (see below)
- `invest/` — second slide deck with its own `slides/`, reusing the pitch deck's `Navigation` component and `useSlideNavigation` hook
- `brochure/`, `proposal/`, `sow/`, `tech-solution/` — page-based documents (`pages/` folder each) that the matching `scripts/generate-*-pdf.mjs` export to PDF

### Pitch Deck (`src/app/pitch/`)
- **Fixed 1440×900 viewport** with `transform: scale` — NOT responsive, behaves like PowerPoint
- All slides in `slides/` directory, registered in `slides/index.ts` — currently 20 active slides (`TeamSlide` is commented out). `docs/prd-pitch-deck.md` and `README.md` still say 18.
- Navigation: keyboard (arrows/space), click (right 70% = next), touch swipe, dot navigator
- Shared animation components: `FadeIn`, `StaggerContainer`, `AnimatedCounter`
- Interactive map demo: Leaflet + CartoDB dark tiles (no API key needed)
- PDF generation via Puppeteer (`scripts/generate-pdf.mjs`)

### Config Notes
- `basePath: '/landq-ai-brand'` in `next.config.ts` — all URLs prefixed with `/landq-ai-brand/`
- `output: 'export'` for static GitHub Pages deployment
- Image paths must use `/landq-ai-brand/images/...` prefix
- `images.unoptimized: true` required for static export

## Key Domain Context

The platform integrates with Australian government open data APIs (all free, no auth):

**NSW (ArcGIS REST, ~130-230ms):**
- Planning: `mapprod3.environment.nsw.gov.au/.../Planning_Portal_Principal_Planning/MapServer`
  - Zoning L19, FSR L11, Height L14, Heritage L16+L221, Lot Size L22
- Hazards: `.../Planning_Portal_Hazard/MapServer` — Bushfire L229, Flood L230
- Cadastre: `portal.spatial.nsw.gov.au` FeatureServer L8
- Use `/query` not `/identify`. Even layers are groups (400 error) — use odd child layers.

**VIC (WFS GeoJSON):**
- All via `opendata.maps.vic.gov.au/geoserver/wfs`, namespace `open-data-platform`
- Use BBOX queries (CQL INTERSECTS has axis-order issues)

## PRD (PRD-First)

**开发前必须先有 PRD** —— 全项目硬规则，见 meta repo `rules/prd.md`。

- 本仓库的执行层 PRD：[`docs/prd-pitch-deck.md`](docs/prd-pitch-deck.md) —— deck 的 slide、导航、viewport、动画、PDF 导出、部署规格。改 deck 前先读它、先更新它。
- 战略层方向（融资口径、平台路线图）在 meta repo：https://github.com/LandGear/LandQ-AI/tree/main/docs/prd 。本仓库不再存战略 PRD 副本 —— `prd-business-plan.md`、`prd-platform.md` 于 2026-07-20 移出（前者与 meta 重复、后者讲的是产品平台不是 deck）。

## Design System

Reference: `docs/design-guidance.md` — Orens Capital-inspired institutional style.

- **Primary:** `#134A32` (Deep Forest Green)
- **Accent:** `#C4952A` (Gold)
- **Fonts:** DM Serif Display (display) + Source Serif 4 (body) + Inter (UI)
- **Style:** Capsule outline buttons, line-separated stat cards, generous whitespace

## Research Documents

All in `docs/research/`:
- `README.md` — API capability matrix and consolidated findings
- `api-testing-nsw.md` / `api-testing-vic-national.md` — verified endpoints
- `australian-land-data-sources.md` — data source survey
- `competitive-analysis.md` — Landchecker, Archistar, CoreLogic, PropCode, Feasly
- `due-diligence/` — 8-category developer checklist (`01`–`08` plus `09-summary.md`); related: `developer-due-diligence-checklist.md`, `due-diligence-process.md`
