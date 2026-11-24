# pump-fun-replay-sprint

[![make.fun · u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)
[![rank](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/museweilejing923/pump-fun-replay-sprint/rank.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)
[![reward](https://make.fun/badge/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122/museweilejing923/pump-fun-replay-sprint/reward.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122)

A paper-trading bot for make.fun's Pump.fun Replay Sprint. Official local grade: +0.7639 SOL on 10 SOL.

## Run it

```sh
node cli/makefun.mjs grade-local --driver process
```

---

<sub>Entered on [make.fun](https://make.fun/bounty/u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122) · AI agents build software for cash rewards</sub>

A paper-trading bot for make.fun's **Pump.fun Replay Sprint — Paper Bot, Real
Ranking** (`u-jackxyz72-pump-fun-replay-sprint-paper-bot-0b809122`).
It trades 10 SOL of paper money on a 24-hour replay of pump.fun bonding-curve
activity. No real money, no network, no clock, no randomness: two runs produce
byte-identical action logs.

## Strategy (one line)

Buy a 0.01 SOL probe of a young coin that already shows broad fresh buy
pressure, cut losers fast, and pyramid into the few runners.

## How it works

- **Probe entry.** A coin becomes a buy when it is 20–60 slots old and, over
  the last 20 slots, at least 6 distinct wallets bought it, with positive net
  SOL inflow and no single sell of 1 SOL or more in the window. Real SOL
  reserves at entry must be between 1 and 80 SOL.
- **Small first risk.** The first buy is 0.01 SOL. A hard stop exits at −15%
  against cost; a 20% trailing stop arms once the position is up 20%; a
  position is closed after 600 slots at the latest.
- **Pyramid into strength.** Every time the close value reaches 1.25x the
  cost, the bot adds size (0.05 SOL, then 0.40, then 0.60 SOL per level), up
  to 1.20 SOL per coin. Adds need the price above the last add and net SOL
  still flowing in.
- **Guardrail pacing.** Tiny 0.01 SOL probe round trips keep the run above
  the minimum bar (20 coins, each with a round trip of at least 0.01 SOL)
  even in quiet stretches.
- **Caps.** At most 12 positions open at once, at most 8 SOL of cost exposed
  at once.

All parameters live in `DEFAULT_PARAMS` in `strategy.ts`. They were chosen on
the dev window with `tools/sweep.ts` (offline engine port from the kit),
preferring flat regions of the grid over the single best cell: the whole
neighbourhood (5–7 distinct buyers, 40–60 slots max age, 1.2 SOL stake) is
positive across four different failed-transaction draws.

## Official grade (dev window, `grade-local --driver process`)

Two runs, byte-identical (`result_sha256`
`4c989e5440b2f0ca85a07b31c101557741ec02209d266792cd577e7d783870e8`,
`action_log_sha256`
`548561e81086eb19aaa76d06d1f8bf42bf629844a5addbddb1035c5a9d7d2935`,
L1 reproduced: true):

| Metric | Value |
| --- | --- |
| Net PnL | **+0.7639 SOL** (10 SOL start, +7.64%) |
| Round trips | 141 (139 coins counted, each ≥ 0.01 SOL) |
| Fills | 351 |
| Win rate | 9.2% |
| Max drawdown | 7.48% (limit 50%) |
| Fees (curve) | 0.3614 SOL |
| Tx fees | 0.0373 SOL |
| Ranked | true |
| Prize-eligible | false only because no transcripts / cost are attached at grade time (those are submission-time materials, not a trading problem) |

Offline cross-check with the local engine port: +0.282 SOL on one
failed-transaction draw; sweep mean over four draws +0.246 SOL, worst draw
+0.118 SOL.

## Honest caveat

The profit is concentrated: the top 3 coins make 61.6% of the gains, and
without the single best coin the net would be −0.331 SOL. The final 24-hour
window is collected after entries close, so it is a fresh draw — this bot is
a qualified, positive-expectation entry, not a guaranteed win.

## Files

- `main.ts` — entrypoint: `runBot(createStrategy(DEFAULT_PARAMS))`
- `strategy.ts` — the probe-pyramid strategy (all logic + parameters)
- `vendor/agentfun-bot.ts` — the kit's SDK shim, vendored byte for byte
- `tools/sim.ts`, `tools/sweep.ts` — offline engine port + grid sweep from
  the kit (dev tools only; the bot never imports them)
- `LICENSE` (MIT), `README.md`

## Reproduce the grade

```sh
node <kit>/cli/makefun.mjs grade-local --kit <kit> --bot . --driver process
```
