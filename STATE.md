# Trade By Wire — Project State

> CRT-monitor / fighter-jet-HUD trading game for Vibe Jam 2026 (deadline 2026-05-01 13:37 UTC). Reuses the Plushie Rush data backbone (bundled bars, Firebase leaderboard, prefetch cron) with a fresh, focused frontend.

## Current state — Day 1 scaffold (2026-04-27 evening)

Just-shipped: `index.html` with the visual frame, boot loader, HUD layout, scanlines, real chart drawing.

**Working now:**
- 4:3 inner viewport with corner brackets
- Scanlines + slow vertical sweep + vignette + barely-perceptible flicker
- Phosphor green palette (`#00ff85`) with amber accents, RGB chromatic aberration on text via `text-shadow`
- VT323 + Share Tech Mono web fonts
- Boot animation (5 lines fade in over 2.4s)
- HUD positions: EARTH DATE, SESSION timer, ● REC indicator, TYPE, TARGET, FREQ, POSITION, P/L, BATTERY (with bar)
- Chart canvas in center frame:
  - Loads the most recent SPY bundle from `data/`
  - Draws past-as-bright-phosphor + future-as-dim-line
  - Reticle (cross + circle) tracks current bar
  - Min/max price labels in corners
- Demo playback: bar pointer advances every 200ms, looping
- Three trade buttons (SHORT / FLAT / LONG) — keyboard ↓/Space/↑ — placeholder, just flashes the position label

**Not yet wired (Day 2 work):**
- Real P&L calculation from trades
- Battery gauge actually depleting
- Tape log scrolling at bottom
- Rival trace (optimal path) overlay on chart
- Firebase score submission
- Onboarding modal (name + flag — copy from plushies)
- Summary modal at end-of-session
- Stock of the Day rotation logic
- Live mode (WS price feed during 6:30–7:30 PT)

## Files

| File | Purpose |
|---|---|
| `index.html` | The game. Single file, all CSS/JS inline. Day 1 visual scaffold. |
| `data/` | 91 prefetched morning-hour bundles (SPY/BTC/GLD × 30 days). Copied from plushies. |
| `STATE.md` | This file. |
| `README.md` | Public-facing project intro. |

## Run

```bash
cd trade_by_wire
python -m http.server 8088
# open http://localhost:8088/
```

Mobile: same URL on phone (must be same Wi-Fi).

## Confirmed design choices

- **Name**: TRADE BY WIRE
- **Trade input**: 3 buttons (SHORT / FLAT / LONG) + keyboard `↓ S / Space / ↑ W`
- **Symbol**: locked = today's Stock of the Day (no picker on main screen)
- **Health**: BATTERY gauge (drains on drawdown, refills on profit ticks)
- **Audio**: modem dial-up boot, ambient hum, ATC chatter on trade events, low-battery alarm
- **Plushie Rush**: published as portfolio piece, NOT submitted to jam (slot reserved for TBW)

## Next session

Day 2 plan:
1. Wire P&L computation from button presses against bar `c` price
2. Battery drains: every frame while position open and price moving against you
3. Build tape log scrolling area inside the screen frame
4. Add rival trace = optimal greedy path overlay (amber dashed line)
5. Port `getProfile()`, `FLAGS`, `flagImgHTML()`, Firebase REST helpers from plushies/game_001.html
6. Onboarding modal — same name+flag pattern, restyled CRT
7. Stock of the Day rotation by PT weekday (same map as plushies)

After Day 2 the game is end-to-end playable. Day 3 = polish + audio + summary card. Day 4 = deploy + submit.

## Design references

The reference frames the user shared:
- NGC 4459 black-hole CRT monitor view
- Corner brackets framing the central image
- "EARTH DATE / RECORDING / TARGET / TYPE / BATTERY" labels
- Phosphor + RGB chromatic fringe + scanlines aesthetic

Translated 1:1 to the trading domain: TARGET = symbol, TYPE = asset class + timeframe, BATTERY = drawdown-tolerant cash, EARTH DATE = today, SESSION = clock against the 1-hour level.

## Reuse from plushies/

When porting features in Day 2:
- **Same Firebase project** (`games-8d8a6`) — entries tagged `game: 'tbw'` so leaderboards can filter
- **Same FLAGS array** (45 country flag emoji + ISO codes)
- **Same flagcdn.com images** (since Windows can't render flag emojis)
- **Same TraderVue/IBKR CSV export format** (Date, Time, Symbol, Quantity, Price, Side)
- **Same bar-loader fallback chain** (bundle → live Alpaca → Binance for crypto / Yahoo for stocks)
- **Same prefetch cron** (`.github/workflows/prefetch-bars.yml` — keep one in plush-rush repo, both games' deploys can reference its data via the deployed netlify URL or copy on commit)

## Open items

- [ ] Decide repo layout: copy `data/` here on commit, or have TBW fetch from `plush-rush.netlify.app/data/`? Currently copied locally for self-contained local dev.
- [ ] Decide Firebase namespace: `tbw/` separate, or same `plushrush/` with a tag? Current plan: same DB, tag entries.
- [ ] Find ATC chatter clips — short royalty-free wav samples
- [ ] Decide on submitting both games (separate slots) or just TBW
