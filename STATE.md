# Trade By Wire — Project State

> CRT-monitor / fighter-jet-HUD trading game for **Cursor Vibe Jam 2026** (deadline **2026-05-01 13:37 UTC**, $35K prize pool). Reuses the Plushie Rush data backbone (bundled bars, Firebase leaderboard, prefetch cron) with a fresh focused frontend.
>
> **GitHub repo:** https://github.com/lucyellu/trade-by-wire
> **Sister project:** https://github.com/lucyellu/plush-rush (Plushie Rush — public test build, not jam-submitted)

---

## Quickstart for the next session

1. **Read this file in full** before doing anything.
2. **Day 1 is done.** Day 2 work is queued — see "Next session" section at bottom.
3. **Run locally:** `python -m http.server 8089` from `trade_by_wire/`, open http://localhost:8089/. Or double-click `Trade By Wire.lnk` on Desktop (port 8089, ASCII-only .bat with auto-zombie-kill on launch).
4. **Sister desktop shortcut** for Plushie Rush is also there (port 8088).
5. **Files tracked in git:** 96 (mostly `data/bars_*.json` × 91, plus `index.html`, `tbw.ico`, `STATE.md`, `README.md`, `.gitignore`, `data/index.json`).
6. **Files local-only (gitignored):** `tbw_launch.bat`, `_make_shortcut.ps1`, `tbw_icon_preview.png`.

## What `index.html` does (Day 1 scaffold, ~440 lines)

- 4:3 inner viewport (auto-pivots to 3:4 on mobile portrait <600px)
- Corner brackets in phosphor green at all four corners
- Scanlines overlay (1px stripes at 3px stride, 20% opacity, multiply blend)
- Slow vertical sweep band (8s loop, gradient line moving down)
- Vignette (radial gradient)
- 0.985-opacity flicker every 0.12s
- VT323 + Share Tech Mono web fonts (Google Fonts CDN)
- RGB chromatic fringe on text via `text-shadow: -0.5px 0 red, 0.5px 0 cyan`
- Boot sequence — 5 lines of phosphor text fading in over 2.4s, then hides
- HUD blocks (absolutely positioned):
  - Top-left: EARTH DATE label + value
  - Below it: SESSION timer label + value (ticks during demo)
  - Top-right: ●REC pulse + VIDEO label
  - Below it: TYPE label + value (amber)
  - Bottom-left: TARGET (big) + FREQ (06:30 · 07:30 PT)
  - Bottom-center: POSITION + P/L (big)
  - Bottom-right: BATTERY label + percent + bar (140px wide, animated width)
  - Bottom row: 3 buttons (SHORT amber-red / FLAT amber / LONG phosphor) with keyboard hints (↓/S, SPACE, ↑/W)
- Chart in center frame:
  - Loads most recent SPY bundle from `data/bars_SPY_<date>.json` (walks back up to 14 days, skips weekends/holidays automatically)
  - Phosphor line for past + dim line for future, faint horizontal grid
  - Reticle (cross + circle) at current bar with dashed vertical guide and amber price label
  - Min/max price corners
  - Demo timer advances `curBarIdx` every 200ms, looping
- Trade buttons + keyboard listeners are **wired but stubs** — they just flash the POSITION label. Day 2 connects them to real P/L.

## What's NOT yet wired (Day 2/3 work)

- Real P/L calculation from button presses against bar `c` price
- Battery gauge actually depleting on losses, refilling on profit ticks
- Tape log scrolling area inside the screen frame
- Rival trace (optimal greedy path) — amber dashed line over the price chart
- Onboarding modal (name + flag, port from plushies/game_001.html)
- Firebase REST helpers (`fbWrite`, `fbRead`, `submitToLeaderboard`) — port from plushies, tag entries with `game: 'tbw'`
- Stock of the Day rotation (same Mon-Fri map as plushies: Mon=SPY, Tue=BTC, Wed=GLD, Thu=SPY, Fri=BTC)
- Live mode (Alpaca crypto WS for BTC, stock WS for SPY/GLD between 6:30-7:30 PT)
- Summary modal at session end (CRT-styled, with TraderVue/IBKR CSV export)
- ATC chatter SFX (Web Speech API or pre-recorded clips)
- Modem dial-up boot sound, ambient hum, beep on trade events, low-battery alarm

