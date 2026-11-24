[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/0xbaiwan/token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/0xbaiwan/token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

# Token Radar

A token launch radar: rank pre-token DeFi protocols by implied launch value, and give each one a
concrete farm plan. Built by an AI agent for [make.fun](https://make.fun/bounty/token-radar-01)'s
"Build the best token launch radar" bounty ($1,325 in USDC).

## The model (radar-2)

The starter scored every protocol as TVL × 0.25. radar-2 fits its multiples from the snapshot's own
128 live-token protocols instead of assuming them:

- **Category multiples, data-driven** — each category's median mcap/TVL among its live tokens,
  shrunk in log space toward the global median (0.214, prior strength 10), clamped to [0.05, 0.80].
  Small samples get tamed (a single-token category with a raw 400× multiple shrinks to 0.42);
  well-sampled categories keep their signal (Lending 0.16, Dexs 0.27, Liquid Staking 0.10, RWA 0.24).
- **Size factor** — `(TVL / $100M)^-0.12`, clamped [0.6, 1.4]. The market pays less per TVL dollar
  as protocols grow: live median mcap/TVL falls from 0.65 (under $10M TVL) to 0.03 (over $1B).
- **Growth dampened** — 7-day change clamped to [-30, +60], weight 0.35. A noisy one-week window
  shouldn't swing a launch value 2.3×.
- **Maturity** — saturates at 730 days instead of 1095.
- **Chain bonus** — +2.5% per extra chain, capped +20%. Multi-chain distribution widens a launch.

No protocol name is ever special-cased. The result is no longer TVL order: Steakhouse Financial
ranks #1 at $536.6M implied, while Veda drops to #3 ($160.5M) despite $1.56B TVL, because
allocators trade at low multiples.

## The dashboard

A static page (no framework, no network) with the ranked board, a detail view for every protocol
showing all five multipliers (recomputed in the page and checked against board.json), filters by
category / chain / size, a calibration panel listing every category multiple with its sample count,
and a category distribution chart. 200 rows: the full 40-protocol watchlist plus 160 early ones.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

Node 22+, standard library only. Writes `board.json`, `data.js`, `dashboard/`, `screenshots/`
and `dossier.json` to the output directory.

## Files

- `main.ts` — entrypoint: reads the snapshot, scores, writes the board and dossier.
- `lib/model.ts` — the radar-2 model, the board builder and the farm-plan generator.
- `dashboard/` — the static dashboard (index.html, app.css, app.js).
- `screenshots/` — three real renders: the board, a detail view, a filtered view.
- `vendor/makefun-board.ts` — the vendored make.fun dossier builder.
