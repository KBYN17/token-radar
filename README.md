[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/KBYN17/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/KBYN17/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

# Token Radar

A token launch radar for make.fun's **token-radar-01** bounty: a dashboard
that finds DeFi protocols whose token does not exist yet, ranks them by an
implied value, and shows a reader how to farm each one.

## What it does

`main.ts` reads a frozen DefiLlama snapshot (`snapshot.json`), scores every
pre-token protocol with the model in `lib/model.ts`, and writes:

- `board.json` — the ranked board (178 rows: the 40-protocol watchlist plus
  138 pre-token protocols below the $50M floor), each row with an implied
  value in dollars and a 6-step farm plan.
- `data.js` + `dashboard/` — a static page (no network, no framework) with
  a sortable, filterable board, a detail view per protocol, and a separate
  Live-token view for protocols whose token already trades.
- `dossier.json` — built with the vendored `vendor/makefun-board.ts`.

## The model (radar-2)

Implied value is computed from the snapshot's own figures only:

```
implied = round( tvl_usd x peer_multiple
               x (1 + clamp(change_7d, -50, 100) / 100 x 0.5)
               x (0.7 + min(age_days, 1095) / 1095 x 0.3)
               x (1 + 0.05 x min(chains - 1, 4)) )
```

`peer_multiple` is the market's own pricing: the median market-cap-to-TVL
ratio of the live-token protocols in the same category (at least 3 peers),
else the global median across all live tokens, clamped to 0.05x–2.5x. No
protocol name is special-cased; the dashboard recomputes every row from the
same figures and shows the peer table the multiples came from.

A protocol is rankable when it records no token (no market cap, no CoinGecko
id, no symbol) and is not a wrapped, staked or bridged product of a token
that already trades. The board never predicts whether or when a token is
issued; implied value is the entry's own claim about what a token would be
worth if the protocol's record stayed as it is.

## Run it locally

Node 22 or later, standard library only, no network at run time:

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

Then serve `out/` statically (for example `python3 -m http.server --directory out`)
and open `out/dashboard/index.html`. The page reads `out/data.js`; it makes
no network request.

## Layout

```
main.ts                 entrypoint: snapshot in, /output out
lib/model.ts            the radar-2 model, farm plans, snapshot parsing
vendor/makefun-board.ts make.fun's dossier library (verbatim copy from the kit)
dashboard/              index.html, app.css, app.js (static, offline)
screenshots/            1-board.png, 2-detail.png, 3-filtered.png
```

## Screenshots

The three screenshots are taken from the dashboard itself at 1440x900:
the board, a protocol's detail view, and a filtered view.
