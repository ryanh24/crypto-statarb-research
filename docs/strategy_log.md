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
| **13** | **Cross-sectional momentum** (rank trailing return → demean → normalize), 9 liquid coins | net-of-cost Sharpe, robustness slices, rolling walk-forward on point-in-time universes | **in-sample edge; OOS = one trade** | net 0.46 on TRAIN; 30-day real-time screen 0.34 OOS (SE 0.43), ≈0 without the ZEC run |
| 13a | — volume as a *sizing* tilt (size down on relative volume, λ<0) | four (λ×h) heatmaps; 139-window rolling walk-forward | **dead** | every tilt loses to λ=0 at h=5 on TRAIN; down-tilt worst over the walk-forward (0.13 vs 0.58) |
| 13b | — universe look-ahead after 2025-07 | ZEC liquidity history; 7/14/30/90-day screen sweep; post-jump window | **about half load-bearing** | ZEC became liquid early in its run; 30-day screen keeps +30% of the +82% burst |

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

### 13.2 The volume tilt: four heatmaps at n = 15

Hypothesis from Tests D/E: size **down** on relative volume, `conviction × exp(λ·volZ)` with λ < 0, through
`leg_norm` so the book stays dollar-neutral. Swept over λ ∈ [−3, +3] × h ∈ {1, 2, 3, 5, 7, 10} on TRAIN, net of
20 bps, lookback fixed at 15. λ = 0 is the conviction-sized book (net 0.44 at h=5), so every row shares one
construction.

| panel | reading |
|---|---|
| (a) gross Sharpe | smooth, monotone toward λ < 0 and long h; peaks at 0.94 (λ = −1.5, h = 10). This is why it was persuasive |
| (b) Sharpe lost to costs | rises with \|λ\| on **both** sides — turnover is symmetric in \|λ\| (h=1: 0.809 at λ=−1, 0.778 at λ=+1), because the churn comes from the z-score *moving* |
| (c) net Sharpe | (a) − (b); the gross gradient mostly cancels. Best tilt 0.49 (λ=−1, h=10) vs 0.44 no tilt at h=5 |
| (d) edge over no tilt, same h | at the frozen **h = 5 every tilt loses** (best −0.07); only a sliver at h = 10 is positive (+0.08) |

The walk-forward carries λ ∈ {−1, −0.5, 0, +0.5, +1}: beyond |λ| = 1 the cost panel keeps climbing. That range
is a judgment made on TRAIN data overlapping most walk-forward windows.

> The earlier matched-turnover frontier, random-control comparison and joint (n, h, λ) walk-forward were removed
> from the notebook in the cleanup; they remain in git history at `f513091`. Their method lesson stands — **a
> control must be matched on what the intervention spends** — and panel (b) now carries the mechanism.

### 13.3 Robustness: not load-bearing *through 2025-07*

Same frozen spec, three slices, nothing re-optimized, through 2025-07-01:

| slice | net SR | turnover | beta |
|---|---|---|---|
| headline — spread < 10 bps, from 2022-07 | 0.460 | 0.151 | −0.015 |
| **2022-H1 put back in** (from 2022-01) | 0.402 | 0.148 | −0.029 |
| **point-in-time top-9 by trailing $ volume** | 0.474 | 0.155 | +0.005 |

Differences of −0.06 and +0.01 against a ~0.48 standard error: neither choice moves the TRAIN result.
Survivorship remains, stated not solved — all 60 coins first list in 2022, so the panel is the current listing
set.

> **This conclusion does not extend past 2025-07.** See §13.4: after the split, the universe look-ahead
> becomes the whole result.

### 13.4 Rolling walk-forward — and one coin

The old split is **retired**. Fixed n = 15, h = 5, 20 bps; five tilts; **139 windows of 12 months, stepped 7
days, from 2023-01-01** (last 2025-08-24 → 2026-08-24). Nothing is re-fitted, so this is a rolling stability test
of fixed strategies. Stated biases: n and h are in-sample before 2025-07; the λ range came from TRAIN; windows
overlap 51/52 weeks, so ~3–4 independent windows, not 139.

| λ | SR full span | SR after 2025-07-01 | window SR mean | % windows beat λ=0 |
|---|---|---|---|---|
| −1.0 | 0.13 | −0.06 | 0.53 | 33% |
| −0.5 | 0.37 | 0.50 | 0.73 | 40% |
| **0** | **0.58** | 1.07 | **0.86** | — |
| +0.5 | 0.57 | 1.32 | 0.76 | 32% |
| +1.0 | 0.53 | 1.40 | 0.66 | 31% |

**No tilt wins.** Highest full-span and mean-window Sharpe; every tilt loses to it in 60–69% of windows; the
hypothesised down-tilt is monotonically worst. Frozen spec: **n = 15, h = 5, no volume layer**.

**But the post-split column is one trade.** ZEC rose ~11× (56 → 614) from 2025-09-26 to 2025-11-09; the
untilted book made +51% on ZEC alone over those 45 days. The post-split ranking of λ is just the ranking of ZEC
weight — ZEC's volume was high, so positive tilts leaned into it.

