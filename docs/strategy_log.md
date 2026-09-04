# Strategy Log — what we tried and why it died

A running graveyard of the strategies explored in [`research.ipynb`](../research.ipynb), the test that
settled each one, and the lesson we carried forward. "Dead" = not significantly profitable **after
realistic costs / out-of-sample scrutiny**. The most promising lead — long-only momentum — is
**parked** (unproven out-of-sample), not dead. We've stepped back to pure data analysis for now.

## Summary table

| # | Strategy | Core test | Verdict | Killing result |
|---|----------|-----------|---------|----------------|
| 1 | Hourly lag-1 reversal, all 60 | corr(alpha, illiquidity) | **dead** | alpha ∝ bid-ask bounce, not real |
| 2 | BTC/ETH factor model (leader on followers) | residual stationarity | **dead** | causality backwards; residual not tradeable |
| 3 | Directional lag-1 reversal (daily) | Welch t on conditional means | **dead** | few coins significant; edge in illiquid names |
| 4 | Cross-sectional reversal (rank-demean, bal-32 / all-60) | net-of-cost Sharpe | **dead as taker** | 20 bps market cost erases it; only maker survives |
| 5 | Ranked-threshold reversal (zero the middle) | net Sharpe vs #4 | **dead** | no material lift; still cost-bound |
| 6 | Market-residualized reversal (rolling-beta) | IC vs raw reversal | **dead** | IC lift marginal; still liquidity-driven |
| 7 | Corwin-Schultz / Abdi-Ranaldo spread estimators | corr with real quotes | **dead** | corr ≈ −0.19 — measures volatility, not spread |
| 8 | Intraday hour→hour predictability (Wen) | t-stat grid, cross-coin consensus | **dead** | ≈ false-positive rate; no consistent edge |
| 9 | Intraday seasonality GAM (returns/vol/volume) | signed-return CI vs 0 | **descriptive only** | majors hug 0; \|ret\| humps = volatility, not drift |
| 10 | Liquid-coin momentum, two-sided sign | pooled Sharpe | **weak** | 0.18–0.24 pooled; dragged by short leg + LTC |
| 10a | — short leg (short the losers) | Sharpe over n×h grid | **dead** | negative almost everywhere; crypto drift bounces losers |
| 10b | — long leg (long the winners) | vs buy-hold + alpha regression | **parked** | +0.82 SR, α t≈2.6, but crash-avoidance & in-sample; OOS test not run |

---

## The reversal track (strategies 1–6)

The recurring theme: **crypto reversal alpha lives in illiquid coins, and that alpha is mostly the
bid-ask bounce you have to pay to harvest it.**

### 1. Hourly lag-1 reversal, all 60 coins — *bid-ask bounce*
- **Idea:** yesterday's/last-hour's move reverses; go against it.
- **Test:** cross-sectional correlation of per-coin reversal strength with liquidity.
  Found **corr(reversal, log volume) ≈ −0.67** — the reversal is strongest exactly where coins are
  least liquid.
- **Why dead:** Roll (1984) — a bouncing trade between bid and ask *manufactures* negative
  autocorrelation equal to the spread. The "profit" is the spread, and you cross that same spread to
  trade it. It nets to ~zero (worse after fees). This is a measurement artifact, not an edge.
- **Lesson:** any reversal signal must be judged **net of the real spread**, and daily bars are far
  less bounce-contaminated than hourly.

### 2. BTC/ETH factor model — *wrong causal direction*
- **Idea:** regress a coin's return on 20 correlated coins (backwards elimination), trade the residual
  if it's stationary / mean-reverting.
- **Test:** ran it with **BTC/ETH as the regressed (dependent) variable**; checked residual stationarity
  and mean-reversion.
- **Why dead:** we regressed the **liquid leader on illiquid followers** — backwards. BTC/ETH lead; the
  followers lag. The residual wasn't a tradeable spread. Correct framing (follower on leader) is the
  liquid-beta-basket idea, parked for later.
- **Lesson:** in a lead-lag system, regress the **follower on the leader**, not the reverse.

### 3. Directional lag-1 reversal, daily conditional means — *thin significance*
- **Idea (HW8 style):** group each coin's daily return by the **sign of yesterday's** return; if
  mean-after-down > mean-after-up, the coin reverses.
