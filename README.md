# RetailMart Data Pipeline

Junior Data Engineer — Technical Assignment  
**Jignesh Prajapat | Roll No. 23EJICS071 | GitHub: jignesh7583**

---

## What I Built

RetailMart operates retail stores all across India and every day it collects sales records from each of them. The problem is the raw data is not in a usable state — there are missing values, duplicate entries, wrong column types, and the dates are not in a consistent format. The business team cannot run any reports on this directly.

So this assignment was about writing a Python pipeline that takes that raw messy data, fixes it step by step, combines everything into one clean table, saves it to a database, and finally answers some business questions using SQL.

The whole thing runs with a single function call — `run_pipeline()`.

---

## Folder Contents

```
RetailMart-data-pipeline/
├── pipeline.ipynb       ← Jupyter notebook with all 6 tasks and outputs
├── sales_data.csv       ← Raw transactions file (has duplicates and missing values)
├── products.csv         ← Product names, categories and prices
├── stores.csv           ← Store names, cities and regions
├── retailmart.db        ← SQLite database created automatically when pipeline runs
└── README.md
```

---

## Setup and Running

Install the two libraries needed (sqlite3 and os come built-in with Python):

```
pip install pandas numpy
```

Then just run the notebook top to bottom or call the function directly:

```python
run_pipeline()
```

Everything — cleaning, merging, database creation, and the final report — happens automatically. No extra steps needed.

---

## How the Pipeline Works

The code is divided into separate functions, one for each task. Here is what each one does:

**`load_data()`**  
Opens all three CSV files using pandas and loads them into DataFrames. Prints the shape and first 5 rows of each file. Also prints a null count for every column so you can see exactly where the data problems are before touching anything.

**`clean_data(sales)`**  
This is where the messy data gets fixed. First it removes all exact duplicate rows and prints how many were found. Then missing quantity values are filled with 0 — the assumption being that no quantity recorded means nothing was sold. Rows where the amount column is null are dropped entirely since revenue cannot be calculated without it. Finally the sale_date column is converted to proper datetime and amount is cast to float.

**`transform_data(sales, stores, products)`**  
Merges all three DataFrames into a single wide table using left joins — first on store_id and then on product_id. After merging, a new column called `total_revenue` is added by multiplying quantity with price. NumPy is used to print the mean, max and min of that column. The data is also grouped by city and sorted by total revenue in descending order to show which cities are performing best.

**`load_to_db(merged)`**  
Writes the final cleaned and merged DataFrame into a SQLite database file called `retailmart.db`, inside a table named `retail_sales`. Then runs a SQL query to pull the top 3 best-selling products ranked by total quantity sold.

**`reporting(conn, merged, city_revenue, top3)`**  
Runs a SQL query to get revenue broken down by store and by day. Then prints a plain text summary showing total number of transactions, total revenue, the top performing city, and the top selling product.

**`run_pipeline()`**  
The master function. It first checks that all 3 CSV files are actually present in the folder before doing anything — if any file is missing it raises a FileNotFoundError with a clear message instead of crashing. Then it calls each of the above functions in order. A finally block at the end makes sure the database connection is closed properly even if something goes wrong halfway through.

---

## Data Issues That Were Handled

The sales_data.csv file was set up with real-world style problems on purpose:

- 3 rows with missing quantity → filled with 0
- 2 rows with missing amount → dropped from the dataset
- 3 exact duplicate rows → detected and removed
- sale_date stored as string → converted to datetime
- amount stored as object type → converted to float

---

## SQL Queries Used

**Top 3 products by quantity sold:**

```sql
SELECT product_name, SUM(quantity) AS total_qty_sold
FROM retail_sales
GROUP BY product_name
ORDER BY total_qty_sold DESC
LIMIT 3;
```

**Revenue per store per day:**

```sql
SELECT store_name, DATE(sale_date) AS sale_date, SUM(total_revenue) AS daily_revenue
FROM retail_sales
GROUP BY store_name, DATE(sale_date);
```

---

## Tech Used

- Python 3
- pandas — loading, cleaning, merging and grouping the data
- NumPy — calculating stats on the revenue column
- sqlite3 — saving data to a local database and running SQL queries
- os — checking if required files exist before the pipeline starts
