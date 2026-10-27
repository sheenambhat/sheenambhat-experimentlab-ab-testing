# ExperimentLab: Conversion \& A/B Testing

**Did an advertising treatment increase conversion enough to justify continued investment?**
An A/B test analysis of 588,101 users built with **Excel and statistics only** (no code): effect size, statistical uncertainty, segment diagnostics, sample-size planning, and the business break-even, with the limits of the evidence stated up front.

!\[Conversion rate by group](docs/images/conversion\_by\_group.png)

## Business problem

A growth team ran an advertising campaign and kept a public-service-announcement (PSA) comparison group. Before deciding whether to keep investing, they need to know:

1. How large is the improvement in conversion?
2. How certain is that result?
3. What does it mean for the business, and what is still unknown?

## Tools

|||
|-|-|
|**Excel**|Live formulas (`COUNTIFS`, `SUMIFS`, `AVERAGEIFS`, `NORMSINV`, `NORMSDIST`, `CHIDIST`, `CHITEST`, `SUMPRODUCT`), native charts, conditional formatting|
|**Statistics**|Two-proportion z-test, confidence intervals, relative-risk (delta method), Cohen's h, power and sample size, Bonferroni correction, Mantel-Haenszel standardisation, chi-square tests|

No macros, scripts, or add-ins. Every number in the workbook is a visible formula that traces back to the `Inputs` sheet.

## Headline results

|Metric|Ad group|PSA group|
|-|-|-|
|Users|564,577|23,524|
|Conversions|14,423|420|
|Conversion rate|**2.555%**|**1.785%**|

