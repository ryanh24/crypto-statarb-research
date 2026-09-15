# CryptoStatArb — Cross-Sectional Momentum in Crypto

**WSQ course project: Statistical Arbitrage in Cryptocurrencies.** A dollar-neutral cross-sectional momentum strategy on
liquid Binance.US coins, built and then stress-tested against transaction costs, a rolling walk-forward, and
point-in-time universe selection.

**Full writeup: [`docs/report.md`](docs/report.md)**

## Result

On liquid coins, ranking by trailing 15-day return and holding 5 days produced a gross in-sample Sharpe of 0.85 with
near-zero BTC beta. Each stress test then took part of it away:

1. **Costs.** 20 bps per dollar traded cut net Sharpe to 0.46 and chose the holding period.
2. **Volume tilt.** A volume-sizing overlay did not survive those costs.
3. **Walk-forward.** A 139-window rolling walk-forward traced most of the out-of-sample gain to one 45-day ZEC trend.
4. **Point-in-time universe.** A real-time liquidity universe kept part of that trade.

With the episode removed, net Sharpe is roughly zero under every universe definition. **It caught one real momentum
episode; there is no evidence of a persistent edge.**

| n = 15, h = 5, net of 20 bps, point-in-time 30-day liquidity universe | TRAIN 2022-07 → 2025-07 | walk-forward 2023-01 → 2026-08 | full history 2022-01 → 2026-08 |
|---|---|---|---|
| gross / net Sharpe | 0.98 / 0.48 | 0.82 / **0.34** | 0.83 / 0.38 |
| net Sharpe, 45-day ZEC burst removed | — | **0.04** | 0.16 |
| turnover / day | 0.161 | 0.162 | 0.160 |
| ann. volatility / max drawdown | 16.3% / −24.9% | 17.0% / −29.8% | 17.9% / −29.8% |
| alpha (ann.) / t-stat vs BTC | 7.7% / 0.98 | 6.7% / 0.91 | 7.3% / 1.06 |
| beta / correlation vs BTC | 0.00 / 0.01 | −0.02 / −0.06 | −0.03 / −0.07 |

Sharpe = daily mean / std × √252, the course convention. The panel is survivors only (see the report's limitations).

## Structure

```
CryptoStatArb/
├── download_data.ipynb       # run once -> writes the canonical data pickles
├── research.ipynb            # main notebook: EDA -> protocol -> XS momentum -> costs -> robustness
│                             #   -> volume tilt -> walk-forward -> point-in-time universes -> final results
├── research_failed.ipynb     # executable graveyard: reversal, pairs, intraday, spread estimators
├── docs/
│   ├── report.md             # final writeup
│   ├── figures/              # figures used in the report (exported from research.ipynb)
│   ├── strategy_log.md       # index of every strategy tested, the test that settled it, and why
│   ├── fees.md               # Binance.US fee schedule + cost assumptions
│   ├── papers.md             # reading list
│   └── market_making_idea.md # parked idea: cross-venue lead-lag market making
├── data/                     # price-volume pickles (gitignored; regenerate via download_data.ipynb)
└── .gitignore
```

## Data workflow

Run **`download_data.ipynb`** once to build the shared dataset. Universe construction:

1. Keep coins listed on **both Binance.US and Coinbase** (USD/USDT), stablecoins excluded.
2. Rank by Binance.US 24h volume, take the **top 200** (~140 exist on both).
3. Download **hourly OHLCV from 2022 → now** (UTC), pruning coins listed too late.
4. Drop any coin with **< 90%** hourly-bar coverage over the window.

This leaves 60 coins. Output goes to `data/`, which is gitignored; regenerate by re-running:

- `binance_us_hourly_ohlcv.pk` — full Open/High/Low/Close/Volume panel, columns `(Field, Ticker)`.
- `binance_us_hourly_close.pk` — close-only convenience view.
- `universe.csv` — final list of kept symbols.

The order-book spread snapshots (`data/orderbook_spread_snapshots.csv`) that define the liquid universe were
collected live in August–September 2026.

## Reproducing

```bash
pip install python-binance pandas numpy matplotlib seaborn scipy statsmodels pygam
```

Then run `download_data.ipynb`, then `research.ipynb` top to bottom. The final statistics cell (`rep0`) asserts that it
reproduces the walk-forward and screen-sweep books exactly.

API keys go in a local `.env` (gitignored) — never commit keys.
