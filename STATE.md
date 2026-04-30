# Trade By Wire — Project State

> CRT-monitor / fighter-jet-HUD trading game for **Cursor Vibe Jam 2026** (deadline **2026-05-01 13:37 UTC**, $35K prize pool). Reuses the Plushie Rush data backbone (bundled bars, Firebase leaderboard, prefetch cron) with a fresh focused frontend.

---

## Day 3 update (2026-04-29, v4) — chart-dominant layout + custom tickers + Mario SFX + jam widget removed

`index.html` ~2550 lines. Round 4 changes:

- **Chart now dominates the viewport.** Bottom HUD compacted aggressively. Numbers:
  - Chart frame: was `top:116 bottom:260` → now `top:110 bottom:168`. On a 4:3 768px screen, chart went from ~392px tall (51% of vp) → ~490px tall (~64% of vp). Mobile portrait went from ~422px → ~524px chart.
  - Portfolio total: 56px → 36px. Row $/% pair: 28px → 18px. Position line: 14px → 12px.
  - Trade buttons: 78px tall (font 28px) → 48px tall (font 18px). Mobile keeps tap-friendly 92px.
  - Side T&S / TAPE panels: 158px wide → 110px. MACD sub-pane: 36% of center → 22%.
  - End result: chart ~25% larger on desktop, ~24% larger on mobile. (Couldn't go full 2× on a fixed viewport without unusable buttons; pushed as far as is sane.)

- **Insufficient-buying-power toast.** Anywhere `applyOrder()` rejects (cap hit), a red toast pops above the portfolio block: "INSUFFICIENT BUYING POWER". Plays the new `sfxAttack()` so the rejection has audible weight. Auto-fades in ~1.4s.

- **Custom ticker search.** Menu's second card replaced with an input + GO button. Type AAPL/NVDA/MSFT/whatever, presses Enter or clicks GO, fetches live 1-min bars from Yahoo Finance via `corsproxy.io`. New helper `fetchYahooBars(ticker)`:
  - Validates `^[A-Z][A-Z0-9.\-]{0,5}$`
  - Hits `query1.finance.yahoo.com/v8/finance/chart/{T}?interval=1m&range=1d`
  - Caches per-ticker for the session
  - Registers ticker in `SYM_INFO` so HUD labels render
  - Falls back gracefully — `loadBundle()` now tries local bundle → Yahoo if not present
  - Status text in the card: "fetching AAPL...", "loaded 60 bars", or "no data for XYZ" (red)

- **Vibe Jam widget removed from HTML.** Was: `<script async src="https://vibej.am/2026/widget.js"></script>` in `<head>`. Out per user request — they want the game free of jam-specific runtime code. **Re-add this single `<script>` tag before submitting to vibej.am/2026 if you still want to enter; the jam tracker requires it.**

- **Portfolio direction SFX = Mario-style.**
  - **UP:** classic NES coin grab — two ascending square-wave notes, B5 (987.77 Hz) → E6 (1318.51 Hz), 60 ms gap. (`sfxCoin()`)
  - **DOWN:** punchy descending attack — sawtooth glide 420→110 Hz then a 90 Hz sub-bass thud. (`sfxAttack()`)
  - Threshold raised to 0.005% of starting balance (= $50 on a $1M book) to keep it musical, not noisy.

- **Menu cleanup.** Removed the redundant "OPEN MARKET" button (SPY card already does the same). Settings moved to its own row at the bottom of the menu.

- **60 unit tests still passing.** No test changes needed for this round (changes are HTML/CSS/audio + new fetch helper that's not exercised by the test stubs).

### Quick test plan
1. Hard-refresh `http://localhost:8089/`.
2. Menu shows two cards: SPY (left) and CUSTOM input (right). Type "AAPL", press Enter — status shows "fetching AAPL..." then "loaded N bars" and game starts on AAPL's last trading day.
3. Top HUD shows "TICKER AAPL" and the chart fills most of the screen.
4. Press BUY 4× rapidly — exposure goes 25%→50%→75%→100%. 5th BUY shows red "INSUFFICIENT BUYING POWER" toast + plays attack SFX.
5. As portfolio rises bar-to-bar, you hear the Mario coin ding. As it falls, the attack hit.
6. Open in mobile DevTools (or actual phone). Chart still dominant, buttons still tappable.
7. Confirm no Vibe Jam tracker call in network panel.

### Day 3 still pending
- (5) Netlify deploy from `lucyellu/trade-by-wire`
- (6) Submit URL to vibej.am/2026 before 2026-05-01 13:37 UTC — **remember to re-add the `<script async src="https://vibej.am/2026/widget.js">` tag in `<head>` before deploying for jam submission, or skip the jam.**

---

## Day 3 update (2026-04-29, v3) — Bloomberg pivot + stacking orders + settings + speed slider

`index.html` ~2410 lines. Big overhaul of the game model and the UI lexicon:

- **Theme: fighter-jet → Bloomberg-terminal trading desk.** Same CRT/phosphor/scanlines aesthetic, but every label is now market-language. Rename map:
  - REC dot + VIDEO → green LIVE dot + MARKET
  - EARTH DATE → trade date in a small `#hud-meta` line under the TICKER
  - TARGET (HUD) → TICKER (HUD, top-left big)
  - TYPE → exchange line in `#hud-meta` ("NYSE · 1MIN")
  - EJECT → CLOSE OUT (still 10s lockout)
  - PILOT EJECTED → TRADER EXITED
  - RECORDING ENDED / RECORDING TERMINATED → MARKET CLOSED / END OF SESSION
  - SESSION END → DAY CLOSED
  - NEW PILOT → NEW TRADER · CALLSIGN → NAME · SECTOR → COUNTRY
  - TARGET ACQUIRED → TICKER LOADED · CRT MONITOR ONLINE → TERMINAL ONLINE · RECEIVING SIGNAL → MARKET DATA SYNC · AWAITING TRANSMISSION → LIVE FEED · STAND BY
  - TOP TRANSMISSIONS · LIVE FROM ALL PILOTS → TOP TRADERS · LIVE FROM ALL DESKS
  - SUBMIT/TRANSMIT → POST · TRANSMITTING → POSTING · TRANSMITTED → POSTED · NO TRANSMISSIONS → NO TRADES POSTED YET
  - VS RIVAL stat / chart label → VS BENCHMARK / BENCHMARK
  - ATC chatter phrases → trading-floor phrases ("buy filled", "long order in", "market closed")

- **Layout overhaul to kill overlaps.** Top now carries TICKER (big) + DATE/TIMER/EXCHANGE in a single small row underneath; top-right has the green MARKET LIVE indicator + a SPEED slider. Top-center toolbar has SET / SOUND ON / CLOSE OUT buttons (CLOSE OUT only appears mid-session). Bottom is a clean stack: a single full-width POSITION line, then the PORTFOLIO centerpiece (LBL / huge total / $P/L · %P/L row), then SELL / BUY buttons. Chart frame moved from bottom:210 → bottom:260 to give the portfolio block real breathing room.

- **Position model rewrite — orders STACK.** Previous version was a binary toggle (one press = enter at 25%, same direction = no-op). Now:
  - Each click of BUY / SELL fills one order at the current step size (`settings.stepPct`, default **25%** of `STARTING_BALANCE`).
  - Same-direction stacking weights the average entry price.
  - Opposite direction: partially closes (FIFO equivalent on net), realizes P/L on the closed portion. If the order is bigger than the existing position, it closes everything and opens the remainder in the new direction at the current price.
  - Hard cap at **±100% of starting balance** notional (`MAX_EXPOSURE_PCT`). Orders that would push past the cap are silently rejected (no partial fill — keeps the math clean).
  - State now tracks `state.netShares` (signed) + `state.avgEntry`. Legacy fields (`state.position`, `state.entryShares`, `state.entryPrice`) are kept in sync for back-compat with chart-arrow rendering and the test suite.
  - Position readout updated: "LONG 75% · 7,500 sh · avg $451.20" instead of single-position info.

- **Settings overlay.** Opens from the menu's "> SETTINGS" button or the HUD's "SET" button. Two controls, both persisted to `localStorage.tbw_settings`:
  - **Order Size**: 4-button radio for 10% / 25% / 50% / 100% of buying power per click.
  - **Playback Speed**: 1× to 60× slider (1× = real-time 60-min session, 60× = 1-min session).
  - "SAVE & CLOSE" returns to menu if not in session, or back to live game if mid-session.

- **Speed slider in HUD.** Top-right shows current speed and a slider; changing it mid-game restarts the tick interval immediately. Also reflected in the settings overlay (kept in sync).

- **Portfolio direction SFX.** Added `sfxPortfolioTick()` — fires on every bar tick, plays a brief 45ms blip whose pitch reflects portfolio direction (720Hz triangle if up, 280Hz square if down) with volume scaled by magnitude. Capped low so it stays ambient. Skipped when delta is < $1 to avoid noise from tiny float drift.

- **Flag images in leaderboard.** Replaced raw emoji rendering with `flagImgHTML(flag)` so flags show as actual `<img>` tiles from `flagcdn.com` — uniform across operating systems instead of hit-or-miss emoji glyphs.

- **MACD = bars only** (already in v2; bumped alpha 55% → 65%).

- **60 unit tests passing** (was 45). New coverage: settings.stepPct override, tickMs at 1× / 30× / 60×, stacking weighted-avg entry, partial reduce vs flip-through-zero, exposure cap rejection.

### Quick test plan
1. Hard-refresh `http://localhost:8089/` (Ctrl+Shift+R).
2. PORTFOLIO shows $1,000,000 in white. Top has TICKER SPY + LIVE dot + SPEED 60×.
3. Click SET → pick 10% → SAVE & CLOSE. Click PLAY.
4. Click BUY 4× rapidly. Position grows: LONG 10% → 20% → 30% → 40%. PORTFOLIO updates, $P/L pulses on cross-zero.
5. Drag SPEED slider down to 5×. Bars start ticking every 12 seconds; a 1-hour session would take 12 minutes at this pace.
6. Click SELL once. Position partially reduces (e.g. LONG 40% → 30%); subsequent SELLs continue to reduce, then flip to short.
7. Stack BUY×4 then try BUY again — rejected (capped at 100% exposure).
8. Wait 10s+, hit ESC. "TRADER EXITED" → summary shows "✓ POSTED TO LEADERBOARD".
9. Open LEADERBOARD — flags render as flag image tiles, not emoji.

### Day 3 still pending
- (5) Netlify deploy from `lucyellu/trade-by-wire`
- (6) Submit URL to vibej.am/2026 before 2026-05-01 13:37 UTC

---

## Day 3 update (2026-04-29, v2) — game-design overhaul

`index.html` ~2140 lines. Major rewrite of the game model on top of the audio/polish pass:

- **$1M starting portfolio** (`STARTING_BALANCE = 1_000_000`). Each move sizes at **25% buying power** (`POSITION_PCT = 0.25`) via `sharesAt(price) = floor(0.25 * 1_000_000 / price)`. Fixed sizing against starting balance — deterministic, easy to reason about. `state.entryShares` carries the per-trade qty.
- **Two buttons only — BUY (long) / SELL (short).** No FLAT button. Pressing same direction = no-op. Pressing opposite = close + flip. Player is either short, long, or pre-trade flat; can't go flat by clicking once they've started.
- **Battery removed entirely.** No drain, no alarm, no `bat-crit` screen flash. The only end-conditions are bars exhausted, ESC/EJECT, or session-complete timer.
- **Bigger keyboard map.** Up / W / Right / D = BUY. Down / S / Left / A = SELL. **Esc** (or Q legacy) = EJECT. Spacebar binding gone.
- **PORTFOLIO centerpiece** dominates the bottom HUD: huge total ($1,000,000), then $P/L and %P/L on the row below (color-coded green/red, glow pulse on cross-zero). Position readout shrinks to a small top-right line ("LONG · 2,500 @ $452.10").
- **Auto-submit to leaderboard** on any session that lasted >10s (`MIN_LOG_MS = 10000`). The 10-second EJECT lockout is unchanged. The summary modal's submit button is now read-only — shows ✓ TRANSMITTED or × TOO SHORT TO LOG.
- **MACD bars only** (no MACD line, no signal line). Bars get bumped from 55% → 65% alpha to compensate.
- **Mobile portrait pass.** Below 700px portrait: full-screen viewport (no aspect ratio), MACD hidden (chart-only), side panels hidden, big tap targets (110px tall buttons), portfolio readout scaled down. Mute button moves to top-center, EJECT moves below it.
- **45 unit tests passing** (was 39). Removed the battery-drain test, added `sharesAt` helper tests, updated the P/L expected values for $1M sizing.

### Audio + polish (still in from v1)
- Web Audio: dial-up modem boot at session start, ambient hum, BUY/SELL beeps, end-of-session klaxon.
- ATC chatter via Web Speech (LONG / SHORT / END phrases — battery chatter removed).
- P/L cross-zero glow pulse now fires on the PORTFOLIO TOTAL (was the old `#val-pnl`).
- 0.55s screen shake on session end.
- Mute toggle: `#hud-mute` button + M key, persisted in `localStorage.tbw_muted`.

### Quick test plan
1. Hard-refresh `http://localhost:8089/` (Ctrl+Shift+R).
2. Click PLAY. PORTFOLIO shows $1,000,000 in white.
3. Press ↑ or → (or D / W). Big BUY beep + ATC voice. Portfolio number turns green/red as price moves; $ and % both update; cross-zero pulses the total.
4. Press ↓ or ← (or A / S). Position flips to short.
5. Hold for 10+ seconds, press ESC. Eject works, summary shows "✓ TRANSMITTED TO LEADERBOARD".
6. Replay, eject before 10s — summary shows "× TOO SHORT TO LOG".
7. Resize browser to phone width (or open on phone). Side panels + MACD vanish, buttons are huge, portfolio still readable.

### Day 3 still pending
- (4) Live mode (Alpaca WS) — defer post-jam unless time
- (5) Netlify deploy from `lucyellu/trade-by-wire`
- (6) Submit URL to vibej.am/2026 before 2026-05-01 13:37 UTC

---

## Day 3 v1 (audio + polish, superseded by v2 above but still in code)

Items 1 + 2 from the original Day 3 priority queue:

- **Web Audio synth module** (no external files, all single-file). Lazy-init AudioContext on first user gesture (Chrome autoplay policy).
- **Modem dial-up handshake** plays at session start (`startSession()` → `sfxModemBoot()` → `sfxStartHum()` 1.7s later).
- **Ambient hum** — 60 Hz sine + 0.3 Hz LFO, gain 0.018. Stops on `endSession`.
- **Trade SFX** — `sfxBuy()` rising triangle, `sfxSell()` falling triangle, `sfxFlat()` short square. Wired into `setPosition()`.
- **Low-battery alarm** — pulsing 820/620 Hz at <=20% battery, with red screen-edge flash (`#screen.bat-crit` keyframes). Stops above 20%.
- **End-of-session klaxon** — descending sawtooth pair, plus 0.55 s screen shake (`#screen.shake`).
- **ATC chatter** via Web Speech API (`sfxAtc()`). Throttled (600 ms min gap), low pitch (0.6), fast rate (1.5), volume 0.55. Picks lowest-pitched English voice. Phrases: ATC_LONG / ATC_SHORT / ATC_FLAT / ATC_BAT / ATC_END.
- **Mute toggle** — `#hud-mute` button top-center, M key, persisted in `localStorage.tbw_muted`. Stops alarm + hum + speech immediately; resumes hum/alarm if toggled back on mid-session.
- **P/L glow pulse** on cross-zero — `#val-pnl.pop-profit` / `.pop-loss` keyframes. Tracks previous sign in `_lastPnLSign`, only triggers on actual crossings (not the first non-zero).
- **39 unit tests still green** (`node _tbw_test.mjs`).

### Quick test plan for the next session
1. Hard-refresh the page (Ctrl+Shift+R; Python's HTTP server caches hard).
2. Click PLAY — modem handshake should play, then hum drones underneath.
3. Click LONG (or ↑/W) — quick rising beep + ATC voice line.
4. Watch P/L cross from negative to positive (or vice versa) — number pulses + glows.
5. Drain battery below 20% — pulsing alarm + red screen-edge flash; ATC says "battery critical".
6. Hit EJECT (or let session end) — descending klaxon + screen shake + ATC sign-off.
7. Hit M — everything mutes instantly, button shows "SOUND OFF" in red. Persists across reload.

### Known minor things (not blockers)
- ATC voice quality is browser-dependent; on Edge/Chrome desktop it's serviceable, on Firefox the voice list can be empty for ~1 s after load (handled via `onvoiceschanged`).
- Mute button sits at `left: calc(50% - 80px)` (top center, just left of EJECT). On very narrow viewports they can crowd; cleanup deferred to mobile pass.

### Day 3 still pending
- (3) Mobile portrait final pass (real-device check)
- (4) Live mode (Alpaca WS) — may defer post-jam
- (5) Netlify deploy from `lucyellu/trade-by-wire`
- (6) Submit URL to vibej.am/2026 before 2026-05-01 13:37 UTC

---
>
> **GitHub repo:** https://github.com/lucyellu/trade-by-wire
> **Sister project:** https://github.com/lucyellu/plush-rush (Plushie Rush — public test build, not jam-submitted)

---

## Quickstart for the next session

1. **Read this file in full** before doing anything.
2. **Day 1 + Day 2 are done.** Day 3 work is queued — see "Next session" section at bottom.
3. **Run locally:** `python -m http.server 8089` from `trade_by_wire/`, open http://localhost:8089/. Or double-click `Trade By Wire.lnk` on Desktop (port 8089, ASCII-only .bat with auto-zombie-kill on launch).
4. **Sister desktop shortcut** for Plushie Rush is also there (port 8088).
5. **Run tests:** `node _tbw_test.mjs` — 39 unit tests covering pure logic (SOTD map, optimal curve, position math, profile, battery drain, tape log).
6. **Files tracked in git:** 96 (mostly `data/bars_*.json` × 91, plus `index.html`, `tbw.ico`, `STATE.md`, `README.md`, `.gitignore`, `data/index.json`).
7. **Files local-only (gitignored):** `tbw_launch.bat`, `_make_shortcut.ps1`, `tbw_icon_preview.png`, `_tbw_test.mjs`, `_served.html`.

## What `index.html` does (Day 2 build, ~1180 lines)

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
- Trade buttons + keyboard wired to **real P/L** (long/short/flat, close-and-flip on opposite-direction press, $10k notional × 100 shares).
- **Stock of the Day** rotation applied at boot (Mon/Thu=SPY, Tue/Fri=BTC/USD, Wed=GLD; weekend defaults to BTC).
- **Onboarding modal** (CRT-styled phosphor card) on first visit — captures callsign + flag, persists to `localStorage` key `tbw_profile`. Browser locale used for default flag pick.
- **Tape log** strip at top-right of chart area, monospace, last 6 trades, color-coded BUY/SELL/CLOSE.
- **Battery gauge** drains tick-by-tick on equity drawdown (full rate), refills on profit at half rate. Hits 0% → "RECORDING ENDED" stamp → summary modal.
- **Rival trace** — amber dashed equity curve from `computeOptimalCurve()` (greedy: enter long every rising bar, exit on first dip). Scaled to chart bounds, "RIVAL" tag at right edge.
- **Summary modal** (CRT-styled) on end-of-session: big P/L number, trade count, win rate, vs-rival %, CSV export, replay, "TRANSMIT TO LEADERBOARD" button.
- **Firebase REST helpers** ported (`fbWrite`/`fbRead`/`submitToLeaderboard`/`trackVisit`). Same `games-8d8a6` project, `plushrush` namespace, every entry tagged `game: 'tbw'` so leaderboard reads can filter.
- **CSV export** in TraderVue/IBKR-compatible format.

## What's NOT yet wired (Day 3 work)

- Live mode (Alpaca crypto WS for BTC, stock WS for SPY/GLD between 6:30-7:30 PT)
- ATC chatter SFX (Web Speech API or pre-recorded clips)
- Modem dial-up boot sound, ambient hum, beep on trade events, low-battery alarm
- Visual polish: end-stamp shake animation, button-press flash, SFX on trade
- Mobile portrait final pass (especially tape log + summary modal)
- Netlify deploy connected to repo, vibejam submission

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
| Day 1 | Apr 27-28 | Scaffold ✓ done | commit `1847722` |
| Day 2 | Apr 28 | Trade input wired, P/L, battery, tape log, rival trace, Firebase port, SOTD ✓ done | this build (`index.html` ~1180 lines, 39 unit tests passing) |
| **Day 3** | Apr 29-30 | Polish: ATC chatter, low-battery alarm, mobile pass, Netlify deploy | "feels finished, share-able URL" |
| Day 4 | May 1 (before 13:37 UTC) | Final phone+laptop test, submit URL to vibej.am | "submitted" |

## Next session: kick off Day 3

The game is end-to-end playable. Open in browser, do a full session (let it auto-tick or trade actively), confirm:
- Onboarding modal appears on first visit, is dismissable with Enter
- Boot sequence shows correct symbol (e.g. on a Tuesday, BTC)
- Tape log fills with newest-first trades
- Battery drains on losing ticks, recovers on winners
- Rival trace draws as amber dashed curve, "RIVAL" label at right
- Session-end shows summary modal with stats, CSV button works
- "Transmit to Leaderboard" actually writes to Firebase

Then Day 3 priority queue:

1. **Audio pass** — ATC chatter on trade events (Web Speech API), modem-dial boot sound, ambient hum, low-battery alarm beep at <20%.
2. **Polish micro-feedback** — flash on trade button press, screen shake at session end, brief glow pulse on big P/L number when it goes positive/negative.
3. **Mobile portrait pass** — verify tape log readable, summary modal scrollable, buttons hit-targetable. Already responsive but needs a real-device check.
4. **Live mode** (optional, may defer to post-jam) — Alpaca WS subscribe between 06:30-07:30 PT for SPY/GLD/BTC, replace bundled bars with real ticks.
5. **Netlify deploy** — connect `lucyellu/trade-by-wire` to a new Netlify site, custom subdomain, every push auto-deploys.
6. **Submit to vibej.am/2026** — paste live deploy URL, confirm widget script is firing.

Useful one-liners while iterating:
- `python -m http.server 8089` (or double-click Trade By Wire.lnk)
- `node _tbw_test.mjs` (unit tests for pure logic, must stay green)
