# Strategy Log — what we tried and why it died

The prose index of every strategy we tested and rejected, the test that settled each one, and the
lesson we carried forward. "Dead" = not significantly profitable **after realistic costs /
out-of-sample scrutiny**. The most promising lead — long-only momentum — is **parked** (unproven
out-of-sample), not dead.

**Two-notebook convention:**
- [`research.ipynb`](../research.ipynb) — live EDA and current work only. Kept uncluttered.
- [`research_failed.ipynb`](../research_failed.ipynb) — the **executable graveyard**: runnable code and
  output for every rejected strategy below. New failures go here, not in the research notebook.

This file is the summary; the notebook is the evidence.

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
| 11 | Correlation / pairs mean-reversion (daily, BTC-residual) | λ gate vs **simulated null** | **dead** | rolling de-meaning manufactures the reversion; a random walk scores t≈−4.2 |
| 11a | — LTC/BCH specifically | null test + cost + lag decay | **lead, unproven** | passes null, ~10 bps cost, SR 1.23 — but n≈24 trades, SE≈0.7, selected from many |
| 12 | Hourly pair reversion | lag-decay test + cost/illiquidity corr | **dead** | bid-ask bounce: 1-hour execution delay erases 77–114% of gross SR |
| **13** | **Cross-sectional momentum** (rank trailing return → demean → normalize), 9 liquid coins | net-of-cost Sharpe, robustness slices, walk-forward | **LIVE — the project's result** | gross 0.90 → **net 0.46** at 20 bps; costs moved h from 2 to 5 |
| 13a | — volume as a *sizing* tilt (size down on relative volume, λ<0) | **matched-turnover** frontier + λ inside the walk-forward | **dead, twice** | +20% gross was bought with +45% turnover; chosen 8/8 blocks OOS and costs 0.10 Sharpe |
| 13b | — re-selecting (n, h) quarterly | walk-forward under 4 selection rules | **freeze instead** | rules span −0.07 to +0.55; n=15 is rank 1–2 of 45 every block but no argmax finds it |

> **Momentum figures caveat:** strategy 10's cells were never committed and were later rebuilt from
> spec in `research_failed.ipynb`. The rebuild reproduces the key results closely (buy-hold −0.18,
> corr 0.63, α t≈2.3) but individual Sharpes differ slightly from the numbers in this table. Trust
> the notebook.

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

---

## 11–12. Correlation / pairs, and why hourly made it worse

Full code and output: [`research_failed.ipynb`](../research_failed.ipynb), sections 5–6.

### 11. Pairs mean-reversion on BTC-neutral residuals — *the reversion was an artifact*
- **Idea:** raw correlations mostly measure shared BTC beta, so residualize each coin on BTC (own β),
  then trade pairs whose *residuals* still co-move. Spread trends, so de-mean on a trailing window and
  trade the z-score — which only needs reversion to the *recent* mean.
- **The trap:** subtracting a trailing moving average **manufactures** mean reversion. Any random walk
  minus its own rolling mean looks stationary.
- **Test:** score the λ gate against a **simulated null** (matched random walks + shuffled increments)
  instead of the Dickey-Fuller −2.86. A pure random walk scores **median t = −4.15**, so −2.86 flags
  everything. Against the proper null only **5 of 10** pairs pass — and the appealing ones
  (ZEC/ZEN privacy coins, DOGE/SHIB memes) **fail**. Co-movement without reversion is untradeable.
- **The screen does work:** mean net Sharpe **+0.48 for pairs that pass** vs **−0.25 for those that
  fail**. Most survivors are then killed by 55–90 bps round-trip costs.
- **Lesson:** simulate the null for *any* transformed series. Ranking, normalizing and rolling
  de-meaning all create structure; the textbook critical value is usually the wrong one.

### 12. The same idea at hourly frequency — *bid-ask bounce, again*
- **Idea:** daily holding is too slow; rerun on hourly bars for more observations and faster turnover.
- **Immediate problem:** at hourly, **30 of 60 coins** have no trade in over half of all hours
  (staleness correlates **0.80** with the real spread). The universe collapses to 18 coins / 153 pairs.
- **The tell:** **143 of 153 pairs passed** the null — 19× the chance rate. When 96% of candidates pass,
  the method is measuring itself.
- **Four confirmations it was bounce:** (1) all 153 pairs have negative lag-1 autocorrelation;
  (2) reversion strength rises with pair cost (**+0.51**) as does bounce size (**+0.45**); (3) the
  "pairs" weren't pairs — single coins alone scored the same as any pair containing them; (4) delaying
  execution by **one hour erases 77–114%** of the gross Sharpe.