- **Test:** Welch t-test on the two conditional means, all 60 coins.
- **Why dead:** only a handful were significant, and the reversal spread again concentrated in the
  illiquid names (same bounce story as #1, just cleaner on daily bars).

### 4. Cross-sectional reversal portfolio (rank → demean → normalize) — *dies as a taker*
- **Idea:** dollar-neutral book — long yesterday's relative losers, short its winners; gross 1.
  Ran on the 32 "balanced" coins and on all 60.
- **Test:** gross vs net Sharpe under 7 bps (limit) and 20 bps (market all-in), train/test split.
- **Why dead:** turnover is high, so **20 bps market cost erases the gross Sharpe**. It only survives as a
  **maker** (0% fee, *earning* the spread) — but that's a market-making strategy, not the reversal
  signal itself. As a taker it's dead.
- **Lesson:** high-turnover reversal is a liquidity-provision trade in disguise; the P&L is the spread
  you provide, not a forecast.

### 5. Ranked-threshold reversal (percentile rank, zero the middle) — *no lift*
- **Idea:** trade only the extremes (top/bottom percentiles), flat in the middle, to cut turnover/cost.
- **Test:** net Sharpe vs the plain rank book.
- **Why dead:** no material improvement; still bounded by the same cost/bounce problem.

### 6. Market-residualized reversal (rolling-beta) — *marginal*
- **Idea:** rank residuals `ε = r − β·mkt` (rolling beta) instead of raw returns — bet on
  idiosyncratic reversion, not market noise (Kakushadze-style).
- **Test:** IC (corr of signal with next-day return) vs the raw-return version, on the all-60 breadth.
- **Why dead:** IC improved only marginally and the book was still liquidity-driven.

---

## Measurement dead-ends (strategy 7)

### 7. OHLC spread estimators (Corwin-Schultz, Abdi-Ranaldo) — *measured the wrong thing*
- **Idea:** with no historical order book, estimate each coin's spread from OHLC to decompose reversal
  into "real" vs "bounce".
- **Test:** correlate the estimates against **real** best bid/ask pulled from `get_orderbook_tickers()`.
- **Why dead:** **corr ≈ −0.19** — the estimators track the high-low *range* (volatility), not the
  spread. They **overstate** liquid coins (big range from volatility) and **understate** dead coins
  (range collapses because nothing trades). BTC real ≈ 1–2 bps; AR estimated ≈ 18 bps.
- **Lesson:** use real quotes. Built [`collect_spreads.py`](../collect_spreads.py) to accumulate live
  top-of-book snapshots. Reality: only **8–9 coins < 10 bps** (tradeable); **~39 coins > 50 bps**
  (untradeable). This reshaped the whole project toward a small liquid universe.

---

## Intraday structure (strategies 8–9)

### 8. Intraday hour→hour predictability (Wen et al.) — *noise*
- **Idea:** does an earlier hour's return predict a later hour's return (same UTC day)? Momentum or
  reversal by hour-pair.
- **Test:** t-stat grid over all (predictor hour × target hour) pairs for BTC/ETH/XRP/SOL/LTC; only
  trust cells consistent across coins (276 pairs/coin ⇒ ~14 false positives each at 5%).
- **Why dead:** significant cells came in at roughly the false-positive rate with no cross-coin
  consensus. No standalone intraday timing edge.

### 9. Intraday seasonality GAM (Härdle "Rise of the Machines") — *descriptive, not a signal*
- **Idea:** GAM smooths of activity over hour-of-day × weekday — the paper's "proof of human."
- **What it shows:** volume and volatility surfaces have clear working-hour humps (real, but that's
  *when* people trade, i.e. volatility — not *which way* price goes). The absolute-return curves are
  volatility by construction (can't be negative).
- **Test:** re-fit the GAM on **signed** returns with CI bands. The majors' curves **hug zero** (±2–3
  bps, confidence bands straddling 0); only 2–4 of 48 hours excl. zero — noise. Only XRP swings wide,
  and it's the noisiest coin.
- **Why not a strategy:** no reliable directional intraday drift to trade. Kept as EDA context.

---

## Momentum track (strategy 10) — parked

**Status (2026-09-04): parked.** The long leg is a genuine lead (see below) but the deciding
out-of-sample test was never run, and the edge is largely crash-avoidance rather than a persistent
forecast. Set aside to return to pure data analysis; the momentum cells were removed from
[`research.ipynb`](../research.ipynb) (recoverable from git history). Revisit by carrying the long book
(n=15, h=1) onto the held-out 20% before trusting it.

### 10. Liquid-coin momentum (vol-normalized, drift-adjusted z-score)
Signal: `√n · (r̄_n − r̄_365) / σ_365`, lagged, on the 9 liquid coins. Held h days via overlapping
tranches; swept lookback n × hold h.

- **Two-sided sign book: weak.** Pooled equal-weight Sharpe **0.18–0.24** — dragged down by (a) LTC,
  which reverses rather than trends, and (b) the short leg.
- **Short leg (10a): dead.** Shorting the bearish "losers" is **negative across almost the whole
  (n × h) grid** (best only ~0.25). Crypto's positive drift means losers bounce — you short into an
  upward tide.
- **Long leg (10b): alive but unproven.** Longer lookback (n ≈ 10–30), fast hold (h ≈ 1–3); peak
  n=15/h=1 **Sharpe 0.82**. Checked against the obvious null — *is it just beta?*
  - Buy-and-hold on the same coins has a **negative** Sharpe (−0.18) over 2022–2026, so the edge is
    **not** market exposure.
  - Correlation with buy-hold is only **~0.62** (not a repackaged long).
  - Regressing the long book on buy-hold leaves **α ≈ +0.30/yr, t ≈ 2.6** — timing adds value, at
    **~39 %** average exposure (flat when bearish).
- **The honest caveat:** most of the edge is **crash avoidance** — momentum sat out the 2022 bear.
  Buy-hold's negative Sharpe is very start-date-dependent (begin in 2023 and buy-hold wins). All
  numbers are **in-sample / gross**.
- **Next test that decides it:** carry the long book (n=15, h=1) onto the **held-out 20%** and confirm
  the alpha survives out-of-sample. Until then it's a promising lead, not a proven strategy.

---

## Cross-cutting lessons

1. **Net of the real spread, or it isn't real.** Most of our early reversal "alpha" was bid-ask bounce.
2. **Illiquidity is where crypto reversal hides — and why you can't harvest it.** The strongest signals
   sat in coins with 50–1000+ bps spreads.
3. **Estimate nothing you can measure.** OHLC spread proxies were −0.19 correlated with truth; real
   quotes reshaped the universe.
4. **Separate long from short.** They are not symmetric in a positively-drifting, hard-to-borrow market.
5. **Beat the right benchmark.** A long-biased crypto strategy must be judged against buy-and-hold and
   have alpha after regressing it out — not just a positive Sharpe.
