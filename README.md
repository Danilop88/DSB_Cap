# Predicting Next-Quarter Revenue and Profit Margin of U.S. Companies

*Using only what a company has already reported, can we estimate its revenue and profit margin for the next quarter?*

## The problem

Analysts and finance teams often need an estimate of a company's next quarter before the official report is published. This project tests whether a company's latest quarterly figures (revenue, profit, cash flow, assets, equity, cash) are enough to predict two things: next-quarter **revenue** and next-quarter **profit margin** (net income divided by revenue). A useful model must beat the simplest rule of thumb: "next quarter will look like this quarter."

## Key result

**Revenue is easy to predict; profit margin is not.** On the training data, this quarter's revenue and next quarter's revenue are almost perfectly correlated (r = 0.99, on a log scale), while for profit margin the correlation is only 0.60. In plain terms: a company's revenue barely moves from one quarter to the next, but its profit swings much more. The real challenge, and the benchmark every model must beat, is the "same as last quarter" baseline, especially for margin.

## The data

- **Source:** [SEC EDGAR Company Facts API](https://www.sec.gov/search-filings/edgar-application-programming-interfaces), quarterly figures from 10-Q and 10-K filings. Public U.S. government data.
- **Coverage:** 400 U.S.-listed companies (selected from an S&P 500 snapshot), quarters from 2018 to mid-2026; file built on 9 November 2026.
- **Size:** 13,966 company-quarters and 66 columns in the raw file (`raw_data_400.csv`); **11,682 rows and 357 companies** after cleaning (`clean final.csv`).
- **In one sentence:** each row shows how much one company sold, earned and held in one quarter.

**Main cleaning decisions:**
- **Banks, brokers and insurers (30 companies) are excluded.** Their "revenue" means something different (interest and investment income), and in this download it is often missing or wrongly tagged. The model applies to **non-financial companies only**.
- Revenue stored as text was converted to numbers; 33 zero or negative revenues were set to missing.
- 403 quarters with margins outside -100% to +100% were set to missing (mostly bad revenue tags, but also real crises such as cruise lines in 2020–21, so the model has not seen the most extreme quarters).
- Split **by time** before any analysis: train = quarters before 2025 (9,627 rows), test = 2025 onwards (2,055 rows). All EDA uses train only.

How to obtain the data: the raw file `raw_data_400.csv` is included in this repository; the cleaned file `clean final.csv` is produced by the EDA notebook.

## How to navigate this repo

Start here, then read the [EDA notebook](01_EDA_clean%20final.ipynb) (data checks, cleaning, findings). The raw data is in [`raw_data_400.csv`](raw_data_400.csv) and the cleaned data, exported after EDA, is in [`clean final.csv`](clean%20final.csv).

## How to reproduce

The raw data file `raw_data_400.csv` is small enough to be included in the repository root; no account or API key is needed.

```bash
git clone https://github.com/Danilop88/DSB_Cap.git
cd DSB_Cap
python -m pip install pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook
```

Open `01_EDA_clean final.ipynb` and run it from a fresh session: it reads the raw `raw_data_400.csv` and exports the cleaned `clean final.csv`. In Colab, first clone the repository, enter its folder and install the packages in a code cell (`!git clone ...`, `%cd DSB_Cap`, `%pip install pandas numpy matplotlib seaborn scikit-learn`). Opening the badge alone does not copy the repository or its data.

[![Open EDA in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Danilop88/DSB_Cap/blob/main/01_EDA_clean%20final.ipynb)

## Repository structure

```text
README.md                    Public project story
01_EDA_clean final.ipynb     EDA notebook: explore, clean, document decisions, export
raw_data_400.csv             Raw quarterly data from SEC EDGAR (untouched)
clean final.csv              Clean data, exported after EDA
```

## Author & credits

Danilo Pavicevic, General Assembly Data Science Bootcamp, Capstone project.
Data: U.S. Securities and Exchange Commission (SEC), EDGAR Company Facts. Libraries: pandas, NumPy, matplotlib, seaborn, scikit-learn.

## License

Code: MIT License, © 2026 Danilo Pavicevic.
Data: SEC EDGAR data is public U.S. government information; use follows the [SEC access rules](https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data).

