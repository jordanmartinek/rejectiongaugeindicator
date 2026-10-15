# Rejection Probability Gauge

A TradingView **Pine Script v5** indicator for **intraday scalping** that estimates
how likely price is to **reject off a key level** versus **push through it**, by
reading the character of the candles approaching the level.

> ⚠️ This is a **heuristic confidence score**, not a statistically backtested
> probability. Treat the 0–100 reading as a tunable gauge of pressure, and
> validate/adjust the weights against real charts.

## The idea

The gauge reads the "fuel" behind a move into a level:

- **Strong solid bodies** pushing into a level → conviction → *more likely to break through*.
- **Lower-wick dominance** on the approach ("buoy" — buyers defending every dip)
  → support building underneath → *more likely to break through*.
- **Upper-wick dominance / stalling bodies** → exhaustion → *more likely to reject*.
- **Level-touch candle that wicks through but closes back into range** → *rejection*.

So the key refinement over "lots of wicks = X" is **which side the wicks are on**,
measured relative to the direction price is approaching the level.

## How the score works

The last *N* approach candles (default 5) are scored on four factors, each
normalized to `0..1` where **1 = more likely to REJECT**:

| Factor | Signal |
|---|---|
| **Body / range ratio** | Big bodies into the level = conviction = *lower* reject score |
| **Wick asymmetry** | Exhaustion-side wicks (upper at resistance / lower at support) raise the score |
| **Direction consistency** | Candles moving *with* the push = conviction = *lower* reject score |
| **Close location at level** | Touch candle closing back in range raises the score |

The factors are combined using **editable weights** (auto-normalized) into a
**0–100 rejection likelihood**:

- **≥ High threshold (default 70):** REJECT likely (red)
- **≤ Low threshold (default 30):** BREAK likely (green)
- **In between:** Neutral (orange)

## Levels

- **Manual:** enter a price in the inputs.
- **Auto:** nearest confirmed swing-high (resistance) / swing-low (support) pivot.
- The **active level** is whichever valid level price is currently closest to.

### Engagement latch (stays alive through the resolution)

The gauge uses an **engagement latch** so it doesn't blink to N/A the moment a
candle pushes past the level — essential on 5m and higher timeframes:

1. **Touch is range-based.** A level engages when the candle's **range** (wick
   included) reaches within `Engage proximity × ATR(14)` — not just the close.
2. **Hold period.** Once engaged, the gauge **latches** onto that level and keeps
   scoring it for at least `Hold bars after engage` bars, even if price trades
   beyond the level, so you can read the full rejection/break-and-retest.
3. **Release.** The level is released only after the hold period has elapsed
   **and** price has moved beyond `Release distance × ATR` from it (or a new,
   nearer level supersedes it).

Set `Hold bars after engage = 0` to restore the old behavior (drop immediately
when price closes outside the proximity band).

## Direction modes

- **Resistance (from below)** — longs pushing up into a level.
- **Support (from above)** — shorts pushing down into a level.
- **Both** — auto-selects the nearer level.

## Outputs

- **Gauge meter** (on-chart table): score, verdict, direction, level, in-range status.
- **Candle coloring** of the approach candles by score (red/orange/green).
- **Active level line.**
- **Alerts:** `alertcondition`s + runtime `alert()` calls when the score crosses
  the high (rejection) or low (breakthrough) thresholds.

All visuals are toggleable and all thresholds/weights are editable in the inputs.

## Install

1. Open TradingView → **Pine Editor**.
2. Paste the contents of [`RejectionGauge.pine`](./RejectionGauge.pine).
3. **Add to chart**, then open settings to set your level(s), direction, window,
   weights, and alert thresholds.

## Tuning tips (scalping)

- Start with the **5-candle window** on 1–5m charts; shorten to 3 for very fast tape.
- If you get too many "reject" flags on strong trends, **raise the body/direction
  weights** (conviction should suppress false rejects).
- Use `proximity` to control how early the gauge wakes up near a level.
