# Forecasting Business and Financial Metrics

This Data Science Capstone Part 1 pitch proposes three research questions about U.S. public companies. All three use quarterly financial statements, with a different outcome for each question.

## Question 1 — How will revenue change over the next four quarters?

- **Problem statement** — Can recent financial results predict the amount and direction of year-over-year revenue change for each of the next four fiscal quarters?
- **Audience** — Financial planning teams and equity research analysts who need a baseline forecast.
- **Goal** — Build forecasts one to four quarters ahead for a subset of companies with complete revenue histories. Use lagged revenue and year-over-year growth as inputs. Use revenue from the same fiscal quarter in the previous year as the baseline.
- **Success metrics** — Report median absolute percentage error and directional accuracy separately for each forecast horizon on held-out periods. Compare both with the seasonal baseline, which predicts flat year-over-year revenue. Report the number of zero or missing revenue observations excluded from percentage errors.
- **Data source** — Public SEC EDGAR Company Facts JSON, using quarterly revenue from 10-Q and 10-K disclosures. Start with a subset of the 400-company panel and verify that each issuer's revenue definition is consistent across periods.

## Question 2 — Which companies are at risk of weakening financial stability?

- **Problem statement** — Can liquidity, leverage, and cash flow ratios identify nonfinancial companies likely to show both negative trailing four-quarter operating cash flow and a current ratio below 1 within the next four quarters? This is a measurable proxy for financial strain.
- **Audience** — Credit analysts and finance teams deciding which companies warrant closer review.
- **Goal** — Build a risk ranking using current assets divided by current liabilities, debt relative to assets, and trailing operating cash flow relative to assets. Focus on companies that do not meet both strain conditions at the forecast date. Check outcome frequency before training and narrow the scope if too few cases are available for evaluation.
- **Success metrics** — Compare average precision with a baseline that ranks companies by current ratio alone. Report precision and recall for the highest-risk 10% in held-out quarters. Evaluate only forecasts with four observable follow-up quarters.
- **Data source** — Public SEC EDGAR Company Facts JSON for operating cash flow, current assets, current liabilities, total assets, and debt. Derive standalone cash flow quarters from cumulative figures only when the periods, accounting concepts, and units match. Retain both source filings.

## Question 3 — Will operating profitability improve or decline next quarter?

- **Problem statement** — Can past margins and changes in revenue and operating costs predict next-quarter operating margin better than using the margin from the same quarter last year?
- **Audience** — Corporate planning teams and analysts tracking whether sales growth translates into operating profit.
- **Goal** — Forecast operating margin, defined as operating income divided by revenue, for companies in one sector. Use historical margins, revenue growth, and operating expense ratios where consistently reported. Restrict the analysis to quarters with positive revenue.
- **Success metrics** — Compare mean absolute forecast error in percentage points against the same-quarter-last-year baseline. Report how often the model correctly predicts whether margin rises, falls, or stays flat year over year on held-out periods.
- **Data source** — Public SEC EDGAR Company Facts JSON for operating income, revenue, and operating expenses. Use public SEC industry classifications to select a sector with adequate coverage.

## Data access and project boundaries

The working panel covers 400 U.S.-listed companies selected from an S&P 500 snapshot, including Dow Jones members and Nasdaq-listed companies. These groups overlap. The panel starts in 2018 and includes 2026 filings available through November 9, 2026. Use actual company fiscal periods. Treat 2026 as partial coverage, and record missing disclosures as missing values. The sample favors companies with longer filing histories, so the findings apply to the selected cohort.

SEC Company Facts is available as public JSON without an API key. Retrieve a company's data using its SEC Central Index Key (CIK), or use the prepared CSV for exploration. Automated requests must follow SEC access rules. The project uses public corporate financial figures only. Share the project's code, aggregate results, and links to the public sources. Before redistributing any third-party constituent list, verify its terms or omit it from the submission.

Use the prepared panel to explore coverage. For all three questions, rebuild the modeling inputs from source facts using filing dates so each forecast uses information public at the time. Split chronologically, keep a final test period separate, and compare models and baselines on the same observations. For the four-quarter stability outcome, leave enough time between training and testing for all training outcomes to be observable before the test begins.

- [SEC EDGAR API documentation](https://www.sec.gov/search-filings/edgar-application-programming-interfaces)
- [Example SEC Company Facts JSON — Apple](https://data.sec.gov/api/xbrl/companyfacts/CIK0000320193.json)
- [SEC automated access guidance](https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data)
- [SEC industry classification codes](https://www.sec.gov/search-filings/standard-industrial-classification-sic-code-list)
- [Course data requirements](https://github.com/juanatga/ga_capstone_guideline_dsb/blob/main/README.md#important-no-proprietary-private-or-sensitive-personal-data)

