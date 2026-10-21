# Retail demand forecasting: five simple methods compared

A comparison of five simple forecasting methods on Eurostat monthly retail sales for five countries. **Seasonal naive** (repeat the same month of last year) wins almost every test.

## Why this project

- A forecast is only useful if it beats a **simple baseline**. If it does not beat the baseline, the extra work adds nothing.
- I tested which simple method beats the baseline, and by how much, on five countries and four test years.
- Skills shown: forecasting, forecast accuracy measures (**WAPE**, **MAPE**, **bias**), Excel and Google Sheets formulas, and attention to detail.

## The question

Which simple forecasting method should a demand planning team use as its starting point, and how much better is it than the most basic forecast?

The audience is a **demand planning team** that has to explain and defend its choice of forecast method.

## Key results

Netherlands, test year 2025:

| Method | WAPE | MAPE | Bias | Improvement over naive |
|---|---|---|---|---|
| naive | 8.56% | 8.88% | +8.23% | 0% |
| seasonal_naive | 1.52% | 1.54% | −1.08% | 82.2% |
| moving_average (3 months) | 5.39% | 5.64% | +4.24% | 37.0% |
| 12_month_average | 4.43% | 4.45% | −1.08% | 48.3% |
| linear_trend | 4.90% | 5.12% | +3.64% | 42.8% |

Lower WAPE and MAPE are better. Bias close to 0 is better. Higher improvement is better.

- **Seasonal naive had the lowest WAPE in 19 of 20 tests**, across 5 countries and 4 test years (2022 to 2025).
- **The one exception is Germany 2022.** There the 12-month average won (WAPE 4.38% against 4.84%).
- **Naive is always the worst.** It repeats the December peak for all 12 months, so it is always too high.
- Seasonal naive removed **60% to 95%** of the naive error across all countries and test years.

## Data

- **Source:** Eurostat, dataset `sts_trtu_m` (turnover and volume of sales in wholesale and retail trade, monthly). [Link to my Eurostat view](https://ec.europa.eu/eurostat/databrowser/view/STS_TRTU_M__custom_23441896/default/table).
- **Downloaded:** 2026-10-21.
- **Countries:** Netherlands (NL), Germany (DE), France (FR), Spain (ES), Italy (IT).
- **Period:** January 2015 to August 2026. That is **140 monthly values per country**, with no gaps.

Filters used:

| Filter | Code | Meaning |
|---|---|---|
| Indicator | `VOL_SLS` | Volume of sales |
| Activity | `G47` | Retail trade, except motor vehicles and motorcycles |
| Unit | `I21` | Index, 2021 = 100 |
| Seasonal adjustment | `NSA` | Unadjusted |

What the numbers mean:

- **Index, 2021 = 100.** The average month in 2021 is 100, so a value of 110 means 10% more sales volume than that.
- **Volume, not euros.** The index tracks how much was sold, not the money value of the sales.
- **Unadjusted.** The seasonal pattern is still in the numbers, with high months such as December and low months such as February.

**Why unadjusted data?** Seasonally adjusted data has the seasonal pattern removed, so a test of seasonal methods would be meaningless on it.

The `raw` tab holds the full Eurostat download, which also contains other countries and other activity codes, including food and non-food. I only used `G47` for the five countries above.

### Data quality flags

Eurostat marks some values with a flag, where **p** means provisional and **e** means estimated.

| Country | Flags (activity G47) | Effect on the test years |
|---|---|---|
| NL | p from January 2026 to August 2026 | None. 2022 to 2025 are not flagged. |
| DE | p in every month | All German results use provisional data. |
| FR | p from August 2024 to July 2026, e in August 2026 | The whole 2025 test year is provisional. |
| ES | p in July and August 2026 | None. |
| IT | No flags | None. |

- **No rows were deleted.** Flagged values are used as they are.
- **Provisional numbers can still change**, so the results for Germany and France may change when Eurostat revises its data.
- The Netherlands flags come from the `raw_NL` tab, but my notes in the `note` tab do not mention them.

## Method

### Training and test rules

- **Test years:** 2022, 2023, 2024 and 2025. Each one is a separate test (a **backtest**: a test on past data where the answer is already known).
- **Training data:** always starts in **January 2015** and ends in the **December before the test year**. For 2025 that is January 2015 to December 2024 (120 months).
- **One forecast per test year.** All 12 months are forecast at once at the end of December, and the forecast is not updated month by month.

### The five methods

| Method | Rule in plain English | Example (Netherlands, forecast for 2025) |
|---|---|---|
| naive | Repeat the last training month for every month. | Every month of 2025 = December 2024 = 109.5 |
| seasonal_naive | Use the same month one year earlier. | January 2025 = January 2024 = 95.6 |
| moving_average (3 months) | Average of the last 3 training months, for every month. | Average of October to December 2024 = 105.47 |
| 12_month_average | Average of the last 12 training months, for every month. | Average of January to December 2024 = 100.08 |
| linear_trend | A straight line fitted on all training months, then extended (Excel `FORECAST.LINEAR`). | January 2025 = 104.04 |

### The four scores

Error = forecast − actual. A positive error means the forecast was too high.

| Score | Formula | What it tells you |
|---|---|---|
| **WAPE** (weighted absolute percentage error) | sum of absolute errors / sum of actuals | How wrong the forecast was overall, in %. Lower is better. |
| **MAPE** (mean absolute percentage error) | average of (absolute error / actual) | The average monthly % error. Lower is better. |
| **Bias** | sum of errors / sum of actuals | Too high (+) or too low (−) on average. Close to 0 is better. |
| **Improvement over naive** | 1 − (method WAPE / naive WAPE) | Share of the naive error that the method removes. Higher is better. |

**Why WAPE is the main score.** MAPE divides each error by that month's own actual, so errors in small months such as February count for more. WAPE divides the total error by total sales, so every unit of sales counts the same.

