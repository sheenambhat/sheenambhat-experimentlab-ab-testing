# Decision memo: ExperimentLab

**To:** Growth team  **Re:** Should we continue investing in the advertising treatment?

## Decision question
Did the ad group convert at a different rate from the PSA comparison group, and is the improvement large enough to justify continued investment?

## Findings
- Ad group: 14,423 conversions of 564,577 users (**2.555%**).
- PSA group: 420 conversions of 23,524 users (**1.785%**).
- Absolute difference: **+0.769 percentage points** (95% CI 0.595 to 0.943).
- Relative lift: **+43.1%** (95% CI 30.0% to 57.5%).
- Two-sided z-test: z = 7.370, p = 1.71e-13. The result is robust to sampling noise.
- If the effect is causal, the ad produced about **4,343 incremental conversions** (about 30% of ad-group conversions).

## Where the lift comes from
- Below 21 ads seen there is **no significant difference** (the PSA group is nominally higher at 2-10 ads).
- The lift is concentrated at **21+ ads**, and is largest at 51+ ads (about +5 to +6 pp). These bands stay significant after Bonferroni correction.
- Standardising for ad-frequency band gives +0.819 pp, so the result is not explained by one group simply seeing more ads.
- Weekday effects (significant Mon to Wed, not Thu to Sun) are exploratory.

## Business implication
ROI cannot be calculated: the dataset has no campaign spend or value per conversion. The workbook's `Break_Even` sheet shows the economics under placeholder assumptions. At a hypothetical $5 CPM (about $70k for 14.0M impressions), each incremental conversion must be worth more than **$16.13** for the campaign to pay back (range $13.16 to $20.86 across the confidence interval).

## Recommendation
**Continue conditionally; do not scale the budget yet.**

1. **Verify the experiment design.** Confirm how users were assigned and what the PSA group saw. The groups' exposure profiles differ (for example the 11-20 ad band is 21.8% of the ad group vs 17.6% of PSA), which may be a delivery effect or a sign that assignment was not random.
2. **Add real economics.** Replace the placeholder CPM and conversion value; proceed only if value per conversion clears the break-even.
3. **Test frequency.** Low-frequency exposure shows no measurable benefit. Run a follow-up test with frequency-capped arms and pre-registered segments.
4. **Replicate** before a major budget shift. At an even split, replicating the observed 0.769 pp lift needs about 5,600 users per arm, and detecting a 0.5 pp lift needs about 12,500 per arm (`Power_Planner` sheet).

## What cannot be concluded
- **ROI or profit:** spend and value per conversion are not supplied.
- **Causal impact with certainty:** randomisation is undocumented and exposure profiles differ between groups.
- **Reliable subgroup winners:** segments are defined after exposure and are exploratory (Bonferroni-adjusted p-values are reported).
- **Effects at low frequency:** the PSA group has few users in these bands, so estimates are imprecise.
