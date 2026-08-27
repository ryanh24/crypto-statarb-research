# CryptoStatArb — WSQ Course Project

**Statistical Arbitrage in Cryptocurrencies**

Research project for The Wall Street Quants course. Goal: research profitable
**momentum and/or reversal** strategies in crypto, backtest them (unconstrained
style), account for realistic execution costs, and evaluate performance.

## Exchange & Costs

- **Exchange:** [Binance.US](https://www.binance.us/fees)
- **Fee tier:** Tier 1 (volume not yet determined)
- Commission fees are tracked against the live Binance.US fee schedule.
  Bid/ask spread and market-impact t-costs are **asset-dependent** and set per symbol.
- See [`docs/fees.md`](docs/fees.md) for the recorded fee schedule and cost assumptions.

## Research Threads (from project brief)

- **Momentum:** time horizon, activity/new-info indicators, seasonality
  (weekday/weekend, day/night), investment themes, technical/mechanical rebalancing.
- **Reversal:** shorter horizons, uninformed/liquidity-driven trading, correlated
  pairs & baskets, macro volatility/dislocation regimes.

## Structure

```
CryptoStatArb/
├── research.ipynb      # main research notebook (imports, data pull, EDA)
├── docs/
│   ├── fees.md         # Binance.US fee schedule + t-cost assumptions
│   └── papers.md       # research paper reading list (arXiv / SSRN)
├── data/               # price-volume data (gitignored; regenerate via API)
└── .gitignore
```

## Setup

```bash
pip install python-binance pandas numpy matplotlib seaborn scipy statsmodels
```

API keys go in a local `.env` (gitignored) — never commit keys.

## Evaluation Metrics

Returns, volatility, Sharpe, max drawdown, alpha/beta.
