<img width="1457" height="781" alt="Screenshot 2026-09-09 093357" src="https://github.com/user-attachments/assets/6e2e8e29-e89c-406a-9196-6f97fac11a87" /><img width="1457" height="781" alt="Screenshot 2026-09-09 093357" src="https://github.com/user-attachments/assets/96e23c2d-27cf-47d6-bf38-cf2c7dc50b1a" /># Customer-Behaviour-Analysis
End-to-end retail customer behaviour analysis using SQL, Python, Excel &amp; Power BI — segmentation, spending trends, and dashboard insights.

**End-to-end retail customer behaviour analysis using SQL, Python, Excel, and Power BI — uncovering spending patterns, customer segments, and revenue drivers from 3,900 transactions.**

---

## 📌 Overview

Retail businesses generate large volumes of transactional data, but without structured analysis that data rarely turns into decisions. This project analyzes a retail customer shopping dataset end-to-end — from raw CSV to an interactive Power BI dashboard — to answer a core business question:

> *How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?*

The workflow mirrors how a data analyst operates in industry: clean and prepare data in **Python**, push it into a **MySQL** database to answer structured business questions in **SQL**, validate and cross-check figures in **Excel**, and finally communicate the findings through an interactive **Power BI** dashboard, closing with concrete business recommendations.

**Tools used:** Python (Pandas) · SQL / MySQL · Microsoft Excel · Power BI (DAX)

**Outcome:** A working analytics pipeline and dashboard that identifies the company's highest-value customer segments, the products and discount patterns driving sales, and where marketing and loyalty spend would have the most impact.

---

## 🎯 Objectives

- Clean and prepare raw transactional data for analysis
- Load the cleaned dataset into a MySQL database and answer business questions using SQL
- Perform exploratory data analysis (EDA) in Python to understand distributions and data quality
- Segment customers into New, Returning, and Loyal groups based on purchase history
- Identify the products and categories that drive the most revenue and engagement
- Analyze the impact of discounts, subscriptions, and shipping type on spending
- Build an interactive Power BI dashboard with KPIs, segmentation views, and filters
- Translate analytical findings into actionable business recommendations

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **Python (Pandas)** | Data loading, cleaning, missing-value imputation, feature engineering |
| **MySQL (SQLAlchemy)** | Structured storage of cleaned data; querying for business insights |
| **SQL** | Aggregation, window functions, and segmentation queries |
| **Microsoft Excel** | Manual validation, pivot tables, and supporting summary views |
| **Power BI** | Interactive dashboard, KPI cards, and drill-down visuals |
| **DAX** | Calculated measures for the Power BI data model |

---

## 📂 Project Structure

```
customer-behaviour-analysis/
│
├── data/
│   ├── raw/
│   │   └── customer_shopping_behavior.csv
│   └── cleaned/
│       └── customer_shopping_behavior_cleaned.csv
│
├── python/
│   └── Customer_Shopping_Behavior_Analysis.ipynb
│
├── sql/
│   └── customer_analysis_queries.sql
│
├── excel/
│   └── customer_analysis.xlsx
│
├── powerbi/
│   └── Customer_Report.pbix
│
├── reports/
│   └── Customer-Shopping-Behavior-Analysis-Report.pdf
│
├── screenshots/
│   └── dashboard_preview.png
│
├── README.md
└── LICENSE
```

| Folder | Purpose |
|---|---|
| `data/raw/` | Original, unmodified dataset |
| `data/cleaned/` | Output of the Python cleaning pipeline, ready for SQL/Power BI |
| `python/` | EDA, cleaning, and feature-engineering notebook |
| `sql/` | All SQL business-question queries |
| `excel/` | Supporting pivot-table validation workbook |
| `powerbi/` | The `.pbix` dashboard file |
| `reports/` | Written project report summarizing methodology and findings |
| `screenshots/` | Dashboard image(s) for the README preview |

---

## 📊 Dataset

- **Records:** 3,900 customer transactions
- **Columns:** 18
- **Missing data:** 37 null values in `Review Rating` (imputed with the per-category median)
- **Source:** Retail customer shopping behaviour dataset (CSV)

**Key columns:**

| Category | Fields |
|---|---|
| Demographics | Customer ID, Age, Gender, Location |
| Purchase details | Item Purchased, Category, Purchase Amount (USD), Size, Color, Season |
| Shopping behaviour | Discount Applied, Promo Code Used, Previous Purchases, Frequency of Purchases, Review Rating, Payment Method, Shipping Type, Subscription Status |

---

## 🧹 Data Cleaning & Preparation

Performed in Python (`pandas`):

