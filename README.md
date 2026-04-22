# 🛍️ Customer Shopping Behavior — Data Analytics Project

> An end-to-end data analytics project analyzing retail customer behavior to uncover trends, improve engagement, and optimize marketing strategy.

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Processing-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-2ECC71?style=for-the-badge)

---

## 📌 Business Problem

> *"How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?"*

A leading retail company wants to better understand its customers' shopping behavior in order to improve sales, customer satisfaction, and long-term loyalty. This project analyzes 3,900 customer transaction records to answer that question across three analytical layers.

---

## 🗂️ Project Structure

```
customer-shopping-behavior/
│
├── 📓 Customer_shopping_behavoir_Analysis.ipynb   # Python: EDA & Preprocessing
├── 🗄️ Customer_shopping_behavior_DB.sql           # MySQL: 10 Business Queries
├── 📊 customer_shopping.pbix                      # Power BI: Interactive Dashboard
├── 📁 customer_shopping_behavior.csv              # Raw Dataset
└── 📄 README.md                                   # Project Documentation
```

---

## 🔧 Tech Stack

| Layer | Tool | Purpose |
|---|---|---|
| **Data Preprocessing** | Python (Jupyter Notebook) | Cleaning, feature engineering, null handling |
| **Data Analysis** | MySQL Workbench 8.0 | 10 business-driven SQL queries |
| **Visualization** | Microsoft Power BI | 2-page interactive dashboard |
| **Libraries** | pandas, numpy, sqlalchemy, pymysql | Data manipulation & DB connection |

---

## 📊 Dataset Overview

| Attribute | Details |
|---|---|
| Records | 3,900 customer transactions |
| Columns | 18 features |
| Target Variable | Purchase Amount (USD) |
| Date Range | Multi-season (Fall, Spring, Winter, Summer) |
| Geography | 50 US States |

**Key Columns:**
`Customer ID` · `Age` · `Gender` · `Item Purchased` · `Category` · `Purchase Amount` · `Location` · `Season` · `Review Rating` · `Subscription Status` · `Shipping Type` · `Discount Applied` · `Previous Purchases` · `Payment Method` · `Frequency of Purchases`

---

## 🐍 Python — Data Preprocessing

### Steps Performed

**1. Data Loading & Exploration**
```python
df = pd.read_csv("customer_shopping_behavior.csv")
df.info()        # 3900 rows × 18 columns
df.describe()    # avg purchase: $59.76, rating: 3.75/5
df.isnull().sum()  # 37 nulls in review_rating only
```

**2. Null Value Treatment**
```python
# Group-wise median imputation by Category
df['review_rating'] = df.groupby('Category')['review_rating'] \
                        .transform(lambda x: x.fillna(x.median()))
```

**3. Column Standardization**
```python
df.columns = df.columns.str.lower().str.replace(' ', '_')
df = df.rename(columns={'purchase_amount_(usd)': 'purchase_amount'})
```

**4. Duplicate Column Detection & Removal**
```python
# discount_applied == promo_code_used (100% identical)
(df['discount_applied'] == df['promo_code_used']).all()  # True
df = df.drop('promo_code_used', axis=1)
```

**5. Feature Engineering**
```python
# Age segmentation using quantile-based binning
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)

# Purchase frequency in days
frequency_mapping = {
    'weekly': 7, 'fortnightly': 14, 'bi-weekly': 14,
    'monthly': 30, 'quarterly': 90, 'annually': 365
}
df['purchase_frequency_days'] = df['frequency_of_purchases'] \
                                  .str.lower().map(frequency_mapping)
```

**6. Export to MySQL**
```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://user:password@host:port/db")
df.to_sql("customer_shopping", con=engine, if_exists="replace", index=False)
```

---

## 🗄️ MySQL — Business Analysis

### 10 SQL Queries

| # | Question | Technique |
|---|---|---|
| Q1 | Revenue by Gender | GROUP BY + SUM |
| Q2 | High-spend discount users | Correlated Subquery |
| Q3 | Top 5 rated products | GROUP BY + AVG + ORDER BY |
| Q4 | Shipping type vs spend | WHERE IN + GROUP BY |
| Q5 | Subscriber vs non-subscriber revenue | GROUP BY + COUNT + AVG |
| Q6 | Products with highest discount rate | CASE WHEN + GROUP BY |
| Q7 | Customer segmentation (New/Returning/Loyal) | CTE + CASE WHEN |
| Q8 | Top 3 products per category | CTE + Window Function (ROW_NUMBER) |
| Q9 | Repeat buyers & subscription link | WHERE + GROUP BY |
| Q10 | Revenue by age group | Engineered column + GROUP BY |

### Sample Query — Customer Segmentation (CTE)
```sql
WITH customer_type AS (
  SELECT customer_id, previous_purchases,
    CASE
      WHEN previous_purchases = 1 THEN 'New'
      WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
      ELSE 'Loyal'
    END AS customer_segment
  FROM customer_shopping
)
SELECT customer_segment, COUNT(*) AS customer_count
FROM customer_type
GROUP BY customer_segment;
```

### Sample Query — Top 3 Products per Category (Window Function)
```sql
WITH item_count AS (
  SELECT category, item_purchased,
    COUNT(customer_id) AS total_orders,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
  FROM customer_shopping
  GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_count
WHERE item_rank <= 3;
```

---

