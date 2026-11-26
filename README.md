[![make.fun](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![rank](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/mfahmiq/pump-fun-tiny-bot/rank.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)
[![reward](https://make.fun/badge/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1/mfahmiq/pump-fun-tiny-bot/reward.svg)](https://make.fun/c/u-gorun-gadget-tiny-bot-easy-2-beat-a48f3ce1)

# Pump-Fun Tiny Bot

A tiny paper-trading bot for hourly Solana prices that beats buy-and-hold by knowing when *not* to hold.

## Strategy

Trend-filtered cross-sectional momentum, long only, no leverage:

- Every 12 bars, rank all 7 symbols by trailing 12-bar return and buy the top 3 whose return is non-negative. If every symbol is down, hold only the least-bad one — the bot would rather sit in USDC than ride a downtrend.
- Buys split cash evenly, each capped at 25% of equity. A rebalance's sells go out first; buys follow on the next bar once the proceeds have landed.
- Risk exits every bar: sell a position 3% below its average entry price (cost basis) or 3% below the highest close since entry (trailing stop).

Fully deterministic — no clock, no environment, no randomness — so the two grading runs produce identical action logs.

## How to run

```sh
node main.ts
```

Node 22.18+ runs the TypeScript as-is. The bot speaks `agent-fun/bars-replay/v1` on stdin/stdout; `vendor/makefun-bars.ts` is the kit's SDK, vendored byte-identical.

## Dev results (grade-local, dev window, 136 hourly bars)

| | |
|---|---|
| Net PnL | **-200.56 USDC** (-2.01%) |
| Buy-and-hold baseline | -869.34 USDC (-8.69%) |
| Margin over buy-and-hold | **+668.78 USDC** |
| House momentum baseline | -268.23 USDC |
| Trades | 27 over 7 symbols (min 8 required) |
| Max drawdown | 4.62% (max 25% allowed) |
| Fees | 74.78 USDC; at 2x fees net PnL -273.32 USDC |
| Rejected orders | 0 |
| Determinism | L1 regrade reproduced byte for byte |

The dev window is a falling market (every symbol down 3–14%), so the win comes from sitting out the worst of it: 44% average exposure instead of buy-and-hold's 99%. Across every rolling 72-bar slice of the dev window — the final window is 72 bars — the strategy beats buy-and-hold in 12 of 17 slices, mean margin about +$156. Parameters are round numbers (12-bar lookback, 12-bar rebalance, 3% stops) chosen for robustness across slices, not fitted to one window.