- **Missing values:** `Review Rating` had 37 nulls, imputed using the median rating **per product category** (not a global median, to avoid biasing ratings across categories).
- **Column standardization:** all column names converted to `snake_case` (e.g. `Purchase Amount (USD)` → `purchase_amt`) for consistency across SQL, Python, and Power BI.
- **Feature engineering:**
  - `age_group` — customers binned into *Young Adult / Adult / Middle-aged / Senior* quartiles using `pd.qcut`
  - `mapped_purchase_frequency` — the categorical `frequency_of_purchases` (e.g. "Weekly", "Monthly", "Quarterly") converted into numeric day intervals for quantitative analysis
- **Redundancy check:** confirmed `discount_applied` and `promo_code_used` were identical across all rows, so `promo_code_used` was dropped.
- **Database load:** the cleaned DataFrame was loaded into a MySQL database via SQLAlchemy for the SQL analysis stage.

| Tool | Role in cleaning |
|---|---|
| Python | Missing-value imputation, renaming, feature engineering, DB load |
| MySQL | Structured storage and querying of the cleaned dataset |
| Excel | Manual spot-checks and pivot-table validation |
| Power BI | Final modeling layer and DAX measures |

---

## 🔍 Exploratory Data Analysis (Python)

Using `pandas`, the notebook covers:

- `df.info()` / `df.describe()` for structure and summary statistics
- Null-value audit (`df.isnull().sum()`)
- Distribution of purchase amount, age, and review rating
- Category and gender breakdowns
- Post-cleaning validation of the feature-engineered columns

> Full detail is available in [`python/Customer_Shopping_Behavior_Analysis.ipynb`](python/Customer_Shopping_Behavior_Analysis.ipynb).

---

## 🗄️ SQL Analysis

Ten business questions were answered directly in SQL against the cleaned `customer` table. Examples:

**Revenue by gender:**
```sql
SELECT gender, SUM(purchase_amt) AS revenue
FROM customer
GROUP BY gender;
```

**Top 3 products per category (window function):**
```sql
WITH item_counts AS (
    SELECT category, item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

**Customer segmentation (New / Returning / Loyal):**
```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
        CASE
            WHEN previous_purchases = 1 THEN 'new'
            WHEN previous_purchases BETWEEN 2 AND 10 THEN 'returning'
            ELSE 'loyal'
        END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS number_of_customers
FROM customer_type
GROUP BY customer_segment;
```

The full set of 10 queries — covering revenue by gender, high-spending discount users, top-rated products, shipping-type comparison, subscriber spend, discount-dependent products, customer segmentation, top products per category, repeat-buyer subscription rates, and revenue by age group — is in [`sql/customer_analysis_queries.sql`](sql/customer_analysis_queries.sql).

---

## 📗 Excel Analysis

Excel was used as a supporting validation layer alongside SQL and Power BI:

- Pivot tables to cross-check SQL aggregation results (e.g. revenue by gender, spend by shipping type)
- Pivot charts for quick visual sanity checks before building the Power BI dashboard
- Conditional formatting to flag high-value customers and discount-heavy products

---

## 📊 Power BI Dashboard

An interactive one-page **Customer Behavior Dashboard** built in Power BI.

**KPI Cards**
- Number of Customers: **3.9K**
- Average Purchase Amount: **$59.76**
- Average Review Rating: **3.75**

**Visualizations**
- Customer segmentation by subscription status (donut: 27% subscribed / 73% not)
- Revenue by category (bar)
- Sales by category (bar)
- Revenue by age group (bar)
- Sales by age group (bar)

**Filters / Slicers**
- Subscription Status (Yes/No)
- Gender
- Category (Accessories, Clothing, Footwear, Outerwear)
- Shipping Type (Standard, Express, Free Shipping, 2-Day, Next Day Air, Store Pickup)

DAX measures drive the KPI cards and the revenue/sales aggregations shown above, letting the dashboard respond dynamically to slicer selections.

### 📸 Dashboard Preview

<img width="1457" height="781" alt="Screenshot 2026-09-09 093357" src="https://github.com/user-attachments/assets/3d691f99-68b9-499c-b899-b90b60ce18cb" />


*(Screenshot to be added from the `.pbix` file — export as PNG and place in `screenshots/`.)*

---

## 📐 Data Model

The Power BI model uses a **single flat table** (`customer`) — one row per transaction, containing customer, product, and behavioural attributes together. This keeps the model simple and is appropriate for the dataset's structure, where a true star schema (separate Customer / Product / Transactions tables) isn't required at this scale.

---

## 🧮 Key DAX Measures

Example measures used in the dashboard (table/column names should be adapted to your model):

```dax
Total Customers = DISTINCTCOUNT(customer[customer_id])

Average Purchase Amount = AVERAGE(customer[purchase_amt])

Total Revenue = SUM(customer[purchase_amt])

Average Review Rating = AVERAGE(customer[review_rating])