## Confirmed design choices (locked in)

- **Name:** TRADE BY WIRE
- **Trade input:** 3 buttons (SHORT / FLAT / LONG) + keyboard (↓ S / Space / ↑ W)
- **Symbol:** locked = today's Stock of the Day (no picker)
- **Health:** BATTERY gauge — drains on drawdown
- **Audio direction:** modem dial-up boot, ambient hum, ATC chatter on trade events, low-battery alarm
- **Plushie Rush (sister project):** published as portfolio piece, NOT submitted to jam (slot reserved for TBW)
- **Repo strategy:** TBW is its own repo `lucyellu/trade-by-wire`. Plushie Rush is at `lucyellu/plush-rush`.
- **Firebase strategy:** same `games-8d8a6` project, same `plushrush/` namespace for now, BUT entries from this game tagged with `game: 'tbw'` so the leaderboard can filter or merge.

## Critical setup notes

### Icon
- Source: `C:\Users\lucyl\Downloads\image-1015312428341233.jpeg` (the white-bg CRT-with-EKG image)
- Auto-cropped via Pillow with white-bg threshold (≥235 RGB → transparent), centered square at 730×730, resized to 256×256
- Saved as multi-res `tbw.ico` (16/24/32/48/64/128/256)
- Preview at `tbw_icon_preview.png` (gitignored)

### Desktop shortcut
- Path: `C:\Users\lucyl\Desktop\Trade By Wire.lnk`
- Target: `tbw_launch.bat` (port 8089)
- Working dir: `trade_by_wire/`
- Icon: `tbw.ico`
- To regenerate: run `_make_shortcut.ps1` via `powershell -NoProfile -ExecutionPolicy Bypass -File _make_shortcut.ps1`

### .bat gotcha (don't repeat)
The original `.bat` had `█` block characters and `—` em-dash saved as UTF-8, plus a piped `for /f` loop with caret-pipes (`netstat -aon ^| findstr ^| findstr LISTENING`). Cmd.exe choked on the combo, threw "... was unexpected at this time", and closed the window before user could see anything. **Fix:** ASCII-only banner using `#` characters, dash instead of em-dash. Pipe netstat output to `%TEMP%\tbw_port.txt` then `type` it inside the for loop — avoids in-line caret-piping which is fragile.

### Port collision (don't repeat)
Both shortcuts originally used port 8088 — clicking Trade By Wire just landed on the running Plushie Rush server. Now: Plushie Rush=8088, Trade By Wire=8089. Each .bat also runs zombie-kill on its own port at startup, so a stale server can't shadow a new launch.

## Files

| File | Tracked? | Purpose |
|---|---|---|
| `index.html` | ✓ | The game. Single-file. Day 1 scaffold. |
| `data/bars_*.json` | ✓ | 91 prefetched morning-hour bundles (SPY/BTC/GLD × ~30 days each) |
| `data/index.json` | ✓ | Bundle catalog |
| `tbw.ico` | ✓ | Multi-res app icon |
| `STATE.md` | ✓ | This file |
| `README.md` | ✓ | Public-facing intro |
| `.gitignore` | ✓ | Ignores config.json, .env, .claude/, _make_shortcut.ps1, tbw_launch.bat, tbw_icon_preview.png |
| `tbw_launch.bat` | ✗ | Local launcher (port 8089) — Windows-specific, no need on GitHub |
| `_make_shortcut.ps1` | ✗ | One-shot to recreate Desktop shortcut |
| `tbw_icon_preview.png` | ✗ | Visual preview of the icon |

## Reuse map from plushies/game_001.html

When porting features in Day 2, copy these from `..\plushies\game_001.html`:

