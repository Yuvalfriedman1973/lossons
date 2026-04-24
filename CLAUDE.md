# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single-file static web app (`index.html`) that displays operational driving lessons from a Google Sheet with 3 tabs (Lebanon, Gaza, Syria). Hosted on GitHub Pages at `https://yuvalfriedman1973.github.io/lossons/`. The owner updates content by editing the Google Sheet — no code changes needed.

## Architecture

Everything lives in `index.html`: HTML structure, all CSS, and all JavaScript. There is no build process, no dependencies, no package manager.

**Data flow:** Google Sheets (published CSV, one tab per zone) → `fetch()` → `parseCSV()` → `rowsToTopics()` → sort → render

**Navigation:** SPA-style — three `<div>` views toggled with `.hidden`. No page reloads:
- `#view-home` — zone selection screen with 3 zone buttons and a universal search bar
- `#view-main` — topic list for a selected zone (with back button to home)
- `#view-topic` — topic detail view (causes / conclusions / actions)

A shared `#main-header` (with `h1` and `#search`) is shown only in `#view-main`.

## Zones

```javascript
const ZONES = [
  { id: 'lebanon', name: 'לבנון', url: '...gid=0...' },
  { id: 'gaza',    name: 'עזה',   url: '...gid=1787717414...' },
  { id: 'syria',   name: 'סוריה', url: '...gid=1613392289...' },
];
let activeZone = null;       // currently selected zone object
const zonesCache = {};       // { zoneId: topics[] } populated by loadAllZones()
```

`loadAllZones()` fetches all 3 zones in parallel (Promise.all) and caches results. Called on page load to populate zone topic counts on home screen buttons.

## Google Sheet Format

The sheet uses a **row-based** format (single column A). Each topic block:

```
נושא N: כותרת הנושא
סיבות: ...
מסקנות: ...
מה יש לעשות: ...     ← label row, action items follow on subsequent rows
פעולה ראשונה         ← continuation rows, no label prefix
פעולה שנייה
מס' פעמים של לקח חוזר: 3
```

The parser (`rowsToTopics`) tracks `currentField` to accumulate unlabeled continuation rows into the last labeled field.

## Deployment

Push to `main` branch → GitHub Pages auto-deploys. The `.nojekyll` file disables Jekyll processing.

Google Sheets published CSV has ~5 minute server-side cache — content changes are not instant.

## JavaScript Function Map

| Function | Purpose |
|----------|---------|
| `parseCSV(text)` | RFC 4180 CSV parser, character-by-character, handles multi-line quoted cells |
| `rowsToTopics(rows)` | Converts row-based Hebrew-labeled sheet to topic objects |
| `loadTopics(url)` | Fetches CSV for a given URL, returns `null` on error |
| `loadAllZones()` | Parallel-fetches all 3 zones into `zonesCache` |
| `sortTopics(topics)` | Sort by `recurrence_count` desc, then Hebrew alpha |
| `filterTopics(topics, query)` | Full-text search across all 4 text fields |
| `renderSection(title, body, extraClass?)` | Renders a section; auto-numbers if body has multiple lines |
| `showView(viewId)` | Toggles between all 3 views + #main-header visibility |
| `showTopicView(topic)` | Populates and shows topic detail |
| `initMainView(zone)` | Loads zone data, renders topic list, wires search + click events |
| `renderHomeResults(results)` | Renders cross-zone search results with a zone badge |

## Topic Object Shape

```javascript
{ title, recurrence_count, causes, conclusions, actions }
```

All fields are strings except `recurrence_count` (integer, defaults to 1).

## CSS Notes

- All colors via CSS variables in `:root` (dark military palette)
- `.section-actions` has distinct styling: green border + green title background (`#8fcc6f`), dark text
- `.section-list` renders multi-line action items as a numbered `<ol>`
- `.hidden` uses `display: none !important`
- `.zone-count` — circular badge in the bottom-left corner of each zone button showing topic count. Green number (`#8fcc6f`), bold, white background, border color matches `var(--border)` (`#3a5030`), 4.5px border width, 44×44px circle via `border-radius: 50%`. Positioned `absolute` relative to `.zone-btn`.
- Home screen search results include a `.zone-badge` (blue pill) indicating which zone each result is from
