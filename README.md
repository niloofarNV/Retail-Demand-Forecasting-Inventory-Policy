# Retail Demand Forecasting and Inventory Policy Evaluation (M5)

Does a better demand forecast actually reduce the inventory a store has to hold for the same service level?
This project answers that question on the M5 (Walmart) data with a day-by-day inventory simulation, instead of
stopping at forecast-accuracy metrics.

## What the code does

One run of the notebook produces accuracy tables with confidence intervals, a Diebold-Mariano test, an inventory
simulation compared at matched fill rate, calibration checks, feature importance, an ablation and eight figures.
Everything is written to `results/`.

**Forecasters (all re-forecast at every origin with the same information):** seasonal naive, 28-day moving
average, SBA, TSB, and LightGBM (Tweedie objective, two seeds, early stopping on a separate validation block).

**Safety-buffer rules:** per-SKU error scale, empirical-Bayes-shrunk error scale, conformal (group-pooled scaled
quantile), and LightGBM quantile spread. Each method's buffer comes from its own calibration errors.

**Inventory system:** daily review, 7-day lead time, order-up-to policy, lost sales. Methods are compared at
matched fill rate (90%, 95%, 98%) by interpolating each method's fill-rate / capital frontier, with
SKU-bootstrap confidence intervals (500 resamples).

**Design rules enforced and tested in the notebook:**
1. The target is demand over days t+1 ... t+P (P = review period + lead time), and every feature uses data up to day t only.
2. Train, validation, calibration and test target windows never overlap (purged splits).
3. Every method is re-forecast at every rolling origin.
4. Each method's safety buffer comes from its own calibration errors.
5. Methods are compared at matched fill rate, not at a nominal service level.

The notebook runs correctness tests (causality, target alignment, purging, simulator limits, weighted fill rate)
before the main run. They must print "all correctness tests passed".

## Data

The M5 data is not included. Download it from the
[M5 Forecasting competition page on Kaggle](https://www.kaggle.com/competitions/m5-forecasting-accuracy/data)
(subject to the competition's terms) and place these three files in `m5_data/`:

- `calendar.csv`
- `sell_prices.csv`
- `sales_train_evaluation.csv` (1,941 days; the older `sales_train_validation.csv` has only 1,913 days and is not enough)

## How to run

```bash
pip install -r requirements.txt
jupyter notebook m5_pipeline.ipynb
```

Run the cells from top to bottom. The last cell is the only slow one (about 2 minutes per rolling origin per
store on the machine used for the reported results). For a quick test set `CFG.n_origins = 1`.

To widen the study, change two settings in the configuration cell: `CFG.stores` (all ten stores are
`CA_1..CA_4, TX_1..TX_3, WI_1..WI_3`) and `CFG.n_origins` (13 origins with the default 28-day step cover one year).

## Results (store CA_1, 3 rolling origins, 9,130 SKU-origins)

These numbers come from running the notebook on the real M5 data. Running the notebook saves all tables and figures to `results/`.

**Forecast accuracy on the test windows**

| Model | WAPE (%) | MAE | RMSE |
|---|---:|---:|---:|
| Seasonal naive | 38.91 | 4.694 | 9.412 |
| Moving average (28 d) | 33.60 | 4.054 | 8.726 |
| SBA | 38.69 | 4.667 | 10.455 |
| TSB | 32.90 | 3.969 | 8.257 |
| **LightGBM** | **32.03** | **3.863** | **7.903** |

LightGBM is better than every benchmark. The Diebold-Mariano tests are positive and significant for all eight
comparisons; the weakest are against TSB. LightGBM is not best for the slowest-selling quartile of SKUs, where
TSB and the moving average are better.

**Capital needed for the same fill rate (LightGBM with per-SKU buffer vs. the best benchmark)**

| Fill rate | Capital saving | 95% CI (SKU bootstrap) |
|---:|---:|---|
| 90% | 3.4% | [2.2, 4.6] |
| 95% | 6.2% | [5.3, 7.3] |
| 98% | 8.0% | [6.2, 9.8] |

The more elaborate buffer rules (empirical-Bayes shrinkage, conformal, quantile spread) each needed more capital
than the plain per-SKU error scale on this data.

## Limitations

- One store, three rolling origins (about 12 weeks, February to May 2016). The results may not hold in other
  stores or seasons, including the year-end peak. The code supports all ten stores and more origins.
- The confidence intervals resample SKUs only. They do not include uncertainty from the choice of time period.
- M5 records sales, not demand. Demand during stock-outs is not observed, so demand is under-measured.
- The benchmark methods use fixed standard parameters (for example SBA and TSB alpha of 0.1), while LightGBM is
  tuned with early stopping. Tuning the benchmarks on the validation block would make the comparison fairer.
- The demand-quartile table forms its groups from realised test demand, so it is descriptive only.
- The simulator assumes a single store, a fixed lead time and no capacity or supplier constraints.

## Author

Niloofar Vahabzadeh Najafi

Research interests: supply chain analytics, machine learning, inventory optimization, operations research.
