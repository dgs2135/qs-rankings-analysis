# QS World University Rankings (2017–2027): Business Intelligence Analysis

A business intelligence project analysing eleven years of QS World University Rankings. Data was cleaned and merged in Python, explored through ten Tableau visualisations, and extended with time series regression and forecasting for English-medium countries.

Built for **CST3340 Business Intelligence (Coursework 2)**, BSc (Hons) Information Technology and Business Information Systems, Middlesex University London, AY 2025–26.

---

## Project Overview

University rankings influence decisions made by students, institutions, and governments. This project asks:

- How is global higher-education performance distributed across countries and regions?
- How has that distribution changed between 2017 and 2027?
- Do elite universities move between tiers, or is prestige static?
- Does English-medium instruction explain competitiveness?
- Which national trends are reliable enough to forecast to 2030?

**Dataset:** Separate annual QS ranking files from Kaggle, merged into a single master dataset of **13,909 records** covering 2017–2027 (included in this repo at `data/QS_Master_2017_2027.csv`).

**Stakeholders:** prospective students, university leaders benchmarking against competitors, and policymakers assessing national competitiveness.

---

## Repository Structure

```

.
├── data/
│   └── QS_Master_2017_2027.csv           # Cleaned, merged master dataset (output of both notebooks)
├── notebooks/
│   ├── 01_standardize_schema.ipynb       # Merge annual files into one standard schema
│   └── 02_normalize_entities.ipynb       # Clean country and university names, add geography
├── QS_Rankings__2017-2027__Insights.twbx  # Tableau packaged workbook (visualisations)
├── README.md
└── requirements.txt                      # Python dependencies

```

---

## Data Pipeline

### 1. Schema standardisation (`01_standardize_schema.ipynb`)

Loads the six source files (2017–2022 combined, then 2023, 2024, 2025, 2026, 2027) and maps their differing column names onto one schema:

`university, year, country, Rank, overall_score, academic_reputation, employer_reputation, faculty_student, citations, sustainability`

Key steps:

- **Rank conversion:** ranges become midpoints (e.g. `601–610` → `605.5`) and the open-ended `1401+` is set to `1401`.
- **Country normalisation:** e.g. "United States" → `USA`, "United Kingdom" → `UK`, "Mainland China" → `China`.
- Concatenation of all years into a single master CSV.

### 2. Entity normalisation (`02_normalize_entities.ipynb`)

Runs in Google Colab on the merged master file:

- Standardises remaining country spellings (e.g. "Russian Federation" → `Russia`, "Republic of Korea" → `South Korea`).
- Audits missing values per column.
- Adds **Continent** and **Region** columns (18 QS sub-regions) via a country-to-geography mapping.
- Cleans university names: trims whitespace, strips parenthetical text, normalises diacritics, and applies a manual mapping for known variants so each institution is tracked as one entity across years.
- Spot-checks consistency for institutions such as ETH Zurich, UCL, and Nanyang Technological University.

### 3. Master dataset schema

`data/QS_Master_2017_2027.csv` is the final output of the pipeline (about 1.3 MB, 13,909 rows × 13 columns).

| Column | Description |
|--------|-------------|
| `university` | Standardised institution name |
| `year` | Ranking year (2017–2027) |
| `country` | Standardised country name |
| `Rank` | Original rank as published (may be a range, e.g. `601-610`, or `1401+`) |
| `NumericRank` | Rank converted to a number (range midpoints; `1401+` → 1401) |
| `overall_score` | QS overall score |
| `academic_reputation` | Academic reputation score |
| `employer_reputation` | Employer reputation score |
| `faculty_student` | Faculty-student ratio score |
| `citations` | Citations per faculty score |
| `sustainability` | Sustainability score |
| `Continent` | Continent (added in notebook 02) |
| `Region` | QS-style sub-region (added in notebook 02) |

Records per year: 933 (2017), 977, 1,018, 1,069, 1,185, 1,300, 1,422, 1,497, 1,503, 1,501, and 1,504 (2027).

**Missing values are expected, not errors.** The reputation, citations, and sustainability columns were not published consistently across years (see Limitations), so they are mostly empty before 2023. Roughly 9,500 of 13,909 rows have no sustainability score, and about 6,500 have no academic reputation, employer reputation, or citations score.

### 4. Derived metric: Percentile Rank

The number of ranked universities grew from **933 (2017)** to **1,504 (2027)**, so raw rank comparisons across years are misleading. A **Percentile Rank** calculated field in Tableau (numeric rank ÷ total ranked universities that year) allows fair comparison over time. Lower values mean stronger performance.

---

