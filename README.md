# Marketing Campaign Effectiveness Analysis

Data cleaning, transformation, exploratory data analysis (EDA) and visualisation of 27 retail coupon campaigns, to find out which campaigns, campaign types, timing and customer groups get the best response. The whole analysis, with its explanations, is in one notebook: [`Marketing_Campaign_Analysis.ipynb`](Marketing_Campaign_Analysis.ipynb).

No predictive model is trained. The project covers loading and exploring the data, cleaning it, creating features, exploring patterns with statistics and plots, and drawing insights. The full process write-up is in [`docs/PROCESS_DOCUMENTATION.md`](docs/PROCESS_DOCUMENTATION.md).

## Objective

Find out how well the coupon campaigns performed and which attributes (campaign type, timing, target audience) are linked to a better response, at what cost.

## Dataset

| | |
|---|---|
| **Name** | The Complete Journey (retail shopping data from the 84.51° Complete Journey study) |
| **Stored in this repository** | `Dataset/` as CSV files, read with `pd.read_csv` (exported once from the Python package [`completejourney_py`](https://pypi.org/project/completejourney_py/) by `download_data.py`) |
| **Contents used** | 1,469,307 transaction lines (2017) for 2,469 households, 27 coupon campaigns, 6,589 household-campaign targeting records, 2,102 coupon redemptions, 116,204 coupon listings, household demographics for 801 households |

**Source note:** the R version of this dataset credits 84.51° and describes the households as frequent shoppers at a retailer. The Python package's README calls it "simulated data based on real grocery shopping patterns" for educational use. This is a learning project, so the findings describe this dataset only.

**The six raw tables are stored in `Dataset/`** and the notebook loads them with `pd.read_csv`. The biggest table (`transactions`, 1.47 million rows) is compressed as `transactions.csv.gz`, which `pd.read_csv` opens directly. The two **cleaned** datasets the notebook produces are saved in the same folder. If a raw file is ever missing, `python download_data.py` recreates all six.

### Task examples and what is used instead

The data has no impressions, clicks or region, so these example metrics from the task were replaced by close equivalents:

| Task example | Used here |
|---|---|
| Response rate | **Redemption rate** = redeeming households / targeted households |
| Cost per acquisition | **Cost per redemption** = coupon discount given / redemptions |
| ROI | **Sales per discount dollar** (a proxy, not incremental return) |
| Engagement | **Baskets per household** |
| Channel | **Campaign type** (A, B, C) |
| Target audience | Age, income, household composition |
| Timing | Start month, duration |

## What the notebook does

| Notebook section | Task requirement | What is done |
|---|---|---|
| 0. Dataset source and background | Dataset overview | Source, metric mapping, table list |
| 1.1 Load | Load with Pandas | `pd.read_csv` for six tables in `Dataset/` |
| 1.2 Structure | Explore structure | Rows, columns, types, missing values, duplicates |
| 1.3 Summary statistics | Summary statistics | `describe()` and value counts |
| 1.4 Missing values | Identify missing values | Gaps in demographics, share of targeted households without them |
| 1.5 Outliers | Identify outliers | Zero-quantity and extreme-quantity lines, large sales values |
| 1.6 Inconsistent records | Inconsistent campaign records | Redemption dates, targeting, campaigns running past the data |
| 2.1 Missing values | Handle missing values | "Unknown" category for demographics |
| 2.2 Tables | Prepare data | Household-campaign table and campaign summary table |
| 2.3 New features | Create features | Redemption rate, cost per redemption, sales per discount dollar, baskets per household, duration, start month |
| 2.4 Outliers | Box plots, IQR | Box plots and IQR for cost, response rate and household sales; cap household sales; flag unreliable campaigns |
| 2.5 Encoding | Label or one-hot | Label encoding only |
| 2.6 Scaling | Scale if needed | Not needed, with the reasoning |
| 2.7 Save | Cleaned datasets | Two CSV files in `Dataset/` |
| 3.1 Descriptive statistics | Response rate, ROI, engagement | `describe()` tables |
| 3.2 Over time | Performance over time | Weekly redemptions, rate by start date, quarter and duration |
| 3.3 Campaign types | Conversion across channels | Redemption rate and cost per redemption by type |
| 3.4 Audience | Target audience | Age, campaigns received, age by income heat map |
| 3.5 Correlation | Heatmaps and correlation plots | Two correlation heat maps |
| 3.6 Top and bottom | Top and underperforming campaigns | Top 5 and bottom 5 bar charts and tables |
| 3.7 Key insights | Which attributes influence success | Findings, actions, limitations |

## Key results

* **Response:** 12.0% of the 6,589 targeted households redeemed at least one coupon (12.7% for the 20 reliable campaigns).
* **Campaign type:** Type A redeemed at 16.6%, against 6.9% (Type B) and 6.6% (Type C), and had the lowest median cost per redemption (about $7.7, against about $19.9 for Type B). Type A campaigns were also much larger (about 900 to 1,100 households and about 200 coupons, against about 150 households and 15 coupons), so type and scale cannot be separated.
* **Audience:** ages 19 to 24 redeemed 6.8%, against 15.6% to 19.1% for older groups. Households with known demographics redeemed 16.7% against 7.2% for the rest.
* **Engagement:** households that received nine or more campaigns redeemed 19.6%, against 6.8% for one or two.
* **Timing:** no clear seasonal pattern; the three big redemption spikes (May, August, November) match the three large Type A campaigns.
* **Top campaigns:** 18, 13 and 8 (Type A). **Weakest:** 6 (Type C, about $300 per redemption), then 7, 11, 21 and 2.
* **Data problems found:** eight campaigns run past the one-year transaction window (their response is understated); demographics are missing for 42% of household-campaign pairs; zero-quantity lines hold 21% of all coupon discount, so they were kept.

## Repository structure

```
.
Marketing_Campaign_Analysis/
├── Dataset/
│   ├── campaign_descriptions.csv       # raw: 27 campaigns
│   ├── campaigns.csv                   # raw: 6,589 household-campaign targeting records
│   ├── coupon_redemptions.csv          # raw: 2,102 redemptions
│   ├── coupons.csv                     # raw: 116,204 coupon listings
│   ├── demographics.csv                # raw: 801 households
│   ├── transactions.csv.gz             # raw: 1,469,307 product lines (compressed)
│   ├── campaign_summary_clean.csv      # 27 campaigns, one row each (created by the notebook)
│   └── household_campaign_clean.csv    # 6,589 household-campaign rows (created by the notebook)
├── Marketing_Campaign_Analysis.ipynb   # the full analysis, with outputs
├── PROCESS_DOCUMENTATION.md            # dataset overview, cleaning, EDA results
├── README.md
└── requirements.txt
```

### Cleaned datasets

| File | Rows | One row is | Main columns |
|---|---|---|---|
| `Dataset/campaign_summary_clean.csv` | 27 | a campaign | type, dates, duration, `coverage`, targeted and redeeming households, discount cost, sales, `redemption_rate`, `cost_per_redemption`, `sales_per_discount_dollar`, `rankable` |
| `Dataset/household_campaign_clean.csv` | 6,589 | a household that received a campaign | campaign details, demographics ("Unknown" where missing), activity, `redeemed` (0 or 1), `total_sales_capped`, label-encoded `*_code` columns, `rankable` |

## Setup and run

Requires Python 3.10 or newer (developed with Python 3.13). Libraries: `pandas`, `numpy`, `matplotlib` and `seaborn`. The data is already in `Dataset/`, so no download is needed. Choose one of the two ways below.

**First, get the code (both options):**

```bash
git clone https://github.com/<your-username>/<repository-name>.git
cd <repository-name>
```

### Option A: Anaconda or an existing Python installation (no virtual environment)

1. Install the libraries into your current environment:

   ```bash
   pip install pandas numpy matplotlib seaborn
   ```

2. Open `Marketing_Campaign_Analysis.ipynb` in Jupyter, JupyterLab or VS Code and select that Python as the kernel.
3. Choose **Run > Run All Cells**.

### Option B: A virtual environment (keeps the project's libraries separate)

```bash
# Create and activate a virtual environment
python -m venv .venv
source .venv/bin/activate          # Windows PowerShell: .venv\Scripts\Activate.ps1

# Install the libraries
pip install -r requirements.txt

# Open the notebook and choose Run > Run All Cells
jupyter lab Marketing_Campaign_Analysis.ipynb
```

In VS Code, select the `.venv` interpreter as the notebook kernel.

### Notes for both options

* Open the notebook from the repository root (the folder that contains `Dataset/`). Running it rewrites the two CSV files in `Dataset/`.
* Running all cells takes under a minute.
* Only if a raw file in `Dataset/` is missing: run `pip install completejourney_py` and then `python download_data.py` to recreate the six raw files. That package is not needed for the notebook itself.

## Design decisions

* **Missing demographics become "Unknown", not dropped or filled.** 42% of household-campaign pairs have none, so dropping them would lose too much, and filling them would invent data.
* **Zero-quantity transaction lines are kept.** They hold 21% of all coupon discount, so removing them would understate campaign cost.
* **`quantity` is not used.** Some products have quantities in the tens of thousands (probably sold by weight). `sales_value` is used instead.
* **Campaigns are flagged, not deleted.** A campaign is `rankable` only if at least 80% of its days fall inside the data and it targeted at least 30 households. The 20 rankable campaigns are used for rankings, correlations and household-level analysis. All 27 stay in the cleaned data.
* **Household total sales is capped** at the IQR upper limit (`total_sales_capped`); the original column is kept.
* **Label encoding only.** `age` and `income` get ordered codes; other columns get Pandas category codes. "Unknown" gets the highest code and is a label, not a size.
* **No scaling.** No distance-based or gradient-based method is used, and correlations do not depend on scale.

## Limitations

* No impressions, clicks or region in the data, so click-through rate and regional analysis are not possible. Campaign type stands in for channel.
* Cost is a proxy (coupon discounts of targeted households during the campaign dates; campaigns overlap). Sales per discount dollar is not an incremental ROI.
* There is no control group, so relationships are associations, not proof of cause.
* Campaign type is mixed up with audience size and number of coupons, and only 20 campaigns could be ranked.
* The Python package describes the data as educational, so findings describe this dataset only.

## Author and licence

**Suhaiv Man Singh.** The code is provided for learning purposes. The data was exported from the `completejourney_py` package (package licence: MIT). Please follow the dataset's own terms when reusing it.
