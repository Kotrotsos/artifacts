# XREV Model Card

**Version:** 1.0
**Task:** Binary classification, will BTC close higher at the end of a 5-minute window
**Output:** P(up) in (0, 1), plus a bet / abstain decision
**Model family:** Gradient-boosted decision trees (HistGradientBoosting), shallow and regularized
**Status:** Research / paper trading. Not financial advice.

## Intended use and scope

Intended as a research signal for Polymarket "Bitcoin Up or Down" 5-minute markets and for studying short-horizon crypto microstructure. Decisions are selective: the model abstains far more often than it bets.

Out of scope: longer horizons (the daily model is a separate system), other assets, and any use as financial advice.

## Training data

- Source: Binance BTCUSDT 5-minute candles
- Span: 1 year, about 105,000 bars
- Label: 1 if close > open of the window, else 0
- Split: expanding walk-forward, weekly retrain, final 20% held out untouched
- Leakage control: every feature shifted to use only bars before the decision instant

## Inputs (29 causal features, 5 families)

1. Mean reversion: deviation from 1-hour and 3-hour moving averages
2. Micro-range position: where price sits inside the recent high-to-low band (novel, worked)
3. Order-flow persistence: rolling taker buy/sell imbalance (novel, worked)
4. Overreaction and volatility regime: last move z-scored by recent volatility
5. Round-number magnetism: distance to nearest 500 and 1000 dollar levels (flopped)
   plus time-of-day and day-of-week encodings

## How it decides

At the open of a window it knows only past closed bars and the current price. It predicts P(up), then checks the live Polymarket ask price and computes expected value at the real fill. It bets only when EV exceeds +0.01, otherwise it stands aside. This selective gate is what turns a thin directional edge into positive expectation.

## Evaluation results

| Metric | Value |
|---|---|
| Full-year out-of-sample accuracy | 51.57% (n = 74,944) |
| Holdout accuracy | 51.69%, z = 4.91 vs 50% |
| Gated (conviction) accuracy | 52.6% to 53.4% |
| Temporal stability | positive every month, flat trend (-0.02 pts/month) |
| Volatility regimes | 51.4% to 51.9% across calm/low/elevated/high |
| Null test (shuffled labels) | 49.5%, confirms no leakage |
| Break-even market efficiency | 58% at a 2-cent spread |

## Top features (permutation importance, holdout)

1. sma_dev_12 (mean reversion vs 1-hour average) +0.00465
2. range_pos_36 (micro-range position, novel) +0.00373
3. taker_persist (order-flow persistence, novel) +0.00315
4. ret_2, vol_ratio, ret_6 (momentum / regime)
5. rn_abs_500 (round-number, flopped) +0.00046

## Limitations and caveats

- The edge is thin, about 1.6 points over a coin toss. A 5-minute move is mostly random.
- Fills are unproven. The backtest assumes entry near 0.50. If the market already prices the same momentum, the edge shrinks. The live panel measures this for real.
- One idea failed: round-number magnetism added almost nothing.
- Decay risk: a small microstructure edge can be arbitraged away as the market matures.

## Bottom line

XREV is a small, real, leak-free directional edge made tradeable by a strict expected-value gate. The open question is the fill, not the prediction.
