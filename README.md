<div align="center">

# 🛍️ Customer Shopping Behaviour Analysis


### End-to-end data analytics project using Python, SQL & Power BI

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=for-the-badge&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Power BI](https://img.shields.io/badge/Power_BI-Dashboard-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com)
[![Pandas](https://img.shields.io/badge/Pandas-2.0-150458?style=for-the-badge&logo=pandas&logoColor=white)](https://pandas.pydata.org)
[![Seaborn](https://img.shields.io/badge/Seaborn-Visualisation-4C8CBF?style=for-the-badge)](https://seaborn.pydata.org)
[![Status](https://img.shields.io/badge/Status-Completed-2ea44f?style=for-the-badge)](/)

<br/>

> **Uncovering actionable insights from 3,900 retail transactions** — covering demographics, product performance, payment behaviour, discount strategies, and customer loyalty.

<br/>

| 📋 Records | 💰 Total Revenue | 🛒 Avg. Purchase | ⭐ Avg. Rating |
|:---:|:---:|:---:|:---:|
| **3,900** | **$233,081** | **$59.76** | **3.75 / 5** |

</div>

---

## 📁 Project Structure

```
customer-shopping-analysis/
│
├── 📓 customer_shopping_analysis.ipynb   ← Main Jupyter Notebook (EDA + SQL + Viz)
├── 📊 Dashboard.pbix                     ← Power BI Interactive Dashboard
├── 📄 Customer_Shopping_Report.docx      ← Full Project Report
├── 📂 data/
│   └── customer_shopping_behavior.csv    ← Raw dataset (3,900 rows × 18 cols)
└── 📂 visuals/
    └── *.png                             ← Exported chart images
```

---

## 🎯 Objective

Perform a **comprehensive analysis** of customer shopping transactions to answer:

- Who spends the most — and why?
- Which products drive the most revenue and satisfaction?
- Are discounts attracting quality buyers or eroding margins?
- What drives (or blocks) subscription uptake?
- How do demographics shape buying patterns?

---

## 🔧 Tech Stack

| Layer | Tool |
|---|---|
| **Data Manipulation** | Python · Pandas · NumPy |
| **Visualisation** | Seaborn · Matplotlib |
| **Database** | PostgreSQL 15 via SQLAlchemy |
| **Dashboard** | Microsoft Power BI |
| **Environment** | Jupyter Notebook |
| **Reporting** | MS Word (docx) |

---

## 🧹 Data Cleaning & Feature Engineering

| Step | What Was Done |
|---|---|
| **Missing Values** | `review_rating` (37 nulls) imputed using **category-wise median** |
| **Column Names** | Standardised to `snake_case` using `.str.lower()` + `.str.replace()` |
| **Rename** | `purchase_amount_(usd)` → `purchase_amount` |
| **New Feature** | `age_group` — 4 quartile bins: `Young-Adult / Adult / Middle-Aged / Senior` |
| **New Feature** | `purchase_frequency_days` — text frequency mapped to numeric days (7/14/30/90/365) |
| **Redundancy** | `promo_code_used` dropped (100% identical to `discount_applied`) |

---

## 🗃️ SQL Analysis (PostgreSQL)

Data was loaded into PostgreSQL using `SQLAlchemy` and queried with `pd.read_sql()`.

<details>
<summary><b>📌 Q1 — Total Revenue by Gender</b></summary>

```sql
SELECT gender, SUM(purchase_amount) AS total_revenue
FROM customer
GROUP BY gender;
```
| Gender | Revenue |
|---|---|
| Male | $157,890 |
| Female | $75,191 |

> **Insight:** Males contribute ~68% of total revenue.
</details>

<details>
<summary><b>📌 Q2 — Discount Users Who Spent Above Average</b></summary>

```sql
SELECT customer_id, purchase_amount
FROM customer
WHERE discount_applied = 'Yes'
  AND purchase_amount >= (SELECT AVG(purchase_amount) FROM customer);
```
> **838 customers** used a discount and still spent above the $59.76 average — discounts attract quality buyers.
</details>

<details>
<summary><b>📌 Q3 — Top 5 Products by Avg Review Rating</b></summary>

```sql
SELECT item_purchased, ROUND(AVG(review_rating), 2) AS avg_rating
FROM customer
GROUP BY item_purchased
ORDER BY avg_rating DESC
LIMIT 5;
```
| Rank | Product | Avg Rating |
|---|---|---|
| 1 | Gloves | 3.86 |
| 2 | Sandals | 3.84 |
| 3 | Boots | 3.82 |
| 4 | Hat | 3.80 |
| 5 | Skirt | 3.78 |
</details>

<details>
<summary><b>📌 Q4 — Standard vs Express Shipping Spend</b></summary>

```sql
SELECT shipping_type, ROUND(AVG(purchase_amount), 2)
FROM customer
WHERE shipping_type IN ('Standard', 'Express')
GROUP BY shipping_type;
```
| Shipping | Avg Spend |
|---|---|
| Standard | $60.48 |
| Express | $58.46 |
</details>

<details>
<summary><b>📌 Q5 — Subscription vs Non-Subscription</b></summary>

```sql
SELECT subscription_status,
       COUNT(customer_id)             AS total_customers,
       ROUND(AVG(purchase_amount), 2) AS avg_spend,
       ROUND(SUM(purchase_amount), 2) AS total_revenue
FROM customer
GROUP BY subscription_status;
```
| Status | Customers | Avg Spend | Total Revenue |
|---|---|---|---|
| Subscribed | 1,053 | $59.49 | $62,645 |
| Not Subscribed | 2,847 | $59.87 | $170,436 |

> Subscription programme has **no spend uplift** — value proposition needs strengthening.
</details>

<details>
<summary><b>📌 Q6 — Top 5 Products by Discount Rate</b></summary>

```sql
SELECT item_purchased,
       ROUND(100.0 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```
| Product | Discount Rate |
|---|---|
| Hat | 50.00% |
| Sneakers | 49.66% |
| Coat | 49.07% |
| Sweater | 48.17% |
| Pants | 47.37% |
</details>

<details>
<summary><b>📌 Q7 — Customer Segmentation</b></summary>

```sql
WITH customer_type AS (
  SELECT customer_id, previous_purchases,
    CASE
      WHEN previous_purchases = 1       THEN 'New'
      WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
      ELSE 'Loyal'
    END AS customer_segment
  FROM customer
)
SELECT customer_segment, COUNT(*) AS count
FROM customer_type
GROUP BY customer_segment;
```
| Segment | Count | Share |
|---|---|---|
| New | 83 | 2% |
| Returning | 701 | 18% |
| **Loyal** | **3,116** | **80%** |
</details>

<details>
<summary><b>📌 Q8 — Top 3 Products per Category</b></summary>

```sql
WITH item_counts AS (
  SELECT category, item_purchased,
         COUNT(customer_id) AS total_orders,
         ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
  FROM customer GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts WHERE item_rank <= 3;
```
</details>

<details>
<summary><b>📌 Q9 — Repeat Buyers & Subscriptions</b></summary>

Of customers with **5+ prior purchases**: 72% are **non-subscribers** — repeat buying does not convert to subscriptions.
</details>

<details>
<summary><b>📌 Q10 — Revenue by Age Group</b></summary>

| Age Group | Total Revenue |
|---|---|
| Young-Adult | $62,143 |
| Middle-Aged | $59,197 |
| Adult | $55,978 |
| Senior | $55,763 |

> Revenue is distributed **evenly** across all age groups — broad demographic appeal confirmed.
</details>

---

## 📊 Python Visualisations

10 charts produced using **Seaborn** & **Matplotlib**:

| # | Chart | Type | Key Takeaway |
|---|---|---|---|
| 1 | Total Revenue by Gender | Bar | Males ~68% of revenue |
| 2 | Top 10 Items by Revenue | H. Bar | Blouse, Shirt, Dress lead |
| 3 | Purchase Share by Category | Pie | Clothing dominates (~40%) |
| 4 | Purchase Amount Distribution | Histogram + KDE | Uniform spread $20–$100 |
| 5 | Season-wise Revenue | Bar | Spring & Fall slightly higher |
| 6 | Avg Purchase by Age Group | Bar | All groups ~$59–$61 |
| 7 | Payment Method Distribution | Count Plot | Cash, Credit Card, Venmo top 3 |
| 8 | Avg Purchase — Category × Gender | Grouped Bar | Minimal gender gap |
| 9 | Review Rating by Category | Box Plot | Consistent 3.5–4.0 across all |
| 10 | Revenue by Season & Subscription | Grouped Bar | Non-subscribers dominate |

---

## 📈 Power BI Dashboard

An interactive **Power BI dashboard** (`Dashboard.pbix`) was built for stakeholder-facing exploration:

- 🔢 **KPI Cards** — Revenue, Avg Purchase, Customers, Avg Rating  
- 🍩 **Revenue by Gender** — Donut chart with slicer  
- 📊 **Top Products** — Ranked bar chart filterable by category & season  
- 👥 **Customer Segments** — Loyal / Returning / New breakdown  
- 💳 **Payment Methods** — Treemap view  
- 🗓️ **Season × Subscription** — Clustered bar matrix  

---

## 💡 Key Insights & Recommendations

| # | Insight | Recommendation |
|---|---|---|
| 1 | Males drive 68% of revenue | Target female customer acquisition campaigns |
| 2 | 80% customers are Loyal | Launch a VIP rewards / loyalty programme |
| 3 | Only 27% subscribed | Offer free shipping or exclusives to drive subscriptions |
| 4 | Hat & Sneakers discounted ~50% | Introduce tiered discounts; reduce blanket promotions |
| 5 | All age groups spend ~$60 | Use product recommendations over price segmentation |
| 6 | Gloves & Sandals highest rated | Feature in homepage banners & marketing assets |
| 7 | 838 discount users > avg spend | Maintain targeted discounts on high-margin items only |

---

## 🚀 How to Run

### 1. Clone the repository
```bash
git clone https://github.com/your-username/customer-shopping-analysis.git
cd customer-shopping-analysis
```

### 2. Install dependencies
```bash
pip install pandas numpy matplotlib seaborn sqlalchemy psycopg2-binary jupyter
```

### 3. Set up PostgreSQL
```bash
# Create the database
createdb customer_behaviour

# Update credentials in the notebook
username = 'your_username'
password = 'your_password'
host     = 'localhost'
port     = '5432'
database = 'customer_behaviour'
```

### 4. Launch Jupyter Notebook
```bash
jupyter notebook customer_shopping_analysis.ipynb
```

### 5. Open Power BI Dashboard

Open `Dashboard.pbix` in **Microsoft Power BI Desktop** (free download at powerbi.microsoft.com).

---

## 📦 Dependencies

```txt
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
seaborn>=0.12
sqlalchemy>=2.0
psycopg2-binary>=2.9
jupyter>=1.0
```

---

## 🗺️ Project Workflow

```
Raw CSV Data
     │
     ▼
Data Cleaning & Feature Engineering (Python / Pandas)
     │
     ├──► EDA & Visualisations (Seaborn / Matplotlib)
     │
     ├──► PostgreSQL Database (SQLAlchemy)
     │         │
     │         └──► SQL Business Queries (10 Questions)
     │
     └──► Power BI Dashboard (Interactive Reporting)
               │
               └──► Project Report (Word Document)
