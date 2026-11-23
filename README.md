# Predicting Next-Quarter Revenue and Profit Margin of U.S. Companies

*Using only what a company has already reported, can we estimate its revenue and profit margin for the next quarter?*

General Assembly Data Science Bootcamp, capstone project by Danilo Pavicevic.

## Executive summary

- **Question:** from a company's latest quarterly report, predict next quarter's **revenue** and **profit margin** (net income / revenue).
- **Data:** public SEC EDGAR filings, 357 non-financial U.S.-listed companies, 2018 to mid-2026 (11,682 company-quarters after cleaning).
- **Bar to beat:** naive persistence, "next quarter = this quarter", which an analyst gets for free.
- **Result:** a tuned **Gradient Boosting** model beats persistence on unseen 2025–2026 data by **32% for revenue** and **23% for margin** (mean absolute error), in every test quarter.
- **Main limits:** non-financial companies only, accounting data only (no sector, macro data or news), and large one-off losses remain unpredictable.

## 1. Problem statement

Analysts and finance teams often need an estimate of a company's next quarter before the official report is published. This project tests whether a company's latest quarterly figures (revenue, profit, cash flow, assets, equity, cash) and its own history are enough to predict next-quarter **revenue** and next-quarter **profit margin**.

- **Audience:** financial analysts, credit and risk teams, finance departments who want an early, data-based estimate.
- **Type of problem:** supervised regression on a panel (many companies, many quarters).
- **When the prediction is made:** right after quarter *t* is reported. Only values from quarter *t* or earlier are used as features, so there is no leakage from the future.

## 2. Goals and success metrics

- **Goal:** an estimate that is clearly better than the rule of thumb "next quarter will look like this quarter".
- **Main metric:** **MAE** (mean absolute error), lower is better. Margin has long tails and MAE is less sensitive to extremes than RMSE. R² is reported for reference only, because for revenue it looks excellent for any method.
- **Baselines:** a null model (always the train median) and **naive persistence** (next = this quarter). A model counts as useful only if it beats persistence.
- **Fair evaluation:** models are chosen only by 5-fold cross-validation on train (folds by company, `GroupKFold`); the test set (2025 onwards) is used once, at the end.

## 3. Data

