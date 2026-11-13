# Tug of War — Final Project Pitch

Lou, Polk & Skouras (2019), *A Tug of War: Overnight Versus Intraday Expected Returns* · FINM 32800 · Nov 11, 2026 · @Hoang Pham

## 1. The claim

The overnight (close-to-open) and intraday (open-to-close) parts of a stock's return each persist on their own and reverse each other, for up to 60 months. As a result, all 14 classic strategies earn their premium either entirely overnight (momentum, reversal) or entirely intraday (value, profitability, investment, beta, idio-vol, and others).

## 2. What the paper does not do

Nobody publishes overnight/intraday factor returns, and the paper's evidence stops in 2013. Three gaps follow:

- **No public dataset.** Ken French posts close-to-close factors only. Anyone who wants the overnight and intraday legs of HML, UMD or RMW has to rebuild them from CRSP and TAQ.
- **No out-of-sample test.** The TugOfWar signal (Eq. 1, Table 5) is fit and tested on 1993–2013 only. Nobody has checked whether it still forecasts factor returns.
- **No test across market-structure changes.** The paper predates zero commissions (Oct 2019), the 2020–21 retail surge, and the closing auction's growing share of daily volume (our TAQ figures will measure it). Each of these shifts who trades at the open and who trades at the close, which is the paper's own mechanism.

## 3. What we will build

We build one `doit` pipeline with three layers: replication, update, and extension. The scaffold is still the template's repo-rate example, so step 0 removes it. Step 0 also sets `START_DATE = 1993-01-01` and `END_DATE = 2025-12-31` in `settings.py`.

| Layer | Exhibit / output | Data (WRDS unless noted) | Planned scripts |
| --- | --- | --- | --- |
| Decomposition | Firm-day and firm-month overnight/intraday returns; month = product of daily legs; open missing → hold to next open | CRSP daily (`dlyopen`, `dlyprc`, `dlyret`, shares, volume) | `pull_crsp.py` → `clean_overnight_intraday.py` |
| Replication | Table 1: 10–1 sorts on last month's overnight / intraday return | CRSP daily (as above); Ken French factors (Mkt-RF, SMB, HML, RF) from WRDS `ff.factors_monthly` | `table1_persistence.py` |
| Replication | Fig. 2: t-stats for the 4 persistence/reversal tests at lags 1–60 | CRSP daily + Ken French factors (as in Table 1) | `fig2_lag_tstats.py` |
| Replication | Table 2: overnight vs intraday CAPM alphas for CRSP + strategies | CRSP, Compustat (BM, ROE, INV, ISSUE, ACCRUALS) merged via CCM link (`crsp.ccmxpf_lnkhist`), I/B/E/S (SUE) via `wrdsapps.ibcrsphist` | `pull_compustat.py`, `build_strategies.py`, `table2_strategies.py` |
| Replication | Figs. 1 and 3: dollar-volume share by 30-min bucket; Fig. 3 split into <$5k vs >$50k trades, 1993–2000 | TAQ: Monthly TAQ (`taq`) for 1993–2003, Daily TAQ (`taqm_*`) after; aggregated to 13 buckets on WRDS before download | `pull_taq_buckets.py`, `fig1_fig3_volume.py` |
| Update | Same exhibits for 1993–2025, plus 2014–2025 on its own (Fig. 3 stays 1993–2000); one sentence per exhibit on whether the result held | CRSP daily, Ken French, Compustat/CCM, I/B/E/S, TAQ, extended to 2025 | Same scripts; only `END_DATE` changes |
| Extension | **Tug-of-war factor library and monitor** (below) | Outputs of the layers above | `build_factor_legs.py`, `tugofwar_oos.py`, notebook |

**Extension: a tug-of-war factor library and monitor.**