% Subscribed = 
DIVIDE(
    CALCULATE(COUNTROWS(customer), customer[subscription_status] = "Yes"),
    COUNTROWS(customer)
)
```

---

## 📈 Key Business Questions Answered

1. What is the total revenue generated by male vs. female customers?
2. Which customers used a discount but still spent more than average?
3. Which products have the highest average review rating?
4. Does shipping type (Standard vs. Express) affect average spend?
5. Do subscribed customers spend more than non-subscribers?
6. Which products have the highest percentage of discounted purchases?
7. How does the customer base break down into New, Returning, and Loyal segments?
8. What are the top 3 products purchased within each category?
9. Are repeat buyers (>5 previous purchases) more likely to subscribe?
10. Which age group contributes the most revenue?

---

## 💡 Business Insights

- **Male customers generated more than double the revenue of female customers** — $157,890 vs. $75,191 — despite a comparable average purchase amount, indicating a volume rather than spend-per-transaction difference.
- **The customer base is overwhelmingly Loyal (79.9%)**, with Returning at 17.97% and New at only 2.13% — suggesting strong retention but a potential bottleneck in new-customer acquisition.
- **Repeat buyers with 5+ previous purchases are far less likely to be subscribers** (2,518 non-subscribed vs. 958 subscribed among repeat buyers), showing subscription conversion isn't keeping pace with loyalty.
- **Discount-dependent products cluster around outerwear and footwear** — Hats, Sneakers, and Coats each have discount rates near 50%, meaning close to half their sales rely on a markdown.
- **Revenue is fairly evenly spread across age groups**, with Young Adults ($62,143) and Middle-aged customers ($59,197) contributing marginally more than Adults and Seniors.
- **Express shipping customers spend slightly more on average** ($60.48) than Standard shipping customers ($58.46), a small but consistent premium.
- **Non-subscribers generate the large majority of revenue** ($170,436 vs. $62,645 from subscribers) simply due to their larger population, even though per-customer averages are close ($59.87 vs. $59.49).

---

## 🎯 Business Recommendations

- **Boost subscription conversion among loyal customers** — the Loyal segment is large but subscription uptake among repeat buyers is low; targeted subscription offers to this group have the clearest upside.
- **Invest in new-customer acquisition** — at 2.13% of the base, the New segment is a bottleneck; the current business is heavily reliant on retaining existing customers rather than growing the funnel.
- **Reassess blanket discounting on outerwear and footwear** — with discount rates near 50% on products like Hats, Sneakers, and Coats, margin impact should be weighed against whether the discount is actually driving incremental volume.
- **Feature top-rated products (Gloves, Sandals, Boots, Hat, Skirt) in marketing** — these categories already have proven customer satisfaction and are a lower-risk focus for campaigns.
- **Prioritize Young Adult and Middle-aged segments in targeting**, as they contribute the most revenue, while monitoring whether Adult/Senior segments are underserved rather than genuinely lower-value.
- **Investigate the male/female revenue gap** — determine whether it reflects category mix, marketing reach, or genuine demand differences before deciding on any rebalancing strategy.

---

## 🚀 Project Workflow

1. Collect and inspect the raw dataset
2. Clean and preprocess data in Python (missing values, renaming, feature engineering)
3. Load cleaned data into MySQL
4. Run SQL queries to answer structured business questions
5. Cross-validate key figures in Excel
6. Build the Power BI data model and DAX measures
7. Design the interactive dashboard
8. Extract insights from SQL results and dashboard visuals
9. Translate insights into business recommendations
10. Document the process in this repository

---

## 📌 Key Learnings

- End-to-end data pipeline design (raw → cleaned → database → dashboard)
- Data cleaning and imputation strategy (category-aware median imputation)
- Feature engineering (binning, categorical-to-numeric mapping)
- SQL: aggregation, subqueries, `CASE` segmentation, and window functions (`ROW_NUMBER() OVER PARTITION BY`)
- Python for data preparation with `pandas` and database integration with `SQLAlchemy`
- Power BI dashboard design: KPI cards, slicers, and DAX measures
- Translating quantitative findings into business-relevant recommendations

---

## 🔮 Future Improvements

- Automate data refresh from source to Power BI
- Add RFM (Recency, Frequency, Monetary) analysis for deeper segmentation
- Build a customer lifetime value (CLV) view
- Add cohort analysis to track retention over time
- Expand the data model beyond a single flat table if additional relational data becomes available
- *(Optional, future scope)* explore predictive modeling for churn or next-purchase likelihood

---

## 📚 Skills Demonstrated

**Data Analysis:** EDA, missing-value handling, feature engineering, segmentation
**SQL:** Aggregations, subqueries, `CASE` logic, window functions, MySQL
**Python:** Pandas, data cleaning, SQLAlchemy database integration
**Excel:** Pivot tables, pivot charts, validation workflows
**Power BI:** Interactive dashboards, slicers, data modeling
**Business Intelligence:** DAX, KPI design, insight-to-recommendation storytelling

---

## 👨‍💻 Author

**Vedant Mahajan**
LinkedIn: [https://www.linkedin.com/in/vedant-mahajan-687b563b1/]
GitHub: [https://github.com/VEX-sudo]

---
