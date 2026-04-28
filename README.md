# TRADE BY WIRE

You are looking through a battered CRT monitor inside a 1980s recording bay. Each session is one trading hour, recorded onto magnetic tape. A target ticker is locked. Your job is to ride the price — long, short, or flat — until the recording stops. The screen flickers, scanlines roll, the battery drains as you lose money. Survive the hour. Beat your past self. Beat the world.

Built for [Cursor Vibe Jam 2026](https://vibej.am/2026/). 100% AI-written.

## Play

- Live: deploy below
- Local: `python -m http.server 8088` from this folder, then open http://localhost:8088/

## Controls

- ↑ or W — go LONG
- ↓ or S — go SHORT
- Space — flatten (close to neutral)
- Three on-screen buttons map to the same actions on touch

## What's in here

| File | Purpose |
|---|---|
| `index.html` | The game. Single-file, all CSS/JS inline. |
| `data/bars_*.json` | Pre-fetched morning-hour OHLCV bars (~30 trading days × SPY/BTC/GLD) |
| `STATE.md` | Internal handoff doc — read first if continuing development |

## Status

**Day 1 scaffold.** Visual frame, scanlines, HUD layout, real chart loading from bundle. Trade input not yet wired (Day 2). See `STATE.md` for full progress.

## Tech

- No build step, no bundler, no npm
- Pre-fetched bars + Binance/Yahoo public-API fallbacks → game is playable without keys
- Firebase Realtime DB (REST) for global leaderboard + visit tracking
- TraderVue / IBKR Flex CSV export of trade history
