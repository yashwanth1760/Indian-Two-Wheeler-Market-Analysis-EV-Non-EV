# 🏍️ Indian Two-Wheeler Market Analysis: EV vs Non-EV

An end-to-end data analysis project in Python: **web scraping → data cleaning → exploratory data analysis (EDA) → insights** on electric and non-electric two-wheelers sold in India.

![Python](https://img.shields.io/badge/Python-3.x-blue)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458)
![BeautifulSoup](https://img.shields.io/badge/BeautifulSoup-Web%20Scraping-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 📑 Table of Contents
1. [Project Overview](#-project-overview)
2. [Objectives](#-objectives)
3. [Dataset Summary](#-dataset-summary)
4. [Project Workflow](#-project-workflow)
5. [Phase 1: Data Extraction](#-phase-1-data-extraction-web-scraping)
6. [Phase 2: Data Cleaning](#-phase-2-data-cleaning)
7. [Phase 3: Exploratory Data Analysis](#-phase-3-exploratory-data-analysis)
8. [Key Insights](#-key-insights)
9. [Conclusion](#-conclusion)
10. [Limitations & Future Scope](#-limitations--future-scope)
11. [Repository Structure](#-repository-structure)
12. [How to Run](#-how-to-run)
13. [Tech Stack](#-tech-stack)
14. [Author](#-author)

---

## 📌 Project Overview

India's two-wheeler market is growing fast, and electric vehicles (EVs) are changing it. Buyers face a lot of choices across brands, prices and specs, and it is hard to know what really drives **price**, **performance** and **efficiency**.

This project collects real listing data from **BikeWale**, cleans it, and analyzes it to answer questions such as:

- Which brands offer the most models, and where do they sit on price?
- What drives the price of a bike: power, engine size, battery, or something else?
- Do heavier bikes really give less mileage?
- Does a higher price buy a longer range in EVs?
- Which bikes give the best value for money?

---

## 🎯 Objectives

| # | Objective | How it was achieved |
|---|---|---|
| 1 | **Collect real-world data** on Indian two-wheelers | Scraped BikeWale with `requests` and `BeautifulSoup`: **555 raw records** (114 EV, 441 non-EV) |
| 2 | **Clean and standardize** the raw data | Stripped units, created `Brand`, removed duplicates and invalid records, imputed missing values. Final data: **108 EV and 335 non-EV bikes** |
| 3 | **Explore and compare** price, performance and efficiency | Outlier checks, univariate, bivariate and multivariate analysis, group-by, pivot and crosstab analysis |
| 4 | **Find the key drivers** of price and performance | Correlation analysis on both datasets |
| 5 | **Analyze brand positioning** across Budget, Mid and Premium segments | Price segments built with `pd.qcut` (3 equal-sized groups) |
| 6 | **Give data-backed buying insights** | Value-for-money picks by use case (mileage, power, range, budget) |

---

## 📊 Dataset Summary

| | **Electric Vehicles (EV)** | **Non-Electric Vehicles** |
|---|---|---|
| Raw records scraped | 114 | 441 |
| After cleaning | **108** | **335** |
| Brands | 39 | 28 |
| Final columns | 10 | 11 |

**Source:** [BikeWale.com](https://www.bikewale.com): "Best Bikes in India" listing pages
**Categories covered:** Scooters, Sports Bikes, Cruiser Bikes

### Final columns

**EV dataset (10):**
`Brand`, `Model`, `Rating`, `Rating_Members`, `Price(Rs)`, `Top_Speed(kmph)`, `Range(km)`, `Battery(kwh)`, `Rating_Imputed`, `Rating_Clean`

**Non-EV dataset (11):**
`Brand`, `Model`, `Rating`, `Rating_Members`, `Price(Rs)`, `Engine(cc)`, `Mileage(kmpl)`, `Power(bhp)`, `Weight(kg)`, `Rating_Imputed`, `Rating_Clean`

> **Note on ratings:** `Rating` holds the final (imputed) value. `Rating_Imputed` is a True/False flag. `Rating_Clean` holds only the original ratings (`NaN` where a rating was imputed), so the real data stays available for analysis.

---

## 🔄 Project Workflow

```
 BikeWale listing pages
          │
          ▼
┌─────────────────────┐
│ 1. Web Scraping     │  requests + BeautifulSoup + regex
└─────────────────────┘
          │   Electric_Vehicles.csv (114)  |  Non_Electric_Vehicles.csv (441)
          ▼
┌─────────────────────┐
│ 2. Data Cleaning    │  units → brand → duplicates → missing values → export
└─────────────────────┘
          │   Electric_Cleaned_DataSet1.csv (108) | Non_Electric_Cleaned_DataSet1.csv (335)
          ▼
┌─────────────────────┐
│ 3. EDA & Insights   │  outliers → univariate → bivariate → multivariate →
└─────────────────────┘  group/pivot/crosstab → correlation
```

---

## 🌐 Phase 1: Data Extraction (Web Scraping)

**Tools:** `requests`, `BeautifulSoup`, `pandas`, `numpy`, `re`, `time`

| Aspect | Details |
|---|---|
| **Pagination** | Scooters: fixed loop over 15 pages. Sports and Cruisers: `while True` loop that stops when a page returns no bike cards (the page count was not known in advance) |
| **Request handling** | Browser-like headers (User-Agent, Accept-Language, Referer) and a **3-second delay** between requests to be polite to the server |

**Extraction pipeline:**
1. Request each listing page and parse the HTML with BeautifulSoup
2. Find every bike card on the page
3. Detect whether each card is an **EV or non-EV** from its HTML tag
4. Pull the fields with CSS selectors, and use regex for ratings and number of reviewers
5. Split the specs line on `|` and match unit keywords (`cc`, `kmpl`, `bhp`, `kmph`, `kwh`, `km`, `kg`)
6. Store missing values as `NaN` so every list stays the same length

**Quality check:** all 8 column lists were verified to have equal length (114 for EV, 441 for non-EV) before building the DataFrames.

**Raw output:**

| | EV | Non-EV |
|---|---|---|
| File | `Electric_Vehicles.csv` | `Non_Electric_Vehicles.csv` |
| Common fields | Model, Rating, Rating_Members, Price, Weight | Model, Rating, Rating_Members, Price, Weight |
| Unique fields | Top_Speed (kmph), Range (km), Battery (kWh) | Engine_CC, Mileage (kmpl), Power (bhp) |

---

## 🧹 Phase 2: Data Cleaning

### 2.1 Initial profiling
Ran `head()`, `info()`, `describe()` and `shape` on both datasets.

| Check | Non-EV (441 rows) | EV (114 rows) |
|---|---|---|
| Numeric columns stored as text | Engine_CC, Mileage, Power, Weight | Top_Speed, Range, Weight, Battery |
| Rating missing | 107 rows | 17 rows |
| Price range | ₹47,922 to ₹1 crore | ₹54,999 to ₹15.25 lakh |
| Price mean vs median | ₹9.04 L vs ₹3.39 L | ₹1.45 L vs ₹1.15 L |

**Issues found:** values still carried units (`"124.8 cc"`, `"82 kmph"`), prices were heavily right-skewed, duplicates existed in both datasets, and Rating, Mileage, Weight and Power had missing values.

### 2.2 Standardization and feature engineering

| Task | Method | Result |
|---|---|---|
| Strip units | `str.replace()` + `pd.to_numeric(errors="coerce")` | `"56.7 kmpl"` becomes `56.7` (float) |
| Create `Brand` | Custom `get_brand()` function | First word of Model, with a list for multi-word brands |
| Rename columns | `rename()` | Units added to names: `Price(Rs)`, `Mileage(kmpl)`, `Range(km)` |
| Reorder columns | Column selection | Brand and Model come first |

**Multi-word brands handled:** Royal Enfield, Moto Morini, QJ Motor, Kinetic Green, Quantum Energy, Joy e-bike, Bounce Infinity, OPG Mobility, Hop Electric, PURE EV

> `errors="coerce"` turns bad values into `NaN` instead of crashing, so they can be handled later.

### 2.3 Duplicates and invalid records

| Issue | Non-EV | EV |
|---|---|---|
| Duplicate rows removed | 104 (441 → 337) | 5 (113 → 108) |
| Invalid or incomplete records | 2 Honda models removed (XR300 Rally, XR300L), each missing 3 of 4 specs | 1 removed: Bajaj Freedom |
| Reason | Too little data to analyze | It is a **CNG** bike (range in km/kg), not electric |
| **Final rows** | **335** | **108** |

Duplicates were reviewed first with `duplicated(keep=False)` and then removed with `drop_duplicates()`, so no valid records were deleted by mistake. The likely cause is the same model appearing on more than one category page.

### 2.4 Missing value handling

| Column | Missing (after dedup) | Strategy |
|---|---|---|
| Power, Weight, Mileage (Non-EV) | 8, 9 and 65 rows | Median by **Brand + Engine group**, then fallback to the Engine-group median |
| Top_Speed (EV) | 1 row (Honda QC3) | Median (77.5 kmph) of similar bikes: ₹1 to 1.5 lakh price and 2.5 to 3.5 kWh battery |
| Weight (EV) | 18 rows (16.7%) | **Column dropped**, as it was not needed for the analysis |
| Rating | Non-EV: 79 (23.6%), EV: 17 (15.7%) | Brand median, then overall median, with a `Rating_Imputed` flag |
| Rating_Members | Non-EV: 30, EV: 6 | Filled with 0 (no reviews) |

**Design choices:**
- The **median** was used instead of the mean because the data is skewed.
- Engine size was grouped into 8 bins (up to 125 cc, 126-160, 161-250, ... up to 1300+ cc), so bikes are compared with similar bikes.
- The temporary `Engine_group` column was dropped after imputation.

### 2.5 Validation
- Nulls remaining: only `Rating_Clean` (23.6% non-EV, 15.7% EV), by design
- Duplicates remaining: 0
- Datatypes and shapes re-checked before export

---

## 🔍 Phase 3: Exploratory Data Analysis

### 3.1 Techniques used

| Technique | Details |
|---|---|
| **Outlier analysis** | IQR method on Price, Speed, Range, Battery (EV) and Price, Engine, Mileage, Power, Weight (non-EV). Every outlier was **checked and kept**, because they are real premium or high-performance models |
| **Feature engineering** | `pd.qcut` segments: Price (Budget/Mid/Premium), plus Range, Battery and Speed (Low/Medium/High) for EV |
| **Univariate analysis** | Mean, median, mode, variance, SD, IQR, histograms with KDE, boxplots, brand frequency tables |
| **Bivariate analysis** | Scatter plots with Pearson and Spearman correlation, median price by brand, Price Segment vs Range |
| **Categorical vs categorical** | Brand vs segment heatmaps (% mix within each brand) |
| **Multivariate analysis** | Bubble chart (Battery vs Range vs Speed vs Price), `lmplot` by price segment |
| **Group-based analysis** | `groupby` summaries by Brand and by each segment |
| **Pivot tables and crosstabs** | Brand vs segment counts and percentages |
| **Correlation analysis** | Standardized covariance matrix (equals the correlation matrix) with heatmap |

### 3.2 EV analysis (108 bikes)

| Feature | Mean | Median | Min to Max |
|---|---|---|---|
| Price | ₹1.40 L | ₹1.12 L | ₹55K to ₹15.25 L |
| Top Speed | 76 kmph | 70 kmph | 25 to 194 |
| Range | 156 km | 145 km | 60 to 579 |
| Battery | 3.57 kWh | 3.24 kWh | 0.6 to 16 |

- Mean is above median in every feature, so all distributions are **right-skewed**.
- The EV market is **fragmented**: Okinawa has the most models (8, or 7.4%), followed by OLA (7) and Odysse (6).
- Cheapest brands (median price): Evolet (₹63.6K), Joy e-bike (₹65.9K), Yo (₹69.4K).
- Most expensive: BMW (₹9.87 L), Ultraviolette (₹3.09 L), Royal Enfield (₹2.79 L).

**Correlation matrix (EV):**

| Pair | Correlation |
|---|---|
| Battery and Range | **0.87** |
| Battery and Top Speed | **0.86** |
| Top Speed and Range | 0.80 |
| Price and Battery | 0.45 |
| Price and Top Speed | 0.40 |
| Price and Range | **0.15** |

**EV price segments (medians):**

| Segment | Price | Speed | Range | Battery |
|---|---|---|---|---|
| Budget | ₹80K | 54.5 kmph | 90 km | 2.1 kWh |
| Mid | ₹1.12 L | 70 kmph | 149.5 km | 3.2 kWh |
| Premium | ₹1.66 L | 95 kmph | 165 km | 4.0 kWh |

### 3.3 Non-EV analysis (335 bikes, 28 brands)

| Feature | Mean | Median | SD |
|---|---|---|---|
| Price | ₹8.25 L | ₹2.89 L | ₹11.3 L |
| Engine | 533 cc | 349 cc | 461 |
| Mileage | 33.5 kmpl | 30 kmpl | 14.7 |
| Power | 54.7 bhp | 36.5 bhp | 51.6 |
| Weight | 177.8 kg | 180 kg | 55.3 |

- All features are right-skewed except **Weight**, which is almost symmetric.
- **Most models:** Kawasaki 37, Triumph 30, Honda 25, Hero 25, KTM 24. The top 5 brands hold 42% of models and the top 10 hold 69%.
- **Most expensive (median):** Harley-Davidson ₹22.9 L, BMW ₹22.5 L, Ducati ₹19.5 L.
- **Cheapest (median):** Hero ₹84.5K, TVS ₹91K, Bajaj ₹1.31 L.
- Ducati's mean (₹28.9 L) is far above its median (₹19.5 L), so a few ultra-premium models inflate it. Honda and Aprilia show the same pattern.

**Correlation matrix (Non-EV):**

| Pair | Correlation |
|---|---|
| Engine and Weight | **+0.89** |
| Power and Price | **+0.85** |
| Engine and Power | +0.82 |
| Weight and Mileage | **−0.77** |
| Engine and Mileage | −0.75 |
| Price and Mileage | −0.60 |

**Non-EV price segments (terciles: Budget ≤ ₹1.84 L, Mid ₹1.84 to 8.23 L, Premium above ₹8.23 L):**

| Median | Budget | Mid | Premium |
|---|---|---|---|
| Price | ₹1.16 L | ₹2.89 L | ₹16.8 L |
| Engine | 125 cc | 349 cc | 999 cc |
| Mileage | 47 kmpl | 30 kmpl | 19 kmpl |
| Power | 11.4 bhp | 36.5 bhp | 106.5 bhp |
| Weight | 123 kg | 182 kg | 221 kg |

**Brand strategy (share of each brand's models):**
- Budget specialists: Hero 96%, Yamaha 93%, Bajaj 85%, TVS 84%
- Mid-segment: Royal Enfield 87.5%, Kawasaki 59.5%
- Premium: Ducati and Indian 100%, BMW and Harley 85%, Triumph 77%
- Diversified: Honda is 56% budget, 24% mid and 20% premium

---

## 💡 Key Insights

1. **Always use the median for price.** Premium bikes pull the mean up in both datasets.
2. **Non-EV: power drives price (r = 0.85).** Every step up in power, size and price lowers mileage.
3. **Non-EV: heavier bikes give less mileage (r = −0.77).**
4. **EV: battery capacity drives performance.** It correlates with both range (0.87) and speed (0.86).
5. **EV: price does not buy range (r = 0.15).** Going from Mid to Premium adds only about 15 km of median range for roughly ₹53K more.
6. **OLA is the EV value leader:** a median price of about ₹1.05 L, with an average range of about 360 km and a 7.3 kWh battery.
7. **Budget non-EV picks:** TVS Sport (80 kmpl, ₹65.7K), Hero Super Splendor Xtec 2.0 and Bajaj Platina 100 (75 kmpl each, under ₹87K), Bajaj Pulsar NS200 (24.1 bhp at ₹1.39 L) for performance.
8. **Outliers were real bikes, not errors** (BMW CE 04, OLA Roadster Pro, Ducati Panigale, Hayabusa), so they were kept after verification.

---

## ✅ Conclusion

| | **Non-EV (335 bikes)** | **EV (108 bikes)** |
|---|---|---|
| Market shape | Two clusters: everyday commuters and premium superbikes | Fragmented: the top brand holds only 7.4% of models |
| Main price driver | Power (r = 0.85) | Battery (r = 0.45, moderate) |
| Performance link | Weight vs Mileage (r = −0.77) | Battery vs Range (r = 0.87) |
| Weak link | Price vs Mileage (r = −0.60) | Price vs Range (r = 0.15) |
| Buying advice | Pick by use case: Hero, Bajaj and TVS for commuting, Bajaj Pulsar for performance | Compare **battery capacity (kWh)**, not just price |

---

## ⚠️ Limitations & Future Scope

**Limitations**
- Ratings were imputed for about 24% of non-EV and 16% of EV bikes. They are flagged in `Rating_Imputed`, and `Rating_Clean` holds the original values only.
- About 19% of non-EV mileage values (65 of 335) were filled by median imputation, so mileage rankings partly rest on imputed values.
- Scope is limited to **Scooters, Sports and Cruiser** bikes from a single source (BikeWale).
- Quantile-based segments (`qcut`) depend on this dataset's distribution, so segment boundaries are not fixed market definitions.
- This is an exploratory analysis. It shows correlation, not causation.

**Future scope**
- Add a **weighted scoring model** for personalized recommendations by use case
- Include more categories (commuter, adventure, touring) and more specs such as torque
- Add sales and registration data for market-share analysis
- Build an interactive dashboard (Power BI, Streamlit or Tableau)
- Try regression models to predict price from specs

---

## 📁 Repository Structure

> Adjust the names below to match your actual files.

```
├── data/
│   ├── raw/
│   │   ├── Electric_Vehicles.csv
│   │   └── Non_Electric_Vehicles.csv
│   └── cleaned/
│       ├── Electric_Cleaned_DataSet.csv
│       └── Non_Electric_Cleaned_DataSet.csv
├── notebooks/
│   ├── 01_Web_Scraping.ipynb
│   ├── 02_Data_Cleaning.ipynb
│   ├── 03_Data_Analysis_Non_Electric_Vehicles.ipynb
│   └── 04_Data_Analysis_Electric_Vehicles.ipynb
└── README.md
```

---


> **Scraping note:** website layouts change, so the scraper may need updated CSS selectors. If you only want to explore the analysis, skip notebook 01 and use the CSVs in `data/`. Please follow the website's terms of use and keep the delay between requests.

---

## 🛠 Tech Stack

| Area | Tools |
|---|---|
| Language | Python |
| Scraping | `requests`, `BeautifulSoup`, `re`, `time` |
| Data handling | `pandas`, `numpy` |
| Visualization | `matplotlib`, `seaborn`, `plotly` |
| Environment | Jupyter Notebook |

---

## 👤 Author

**Your Name**
- LinkedIn: [[your-linkedin-url](https://www.linkedin.com/in/yashwanth-chalamalla/)]
- Email: chalamallan11@gmail.com

---

## 📜 Disclaimer

The data was collected from publicly available listing pages for **educational and portfolio purposes only**. Prices and specifications may change over time. This project is not affiliated with BikeWale.

---

⭐ If you found this project useful, please consider giving it a star!
