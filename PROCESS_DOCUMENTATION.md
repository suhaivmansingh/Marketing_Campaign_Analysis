# Process Documentation: Marketing Campaign Effectiveness Analysis

This document describes what was done, why, and what was found. The code is in `Marketing_Campaign_Analysis.ipynb`; section numbers below match the notebook.

## 1. Dataset overview

**Source:** The Complete Journey (84.51° Complete Journey study), loaded in the notebook with `pd.read_csv`. The R version of the dataset describes frequent shoppers at a retailer; the Python package's README calls the data simulated and educational. This is a learning project, so the findings describe this dataset only.

| Table | Rows | One row is | Used for |
|---|---|---|---|
| `campaign_descriptions` | 27 | a campaign | type (A, B, C), start and end date |
| `campaigns` | 6,589 | a household targeted by a campaign | who received what |
| `coupon_redemptions` | 2,102 | a redeemed coupon | the response |
| `coupons` | 116,204 | a coupon listed in a campaign | coupons offered (unique coupons only) |
| `transactions` | 1,469,307 | a purchased product line (1 Jan 2017 to 1 Jan 2018) | sales, discounts, baskets |
| `demographics` | 801 | a household | age, income, household make-up |

Not used: `products` and `promotions` (not needed to measure the campaigns).


## 2. Data exploration findings (notebook Step 1)

| Check | Result |
|---|---|
| Missing values | None in four tables. Demographics: 370 empty cells (`home_ownership` 233, `marital_status` 137), and only 756 of the 1,559 targeted households (48%) appear in the table at all. 2,754 of 6,589 household-campaign pairs (42%) have no demographics. |
| Duplicates | None, except 4,872 duplicate rows in `coupons` (harmless: only unique coupons are counted). |
| Zero or extreme values | 8,869 lines with quantity 0, 11,226 with sales value 0, 13,494 with quantity above 1,000 (maximum 89,638, mostly one product). Largest single line: $840. |
| Zero-quantity lines | They carry $6,544 of $30,903 total coupon discount (21%), so they are real coupon lines, not errors. |
| Redemption consistency | No redemption outside its campaign dates; every redeemer was targeted; no campaign ends before it starts. |
| Campaigns outside the data | Transactions cover only 2017. Campaigns 15, 20, 21, 22, 23, 24, 25 and 26 run partly outside it (coverage 0.13 to 0.93). Campaign 24 has zero redemptions because most of it is before the data starts. |
| Small campaigns | Campaigns 1, 3, 15, 24 and 25 targeted fewer than 30 households, so their rates are unstable. |

## 3. Cleaning and transformation (notebook Step 2)

### Missing values
Demographic gaps are filled with the label **"Unknown"** (not dropped, not imputed). Dropping would lose 42% of the pairs; imputing would invent household characteristics. Unknown households are analysed as their own group and turn out to behave differently (7.2% redemption against 16.7%).

### Tables built
* **`household_campaign`** (6,589 rows): one row per household and campaign. Added: campaign details, coupons offered, redemptions, demographics, the household's baskets and sales for the year, campaigns received, and the response `redeemed` (1 if at least one coupon was redeemed).
* **`campaign_summary`** (27 rows): per campaign, the targeted and redeeming households, redemptions, sales and discount of the targeted households during the campaign dates, and baskets.

### New features

| Feature | Formula |
|---|---|
| `redemption_rate` | redeeming households / targeted households |
| `redemptions_per_household` | redemptions / targeted households |
| `cost_per_redemption` | discount cost / redemptions |
| `sales_per_discount_dollar` | window sales / discount cost |
| `sales_per_household` | window sales / targeted households |
| `baskets_per_household` | window baskets / targeted households |
| `duration_days` | end date minus start date plus 1 |
| `start_month` | month of the start date |
| `coverage` | share of the campaign's days that fall inside the transaction period |
| `rankable` | `coverage >= 0.8` and at least 30 targeted households |