- **Method lesson:** the shuffled-increment null **cannot** catch this, because shuffling destroys the
  autocorrelation that *is* the artifact. Use the **lag-decay test** for microstructure. A real edge
  degrades gracefully as you delay execution; an artifact falls off a cliff in one bar.
- **Does not contaminate the daily work:** daily residual lag-1 autocorrelations are ≈ −0.03 vs ≈ −0.20
  hourly, because bounce is a fixed size while daily vol is ~5× hourly vol.

---

## 13. Cross-sectional momentum — the one that lived

**The strategy.** Rank the nine liquid coins (real spread < 10 bps) by trailing 15-day return, demean the
ranks so the book is dollar-neutral, normalize to gross 1, hold 5 days via overlapping tranches. Long the
relative winners, short the relative losers. No volume layer, no adaptive parameters.

| | TRAIN, 2022-07 → 2025-07 |
|---|---|
| gross Sharpe | 0.85 |
| **net Sharpe @ 20 bps** | **0.46** |
| turnover | 0.151 / day |
| break-even cost | **43 bps** |
| annual return (net) | 8.8% |
| annual vol | 19.2% |
| max drawdown | −23.1% |
| alpha vs BTC (net) | 9.5%/yr, **t = 1.03** |
| beta / corr vs BTC | −0.015 / −0.031 |

The alpha is positive and **not statistically significant** net of costs. That is the honest headline;
the gross 0.90 is not.

### 13.1 Costs selected the holding period

Turnover falls roughly like 1/h while gross Sharpe is nearly flat across h = 2…10, and that asymmetry is
the whole argument. Choosing h=2 over h=5 buys +2.0%/yr of gross and pays +5.1%/yr of cost — a net loss
of 3.1%/yr. h=1 is the cautionary case: **gross 0.80, net −0.10**, an edge that is entirely a rebate to
the exchange.

Only *h* was chosen with costs in view; *n* stayed at the value picked on gross. The net argmax over all
45 grid cells is n=90, h=10 — deliberately **not taken**, because a turnover penalty is monotone, so the
net optimum always drifts toward longer lookbacks and longer holds. That corner is partly mechanical, not
evidence of a better signal.

### 13.2 The volume tilt died on the *right* test

Sizing down on relative volume (`conviction × exp(λ·volZ)`, λ < 0, through leg-normalization) raised gross
Sharpe from ~0.83 to ~1.00 at h=5. It also raised turnover from 0.151 to 0.219/day.

The naive net comparison says it loses; that comparison is unfair, because it pits books at different
turnover. The correct test asks whether the tilt beats the baseline **at matched turnover** — and the
baseline can buy that same turnover for free, just by shortening h.

It cannot. At the pre-registered λ = −1.0, h = 5: tilt nets +0.236, baseline at the same turnover nets
+0.309, edge **−0.073**, against a random-control band of −0.117 ± 0.171 → **+0.26 control-σ** where the
rule required 2σ. The best matched-turnover edge anywhere on the 18-config grid is +0.041, inside the
control band.

> **Method lesson — the control must be matched on what the intervention *spends*.** An earlier version of
> this test showed the tilt beating a random control by ~2σ, and that result was not fraudulent; it was
> matched on **concentration**. But concentration is not what the tilt costs. Turnover is. Hold the right
> quantity fixed and the edge disappears. Choosing the control variable *is* choosing the hypothesis.

**The full (λ × h) surface says the same thing four ways.** Swept over λ ∈ [−3, +2] and h ∈ [1, 10]:

- **gross Sharpe** is smooth and monotone toward λ<0 and large h, peaking at 0.94 (λ=−1.5, h=10) — which
  is exactly what a real effect looks like, and why this was persuasive;
- **turnover is symmetric in |λ|** — at h=1, λ=−1.0 gives 0.809 and λ=+1.0 gives 0.778. Churn tracks the
  *size* of the tilt, not its direction, because it comes from the z-score **moving**;
- **net Sharpe** is (gross − turnover cost), and the gradient cancels almost exactly: best tilt 0.49
  versus 0.44 for no tilt at all;
- **the matched-turnover edge** has no structure left, and its maximum sits at **λ = +1.5** — the
  *opposite* sign to the hypothesis. An effect that peaks at the wrong sign is not an effect.

**Out-of-sample confirmation.** Put λ into the walk-forward selection set and let the procedure choose it
quarterly on prior data only:

| | OOS net SR | max DD |
|---|---|---|
| λ forced to 0 | **+0.641** | −30.6% |
| λ free to be chosen | **+0.538** | −32.4% |

The free rule picks a tilt in **8 of 8 blocks** — it always looks best on the data available at the time —
and allowing it costs **0.10 Sharpe out of sample**. In-sample attractive, out-of-sample costly: the
signature of fitting noise, not of a weak-but-real effect. Two independent tests now agree, and the
turnover mechanism explains both.

