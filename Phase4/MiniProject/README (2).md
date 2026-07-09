# Business Pipeline & Analytics — Mini Project

An end-to-end PySpark pipeline that cleans raw sales data, derives business
metrics, segments customers, and writes a final report.

## Project structure

```
.
├── data/
│   └── generate_sample_data.py   # creates data/sales.csv (sample input)
├── output/
│   └── report/                   # written by the pipeline (Task 7)
├── pipeline.py                   # the pipeline itself (Tasks 1-7)
├── requirements.txt
└── README.md
```

## Setup

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Run

```bash
# 1. Generate sample input data (skip this if you have your own sales.csv
#    with columns: order_id, customer_id, customer_name, city, order_date, amount)
python data/generate_sample_data.py

# 2. Run the pipeline
python pipeline.py
```

The pipeline prints the result of each task to the console, then writes the
final report table to `output/report/` as CSV.

## Pipeline stages

1. **Load** — read `data/sales.csv` into a Spark DataFrame.
2. **Clean**
   - drop rows with a null `customer_id`
   - drop exact duplicate rows
   - filter out invalid (negative) `amount` values
   - cast columns to their expected types (`customer_id`→int, `amount`→double,
     `order_date`→date)
3. **Task 1 — Daily Sales**: sum of `amount` grouped by `order_date`.
4. **Task 2 — City-wise Revenue**: sum of `amount` grouped by `city`.
5. **Task 3 — Top 5 Customers**: top 5 customers by total spend.
6. **Task 4 — Repeat Customers**: customers with more than one order.
7. **Task 5 — Customer Segmentation**: buckets each customer into
   Gold (>10,000), Silver (5,000–10,000), or Bronze (<5,000) based on total
   spend.
8. **Task 6 — Final Reporting Table**: joins spend, segment, order count, and
   each customer's most frequent city into one row per customer.
9. **Task 7 — Save Output**: writes the final table to `output/report/`.

## Reflection

**Why is cleaning done before joining tables?**
Joins multiply the effect of bad data. A null or duplicate key on one side of
a join either drops rows unexpectedly or creates duplicate matches on the
other side, so any downstream sum or count becomes wrong. It's cheaper and
clearer to fix that once, before the join, than to debug a distorted result
after several joins have already combined the tables.

**What would go wrong if null keys are not removed?**
Rows with a null `customer_id` can't be reliably matched to a customer. In an
inner join those rows are silently dropped, undercounting activity; in an
outer join they can produce a single "unknown" bucket that mixes unrelated
orders together, corrupting aggregates like total spend or order count for
no real customer.

**How did you decide join order?**
Start with the table that has the most reliable, deduplicated keys (here,
cleaned orders keyed by `customer_id`), then join in derived aggregates
(spend, segment, primary city) that only make sense once the base table is
already clean. Doing the cheap filters first also reduces the amount of data
that has to flow into the more expensive joins and aggregations later.

**Which step was most difficult and why?**
Building the final reporting table (Task 6) was the trickiest step, because
a customer can order from more than one city. Simply joining orders to
segmentation data by `customer_name` would multiply rows per customer once
city is involved, so a window function (`row_number` over orders per city,
ranked by frequency) was needed to pick a single representative city per
customer before joining everything together.

**How is SQL logic similar to PySpark?**
PySpark's DataFrame API mirrors relational logic almost one-to-one:
`.filter()` is `WHERE`, `.groupBy().agg()` is `GROUP BY`, `.join()` is `JOIN`,
and `Window` functions correspond to SQL window functions (`ROW_NUMBER() OVER
(...)`). The mental model — filter, aggregate, join, window — transfers
directly; the difference is mostly syntax and Spark's lazy execution model.

**What challenges will appear with large data?**
Shuffles become expensive: `groupBy` and `join` on high-cardinality keys move
large amounts of data across the cluster, and skewed keys (a few customers
with disproportionately many orders) create straggler tasks. Memory pressure
grows too — caching a large cleaned dataset, as this pipeline does, needs to
be weighed against available cluster memory, and small-file output problems
appear if partitioning isn't managed when writing the final report.

**Can you explain your pipeline in simple steps?**
Load the raw sales file → throw out incomplete, duplicate, or invalid rows →
compute daily sales, revenue by city, and the top 5 customers → count each
customer's orders and flag repeat customers → total each customer's spend and
label them Gold, Silver, or Bronze → combine all of that into one summary row
per customer → save that summary as the final report.