Discount cost is the sum of `coupon_disc` and `coupon_match_disc` for targeted households during the campaign dates. Campaigns overlap, so it is a proxy. `cost_per_redemption` for campaign 24 (no redemptions) is left empty instead of infinite.

### Outliers
* **Campaign discount cost:** 3 IQR outliers (campaigns 8, 13, 18), all Type A with over 1,000 households each. Kept; they are real and large.
* **Campaign redemption rate:** no IQR outliers. Unstable and understated rates are handled with the `rankable` flag instead of changing the values.
* **Household total sales:** 296 rows above the IQR upper limit ($8,791). Capped at that limit in `total_sales_capped`.
* **Transaction lines:** kept. `quantity` is not used.

### Encoding
Label encoding only. Ordered dictionaries for `age` and `income`; Pandas category codes for `campaign_type`, `home_ownership`, `marital_status`, `household_size`, `household_comp`, `kids_count`. "Unknown" gets the highest code. A check confirms no code is missing or negative.

### Scaling
Not applied. No distance-based or gradient-based method is used, correlations and group averages do not change with scale, and original units keep the results readable.

### Outputs
`Dataset/campaign_summary_clean.csv` (27 rows, 22 columns) and `Dataset/household_campaign_clean.csv` (6,589 rows, 31 columns, no missing values).

## 4. EDA results (notebook Step 3)

Household-level results use the 20 `rankable` campaigns (6,091 rows, 12.7% redemption).

| Question | Result |
|---|---|
| Overall response | 12.0% of all 6,589 pairs redeemed; mean campaign redemption rate 8% |
| Campaign type | Type A 16.6%, Type B 6.9%, Type C 6.6%. Median cost per redemption about $7.7, $19.9 and $170 (Type C is two campaigns). |
| Type versus scale | Type A: about 912 households and 202 coupons per campaign; Type B: about 154 and 16; Type C: about 145 and 14. Rate correlates 0.80 with audience size and 0.77 with coupons offered. |
| Age | 19-24: 6.8%; 25-34: 15.6%; 35-44: 15.9%; 45-54: 19.1%; 55-64: 18.1%; 65+: 17.3% |
| Income | Highest 150-174K (22.8%) and 50-74K (22.4%); lowest 250K+ (4.7%, 64 rows) |
| Demographics known | 16.7% known, 7.2% unknown |
| Campaigns received | 6.8% for 1-2, 11.4% for 3-4, 10.7% for 5-6, 14.2% for 7-8, 19.6% for 9+ |
| Timing | Start quarter averages 7.2%, 7.2%, 8.9%, 9.7%. No clear pattern; within Type B, duration and rate are unrelated. |
| Over time | Weekly redemptions have three large spikes (May, August, November) matching Type A campaigns 8, 13 and 18. |
| Top campaigns | 18 (18.9%), 13 (18.2%), 8 (14.7%), 19 (11.5%), 9 (11.4%) |
| Bottom campaigns | 6 (1.5%), 7 (2.5%), 11 (2.8%), 21 (3.1%), 2 (4.2%) |
| ROI proxy | Sales per discount dollar about 126 on average; similar across types |

### Plots
Box plots (outliers), weekly redemption line chart, redemption rate by start date (bubble size = audience), type comparison bars, age and campaigns-received bars, age-by-income heat map (cells under 20 rows hidden), two correlation heat maps, top and bottom campaign bars.

## 5. Limitations and caveats
* No impressions, clicks or region, so click-through rate and regional analysis were not possible.
* Cost and sales-per-discount-dollar are proxies, not true campaign cost or incremental return.
* No control group: all findings are associations.
* Campaign type is mixed up with audience size and coupon count.
* Correlations with `age_code`, `income_code` and `kids_count_code` are partly caused by the "Unknown" group sharing the highest code, so the group tables are the clearer view.
* The Python package describes the data as educational; findings describe this dataset only.
