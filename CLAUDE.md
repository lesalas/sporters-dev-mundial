# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quiniela del Mundial 2026 — a single-page prediction game for the FIFA World Cup 2026. Players predict match scores; points are awarded for exact and directional predictions. Includes a jackpot bonus game ("MÁS") and an admin panel for managing results.

## Running the App

No build process. Open `index.html` directly in a browser, or serve it with any static file server:

```
npx serve .
```

## Architecture

The entire application is a **single file: `index.html`**. It is ~712 lines split into:

- **Lines 1–87:** HTML structure + embedded CSS (dark theme, `#0d1b2a` bg, `#e63946` accent)
- **Lines 88–180:** Firebase init + hardcoded match data (`GROUPS`, `MATCH_DATES`, `MATCHES`)
- **Lines 181–712:** All application logic (vanilla JS, no framework)

### Firebase Collections

| Collection | Purpose |
|---|---|
| `users` | Registered player names |
| `picks` | Per-user score predictions, keyed by `userId_matchId` |
| `config/locks` | Per-match lock flags (set when match kicks off) |
| `config/results` | Official results entered by admin |
| `config/mas_state` | Jackpot pool state (pool amount, eligible players, current cycle) |
| `mas` | Archive of settled jackpot rounds |

### Tab Structure

1. **Partidos** — User picks (score inputs, locked when match starts)
2. **Tabla** — Leaderboard (1 pt exact, 2 pts correct direction "Bol.")
3. **Todos los picks** — All players' predictions per match
4. **MÁS** — Jackpot bonus game
5. **⚙️ Admin** — Password-protected results entry, locking, and MÁS settlement

### Authentication

Name-only login stored in `localStorage['quiniela_user']`. No password auth for regular users.  
Admin panel uses `ADMIN_PASS = "mundial2026"` checked against `sessionStorage['admin_ok']`.

### Scoring System

- **Exacto (1 pt):** Predicted the exact score (e.g., 2-1 → 2-1)
- **Boletazo / Bol. (2 pts):** Predicted the correct result direction (Win/Draw/Loss) but not exact score

### MÁS Jackpot Logic (`settleMas`, lines ~316–385)

- Pool accumulates automatically each match (2 pts × number of participants)
- Only **eligible players** (those who haven't won in the current cycle) can win the pool
- If an eligible player gets exact score → wins the full pool, cycle resets for that player
- If an ineligible player gets exact score → receives only their match share, pool continues accumulating
- "Reliquidar" button reverts the last settlement and re-runs it (for corrections)

### Match Data

- `GROUPS`: 12 groups (A–L), 4 teams each
- `MATCHES`: auto-generated from groups + 5 knockout placeholders
- `MATCH_DATES`: lookup by `"home-away"` key → `{ date, time, venue }`

Knockout match slots are placeholders (TBD teams) until group stage completes.