- **Source:** [SEC EDGAR Company Facts API](https://www.sec.gov/search-filings/edgar-application-programming-interfaces), quarterly figures from 10-Q and 10-K filings. Public U.S. government data, company figures only, no personal data.
- **Coverage:** 400 U.S.-listed companies (selected from an S&P 500 snapshot), quarters from 2018 to mid-2026; file built on 9 November 2026.
- **One row:** one company in one quarter (key `cik` + `period_end`).
- **Size:** 13,966 rows and 66 columns in the raw file (`raw_data_400.csv`); **11,682 rows, 357 companies and 22 columns** after cleaning (`clean final.csv`).
- **Targets:** `log_revenue_next` (next quarter revenue, in log because revenue is very right skewed) and `profit_margin_next`.
- **Data dictionary:** section 3 of the [EDA notebook](01_EDA_clean%20final.ipynb) describes all 66 raw columns; section 4 describes the 22 columns of the clean table, including the formulas of the new columns and both targets.

**Main cleaning decisions** (all in one re-runnable `clean_data` function):
- **Banks, brokers and insurers (30 companies) are excluded.** Their "revenue" means something different (interest and investment income), and in this download it is often missing or wrongly tagged. The model applies to **non-financial companies only**.
- Revenue stored as text was converted to numbers; 33 zero or negative revenues were set to missing.
- 403 quarters with margins outside −100% to +100% were set to missing (mostly bad revenue tags, but also real crises such as cruise lines in 2020–21).
- Outliers that are real were kept: negative equity after large share buybacks (e.g. Domino's, Home Depot) and a few very high cash-flow-to-revenue quarters.
- Split **by time** before any analysis: train = quarters before 2025 (9,627 rows), test = 2025 onwards (2,055 rows). All EDA uses train only.

## 4. Approach

1. **EDA** ([`01_EDA_clean final.ipynb`](01_EDA_clean%20final.ipynb)), following the GA capstone EDA template: load, data dictionary and missing values, cleaning, univariate analysis, targets and relationships, correlation and leakage check, outliers, risks and next steps; exports `clean final.csv`.
2. **Feature engineering** ([`02_modelling_.ipynb`](02_modelling_.ipynb)): 9 base features (size, profit and cash ratios, balance sheet ratios, growth, calendar quarter) plus 5 features from the company's own history (`margin_lag1`, 4-quarter average margin, gap to that average, cash flow vs profit gap, last year's seasonal revenue step), built with `groupby` + `shift`.
3. **Baselines:** null model (`DummyRegressor`, median) and naive persistence.
4. **Models:** the best model of each family from a first round of course regressors: **Lasso** (linear, L1 regularisation), **Random Forest** (bagging) and **Gradient Boosting** (boosting). Each is a pipeline with median imputation plus a missing flag (and scaling for Lasso), tuned with `GridSearchCV` on the same company folds.
5. **Evaluation and interpretation:** test MAE vs baselines, error over time, share of close predictions, error analysis by company size and largest misses, permutation importance.

## 5. Results

Test MAE on 2,055 company-quarters from 2025 to mid-2026 (change vs naive persistence in brackets):

| Target | Null model | Naive persistence | Lasso | Random Forest | **Gradient Boosting** |
|---|---|---|---|---|---|
| Next quarter log revenue | 0.967 | 0.089 | 0.068 (−24%) | 0.071 (−20%) | **0.060 (−32%)** |
| Next quarter profit margin | 0.095 | 0.060 | 0.053 (−12%) | 0.047 (−21%) | **0.046 (−23%)** |

- **Gradient Boosting is the final model for both targets**, chosen by train CV and confirmed on test; CV and test rank the three models the same way.
- All three models beat persistence in every test quarter.
- **Feature engineering matters more than the model:** without the 5 history features, default Gradient Boosting beats persistence in CV by only 1.6% (revenue) and 8.0% (margin); with them by 11.0% and 15.0%.
- For margin, R² rises from 0.15 (persistence) to 0.51; 63% of Gradient Boosting revenue predictions are within about 5% of actual revenue (48% for persistence).

## 6. Findings

- **Revenue is easy, margin is hard.** Revenue barely moves between quarters (r = 0.99 in log), margin much more (r = 0.60).
- **What the model relies on:** for revenue, this quarter's revenue plus last year's seasonal step ("if revenue jumped into this season last year, it will probably do so again"); for margin, the company's 4-quarter average margin and this quarter's margin ("next margin lies between this quarter and the company's normal level", so a one-off bad quarter is expected to recover).
- **Where it fails:** sudden one-off losses or gains (write-downs, mergers), e.g. Healthpeak, Hasbro, Kraft Heinz. In most of these cases nothing in the accounting data warned of the shock. Margin errors are largest for the smallest companies.

## 7. Limitations and ethics

- Non-financial companies only; large U.S. listed companies only (results may not hold for small, private or non-U.S. firms).
- Survivorship bias: a recent company list, so failed or delisted firms are missing.
- The most extreme quarters (margins outside ±100%) were removed from both train and test, so the model will underestimate real crises and the test MAE does not cover them.
- Short test period (about six quarters); tuning CV scores are slightly optimistic, the test MAE is the honest number.
- Only accounting data. Predictions should support an analyst's judgement, not replace it in credit, investment or audit decisions.

## 8. Next steps

1. Add sector (SEC SIC code), simple macro data and company guidance.
2. Check stability with a rolling time split (`TimeSeriesSplit`).
3. Retrain every quarter, save the pipeline with `joblib` and monitor MAE against persistence.
4. A separate model for banks and insurers, and prediction intervals for each estimate.

## How to navigate this repo

Start here, then read the [EDA notebook](01_EDA_clean%20final.ipynb) (data checks, cleaning, findings, next steps) and the [modelling notebook](02_modelling_.ipynb) (summary at the top, baselines, models, evaluation, interpretation, limitations). The raw data is in [`raw_data_400.csv`](raw_data_400.csv) and the cleaned data in [`clean final.csv`](clean%20final.csv).

## How to reproduce

The raw data file `raw_data_400.csv` is included in the repository; no account or API key is needed.

```bash
git clone https://github.com/Danilop88/DSB_Cap.git
cd DSB_Cap
python -m pip install pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook
```

1. Run `01_EDA_clean final.ipynb` from a fresh session: it reads `raw_data_400.csv` and writes `clean final.csv` (plus `figures/` and `outputs/decisions.csv`).
2. Run `02_modelling_.ipynb`: it reads `clean final.csv` and writes charts to `figures/` and the results table to `outputs/model_results.csv`. The tuning cell takes a few minutes.

In Colab, first clone the repository, enter its folder and install the packages in a code cell (`!git clone ...`, `%cd DSB_Cap`, `%pip install pandas numpy matplotlib seaborn scikit-learn`). Opening the badge alone does not copy the repository or its data.

[![Open EDA in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Danilop88/DSB_Cap/blob/main/01_EDA_clean%20final.ipynb)

## Repository structure

```text
README.md                    Project summary (this file)
01_EDA_clean final.ipynb     EDA: explore, clean, data dictionary, decisions; writes clean final.csv
02_modelling_.ipynb          Modelling: baselines, features, 3 tuned models, evaluation, interpretation
raw_data_400.csv             Raw quarterly data from SEC EDGAR (untouched)
clean final.csv              Clean data, written by the EDA notebook and read by the modelling notebook
```

## Sources, author and credits

Danilo Pavicevic, General Assembly Data Science Bootcamp, Capstone project.
Data: U.S. Securities and Exchange Commission (SEC), EDGAR Company Facts. Libraries: pandas, NumPy, matplotlib, seaborn, scikit-learn.
Links to every library, the General Assembly course code each step follows, and the data sources are in the Appendix at the end of each notebook ([EDA](01_EDA_clean%20final.ipynb), section 9; [modelling](02_modelling_.ipynb), section 13).

## License

Code: MIT License, © 2026 Danilo Pavicevic.
Data: SEC EDGAR data is public U.S. government information; use follows the [SEC access rules](https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data).
