# SQL to PySpark — Phase 2 Bridge Pack

Sample data plus a runnable solution for all 7 guided exercises, each shown
in SQL and PySpark side by side.

## Files

```
customers.csv    # customer_id, customer_name, city  (1 row has a blank customer_id)
orders.csv       # order_id, customer_id, amount     (1 row has a blank customer_id)
solution.py       # loads data, cleans it, solves all 7 exercises in SQL + PySpark
requirements.txt
```

`customers.csv` and `orders.csv` are set up so every exercise has something
real to show: customers 7 and 8 have no orders (Exercise 3), customer 1 has
3 orders (Exercise 6), and one row in each file has a missing `customer_id`
for the cleaning step to remove.

## Setup

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Run

```bash
python solution.py
```

For each exercise, the script prints the SQL result first (run via
`spark.sql()` against temp views), then the equivalent PySpark DataFrame
result, so you can compare the two outputs directly.

## Exercises covered

1. Total order amount for each customer
2. Top 3 customers by total spend
3. Customers with no orders
4. City-wise total revenue
5. Average order amount per customer
6. Customers with more than one order
7. Sort customers by total spend descending (includes zero-order customers)

## Notes on the SQL ↔ PySpark mapping

| SQL | PySpark |
|---|---|
| `JOIN ... ON` | `.join(other, on="col", how="inner")` |
| `LEFT JOIN` | `.join(other, on="col", how="left")` |
| `WHERE` | `.filter(...)` |
| `GROUP BY` | `.groupBy(...)` |
| `SUM()`, `AVG()`, `COUNT()` | `F.sum()`, `F.avg()`, `F.count()` inside `.agg()` |
| `HAVING` | `.filter(...)` applied *after* `.agg()` |
| `ORDER BY ... DESC` | `.orderBy(F.desc("col"))` |
| `COALESCE()` | `F.coalesce()` |

Exercise 3 and Exercise 7 both need a `LEFT JOIN` (not an inner join),
because they specifically care about customers who have **no** matching
rows in `orders` — an inner join would drop those customers entirely.
