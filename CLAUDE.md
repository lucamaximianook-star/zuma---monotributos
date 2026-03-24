# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Zuma is a single-page monitoring dashboard for Argentine Monotributistas (small taxpayers). It's a standalone `index.html` file (no build system, no bundler) that fetches live data from a published Google Sheets spreadsheet and displays client billing status, category thresholds, and compliance alerts.

## Architecture

- **Single file**: Everything (HTML, CSS, JS) lives in `index.html`. No framework, no dependencies beyond two CDN libraries.
- **External libraries** (loaded via CDN): Chart.js 4.4.1 (bar charts in detail panel), SheetJS/xlsx 0.18.5 (Excel export).
- **Data source**: Published Google Sheets CSV endpoints. The sheet ID is in `PUB`, with tab GIDs in `GIDS` (`TOPES`, `IIBB`, `CICLO1`, `CICLO2`, `CLIENTES`).
- **Language**: All UI text is in Spanish (Argentine locale). Currency formatting uses `es-AR`.

## Key Data Flow

1. `loadAll()` orchestrates startup: calls `loadT()` (category thresholds), `loadI()` (IIBB tax rates), `loadC()` (billing cycles 1 & 2), `loadCL()` (client list).
2. CSV is fetched via `fcsv(gid)` → `fetch()` → `parseCSV()`. Numbers use Argentine format (dots as thousands separator, comma as decimal).
3. Client data merges into `CL[]` array; each client gets billing arrays `c1`/`c2`, missing flags `m1`/`m2`, plus metadata (group, activity type, IIBB regime, employment status).
4. `gFilt()` filters/sorts clients → `render()` builds the table → `openP()`/`buildP()` renders the slide-out detail panel with KPIs, quota breakdown, and monthly bar chart.

## Key Constants & Variables

- `CATS[]` — Monotributo category thresholds and fees (A through K)
- `TK` — Current recategorization threshold (hardcoded, update when AFIP changes it)
- `M1`/`M2` — Month labels for Ciclo 1 (Jan-Dec 2025) and Ciclo 2 (Jul 2025-Jun 2026)
- `PAST_C1`/`PAST_C2` — Number of past months to check for missing data (determines "Incompleto" status)

## Status Logic

Status is determined by `gS(percentage, incomplete)`:
- **Incompleto** (purple): missing billing data in past months
- **Sin datos** (gray): zero total
- **Rojo** (red): ≥80% of threshold
- **Alerta** (yellow): ≥50% of threshold
- **OK** (green): <50%

## CSS Conventions

- CSS variables defined in `:root` use short Spanish names: `--marino` (navy), `--celeste` (light blue), `--ok`/`--al`/`--ro` (status colors).
- Class names are heavily abbreviated: `.b` = badge, `.sp` = slide panel, `.sph` = slide panel header, `.kc` = KPI card, `.cc` = cuota card, `.mc` = month card.
- Responsive breakpoints at 900px and 600px. Print styles for PDF export of the detail panel.

## Development

No build step. Open `index.html` in a browser. Data loads live from Google Sheets (requires internet). To test locally with modified data, you'd need to change the `PUB` constant or mock the fetch calls.
