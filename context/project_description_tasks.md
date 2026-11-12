## Project Description and Tasks

Split every daily return into its overnight (close-to-open) and intraday (open-to-close) halves and a strange thing appears: each half persists strongly on its own, while the two offset each other across periods, and the offsetting reversal runs for years. The punchline is the one every quant remembers—"profits of 14 trading strategies are earned either entirely overnight or entirely intraday." Momentum is an overnight strategy. Several others are entirely intraday. Anyone who trades a factor book at the open or the close has to know which of these they are in.

### Tasks:

- Build the overnight/intraday decomposition
- Replicate the persistence and reversal tests, including Figure 2, the t-statistics of the overnight/intraday persistence test across horizons
- Replicate the strategy table
- Replicate Figures 1 and 3 (trading volume)
- Update: extend through the most recent year, including the period after 2019 when retail participation and closing-auction volume both jumped
- Data sources: CRSP daily stock file (open, close, returns) on WRDS; Ken French's library for the factors and for the strategy definitions; daily TAQ for the intraday volume figures