| Concern | Lines (approximate, will drift) | Notes |
|---|---|---|
| FLAGS array (45 [emoji, code] pairs) | ~3140-3160 | Used by onboarding + leaderboard |
| `flagImgHTML(flagEmoji)` helper | ~3490 | Returns `<img src="https://flagcdn.com/...">` |
| `getProfile()` / `saveProfile(p)` | ~3170-3175 | localStorage `plr001_profile` (or rename to `tbw_profile`) |
| `fbWrite(path, data)` / `fbRead(path)` | ~3315-3340 | Firebase REST helpers |
| `submitToLeaderboard(pnl, pct)` | ~3355-3380 | POST to `plushrush/leaderboard` — add `game:'tbw'` tag |
| `fetchGlobalLeaderboard(force)` | ~3400-3415 | 10s cache |
| Onboarding modal HTML | ~510-540 | Restyle to CRT |
| Summary modal HTML + JS | ~545-590 + ~3220-3290 | The screenshot-worthy card with confetti / sad-trombone / fun fact / CSV export |
| `exportTradesCSV()` | ~3290-3315 | TraderVue/IBKR Flex format |
| `buildFunFact(snap)` compounding math | ~3640-3665 | "+0.5% daily compounds to $3.5M/year" message |
| `spawnConfetti(n)` + `stepConfetti()` | ~3695-3760 | Profit celebration |
| `sfxSadTrombone()` | ~3765-3775 | Loss SFX |
| Stock-of-the-day map + `stockOfTheDayIdx()` | ~1120-1140 | Mon-Fri symbol rotation |
| `ptNow()` / `ptDateStrOffset(daysBack)` / `isWeekend()` / `getAvailableDates()` | ~1240-1300 | Date helpers (already partially in tbw scaffold) |
| `ptToUTC(date, hour, minute)` | ~530-545 | PT→UTC for Alpaca queries (already in scaffold) |
| `fetchAlpacaBarsCached(symInfo, date)` chain | ~545-625 | Bundle → live → Binance/Yahoo fallback |

## Vibe Jam submission status

- ✅ Vibe Jam tracking widget already in `<head>` of `index.html` (line ~9: `<script async src="https://vibej.am/2026/widget.js">`)
- ✅ Required for entry — must NOT be removed
- ☐ Need: live deploy URL (Netlify drop or git-connected)
- ☐ Need: submit URL at vibej.am/2026/ before 2026-05-01 13:37 UTC

User has Plushie Rush already on Netlify (testing). Plan: separate Netlify site for TBW, connected to `lucyellu/trade-by-wire` repo — every git push auto-deploys.

## Day-by-day timeline (3 days remaining)

| Day | Date | Goal | Done = |
|---|---|---|---|
| Day 1 | Apr 27-28 | Scaffold ✓ done | this commit `1847722` |
| **Day 2** | Apr 28-29 | Trade input wired, P/L, battery, tape log, rival trace, Firebase port | "playable end-to-end, score saves" |
| Day 3 | Apr 29-30 | Polish: ATC chatter, summary card restyle, mobile layout, deploy | "feels finished, share-able URL" |
| Day 4 | May 1 (before 13:37 UTC) | Final phone+laptop test, submit URL to vibej.am | "submitted" |

## Next session: kick off Day 2

Open the file in browser first to confirm Day 1 still works. Then in priority order:

1. **Port profile + onboarding** — copy `getProfile()`, `saveProfile()`, FLAGS, `flagImgHTML`, the onboarding modal HTML/CSS/JS from plushies. Restyle modal to CRT (phosphor green border, monospace font, scanlines on the modal too).
2. **Wire trade buttons** — add `position` state (-1/0/1), `entryPrice`, `realPnL`. On button click: if state changes, close existing position into realPnL, open new at current bar's `c` price. Show in P/L block.
3. **Battery gauge** — start at 100%, drain by `(losingDelta / startBalance * 100)` on each tick where you're losing. Refill (slowly) when profit ticks. At 0% → "RECORDING ENDED" → summary.
4. **Tape log** — small monospace strip near bottom (currently empty). Each trade adds a line: `06:42:15  BUY  SPY  $710.27  100`. Newest at top, last 6 visible.
5. **Rival trace** — port `computeOptimal(bars)` from plushies. Draw amber dashed line on chart showing optimal-foresight equity curve scaled to chart bounds.
6. **Firebase** — port `fbWrite/fbRead/submitToLeaderboard`. Tag every entry `game: 'tbw'`. Test write at end of session.
7. **Stock of the Day** — port the rotation map and apply on boot.

After this milestone, the game is end-to-end playable. Day 3 is purely polish.
