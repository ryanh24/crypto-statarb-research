# Research Reading List — Crypto StatArb

Curated papers (arXiv / SSRN) mapped to the project's momentum & reversal threads.
Status: 🔲 to read · 📖 reading · ✅ read+noted.

---

## Start here (directly on point)

- 🔲 **Cryptocurrency Momentum and Reversal** — Dobrynskaya (SSRN 3913263).
  2,000 coins, 2014–2020. Finds **short-horizon momentum (up to ~2–4 weeks)** and
  **longer-horizon reversal (beyond ~1 month)**. Key takeaway: crypto's momentum→reversal
  switch happens ~1 month, *far faster* than equities ("faster metabolism"). Great map of
  where each effect lives by horizon — directly informs our horizon search.
  <https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3913263>

- 🔲 **Time-Series and Cross-Sectional Momentum in Crypto under Realistic Assumptions** —
  Han, Kang, Ryu, 2024 (SSRN 4675565). Critical, transaction-cost-aware take: after t-costs
  and liquidations, **time-series momentum survives; cross-sectional momentum is weak**.
  Read for the *sober* view and for how they model costs — mirrors our 20 bps / 7 bps setup.
  <https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4675565>

- 🔲 **Cryptoasset Statistical Arbitrage (mean-reversion / momentum factor)** —
  Kakushadze & Yu (arXiv 1903.06033). Includes an **algorithm + source code** for a
  dollar-neutral long-short cross-sectional mean-reversion alpha driven by the leading
  momentum factor. Closest to a buildable reference implementation.
  <https://arxiv.org/abs/1903.06033>

---

## Momentum thread

- 🔲 **Machine Learning and the Cross-Section of Cryptocurrency Returns** (AFFI 2023).
  Which features (incl. momentum) predict the cross-section; useful for the "investment
  themes / activity" sub-threads.
  <https://affi2023.eventsadmin.com/Papers/ViewContribution?cid=8390>

- 🔲 **Cross-cryptocurrency Return Predictability** (SSRN 3974583). Lead-lag / spillover
  predictability across coins — relevant to "front-run mechanical rebalancing" and themes.
  <https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3974583>

## Reversal thread

- 🔲 **Intraday Return Predictability in Crypto: Momentum, Reversal, or Both** —
  Wen, Bouri, Xu, Zhao (SSRN 4080253). Intraday horizon — speaks to the day/night,
  weekday/weekend (retail vs. institutional) seasonality sub-thread.
  <https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4080253>

- 🔲 **Copula-Based Trading of Cointegrated Cryptocurrency Pairs** — Tadi et al.
  (arXiv 2305.06961). Pairs / basket construction for the "A − (correlated basket)"
  reversal idea; cointegration + copula dependence.
  <https://arxiv.org/abs/2305.06961>

- 🔲 **Advanced Statistical Arbitrage with Reinforcement Learning** (arXiv 2403.12180).
  Mean-reversion signal construction; skim for the stat-arb spread mechanics, not the RL.
  <https://arxiv.org/abs/2403.12180>

## Microstructure / execution (costs, liquidations, price discovery)

- 🔲 **Price Discovery in Cryptocurrency Markets** (arXiv 2506.08718). Who leads price
  discovery — supports the lead-lag / front-running momentum angle.
  <https://arxiv.org/abs/2506.08718>

- 🔲 **Explainable Patterns in Cryptocurrency Microstructure** (arXiv 2602.00776).
  Order-book/trade-flow features for short-horizon predictability — ties to our
  asset-dependent spread/impact cost modeling.
  <https://arxiv.org/html/2602.00776v1>

---

## Notes template (copy per paper)

```
### <Paper> — <authors, year>
- Signal / horizon:
- Universe & period:
- Costs assumed:
- Result (net of costs):
- What to test in our data:
```