|Effect|Value|
|-|-|
|Absolute difference (Ad − PSA)|**+0.769 percentage points**|
|95% CI for the difference|**\[0.595, 0.943] pp**|
|Relative lift|**+43.1%** (95% CI 30.0% to 57.5%)|
|Two-proportion z-test|z = 7.370, **p = 1.71 × 10⁻¹³**|
|Independent check: 2 × 2 chi-square|χ² = 54.318 = z², same p-value|
|Effect size (Cohen's h)|0.053 (small; the base rate is low)|
|Incremental conversions, *if causal*|≈ 4,343 (95% CI 3,360 to 5,326), about 30% of ad-group conversions|

The ad group converts at a higher rate and the difference is statistically robust. Statistical significance is not profitability, and the dataset cannot prove causation. See [Limitations](#limitations).

## What the segment analysis adds

### 1\. The lift appears only at high exposure

!\[Conversion rate by ads seen](docs/images/lift\_by\_ad\_frequency.png)

For users who saw 1 to 20 ads there is no significant difference (the PSA group is nominally *higher* at 2 to 10 ads). The lift comes from users who saw **21+ ads**, and 51+ ads is where it is largest (about +5 to +6 pp). All three of those bands remain significant after Bonferroni correction.

**Implication:** the headline 0.77 pp average hides a very uneven effect. Frequency caps or targeting tests are the obvious next experiment.

### 2\. Adjusting for exposure mix does not shrink the lift

A frequency-standardised (Mantel-Haenszel) comparison gives **+0.819 pp** (95% CI 0.648 to 0.989) versus 0.769 pp crude, so the result is not an artefact of one group simply seeing more ads. This is a sensitivity check, not a causal adjustment, because ads seen is measured *after* exposure.

### 3\. The groups do not have identical exposure profiles

!\[Exposure balance](docs/images/exposure\_balance.png)

Average ads per user is almost identical (24.8 vs 24.8), but the distributions differ: for example the 11-20 band is 21.8% of the ad group but 17.6% of the PSA group, and 2-5 is 20.5% vs 23.6%. With a group this size those gaps are far beyond chance (chi-square p ≈ 10⁻⁶²). They could be a delivery effect of the treatment *or* a sign that assignment was not purely random; the data cannot tell which. This is the main reason the recommendation says "verify the experiment design first."

### 4\. Day-of-week is exploratory

Monday, Tuesday and Wednesday show significant differences after Bonferroni correction; Thursday to Sunday do not. With 7 tests and a small comparison group, treat this as hypothesis-generating only.

## Planning the follow-up test

The `Power\_Planner` sheet turns the recommendation into numbers, using the standard two-proportion sample-size formula:

* To **replicate** the observed 0.769 pp lift at the PSA baseline (1.785%) with 80% power, an even split needs about **5,600 users per arm**.
* To detect a smaller **0.5 pp** lift at that baseline, it needs about **12,500 users per arm** (about 25,000 in total).
* Halving the lift to detect roughly quadruples the users required. The sheet includes a lift-by-baseline grid and a calculator with an adjustable allocation split.

## Business implication

The dataset has no spend or conversion value, so ROI **cannot** be calculated from it. The workbook instead provides a break-even framework with clearly marked placeholder assumptions:

!\[Break-even sensitivity](docs/images/break\_even\_sensitivity.png)

At a hypothetical $5 CPM, the campaign costs about $70k for 14.0M impressions, so each incremental conversion must be worth more than **$16.13** to pay back (range $13.16 to $20.86 across the confidence interval). Replace the yellow cells in the `Break\_Even` sheet with real figures to get a real answer.

## Recommendation

**Continue conditionally**, and do not scale the budget yet:

1. Verify how users were assigned to groups and what the PSA group actually saw (exposure profiles differ).
2. Replace the placeholder CPM and conversion value with real numbers and confirm the break-even is cleared.
3. Test frequency caps and targeting, since low-frequency exposure shows no measurable benefit.
4. Replicate the result before a large budget increase.

Full write-up: [`docs/decision\_memo.md`](docs/decision_memo.md).

## Workbook guide

`workbook/ExperimentLab.xlsx` has 12 sheets and 531 live formulas (no pasted results). Blue font marks hardcoded inputs, black is formulas, green is cross-sheet links, yellow fill marks assumptions to replace.

|Sheet|Contents|
|-|-|
|`Start\_Here`|Purpose, headline results, sheet guide|
|`Inputs`|Every hardcoded number, with its source|
|`Data\_Quality`|Validation checks and live reconciliation of every table to the group totals|
|`Summary`|Rates, absolute difference, relative lift, Cohen's h, incremental conversions|
|`Statistical\_Test`|Two-proportion z-test, CIs for the difference and for relative lift, minimum detectable effect, power|
|`Break\_Even`|Illustrative ROI, break-even value per conversion, profit sensitivity grid|
|`Segment\_Day` / `Segment\_Frequency`|Per-segment tests with Bonferroni-adjusted p-values; Mantel-Haenszel standardised difference|
|`Balance\_Check`|Chi-square tests of exposure balance between groups|
|`Power\_Planner`|Chi-square cross-check of the headline test; sample size for a follow-up experiment|
|`Charts`|Five native Excel charts|
|`Decision\_Memo`|Stakeholder recommendation, driven by the numbers above|

`workbook/ExperimentLab\_Verification.xlsx` holds the raw 588,101 rows and recomputes every hardcoded input with formulas (see below).

## Method

* **Outcome:** `converted` (0/1). **Comparison:** `ad` vs `psa`.
* **Test:** two-proportion z-test with pooled standard error; 95% CI for the difference using the unpooled standard error; relative-lift CI via the log risk ratio (delta method). Cross-checked with a 2 × 2 chi-square test.
* **Multiple comparisons:** Bonferroni correction within each segment table.
* **Sensitivity checks:** Mantel-Haenszel standardisation across ad-frequency bands; chi-square balance tests.
* **Planning:** minimum detectable effect, power, and required sample size.
* **Formulas:** see [`docs/methodology.md`](docs/methodology.md).

## Data

Marketing A/B Testing dataset, 588,101 rows, six fields (see [`docs/data\_dictionary.md`](docs/data_dictionary.md)). It is a publicly circulated Kaggle dataset; check the original source page for its licence before redistributing the CSV.

Data quality: 0 duplicate IDs, 0 missing cells, 0 invalid values; no rows removed. Only the index-like `Unnamed: 0` column was dropped.

## Reproduce (Excel only)

1. Open `workbook/ExperimentLab.xlsx` and read `Start\_Here`.
2. To confirm the hardcoded counts against the raw data, open `workbook/ExperimentLab\_Verification.xlsx`. The `Data` sheet is the raw CSV (sorted by `user\_id`); the `Verify` sheet recomputes group totals, impressions, means and medians, the data-quality checks (duplicates, blanks, invalid labels and values), and both segment tables with `COUNTIFS` / `SUMIFS`, then compares each to the value reported in the analysis workbook. Cell B3 reads **ALL CHECKS PASSED** when everything matches. Excel may take a few seconds to calculate because each formula scans 588,101 rows.
3. Alternatively, import `data/marketing\_ab\_clean.csv` into Excel and build a PivotTable (rows: `test\_group`; values: count of `user\_id`, sum of `converted`).

## Limitations

* **Association, not proven causation.** The columns do not document randomisation, and the groups' exposure profiles differ.
* **Unequal groups.** The PSA group is 4% of users (23,524), so segment estimates for it are noisy. Some cells have fewer than 10 conversions.
* **Post-exposure segments.** `total\_ads` and `most\_ads\_day` are measured after the ad is served, so segments describe who responds, not randomised subgroups.
* **No cost or value data.** ROI is a framework with placeholder inputs, not a result.
* **Unknowns:** campaign dates, conversion window, and what the PSA group was shown.

## Repository structure

```
experimentlab-ab-testing/
├── README.md
├── LICENSE
├── data/
│   └── marketing\_ab\_clean.csv
├── workbook/
│   ├── ExperimentLab.xlsx
│   └── ExperimentLab\_Verification.xlsx
└── docs/
    ├── methodology.md
    ├── data\_dictionary.md
    ├── decision\_memo.md
    ├── portfolio\_talking\_points.md
    └── images/
```

## Skills demonstrated

A/B test design and analysis · two-proportion hypothesis testing · confidence intervals · effect size, power and sample-size planning · multiple-comparison correction · stratified (Mantel-Haenszel) analysis · chi-square tests · data validation · business framing and break-even analysis · Excel modelling with traceable formulas · honest communication of limitations

\---

*Author: SHEENAM BHAT LinkedIn:www.linkedin.com/in/sheenam-bhat-2b0017215*