1. **Dataset.** Monthly overnight, intraday and close-to-close returns for 12 long-short factors (Table 2's strategies minus time-series momentum), 1993–present. It updates by changing `END_DATE`. We publish the code and summary series on the ChartBook site, not raw CRSP data.
2. **Out-of-sample test.** We re-estimate Table 5's TugOfWar regression (60-month-half-life EWMA) on 1993–2013, then test it on 2014–2025 with no refitting. Measures are OOS R² and the return of a strategy-timing portfolio.
3. **Regime test.** We split each factor's overnight/intraday decomposition and the TugOfWar coefficient into before and after Oct 2019 (zero commissions). We also report 2020–21 (the retail surge) on its own. Fig. 1 is updated for each regime to show the closing-auction shift.
4. **Monitor.** A ChartBook page shows the current TugOfWar reading for each factor and labels each factor overnight or intraday by its latest 60-month alphas.

**Scope cut to agree on:** we drop time-series momentum. It needs Thomson Reuters Tick History futures data, which the course does not provide. SUE is a stretch goal because it depends on I/B/E/S.

## 4. Who would clone it

- **Factor PMs and execution traders** who rebalance at the open or the close. They would check which leg each of their factors earns in today and decide whether to trade in the opening or closing auction.
- **Quant researchers** testing a new anomaly. They would add one signal column and get its overnight/intraday decomposition with the paper's filters and Newey-West t-stats already applied.
- **Academics** studying clienteles or retail flow. They would use the factor-leg series as a ready-made input, much as they use Ken French's library today.

## 5. How we'll know it worked

We commit these tolerances as `pytest` tests before running on the full sample. They are loose on purpose: the paper's open price is the TAQ 9:30–10:00 VWAP, and ours is CRSP's opening price. The paper's own midquote-open robustness check shows the open definition matters. With it, the Table 1 overnight alpha falls from 3.47% to 1.88%.

| Exhibit | Pass criterion (1993–2013) |
| --- | --- |
| Table 1 | All four 10–1 spreads have the paper's sign with \|t\| > 2; each point estimate is within 50% of the paper's |
| Fig. 2 | Each of the 4 curves has the paper's sign at ≥ 90% of lags 1–60 |
| Table 2 | Each strategy falls on the same side (overnight vs intraday) as in the paper; alphas are within 2 of the paper's standard errors |
| Fig. 1 | U-shaped profile; first half-hour share within 2 pp of 14.25% |
| Fig. 3 | Small trades skew toward the open and large trades toward the close |
| Extension | OOS test runs end to end; we report the result whatever its sign (a failed out-of-sample test is still a finding) |

## Data validation plan

Each check below runs as a test with a tolerance fixed in advance. The report gets a filter-count table and a coverage-and-reconciliation figure built from these checks.

| Check | Second source or identity | Tolerance |
| --- | --- | --- |
| Open price | CRSP open vs TAQ 9:30 opening print, on a sample of stock-days | Median absolute gap < 0.5% |
| Volume | Sum of TAQ 30-min buckets vs CRSP daily volume | Within 5% on ≥ 95% of stock-days |
| Market leg | Our CRSP value-weight return vs Ken French Mkt (close-to-close) | Correlation > 0.99 |
| Strategies | Our size, BM and momentum deciles vs Ken French decile portfolios | Correlation > 0.95 |
| Identity | (1 + overnight)(1 + intraday) = 1 + CRSP return, firm-month | Exact to 1e-8 |
| Filters | Share of rows removed by the price < $5 and bottom-NYSE-quintile filters, by year | Within a range we set before the first run |
| Coverage | Share of stock-days with a valid open price, by year and exchange | No unexplained break; 1992–93 start and Nasdaq gaps documented |

## Risks and questions for the consultation

- **TAQ size.** Figs. 1 and 3 need trade-level TAQ for 1993–2025. We plan to aggregate on the WRDS cloud and download only 13 buckets per day, falling back to a sampled day per week if that is too slow. Is a sample acceptable?
- **Open price.** Is CRSP's open an acceptable replication input, or should we build the paper's TAQ VWAP open for a subsample as a comparison (a head-to-head add-on)?
- **CI.** Tests that need WRDS are skipped in GitHub Actions. The rest run on a small synthetic CRSP-shaped fixture checked into `tests/`.
- **Scope.** We drop time-series momentum. SUE and Fig. 3 after 2000 are optional. The paper itself warns that trade size no longer identifies institutions after 2001.
