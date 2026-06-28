# RetailMart Data Pipeline — Assignment Submission

This is my submission for the Junior Data Engineer technical assignment given by RetailMart Pvt. Ltd. The task was to build a small data pipeline that takes raw, messy CSV files and cleans them up so the business team can use the data for reporting.

I have tried to complete all 6 tasks as mentioned in the problem statement. Below I have explained everything — what files I created, how to run the code, and what each part of the code is doing.

---

## What This Project Does

RetailMart collects sales data daily from all its stores but the data is not clean. There are duplicate rows, missing values, and wrong data types. My job was to:

- Load the raw data from 3 CSV files
- Clean it (remove duplicates, fix nulls, fix data types)
- Merge all 3 files into one combined dataset
- Calculate revenue and find which city and product is performing best
- Save everything into a SQLite database
- Write SQL queries to answer business questions
- Wrap the whole thing in one function with proper error handling

---

## Files in This Project

```
retailmart/
│
├── sales_data.csv          ← raw daily transactions (has duplicates and missing values on purpose)
├── products.csv            ← product names, categories and prices
├── stores.csv              ← store names, cities and regions
│
├── retailmart_pipeline.py  ← main Python script with all 6 tasks
├── retailmart.db           ← this gets created automatically when you run the script
└── README.md               ← this file
```

---

## About the CSV Files

**sales_data.csv** — This is the main transactions file. It has 47 rows total out of which 3 are duplicate rows and some rows have missing quantity or amount values. I kept these issues intentionally as the assignment asked us to simulate real-world messy data. Columns are: `sale_id`, `store_id`, `product_id`, `quantity`, `sale_date`, `amount`

**products.csv** — Has 5 products with their name, category and price. Columns are: `product_id`, `product_name`, `category`, `price`

**stores.csv** — Has 5 stores from different cities across India. Columns are: `store_id`, `store_name`, `city`, `region`

---

## How to Run

First make sure you have Python installed. Then install the required libraries by running this in your terminal:

```
pip install pandas numpy sqlalchemy
```

After that just put all the files in the same folder and run:

```
python retailmart_pipeline.py
```

That's it. The script will do everything on its own — load the files, clean the data, merge it, save to database and print the final report. The `retailmart.db` file will also get created in the same folder automatically.

---

## How the Code is Organized

I split the code into separate functions so it is easier to read and debug. Each function handles one task.

**load_data()** — reads all 3 CSV files and prints the shape and first 5 rows of each. Also checks and prints which columns have missing values.

**clean_data()** — this is where the actual cleaning happens. First it removes duplicate rows and tells you how many were found. Then it fills missing quantity values with 0 (because if quantity is missing it probably means nothing was sold). Rows where amount is missing are dropped completely because we cannot calculate revenue without the amount. Finally it converts sale_date to proper datetime format and amount to float.

**transform_data()** — merges all 3 DataFrames into one using left joins on store_id and product_id. Then adds a new column called `total_revenue` which is just quantity multiplied by price. Uses numpy to print the mean, max and min of total_revenue. Also groups the data by city to show which city is generating the most revenue.

**load_to_db()** — saves the final merged DataFrame into a SQLite database file called `retailmart.db` inside a table named `retail_sales`. Then runs a SQL query to find the top 3 best selling products by total quantity.

**reporting()** — runs one more SQL query to get revenue per store per day. Then prints a simple summary report showing total transactions, total revenue, top city and top product.

**run_pipeline()** — this is the main function that calls everything in order. It also checks if all 3 CSV files exist before starting, so if any file is missing it will print a proper error message instead of crashing with a traceback.

---

## Error Handling

I added try-except inside run_pipeline() to handle 3 types of errors:

- If any CSV file is missing, it catches FileNotFoundError and prints a message telling you which file is missing
- If a CSV file exists but is completely empty, it catches EmptyDataError
- For anything else unexpected, there is a general except block that prints the error type and message

There is also a finally block at the end which makes sure the database connection gets closed no matter what happens — even if the script crashes halfway through.

---

## SQL Queries I Used

To find top 3 best selling products:
```sql
SELECT product_name, SUM(quantity) AS total_qty_sold
FROM retail_sales
GROUP BY product_name
ORDER BY total_qty_sold DESC
LIMIT 3;
```

To find revenue per store per day:
```sql
SELECT store_name, DATE(sale_date) AS sale_date, SUM(total_revenue) AS daily_revenue
FROM retail_sales
GROUP BY store_name, DATE(sale_date)
ORDER BY store_name, sale_date;
```

---

## Libraries Used

- **pandas** — for loading CSVs, cleaning data, merging DataFrames and running SQL queries
- **numpy** — for calculating mean, max and min of total_revenue
- **sqlite3** — for connecting to the SQLite database and saving the data
- **os** — just used to check if the CSV files exist before trying to open them

---

## Notes

- The data I used is sample/dummy data that I created myself for this assignment. Store names are from real Indian cities (Mumbai, Delhi, Bangalore, Kolkata, Hyderabad) and product names are common retail items to make it look realistic.
- The sale dates go from 10th January 2024 to 22nd January 2024.
- I tested the full pipeline and it runs without any errors.

