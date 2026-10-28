# Taxpayer Payment Risk and Liquidity: EDA for Modeling

## About

This capstone explores three next-year prediction tasks for synthetic companies:

1. **Will the taxpayer pay late?** Predict whether at least one payment instalment will be late in the next year.
2. **How much will remain unpaid?** Estimate the next year's closing tax debt in EUR, a proxy for outstanding tax exposure.
3. **What is the liquidity tendency?** Predict whether next-year liquidity coverage will improve, remain stable, or deteriorate.

The analysis follows the learning objectives and documentation structure of [GA's EDA for Modeling lesson](https://github.com/ga-curriculum/eda-for-modeling): understand variables and distributions, interpret correlations, prepare features, and standardize data without leakage. It is a project-specific analysis, not an official GA assessment.

**Current stage:** EDA, data-quality assessment, target construction, preprocessing demonstration, and baseline evaluation are complete. No learned classifier or regressor has been trained. The existing liquidity projection is a deterministic trend rule.

**Important distinction:** year-end tax debt is not the amount paid late during the year. A taxpayer can pay late and settle everything before year-end. Exact overdue instalment amounts require payment schedules, due dates, and payment transactions, which this workbook does not contain.

## Learning Objectives

By working through this project, you should be able to:

- Translate a business question into a prediction horizon and measurable outcome.
- Identify continuous, discrete, categorical, and binary variables.
- Examine skewness, structural zeros, losses, and legitimate undefined ratios.
- Compare Pearson's linear correlation with Spearman's rank correlation.
- Recognize accounting identities, redundant predictors, and target leakage.
- Fit transformations using training observations only.
- Establish baselines before fitting predictive models.
- Explain why associations in synthetic panel data do not establish causation.

## Content

| Topic | What was completed | Evidence |
|---|---|---|
| Problem definition | Three next-year outcomes and limitations | [Prediction targets](#prediction-targets) |
| Data audit | Keys, panel completeness, types, missingness, bounds, accounting identities | [Quality checks](tables/quality_checks.csv) |
| Univariate EDA | Summary statistics, zero rates, skewness, outlier review | [Training statistics](tables/descriptive_statistics_train.csv) |
| Bivariate EDA | Numeric correlations and target associations | [Target associations](tables/target_associations_train.csv) |
| Feature preparation | Exclusions, engineered measures, training-only log transformation and scaling | [Scaling evidence](tables/scaling_train_only.csv) |
| Baselines | Payment delay, outstanding balance, and liquidity tendency | [Payment/amount baselines](tables/three_task_baselines.csv), [liquidity backtest](tables/forecast_backtest.csv) |
| Reproduction | Python analysis and exported figures/tables | [EDA script](eda_capstone.py) |

## Business Problem and Intended Use

The intended user is a tax-risk analyst prioritizing cases for review and estimating future payment pressure. Predictions are made **after the end of year t**, when that year's financial and payment fields are available, and concern year t+1.

This is a learning exercise using a synthetic company adaptation of an earlier individual-level dataset. Fictional names, simulated tax assumptions, inherited late-payment patterns, and reallocated financial fields do not represent verified company financial statements.

The project does not establish actual tax policy, creditworthiness, or real-world predictive validity.

## Dataset

| Property | Verified result |
|---|---:|
| Source | companies_2021_2025_liquidity_en=capstone.xlsx |
| Worksheet | companies_2021_2025_liquidity_e |
| Rows | 250,000 |
| Columns | 37 |
| Distinct companies | 50,000 |
| Years | 2021–2025 |
| Records per company | Five, one in each year |
| Currency | EUR |
| Composite key | company_id + year |
| Duplicate rows / duplicate keys | 0 / 0 |
| Stored Excel formula errors | 0 |
| Stored Excel formula cells | 0; derived values are saved as numbers |

All company-year rows were inspected. Financial identities were recomputed independently of the stored values. The source workbook was not modified.

**Source SHA-256:** `2c51af3d1dbb2ed6ec32f797bc9afe55e889e5c94a79499c9ce2463739e152a6`.

The dataset has 50,000 company trajectories, not 250,000 independent companies. Repeated observations must remain grouped for splitting and uncertainty assessment.

### Variable Types and Random Variables

A random variable assigns a numerical outcome to an observation selected from the population. Here, selecting a company-year yields variables such as cash balance X and next-year late-payment indicator Y.

| Type | Examples | Treatment |
|---|---|---|
| Identifier | company_id, company_name | Never use as model predictors |
| Categorical | region_code | Encode as nominal categories |
| Constant | entity_type, assumed_tax_rate | Drop for model training |
| Discrete | year, number_of_late_installments, maximum_days_past_due | Preserve meaningful ordering; year also controls time splits |
| Continuous | revenue, cash, profit, assets, liquidity coverage | Assess scale, skewness, zeros, and negative values |
| Binary | zero_assets_flag, nonpositive_profit_flag, late-payment flags | Preserve as indicators |
| Forecast output | Columns starting with forecast_ | Exclude from an independent predictive model |

See the complete [data dictionary](tables/data_dictionary.csv).

## Prediction Targets

Targets are reconstructed within the analysis by shifting each company's observations one year forward. They are **not added to or written over the supplied workbook**.

| Task | Analysis target | Definition |
|---|---|---|
| Payment lateness | late_payment_next_year | 1 if next year's number_of_late_installments > 0, otherwise 0 |
| Outstanding amount | closing_tax_debt_next_year_eur | Next year's closing_tax_debt_eur |
| Liquidity tendency | realized_proxy_direction_next_year | Direction of next year's liquidity coverage relative to the current year |

For liquidity, let L be cash coverage in months and B = max(0.25 months, 10% of current L). The direction is improving if L(t+1) - L(t) > B, deteriorating if the change < -B, otherwise stable. These thresholds are working assumptions, not an externally validated risk definition.

Cash coverage is:

```text
cash_balance_eur /
((operating_expenses_eur + annual_loan_repayments_eur + tax_assessed_eur) / 12)
```

This is a cash-runway proxy. It is not the standard current ratio, an audited cash-flow measure, or a complete working-capital calculation.

**2025 has no observed 2026 outcomes.** Use 2021–2024 to derive outcomes that can be checked within this panel. Do not convert unavailable 2025 outcomes into zero or stable labels.

The stored `forecast_liquidity_movement_next_year` is an existing rule's prediction, not ground truth. Training against it would reproduce that rule rather than test next-year outcomes.

## Train, Validation, and Test Design

The existing split is company-disjoint: 35,000 training, 7,500 validation, and 7,500 test companies. No company occurs in multiple groups.

The proposed chronological prediction design is:

| Role | Company group | Feature years | Outcome years | Rows |
|---|---|---|---|---:|
| Model-development EDA and future model fitting | train | 2021–2023 | 2022–2024 | 105,000 |
| Validation | validation | 2024 | 2025 | 7,500 |
| Test | test | 2024 | 2025 | 7,500 |
| Future scoring only | Any eligible group | 2025 | 2026, unavailable | Not evaluable here |

Univariate feature EDA, correlations, feature-target associations, and scaling use only the 105,000 development rows. Full-workbook tables audit integrity and coverage; they are not used to fit preprocessing or choose thresholds.

The pre-existing trend rule was backtested without tuning it. Validation/test metrics below are transparent baseline audits. Since these results have now been inspected, they should not be repeatedly reused to choose future models. Freeze the modeling plan or reserve fresh unseen data for a later final evaluation.

## Data Quality Findings

**72 checks passed at the documented tolerances.** This establishes internal consistency, not economic realism or readiness for deployment.

- Keys are unique and all companies have a complete five-year panel.
- Company names and split assignments are stable across years.
- Numeric values have no stored infinities, and flags have valid binary values.
- Total revenue and total assets reconcile to their components.
- Profit, taxable profit, tax assessment, net assets, and tax-debt roll-forwards reconcile.
- Each year's opening tax debt matches the preceding year's closing debt.
- Late-payment flags agree with the day thresholds.
- The stored liquidity formula, forecast, and direction rule were reproduced.
- Monetary checks generally allow EUR 0.021; tax-rate multiplication allows EUR 0.006. Ratios allow 0.000001, and four-decimal coverage values allow 0.000051. These cover stored rounding precision rather than material differences.

### Missing Values Are Defined, Not Random

| Column | Missing rows | Share | Verified reason |
|---|---:|---:|---|
| debt_to_assets_ratio | 2,211 | 0.88% | Total assets are zero |
| debt_service_to_profit_ratio | 81,378 | 32.55% | Profit is nonpositive |
| forecast_liquidity_coverage_months_next_year | 50,000 | 20.00% | First-year records have no prior observation |

These patterns match their definitions exactly. No other input columns are missing. Retain the existing denominator-condition flags and use a training-fitted imputation policy only if a chosen estimator requires it. Do not interpret an undefined debt-service ratio as zero burden.

![Expected missingness](figures/01_missingness.png)

### Modeling Risks and Remediation

| Priority | Finding | Consequence | Action |
|---|---|---|---|
| Critical for external validity | The company data are synthetic adaptations | Good scores do not validate actual taxpayers | Obtain genuine company records and future outcomes before real-world use |
| High | Outstanding balance is not an exact overdue-payment amount | The amount target can answer a different business question | Use the stated proxy or obtain instalment/payment-level data |
| High | Forecast outputs are formula-derived | Using them as predictors can leak the output rule | Exclude all forecast_ fields and score against next-year observations |
| High | Year and company grouping both matter | Random row splits can overstate performance | Preserve company-disjoint and chronological evaluation |
| Medium | Debt-service ratio is undefined for 32.55% of rows | Blind imputation changes its economic meaning | Keep the nonpositive-profit flag and document missingness handling |
| Medium | Several variables are algebraically dependent | Linear models can become redundant or unstable | Select a coherent feature set and compare regularized models |
| Medium | Outstanding amount is mostly zero | Aggregate MAE can reward predicting zero everywhere | Report positive-balance detection and conditional amount errors too |

Negative profit occurs in **81,378 rows** and negative net assets in **3,192 rows**. These are valid losses/negative equity within the synthetic construction, not automatic data errors.

## Univariate EDA: Distributions

Training-only summaries:

| Variable | Mean | Median | 95th percentile |
|---|---:|---:|---:|
| Total revenue, EUR | 31,255.68 | 19,934.24 | 98,355.28 |
| Profit before tax, EUR | 10,397.39 | 5,744.68 | 45,566.90 |
| Cash balance, EUR | 22,616.31 | 13,138.53 | 78,467.09 |
| Liquidity coverage, months | 13.19 | 7.10 | 47.26 |

Revenues, cash, and coverage are right-skewed. Cash is zero in 21,500 development records (20.48%). Profit spans positive and negative values, so plain log transformation is inappropriate for that field.

The charts use log1p for nonnegative amounts and signed log1p(abs(x)) for profit. These are visualization transformations, not claims that the original variables are normally distributed. Classification and regression do not generally require every predictor to be normal.

![Training distributions](figures/02_distributions_train.png)

IQR fences flag 8,584 revenue observations for review, 8.18% of development rows. These are distributional flags, not proven errors. No records were deleted or winsorized automatically. Any future capping thresholds must be chosen on training data and applied consistently.

## Bivariate and Multivariate EDA

Pearson correlation describes linear association. Spearman correlation describes monotonic rank association and is useful when amounts are skewed or nonlinear relationships are plausible.

In development data:

- Revenue and profit: Pearson **0.778**, Spearman **0.860**.
- Operating revenue and total revenue: Pearson **0.996**.
- Taxable profit and tax assessed: Pearson approximately **1.000**, because the latter is calculated using a constant rate.

These relationships partly reflect accounting identities. They are not independent discoveries about firm behaviour.

For **next-year lateness**, training Spearman associations include:

| Current-year predictor | Spearman association |
|---|---:|
| Cash balance | -0.328 |
| Liquidity coverage | -0.313 |
| Profit before tax | -0.302 |
| Maximum days past due | +0.281 |

For the **next-year outstanding amount**, current closing tax debt has Spearman association **+0.253**. Associations are descriptive; they do not establish causation or guarantee multivariable predictive performance.

Repeated company observations also mean that ordinary row-independent significance tests would be misleading. No causal claims or naive correlation p-values are reported.

![Training correlations](figures/03_correlations_train.png)

## Target Distributions and Baselines

### Will the Taxpayer Pay Late?

In development data, **31,954 of 105,000 observations (30.43%)** have at least one late instalment next year.

The untouched-company 2024 test subset has **2,302 late outcomes out of 7,500 (30.69%)**.

| Test baseline | Accuracy | Precision for late | Recall for late | F1 for late |
|---|---:|---:|---:|---:|
| Predict on time for everyone | 69.31% | 0.00% | 0.00% | 0.000 |
| Repeat previous-year lateness | 69.87% | 50.91% | 51.30% | 0.511 |

Accuracy alone hides failure to identify late payers. Future probability models should report average precision/PR-AUC, calibration, precision and recall at an operationally chosen threshold, and the workload implied by that threshold.

### How Much Will Remain Unpaid?

Only **15.81%** of development observations have positive next-year closing tax debt. The training median is **EUR 0**, the mean **EUR 66.48**, and the 95th percentile **EUR 331.43**.

In test data, 1,088 of 7,500 next-year balances are positive. Predicting zero everywhere has **MAE EUR 59.52** and **RMSE EUR 313.29**. Carrying forward the current debt has **MAE EUR 106.54** and **RMSE EUR 445.34**.

The zero baseline is hard to beat under aggregate MAE because most outcomes are zero; it is not necessarily useful for identifying large unpaid exposures. Evaluate positive-balance recall, conditional MAE on positive balances, and portfolio-level error. Consider a two-part model for probability of a positive balance and amount conditional on a positive balance.

Across all 200,000 observable next-year records, **30,346 cases have late payments but zero closing tax debt**. In the test subset, this affects **1,214 of 2,302 late outcomes**. Do not equate the late-payment indicator with year-end debt.

![Training business targets](figures/07_business_targets_train.png)

### What Is the Liquidity Tendency?

The stored forecast categories include 50,000 first-year insufficient-history values. That category is an availability status, not a financial risk class.

![Stored forecast categories](figures/04_forecast_classes.png)

The existing trend rule was compared with later observed coverage values inside the synthetic panel. For test companies, feature year 2024 predicts observed proxy values in 2025:

| Metric | Persistence / stable baseline | Stored trend rule |
|---|---:|---:|
| MAE, months | 2.933 | 2.778 |
| RMSE, months | 5.920 | 6.063 |
| Direction accuracy | 49.32% | 62.29% |
| Direction macro-F1 | 0.220 | 0.582 |

Persistence predicts next-year coverage equal to current coverage; the direction baseline always predicts stable.

The trend rule reduces MAE by approximately **5.3%**, but its RMSE is worse. It improves typical absolute error while remaining vulnerable to extreme misses. No uncertainty intervals have been estimated.

![Liquidity backtest](figures/05_forecast_backtest.png)

Full [backtest results](tables/forecast_backtest.csv) include 150,000 eligible company-year forecasts from 2022–2024. Results are within this synthetic panel, not external validation.

## Feature Engineering and Standardization

Keep candidate inputs that exist by the end of the prediction year: financial components, loan repayments, payment history, cash coverage, and clearly defined debt measures.

Potential engineered features include profit margin, cash-to-expense coverage, prior-year changes, and lagged payment behaviour. Calculate lags within company after sorting by year. Undefined denominators need explicit handling; do not silently divide by one.

Exclude:

- company_id and company_name: arbitrary identifiers;
- company_split: evaluation metadata;
- entity_type and assumed_tax_rate: constants;
- all forecast_ fields: outputs of the existing projection;
- next-year outcomes: target values, not input features.

For an initial linear model, choose financial components or totals thoughtfully. For example, using every revenue component and total revenue together introduces exact dependence.

The executed preprocessing demonstration uses selected nonnegative candidates, training-only medians, log1p, and z-score standardization:

```text
z = (log1p(x) - training_mean) / training_population_standard_deviation
```

The saved [scaling table](tables/scaling_train_only.csv) verifies the fitted parameters and transformed training means/standard deviations. The same fitted parameters must be reused on validation, test, and future data.

Signed profit or net-assets features need a different treatment, such as signed log1p or robust scaling. Binary flags should generally remain binary. Scaling is useful for distance-based and regularized linear models; tree models usually do not require it. Scaling does not remove outliers, establish causality, or fix target leakage. See the official [scikit-learn preprocessing guide](https://scikit-learn.org/stable/modules/preprocessing.html).

## Conclusions and Modeling Plan

The workbook is internally consistent and suitable for a synthetic capstone. The three business questions should remain separate tasks, with shared company-year features:

1. **Lateness classification:** begin with logistic regression and a tree-based alternative; compare against the previous-lateness baseline.
2. **Outstanding-amount regression:** compare zero, persistence, and a two-part positive-balance/amount approach. Report both aggregate and positive-outcome errors.
3. **Liquidity classification or regression:** predict the next observed proxy or its direction; compare against persistence and the current trend rule.

Use company-grouped, forward-in-time evaluation. Fit imputers, scalers, encoders, and any resampling within the training process only. Report task-specific metrics rather than a single overall score.

The missing payment schedule is a substantive limitation for the precise question “how much was paid late?” Adding that data is more important than increasing model complexity.

## Materials We Provide

| Material | Description | Link |
|---|---|---|
| README | Business framing, executed EDA, conclusions | [This document](README.md) |
| Python analysis | Recreates checks, tables, and figures from the workbook | [eda_capstone.py](eda_capstone.py) |
| Environment | Packages used for this analysis | [requirements.txt](requirements.txt) |
| Data dictionary | Column types, roles, units, missingness | [CSV](tables/data_dictionary.csv) |
| Data-quality evidence | All 72 checks | [CSV](tables/quality_checks.csv) |
| Descriptive statistics | Full workbook and development subset | [Full](tables/descriptive_statistics_full.csv), [training](tables/descriptive_statistics_train.csv) |
| Baseline evidence | Payment, amount, and liquidity comparisons | [Payment/amount](tables/three_task_baselines.csv), [liquidity](tables/forecast_backtest.csv) |
| Analysis metadata | Source fingerprint and computed results | [JSON](tables/analysis_summary.json) |
| Figures | Seven static EDA charts | [Directory](figures/) |

The input workbook is intentionally not duplicated in this package. No slides or notebooks are claimed to be included.

## Prerequisites and Reproduction

Use Python 3.12 with pandas, NumPy, openpyxl, and Matplotlib. Familiarity with dataframes, grouping, summary statistics, and basic classification/regression is recommended. scikit-learn is relevant for the next modeling stage, but is not required to run this EDA.

From the extracted package folder:

```bash
python -m pip install -r requirements.txt
python eda_capstone.py --input "/path/to/companies_2021_2025_liquidity_en=capstone.xlsx" --output .
```

In Google Colab, upload the workbook and eda_capstone.py, then run:

```python
!pip -q install -r requirements.txt
!python eda_capstone.py --input "/content/companies_2021_2025_liquidity_en=capstone.xlsx" --output "/content/capstone_eda"
```

Upload requirements.txt too if it is not already in your Colab session. The script reads the workbook, produces the tables and figures, and leaves the source unchanged. README.md is the authored interpretation of this audited version; review it when rerunning on changed data.

## References

- [GA curriculum: EDA for Modeling](https://github.com/ga-curriculum/eda-for-modeling) — structure and learning objectives supplied for this project.
- [pandas: read_excel](https://pandas.pydata.org/docs/reference/api/pandas.read_excel.html) — workbook loading.
- [scikit-learn: preprocessing](https://scikit-learn.org/stable/modules/preprocessing.html) — scaling and transformations.
- [scikit-learn: common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html) — leakage and consistent preprocessing.

## Important Notes

This project uses synthetic corporate data, inherited synthetic payment behaviour, a flat assumed tax rate, and a simplified liquidity proxy. It has no transaction-level evidence of overdue amounts and no 2026 outcomes. It does not contain a trained production model or calibrated probability forecasts.

The source workbook was not modified. Existing removed future-target columns were not reintroduced into the input data. Analysis targets are created in memory for evaluation only. Correlation does not establish causation, and internal consistency does not establish external validity.