| | SR after split | after split, burst removed | full span, burst removed |
|---|---|---|---|
| λ = 0 | 1.07 | **−0.77** | **0.01** |

> **The finding that matters most.** ZEC is in the universe because its spread measured under 10 bps in
> **2026** — after the run made it liquid. By trailing dollar volume it ranked ~32nd of 60 when the burst began,
> sat in the point-in-time top-9 on **0 of 45** burst days, and entered only on **2025-11-15**, six days after the
> run ended. So the universe look-ahead that §13.3 found harmless through 2025-07 is **the entire out-of-sample
> gain** after it. Removing the largest stretch is a deliberate stress test, not a fair estimate — but together
> these mean the evidence for XS momentum beyond TRAIN is one look-ahead-dependent trade. TRAIN (≈0.46) is not
> contradicted, and not confirmed.

### 13.4b Point-in-time universes — was the ZEC profit real?

> **Correction.** An earlier version of this section was titled "the edge was the look-ahead". That came from a
> slow 90-day median liquidity screen, which admitted ZEC only after its run ended. ZEC was illiquid before the
> run but became liquid within days of it starting, so a responsive real-time screen would have held it. The
> claim is withdrawn.

**ZEC's liquidity.** 2022 through Q3 2025: ~$1–7k/day on Binance.US, 15–100× below the 9th-ranked coin, median
rank 31–52 of 60, no trade in up to 60% of hours. Its daily volume then went $8k (09-26) → $221k (10-02) →
$846k (10-11) → $3M (11-07), crossing the top-9 cutoff in early October.

**Screen-speed sweep**, same 139-window walk-forward (n = 15, h = 5, λ = 0, 20 bps). The 30-day screen was fixed
as the headline before any of these Sharpes were computed. One builder for every book; the 90-day screen
reproduces the earlier point-in-time book exactly.

| | look-ahead | 7-day | 14-day | **30-day** | 90-day |
|---|---|---|---|---|---|
| ZEC first eligible | always | 10-05 | 10-10 | **10-16** | 11-15 |
| net Sharpe, full span | 0.58 | 0.35 | 0.31 | **0.34** | 0.12 |
| net Sharpe after 2025-07-01 | 1.07 | 0.59 | 0.34 | **0.38** | −0.37 |
| burst return (45 days) | +82% | +42% | +30% | **+30%** | +9% |
| full span, burst removed | 0.01 | −0.01 | 0.04 | **0.04** | 0.02 |
| turnover | 0.150 | 0.178 | 0.168 | **0.162** | 0.155 |

**Post-liquidity-jump window** (2025-10-05, when a 7-day screen first admits ZEC, → 2026-08-28; 328 days, SE ≈ 0.88):
30-day screen 0.64 (look-ahead 0.78, 90-day −0.43). Every book lost money after the run ended (aftermath Sharpe
−0.48 to −0.86).

**What this means.**
- About half of the look-ahead book's out-of-sample result survives a real-time universe. The part that was not
  real is the ~12% ZEC position held through September while ZEC traded ~$2k/day, before any screen admitted it.
- The real part is still **one trade**. With the burst removed, every universe scores −0.01 to +0.04; after the run,
  every book lost money.
- 0.34 (SE ≈ 0.43) is within one standard error of zero. Out of sample, XS momentum caught one strong trend in a coin
  that became liquid as it trended, which is what momentum should do, and earned roughly nothing otherwise. That is a
  capturable profit, not evidence of a persistent edge.

**Is dollar volume a fair proxy?** Against the real Aug–Sep 2026 spreads, trailing $ volume has rank correlation
−0.90 and today's top-9 matches the spread set on 8 of 9 — validated in 2026 only. Limits: no historical spreads;
the 60-coin panel is a survivor set; Binance.US volume fell from ~$62M/day to ~$8M/day after Q2 2023; ZEC's October
2025 spread and capacity ($50k–$1M/day) are unknown.

### 13.5 Protocol: the split is retired

The 2025-07-01 split was compromised (earlier full-sample work; an old 80/20 boundary at 2025-09-22 inside it),
so it is no longer scored as a holdout. The rolling walk-forward replaces it and runs through 2026-08-28. The
research protocol and cost cells in the notebook still describe a window "scored once"; they were left untouched
by instruction, and the walk-forward section carries the supersession note. `test_only()` remains uncalled.

### 13.6 A convention mismatch worth knowing about

The volume cells (`actD1`, `actD3`, `actE1`) lag the signal one day more than the momentum and cost
sections — `.rolling(n).sum().shift(1)` plus the `w.shift(1)` in the P&L, versus a single lag in
`xs_pnl`. Harmless and conservative, but it makes their Sharpes ~0.02 lower and not directly comparable
(h=5 gross: 0.835 vs 0.855). The heatmap and walk-forward cells use the single-lag convention.

