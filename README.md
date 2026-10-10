# Customer Shopping Behavior Analysis

End-to-end data analytics project: clean a retail dataset with **Python**, store it in **MySQL**, and present the results in an interactive **Power BI** dashboard.

![Dashboard](images/dashboard.png)

## Overview

This project analyses **3,900 customer purchases** worth **$233,081** to understand who buys what, how they pay and ship, and which offers matter.

**Main finding:** spending is remarkably even. Average order value stays between about $57 and $62 across every category, age group, gender, season and subscription status. The real differences are in volume: Clothing and Accessories bring 76.5% of revenue, men account for 67.7% of sales, and only 26.9% of revenue comes from subscribers.

## Tech Stack

| Tool | Used for |
| --- | --- |
| Python 3.13 | Main language |
| pandas | Data cleaning and feature engineering |
| SQLAlchemy + PyMySQL | Connecting Python to MySQL |
| MySQL | Storing the cleaned data and running SQL queries |
| Jupyter Notebook | Documenting the analysis |
| Power BI Desktop | Interactive dashboard |

## Pipeline

```
CSV file  ->  Python (pandas)  ->  MySQL  ->  Power BI
```

## Dataset

3,900 rows, one per purchase, and 18 original columns:

| Column | Description |
| --- | --- |
| Customer ID | Unique customer number |
| Age, Gender | Customer demographics (age 18 to 70) |
| Item Purchased, Category | 25 items in 4 categories |
| Purchase Amount (USD) | $20 to $100 |
| Location | 50 US states |
| Size, Color | Product attributes |
| Season | Spring, Summer, Fall, Winter |
| Review Rating | 2.5 to 5.0 |
| Subscription Status | Yes or No |
| Shipping Type, Payment Method | 6 options each |
| Discount Applied, Promo Code Used | Yes or No |
| Previous Purchases | Earlier orders (1 to 50) |
| Frequency of Purchases | Weekly to Annually |

## Data Cleaning and Feature Engineering

1. **Inspected** the data: only `Review Rating` had missing values (37 rows).
2. **Filled missing ratings** with the median rating of each product category.
3. **Standardised column names** to lowercase with underscores (for example `purchase_amount`).
4. **Created age groups** with `pd.qcut` (equal-sized quartiles): Young Adult (18-31), Adult (32-44), Middle Aged (45-57), Senior (58-70).
5. **Converted purchase frequency to days** (Weekly = 7 up to Annually = 365) in a new `frequency_of_days` column.
6. **Dropped `promo_code_used`**, which was identical to `discount_applied` in every row.

The cleaned table has 3,900 rows and 19 columns.

## MySQL Integration

The cleaned DataFrame is written to a `customer` table in a `customer_behavior` database:

```python
from sqlalchemy import create_engine, text

server_engine = create_engine("mysql+pymysql://USER:PASSWORD@localhost:3306")
with server_engine.connect() as conn:
    conn.execute(text("CREATE DATABASE IF NOT EXISTS customer_behavior"))
    conn.commit()

engine = create_engine("mysql+pymysql://USER:PASSWORD@localhost:3306/customer_behavior")
df.to_sql("customer", engine, if_exists="replace", index=False)
```

Example SQL query (top 5 products by average rating):

```sql
SELECT item_purchased,
       ROUND(AVG(review_rating), 2) AS `Average Product Rating`
FROM customer
GROUP BY item_purchased
ORDER BY AVG(review_rating) DESC
LIMIT 5;
```

## Key Findings

| Area | Result |
| --- | --- |
| Total revenue | $233,081 from 3,900 orders |
| Average order / rating | $59.76 / 3.75 out of 5 |
| Top categories | Clothing (44.7%) and Accessories (31.8%) of revenue |
| Weakest category | Outerwear (7.9%) |
| Age groups | Revenue share ranges only from 23.9% to 26.7% |
| Gender | Men 67.7% of revenue, women 32.3%, with similar spend per order |
| Subscribers | 26.9% of revenue; spend and rate the same as non-subscribers |
| Discounts | Used on 42.7% of orders; discounted orders were slightly smaller ($59.28 vs $60.13) |
| Best season | Fall ($60,018, highest average order at $61.56) |
| Top-rated items | Gloves (3.86), Sandals (3.84), Boots (3.82) |

## Power BI Dashboard

A single-page dashboard connected to the MySQL `customer` table:

- **KPI cards:** Average Purchase Amount, Average Review Rating, Number of Customers
- **Filters:** Subscription Status, Gender, Category, Shipping Type
- **Charts:** subscription share (donut), revenue and sales by category, revenue and sales by age group

Every visual responds to the filters.

## Recommendations

1. Grow Outerwear and Footwear, which bring only 7.9% and 15.5% of revenue.
2. Attract more women: they spend as much per order but make fewer purchases.
3. Rework the subscription programme with real member benefits.
4. Target discounts instead of offering them across the board.
5. Plan stock and promotions around Fall, the strongest season.
6. Promote the best-rated items (Gloves, Sandals, Boots).

## Limitations and Future Work

- The data has no purchase dates, so trends over time cannot be measured.
- Differences between segments are small and were not tested for statistical significance.
- Next steps: significance tests, customer clustering (RFM or k-means), and a model to predict subscription status.

## Project Structure

```
.
├── customer_shopping_behavior.csv   # raw dataset
├── notebook.ipynb                   # cleaning, feature engineering, MySQL load
├── customer_behavior.pbix           # Power BI dashboard
├── images/
│   └── dashboard.png                # dashboard screenshot
└── README.md
```

## How to Run

1. Clone the repository:
   ```bash
   git clone https://github.com/samadkhan-18/customer-behavior-analysis.git
   cd customer-behavior-analysis
   ```
2. Install the Python libraries:
   ```bash
   pip install pandas sqlalchemy pymysql jupyter
   ```
3. Start MySQL and open `notebook.ipynb`. Change the `username` and `password` in the connection cell to your own MySQL credentials.
4. Run all cells. This cleans the data and creates the `customer_behavior` database with a `customer` table.
5. Install the [MySQL Connector/NET](https://dev.mysql.com/downloads/connector/net/) driver, open `customer_behavior.pbix` in Power BI Desktop, and refresh the data (server `127.0.0.1:3306`, database `customer_behavior`).

## Author

**Ahemad Maheed Khan**
B.E. CSE (Data Science), Lords Institute of Engineering & Technology (Osmania University)

- GitHub: [samadkhan-18](https://github.com/samadkhan-18)
- LinkedIn: [Ahemad Maheed Khan](https://www.linkedin.com/in/ahemad-maheed-khan-76a0a1301/)
- Portfolio: [samad-portfolio.onrender.com](https://samad-portfolio.onrender.com/)
