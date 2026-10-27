# Data dictionary

`data/marketing_ab_clean.csv`: 588,101 rows, 6 columns.

| Field | Type | Meaning |
|---|---|---|
| `user_id` | Integer | Unique user identifier (900,000 to 1,654,483) |
| `test_group` | Category | `ad` for the advertising group; `psa` for the public-service-announcement comparison group |
| `converted` | Integer 0/1 | Whether the user converted |
| `total_ads` | Integer | Number of ads seen by the user (1 to 2,065) |
| `most_ads_day` | Category | Weekday on which the user saw the most ads |
| `most_ads_hour` | Integer 0-23 | Hour of day when the user saw the most ads |

## Derived grouping used in the analysis

Ad-frequency bands on `total_ads`: `1`, `2-5`, `6-10`, `11-20`, `21-50`, `51-100`, `101+`.

## Cleaning

The original CSV also included `Unnamed: 0`, an index-like row-number field. It was excluded because it is not a business feature. No rows were removed, and `converted` is stored as 0/1.

## Notes

- `total_ads`, `most_ads_day` and `most_ads_hour` describe behaviour *after* the ad was served, so they are not clean pre-treatment covariates.
- The dataset does not include campaign dates, spend, conversion value, or the assignment mechanism.