## Visualisations (Tableau)

🔗 **[View Interactive Dashboard on Tableau Public](https://public.tableau.com/views/QSRankings2017-2027Insights/Sheet1?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)**

Open the live dashboard on Tableau Public or download `QS_Rankings__2017-2027__Insights.twbx` to open in Tableau Desktop / Tableau Reader.

| # | Visual | Insight |
|---|--------|---------|
| 1 | Filled map | Global distribution of average Percentile Rank (2023–2027) |
| 2 | Heatmap | Regional performance across 18 sub-regions over time |
| 3 | Bar chart | Top 20 countries by average Percentile Rank (min. 5 ranked universities) |
| 4 | Box plot | Ranking stability and spread within the top 20 countries |
| 5 | Highlight table | Country strengths across the five QS metrics |
| 6 | Line graph | Performance trends for eleven English-medium countries, with forecast |
| 7 | Pareto chart | Concentration of the 2027 Top 100 by country |
| 8 | Sankey diagram | Tier mobility (Established Elite / Top Qualifier / Contenders), 2017 → 2027 |
| 9 | Scatter plot | Academic vs Employer Reputation, coloured by 2027 tier |
| 10 | Bump chart | Year-on-year rank volatility of the top 10 universities |

---

## Data Mining: Time Series Forecasting

Linear trend lines and forecasts (to 2030) were produced in Tableau for eleven English-medium countries, using **Year** as the independent variable and **average Percentile Rank** as the dependent variable. R was derived from Tableau's reported R².

| Trend strength | Countries (R) |
|----------------|---------------|
| Very strong, improving | Ireland (0.991), New Zealand (0.990), Australia (0.980), Canada (0.954), South Africa (0.950) |
| Moderate | Philippines (0.754), USA (0.663, downward), UK (0.559) |
| Weak / stable | Singapore (0.195), India (0.150), Malaysia (0.034) |

---

## Key Findings

- **Prestige stays concentrated.** Nine countries (USA, UK, Australia, China, Hong Kong, Canada, France, Germany, Japan) hold 80% of the 2027 Top 100.
- **Small, focused systems can outperform large ones.** The Netherlands leads the top 20 by average Percentile Rank (0.1167), while the UK ranks lower than its reputation suggests.
- **No country dominates every metric.** Nations tend to specialise (e.g. Nordic countries on Sustainability).
- **Elite mobility exists but is asymmetric.** Within the top-200 cohort of 2017, Established Elite grew from 30 to 47, and few elite institutions fell.
- **English-medium status alone does not predict competitiveness.** Trajectories differ sharply among countries sharing a language of instruction.
- **Forecasts suggest a broadly stable landscape through 2030**, with caution advised given only eleven data points per country.

---

## Limitations and Data Ethics

- QS methodology changed over time. Academic reputation, employer reputation, citations, and faculty-student ratio are only consistently available from **2023**; sustainability scores are missing for **2023 and 2025**.
- Averaging effects: countries with many ranked universities (e.g. the USA) can show declining averages as lower-ranked institutions enter the pool.
- Rank-range midpoints are approximations.
- Forecasts rest on short time series and linear assumptions.
- The dataset contains no personal data. Source data comes from public Kaggle repositories, so check the original licensing terms before reuse or redistribution.

---

## Tools and Technologies

- **Python** (pandas, NumPy, `re`, `unicodedata`) in **Google Colab**
- **Tableau** for visualisation, calculated fields (including LOD expressions), and forecasting

---

## Getting Started

**To explore the results:** open `QS_Rankings__2017-2027__Insights.twbx` in Tableau, or load `data/QS_Master_2017_2027.csv` directly:

```python
import pandas as pd
df = pd.read_csv("data/QS_Master_2017_2027.csv")

```

**To reproduce the pipeline:**

1. Install dependencies:
```bash
pip install -r requirements.txt

```


2. Download the raw annual QS ranking files from Kaggle and place the CSVs where `01_standardize_schema.ipynb` expects them.
3. Run `01_standardize_schema.ipynb` to build the merged master file.
4. Upload the merged file to Google Drive, update the file paths, and run `02_normalize_entities.ipynb` in Colab.
5. Open the `.twbx` workbook in Tableau.

> **Tableau note:** Disable **Data Interpreter** on the data source. It misparses university names containing commas, which corrupts the Percentile Rank LOD calculation.

---

## Author

**Earl Campillanos**

BSc (Hons) Information Technology and Business Information Systems, Middlesex University London

---

## Acknowledgements

* Module leader: Stephen Agada, Middlesex University London
* QS World University Rankings data via Kaggle

```

```
