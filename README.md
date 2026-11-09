# Quarterly Financial Data: A Simple EDA

## Project

This student project explores a dataset of quarterly financial information for
public companies. The goal is to practice loading, cleaning, and describing data
with Python, pandas, and simple charts.

## Research question

Can historical quarterly financial data be used to predict future revenue trends 
and identify early warning signals for negative financial trends using machine learning models?

## Project files

- `data/raw/quarterly_financial_data.csv` - the CSV file provided for this
  project.
- `EDA/quarterly_financial_EDA.ipynb` - the data cleaning, exploratory analysis,
  charts, and short presentation notes.

## Dataset

The provided CSV contains 13,966 rows and 66 columns. Each row represents a
company's quarterly financial data. The file includes dates, company tickers,
revenue, net income, cash-flow measures, and other financial fields.

The original source and collection method were not included with the provided
file. They should be added if that information becomes available.

## Basic cleaning

The notebook trims spaces from column names, converts revenue values to
numbers, treats values in parentheses as negative, treats a dash as missing,
and converts the period fields to dates. It reports missing values and exact
duplicate rows; it does not remove rows.

## Setup

Install the required packages:

```bash
python -m pip install -r requirements.txt
```

Open `EDA/quarterly_financial_EDA.ipynb` in VS Code and run the notebook cells
from top to bottom. If VS Code asks for a Python environment, select one that
has the packages from `requirements.txt` installed.

## Scope and limitations

This is a preliminary, descriptive analysis. It does not explain why company
financial results change and should not be used as investment advice. The file
contains missing values, and the original source and definitions of all fields
were not provided. Comparisons across companies and years should therefore be
treated with care.