## 📈 Power BI Dashboard

### Page 1 — Customer Trends & Shopping Behavior

| Visual | Insight |
|---|---|
| KPI Cards | 3.9K customers · $233K revenue · $59.76 avg · 3.75 rating · 27% subscribers |
| Seasonal Revenue | Fall $60K (peak) · Summer $56K (weakest) |
| Age Group Distribution | Even spread across Young Adult, Adult, Middle-aged, Senior |
| Top 5 Rated Products | Jewelry 3.8 · Dress/Pants/Blouse 3.7 · Shirt 3.6 |
| Revenue by Category | Clothing $104K · Accessories $74K · Footwear $36K · Outerwear $19K |
| Customer Segment Revenue | Loyal $185K · Returning $43K · New $5K |
| Location Map | Customer distribution across all 50 US states |

### Page 2 — Engagement, Loyalty & Strategy Insights

| Visual | Insight |
|---|---|
| Loyalty Donut | Loyal 79.9% · Returning 17.97% · New 2.13% |
| Subscription Impact | Non-subscribers $170K (73% of base) · Subscribers $63K (27% of base) |
| Discount Effectiveness | No discount $134K · With discount $99K |
| Payment Methods | All 6 methods equally distributed ~16-17% |
| Shipping Type Revenue | All 6 types generate $38–41K (no significant variation) |
| Product Opportunity Matrix | Scatter plot: Avg Rating vs Total Customers by Category |

> **Dashboard Filters:** Location slicer · Subscription Status toggle

---

## 💡 Key Findings

### 1. 🔴 Subscription Program is Under-Penetrated
Only 27% of customers (1,050) are subscribers. Non-subscribers generate $170K simply due to volume (2,850 customers). **Growing the subscriber base is the #1 revenue opportunity.**

### 2. 💛 Loyalty is Strong but Pipeline is Thin
79.9% of customers are loyal (10+ purchases) — excellent retention. However, only 2.13% are new customers, signaling a weak acquisition funnel that threatens future growth.

### 3. 📦 Discounts Don't Drive More Revenue
Non-discounted customers generate $134K vs $99K from discounted customers. **Blanket discounting is unnecessary** — customers are willing to pay full price.

### 4. 🍂 Seasonality Drives Revenue
Fall is the peak season ($60K) and Summer is weakest ($56K). Marketing spend should be concentrated around Fall campaigns.

### 5. 👕 Clothing Dominates Revenue
Clothing ($104K) and Accessories ($74K) together account for **76% of total revenue**. Outerwear ($19K) significantly underperforms.

### 6. 🚚 Shipping Type Has No Revenue Impact
All 6 shipping types generate nearly identical revenue ($38-41K). This is an opportunity to push customers toward lower-cost shipping options without revenue risk.

---

## ✅ Business Recommendations

| Priority | Recommendation |
|---|---|
| 🔴 HIGH | Launch subscription conversion campaign — offer trial, exclusive discounts for new subscribers |
| 🔴 HIGH | Introduce tiered loyalty rewards program (Bronze / Silver / Gold) |
| 🔴 HIGH | Concentrate 35-40% of annual marketing budget on Fall season campaigns |
| 🟡 MEDIUM | Run Summer-exclusive promotions to close the $4K seasonal revenue gap |
| 🟡 MEDIUM | Stop blanket discounting — shift to targeted offers for at-risk customers only |
| 🟡 MEDIUM | Expand Clothing and Accessories SKU range and inventory depth |
| 🟡 MEDIUM | Feature top-rated products (Jewelry, Dress) on homepage and email campaigns |
| 🟢 LOW | Investigate Outerwear underperformance — consider seasonal bundling |
| 🟢 LOW | Promote Free Shipping / Store Pickup to reduce logistics costs |
| 🟢 LOW | Launch referral program to grow the thin new customer pipeline |

---

## 📁 How to Run This Project

### Prerequisites
```bash
pip install pandas numpy sqlalchemy pymysql jupyter
```

### Step 1 — Run Python Notebook
```bash
jupyter notebook Customer_shopping_behavoir_Analysis.ipynb
```

### Step 2 — Set Up MySQL
1. Open MySQL Workbench
2. Create database: `CREATE DATABASE customer_shopping_behavior;`
3. Run `Customer_shopping_behavior_DB.sql` to execute all 10 queries

### Step 3 — Connect Jupyter to MySQL
```python
from sqlalchemy import create_engine
engine = create_engine("mysql+pymysql://root:your_password@localhost:3306/customer_shopping_behavior")
df.to_sql("customer_shopping", con=engine, if_exists="replace", index=False)
```

### Step 4 — Open Power BI Dashboard
Open `customer_shopping.pbix` in Microsoft Power BI Desktop

---

## 📂 File Descriptions

| File | Description |
|---|---|
| `Customer_shopping_behavoir_Analysis.ipynb` | Jupyter notebook with full EDA, preprocessing, and feature engineering |
| `Customer_shopping_behavior_DB.sql` | 10 MySQL business queries with comments |
| `customer_shopping.pbix` | Power BI interactive dashboard (2 pages) |
| `customer_shopping_behavior.csv` | Raw dataset (3,900 rows × 18 columns) |

---

## 🙋 About

**Divya OS** — Aspiring Data Analyst

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com)

---

*⭐ If you found this project helpful, please consider giving it a star!*
