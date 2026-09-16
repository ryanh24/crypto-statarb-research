# Binance.US Fees & Transaction Costs

Source: <https://www.binance.us/fees> — checked **2026-08-27**. Assuming **Tier 1**
(30-day volume not yet determined). The per-account tier table is login-gated;
figures below are the publicly published standard rates.

## Commission (published, standard pairs — all users)

| Order type | Role   | Fee     | Notes                              |
|------------|--------|---------|------------------------------------|
| Limit      | Maker  | 0.00%   | adds liquidity                     |
| Market     | Taker  | 0.02% (2 bps) | removes liquidity            |

- **Tier 0 pairs** (select premium pairs): 0% maker / 0.01% taker regardless of volume.
- **BNB discount:** additional **5% off** maker & taker when fees paid in BNB.
- **Tier 1+ pairs:** volume-based discounts on a trailing 30-day basis (see logged-in
  fee schedule to confirm exact numbers once volume is known).

## Cost baseline (for cross-checking)

A conservative all-in assumption, used throughout:

| Order type    | Commission | Assumed slippage | All-in    |
|---------------|-----------|------------------|-----------|
| Market order  | ~7 bps    | ~13 bps          | **20 bps** |
| Limit order   | ~7 bps    | 0                | **7 bps** |

## Asset-dependent costs (set per symbol during research)

- **Bid/ask spread** — half-spread paid on each side; varies by symbol/liquidity.
- **Market impact / slippage** — depends on order size vs. book depth and volume.

These are configured per-symbol in the notebook (`FEES` / `PROJECT_COST_ASSUMPTION`),
not global constants, because they differ materially across BTC vs. small-cap alts.

## TODO

- [ ] Log in and record exact Tier-1 maker/taker for target 30-day volume.
- [ ] Estimate per-symbol average spreads from order-book / high-low data.
- [ ] Decide market vs. limit execution assumption per strategy.
