# Methodology

All formulas below are implemented as live Excel formulas in `workbook/ExperimentLab.xlsx`. Legacy function names (`NORMSDIST`, `NORMSINV`, `CHITEST`, `CHIDIST`) are used so the file works in any Excel version and in LibreOffice; they are equivalent to `NORM.S.DIST`, `NORM.S.INV`, `CHISQ.TEST` and `CHISQ.DIST.RT`.

## 1. Outcome and groups
- Outcome: `converted` (1 = conversion, 0 = no conversion).
- Test group: `ad`. Comparison group: `psa`.
- Conversion rate = conversions / users.

## 2. Effect size
- Absolute difference: `p_ad - p_psa` = 0.007692 (0.769 percentage points).
- Relative lift: `(p_ad - p_psa) / p_psa` = 43.09%.
- Cohen's h: `2*(ASIN(SQRT(p_ad)) - ASIN(SQRT(p_psa)))` = 0.053.
- Incremental conversions (only meaningful if causal): `(p_ad - p_psa) * n_ad` = 4,343.

## 3. Two-proportion z-test
Null: `p_ad = p_psa`. Alternative: `p_ad <> p_psa` (two-sided). Alpha = 0.05.

```
p_pool = (x_ad + x_psa) / (n_ad + n_psa)
SE     = SQRT(p_pool * (1 - p_pool) * (1/n_ad + 1/n_psa))
z      = (p_ad - p_psa) / SE
p      = 2 * NORMSDIST(-ABS(z))
```

`2*NORMSDIST(-ABS(z))` is used rather than `2*(1-NORMSDIST(ABS(z)))` because the latter loses precision for very small p-values.

Result: z = 7.370078, p = 1.70528e-13.

## 4. Confidence intervals
Difference (unpooled SE):

```
SE_diff = SQRT(p_ad*(1-p_ad)/n_ad + p_psa*(1-p_psa)/n_psa)
CI      = (p_ad - p_psa) ± NORMSINV(0.975) * SE_diff
```

Result: [0.595%, 0.943%] (percentage points).

Relative lift (delta method on the log risk ratio):

```
RR       = p_ad / p_psa
SE_lnRR  = SQRT((1-p_ad)/x_ad + (1-p_psa)/x_psa)
CI(lift) = EXP(LN(RR) ± z_crit * SE_lnRR) - 1
```

Result: [30.0%, 57.5%].

## 5. Sensitivity and power
- Minimum detectable effect at 80% power: `(z_crit + NORMSINV(0.80)) * SE` = 0.292 pp. The observed effect (0.769 pp) is well above this.
- Retrospective power at the observed effect is shown for completeness; it is effectively 100% and adds little information.

## 6. Segment analysis (exploratory)
For each segment (weekday, ad-frequency band) the workbook computes the difference, unpooled-SE confidence interval, z-test, and a **Bonferroni-adjusted p-value** (`MIN(1, p * number_of_tests)`). Segments are defined by post-exposure behaviour, so they describe who responds rather than randomised subgroups.

## 7. Frequency-standardised difference (Mantel-Haenszel risk difference)
Across the 7 ad-frequency bands, with `a = x_ad`, `c = x_psa`, `n1 = n_ad`, `n0 = n_psa`, `N = n1 + n0`:

```
RD_MH  = SUM( (a*n0 - c*n1) / N ) / SUM( n1*n0 / N )
Var    = SUM( (a*(n1-a)*n0^3 + c*(n0-c)*n1^3) / (n1*n0*N^2) ) / SUM(n1*n0/N)^2    (Greenland-Robins)
```

Result: 0.819 pp (95% CI 0.648 to 0.989). This checks that differences in exposure mix do not explain the lift. Because ads seen is a post-exposure variable, it is a sensitivity check, not a causal adjustment.

## 8. Balance check
Chi-square tests of independence (group x ad-frequency band; group x weekday) using `CHITEST` on observed vs expected counts. With ~588k users the tests flag small gaps, so the workbook also reports the largest share gap in percentage points. Result: gaps of up to 4.2 pp per band, which warrants verifying the assignment mechanism.

## 9. Break-even analysis
```
cost                   = ad_impressions / 1000 * CPM
cost per incremental conversion = cost / incremental_conversions
break-even value       = cost per incremental conversion
net profit             = incremental_conversions * value_per_conversion - cost
```

CPM and value per conversion are **placeholder assumptions** (yellow cells); the dataset contains neither. The model assumes the comparison group is not billed; if PSA impressions were also purchased, incremental cost is lower.

## 10. Data-quality checks
Rows 588,101 · unique user IDs 588,101 · duplicate IDs 0 · missing cells 0 · unexpected group labels 0 · invalid conversion values 0 · negative or zero `total_ads` 0 · invalid hours 0 · invalid weekday labels 0. The workbook also reconciles every segment table to the group totals with live formulas, and `workbook/ExperimentLab_Verification.xlsx` reproduces every check and every input from the raw rows with `COUNTIFS` / `SUMIFS` / `SUMPRODUCT`.

## 11. Cross-check: chi-square test on the 2 x 2 table
Observed counts (converted / not converted by group) against expected counts under independence:

```
E_ij = row_total_i * column_total_j / grand_total
chi2 = SUM( (O_ij - E_ij)^2 / E_ij )          df = 1
p    = CHIDIST(chi2, 1)
```

For a 2 x 2 table, chi2 equals the squared pooled z statistic, so this is an independent check of the headline test. Result: chi2 = 54.318 = 7.370^2, p = 1.705e-13 (`Power_Planner`, section A).

## 12. Sample size for a follow-up test
For baseline rate `p1`, target rate `p2 = p1 + delta`, equal arms, two-sided alpha and power `1 - beta`:

```
p_bar = (p1 + p2) / 2
n     = ( z_(1-alpha/2) * SQRT(2 * p_bar * (1 - p_bar)) + z_(1-beta) * SQRT(p1*(1-p1) + p2*(1-p2)) )^2 / delta^2     (per arm)
```

With allocation ratio `k = n_treatment / n_control` the control size becomes
`( z_a * SQRT((1 + 1/k) * p_bar_k * (1 - p_bar_k)) + z_b * SQRT(p1*(1-p1) + p2*(1-p2)/k) )^2 / delta^2`, where `p_bar_k = (p1 + k*p2) / (1 + k)`, and the treatment arm is `k` times larger. Results at the PSA baseline (1.785%): 5,631 users per arm to replicate the observed 0.769 pp lift; 12,520 per arm to detect 0.5 pp.

## 13. Interpretation cautions
- Statistical significance does not establish commercial significance.
- Randomisation is not documented, and exposure profiles differ between groups, so the result is evidence of association, not proof of causal impact.
- The large-sample normal approximation is appropriate for the overall test; segment cells with very few conversions (for example 3 PSA conversions in the 1-ad band) are unreliable.
- Segment results are exploratory.