### 13.3 Robustness: the two arbitrary choices are not load-bearing

Same frozen spec, three slices, nothing re-optimized:

| slice | net SR | turnover | beta |
|---|---|---|---|
| headline — spread < 10 bps, from 2022-07 | 0.460 | 0.151 | −0.015 |
| **2022-H1 put back in** (from 2022-01) | 0.402 | 0.148 | −0.029 |
| **point-in-time top-9 by trailing $ volume** | 0.474 | 0.155 | +0.005 |

- **2022-H1.** The exclusion was justified by BTC's crash — a bad reason for a book that hedges BTC
  direction out. Putting it back costs 0.06 Sharpe and **does not change the parameter selection**
  (gross argmax is n=15, h=2 either way). The cut is not load-bearing, which is the useful outcome.
- **Universe look-ahead.** The nine coins were chosen on spreads measured in **2026**. A genuinely
  point-in-time universe — top 9 by trailing 90-day median dollar volume, re-formed daily, overlapping
  the spread set only 6.88 of 9 on average and containing APE/SHIB/VET in 2022 and LINK/XLM by 2025 —
  scores **0.474**, slightly *better*. The result does not depend on the look-ahead.
- **Survivorship remains, and is not fixable here.** All 60 coins have a first-valid date in 2022, so the
  panel is the *current* Binance.US listing set: anything delisted 2022–2026 is absent. The bias favours
  momentum. It is weaker for a dollar-neutral book that ranks *within* survivors than for a long-only
  one, but it is not zero. Stated, not solved.

### 13.4 Walk-forward: the selection *rule* is the variable

Expanding window, quarterly refits, 18-month minimum train, truncated at 2025-07-01. Eight refits,
732 OOS days.

> **Correction.** The first version of this section reported a single number — 0.55 — and concluded "the
> process transfers." That was one of several defensible selection rules, and it happened to be the best.
> A walk-forward does not test *a strategy*; it tests *a selection rule*, and the rule must be named
> before the number means anything.

| rule | OOS net SR | what it picks |
|---|---|---|
| **R1** n on gross argmax, then h on net — *the rule we actually used* | **−0.07** | n=3 in 5 of 8 blocks |
| **R2** net argmax over all 45 cells | **+0.55** | n=90 in 8 of 8 |
| **R3** net argmax on a 3×3-smoothed grid | **+0.20** | n=20 → n=90 |
| **R4** n by mean net across h, then h on net | **+0.25** | n=30 → n=90 |

**The spread across four reasonable rules is 0.62 Sharpe — larger than most effects in this notebook.**
The top of the grid is flat (top-5 cells within ~0.1 Sharpe), so argmax re-selection is mostly amplifying
noise.

**But n=15 itself is robust.** Rank of the best n=15 cell out of 45, per block: **1–2 on gross in every
single block**, 4–9 on net. The lookback is well-supported; what fails is any automatic rule that tries to
rediscover it:

- **R1 picks n=3** because gross ignores turnover, so the h=1 column — where trading is free — dominates
  the grid, and n=3/h=1 edges out n=15/h=1 by a hair. Then h is chosen on net and the damage is done.
- **R2 picks n=90**, which is *worse gross everywhere* (best 0.64 vs 0.90) but carries ~2.5× lower
  turnover, so a 20 bps penalty flips the ranking. Pure-net argmax is a turnover minimizer wearing a
  signal-selector costume.

**Conclusion: this is an argument for freezing parameters, not for re-selecting them.** Same destination
the mentor reached about adaptive layers, by a different route. The frozen n=15/h=5 line scores 0.63 over
the same span but is **not comparable** — its parameters were chosen on a window containing those 732 days.

### 13.5 Protocol correction

The 2025-07 → 2026-08 window is now called a **validation set**, not a sealed holdout. Earlier pairs and
hourly work ran on the full sample, and the previous 80/20 boundary sat at 2025-09-22, inside it. That
history cannot be un-run. Walk-forward is the stronger out-of-sample evidence; the single split is one
more cut, not the verdict. `test_only()` remains uncalled.

### 13.6 A convention mismatch worth knowing about

The volume cells (`actD1`, `actD3`, `actE1`) lag the signal one day more than the momentum and cost
sections — `.rolling(n).sum().shift(1)` plus the `w.shift(1)` in the P&L, versus a single lag in
`xs_pnl`. Harmless and conservative, but it makes their Sharpes ~0.02 lower and not directly comparable
(h=5 gross: 0.835 vs 0.855). The frontier test rebuilds both arms on the single-lag convention.

