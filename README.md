# Quarterly Financial Data: A Simple EDA

## Project overview

This capstone project is an exploratory data analysis (EDA) of a dataset
containing quarterly financial information for public companies. It was created
to practice the first steps of a data science project: loading a real dataset,
checking its quality, cleaning selected fields, summarizing the results, and
communicating observations with charts.

The analysis is descriptive. It looks for patterns in the available numbers,
but does not try to explain what caused them.

## Research question

**How do reported revenue, net income, and free cash flow vary over time in this
dataset?**

To explore this question, the notebook calculates yearly median values for the
three measures. It also shows quarterly revenue and net income for Apple
(AAPL), when those records are present in the dataset.

## Project contents

| Path | Description |
| --- | --- |
| `data/raw/quarterly_financial_data.csv` | Copy of the CSV dataset provided for this project. |
| `EDA/quarterly_financial_EDA.ipynb` | Notebook with data loading, basic cleaning, quality checks, summary statistics, charts, findings, limitations, and a five-minute presentation outline. |
| `requirements.txt` | Python packages used by the notebook. |

## About the dataset

The provided CSV contains **13,966 rows and 66 columns**, covering **400
company tickers**. Each row represents quarterly financial information for a
company. The columns include company identifiers, reporting periods, revenue,
income, assets, liabilities, cash-flow measures, and other financial fields.
The period-end dates in this file range from **2018-01-26 to 2026-10-04**.

The original source, collection method, and definitions for every field were
not included with the file. This project therefore describes the dataset as
provided and does not claim independent verification of its contents.

## Data preparation

The notebook applies a few targeted cleaning steps:

1. Removes leading and trailing spaces from column names.
2. Converts revenue from formatted text to numeric values.
3. Interprets values in parentheses as negative numbers and a dash as a missing
   value.
4. Converts the period start and end columns to date values.
5. Checks the number of exact duplicate rows and reports the columns with the
   most missing data.

Rows are not removed, and missing values in the other columns are not filled
in. This keeps the original records intact while making the selected fields
easier to inspect and analyze.

## Preliminary observations

The notebook calculates yearly medians for revenue, net income, and free cash
flow and presents these measures in billions of dollars. In the provided data,
each of the three median values is higher in 2026 than in 2018. This is a
description of the values in those years, not evidence that the measures
increased steadily or an explanation for the difference.

The dataset includes 2026 records only through October. The companies included
may also differ from year to year, so yearly comparisons should be interpreted
carefully. Run the notebook to see the summary tables, charts, and exact
results.

## Requirements

- Python 3
- pandas
- matplotlib
- Jupyter support in VS Code or another notebook environment

## How to run

1. Download or clone the project folder.
2. Open the project folder in VS Code.
3. Install the required packages in a terminal:

   ```bash
   python -m pip install -r requirements.txt
   ```

4. Open `EDA/quarterly_financial_EDA.ipynb`.
5. Select a Python environment with the required packages installed.
6. Run the notebook cells from top to bottom.

The notebook looks for the CSV in the project's `data/raw` folder. Keep the
data file in that location so the notebook can load it.

## Five-minute presentation

The final section of the notebook includes a short presentation outline. It
covers the research question, the dataset, the cleaning steps, the main chart,
the Apple example, and the main limitations. Use the results and charts
produced by the notebook when presenting.

## Limitations

- The original data source and the definitions of all columns were not
  supplied with the CSV.
- Many fields contain missing values. Different analyses may therefore use
  different numbers of records.
- The set of companies may change over time, and 2026 does not include a full
  calendar year.
- Company size and industry can vary considerably, so the median across
  companies is only a simple overview.
- The analysis identifies patterns but does not establish cause and effect.
- This project is for learning and is not investment advice.