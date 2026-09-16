# Idea (parked): Cross-Venue Lead-Lag Market Making

**Status:** parked — revisit after the momentum/reversal track is working.
A second strategy to explore later. Not being built yet.

## The concept

A form of stat arb. Use the **most liquid venue (Binance)** as the fair-value
anchor and **market-make on a less-liquid venue** around that fair value.

- **Binance = price-discovery leader.** Most liquid → tightest spread → incorporates
  new info first. Its mid / microprice = best estimate of **fair value**.
- **Less-liquid venue = laggard.** Wider spread, slower-updating quotes. Post two-sided
  quotes there, skewed around the Binance-derived fair value.
- You earn the **slow venue's spread**; Binance tells you where price really is and lets
  you **hedge / offload inventory**.

## Distinction to keep straight

| | Pure cross-exchange arbitrage | Cross-venue market making (this idea) |
|---|---|---|
| Role | Taker on both sides | **Maker** on slow venue, hedge on Binance |
| Edge | Buy cheap venue / sell dear venue on gaps | Earn slow-venue **spread**; Binance = fair-value anchor + hedge |
| Fees | Pay taker twice | Earn/pay **maker** fees (often rebated) |

The profit is the **spread captured as a liquidity provider**, not the raw price gap.

## Why speed matters (the core risk: adverse selection)

Resting quotes on the slow venue go **stale** the instant Binance moves. Example:
Binance jumps up → your resting ask is now below fair → an informed trader lifts it →
you sold below fair value. Speed = ability to **cancel/reprice before being picked off**.
The edge is being *faster than other participants at reflecting Binance onto the slow venue*.

## Feasibility (be honest)

The strategy as described by pros assumes capital + colocated cloud infra (AWS near the
exchange, fastest feeds). **A daily-bar research project cannot win a live latency race.**
The realistic, still-valuable *research* version:

1. **Measure lead-lag** between Binance and a smaller venue (how much / what horizon does
   the small venue follow Binance?). Ties to the price-discovery / lead-lag papers in
   [`papers.md`](papers.md).
2. **Construct fair value** — Binance mid vs. microprice.
3. **Simulate a quoting + inventory policy** on historical data; estimate P&L **net of fees
   and adverse selection**.

## Open decisions before building

- **Data granularity:** (a) minute klines — cheap, reuses our fetch pattern, but can only
  do coarse fair-value / lead-lag, not realistic bid/ask quoting; (b) **L2 order book +
  trades** (python-binance websockets / depth snapshots) — what MM actually needs, but a
  real data-eng lift and must be *recorded going forward* (free history is scarce).
- **Which less-liquid venue** pairs against Binance (note: Binance.US liquidity map differs
  from global Binance).

## Source

Sparked by a practitioner quote on connecting to a less-competitive venue and market-making
using data from the most liquid exchange (Binance) as fair value.
