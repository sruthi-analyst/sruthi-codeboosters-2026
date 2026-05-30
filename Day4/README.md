# Day 4 — Big Data + PySpark + Modern Data Architecture
### Codeboosters Tech — Data Engineering + GenAI Internship

---

## Continuity from Days 1–3
- Days 1–3 processed 30–5000 rows using **Pandas**
- Today we use **Apache Spark** — the industry standard for millions/billions of rows
- Same sales data, same cleaning concepts — just at a completely different scale

---

## The Big Data 5 Vs

| V | What It Means | Example |
|---|---------------|---------|
| **Volume** | How MUCH data | Flipkart: 8 million orders/day during sale |
| **Velocity** | How FAST data arrives | Ola: 1 million GPS pings per second |
| **Variety** | How MANY formats | Zomato: text + images + GPS + JSON payments |
| **Veracity** | How TRUSTWORTHY | Paytm: detecting fraudulent transactions |
| **Value** | How USEFUL | IRCTC: predictive pricing from 80M searches/day |

**When is it Big Data?** When one machine cannot process it in acceptable time.

---

## Pandas vs PySpark — When to Use Which

| | Pandas | PySpark |
|--|--------|---------|
| **Data size** | Up to a few GB (fits in RAM) | TB to PB (distributed across many machines) |
| **Speed** | Fast for small data | Fast for large data |
| **Setup** | `import pandas as pd` | Install + SparkSession required |
| **Operations** | Immediate (eager) | Lazy — builds a plan, executes on action |
| **Use when** | Single machine, <1GB | Multiple machines, >1GB |

---

## PySpark Quick Start

```python
# ── INSTALL (run once in Colab) ──
!pip install pyspark --quiet
# If Java error: !apt-get install -y default-jdk

# ── IMPORTS ──
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.functions import col, year, month, to_date

# ── CREATE SESSION ──
spark = SparkSession.builder \
    .appName("MyApp") \
    .getOrCreate()
# Always .getOrCreate() — never create twice

print(spark.version)   # Verify it works

# ── LOAD DATA ──
df = spark.read \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .csv("large_sales_data.csv")

# ── INSPECT ──
df.count()          # ACTION — total rows
df.printSchema()    # Column names and types
df.show(5)          # ACTION — display first 5 rows
print(df.columns)   # List of column names
```

---

## Transformations vs Actions

```
TRANSFORMATIONS (LAZY — no execution, just planning)
──────────────────────────────────────────────────────
df.select("col1", "col2")              # Pick columns
df.filter(col("revenue") > 50000)      # Filter rows
df.filter((col("cat")=="Electronics") & (col("rev")>40000))
df.withColumn("new_col", expression)   # Add/modify column
df.groupBy("category")                 # Group rows
df.orderBy("revenue", ascending=False) # Sort
df.dropDuplicates()                    # Remove exact duplicates
df.dropna(subset=["order_id"])         # Drop null rows
df.limit(10)                           # Keep top n rows
df.drop("col")                         # Remove a column

ACTIONS (TRIGGER execution)
──────────────────────────────────────────────────────
df.show(10)            # Print rows to screen
df.count()             # Return total row count
df.first()             # Return first row
df.collect()           # Return ALL rows — dangerous on big data!
df.toPandas()          # Convert to Pandas DataFrame
df.write.parquet(path) # Save to Parquet
df.write.csv(path)     # Save to CSV
```

**Rule:** Chain many transformations (free). Call ONE action at the end.

---

## Core PySpark Operations Cheat Sheet

```python
# ── SELECT columns ──
df.select("product", "revenue", "city").show(5)

# ── FILTER rows ──
df.filter(col("revenue") > 50000).show()
df.filter((col("category") == "Electronics") &
          (col("revenue") > 40000)).show()
# Each condition in its own parentheses; & for AND, | for OR

# ── GROUP BY + aggregate ──
df.groupBy("category") \
  .agg(
      F.sum("revenue").alias("total_rev"),
      F.count("order_id").alias("num_orders"),
      F.avg("revenue").alias("avg_rev"),
      F.countDistinct("customer_name").alias("unique_customers")
  ) \
  .orderBy("total_rev", ascending=False) \
  .show()

# ── ADD calculated column ──
df = df.withColumn(
    "revenue_category",
    F.when(col("revenue") > 40000, "High")
     .when(col("revenue") > 10000, "Medium")
     .otherwise("Low")
)

# ── PARSE dates ──
df = df.withColumn("order_date", to_date(col("order_date"), "yyyy-MM-dd"))
df = df.withColumn("order_year",  year(col("order_date")))
df = df.withColumn("order_month", month(col("order_date")))

# ── SAVE as Parquet ──
df.write.mode("overwrite").parquet("output.parquet")

# ── READ Parquet ──
df2 = spark.read.parquet("output.parquet")
# No inferSchema needed — schema is embedded in Parquet

# ── CONVERT to Pandas (only after aggregation!) ──
result_pd = df.groupBy("region").agg(F.sum("revenue")).toPandas()

# ── STOP session when done ──
spark.stop()
```

---

## Aggregate Functions Reference

| Function | What It Does | Example |
|----------|-------------|---------|
| `F.sum("col")` | Total sum | `F.sum("revenue").alias("total")` |
| `F.count("col")` | Count rows | `F.count("order_id").alias("orders")` |
| `F.avg("col")` | Average | `F.avg("revenue").alias("avg_rev")` |
| `F.min("col")` | Minimum value | `F.min("revenue").alias("min_rev")` |
| `F.max("col")` | Maximum value | `F.max("revenue").alias("max_rev")` |
| `F.countDistinct("col")` | Unique count | `F.countDistinct("customer")` |
| `spark_round(F.avg("col"), 2)` | Rounded average | Round to 2 decimal places |
| `F.lit(value)` | Constant value | `F.lit(100)` for division |

---

## CSV vs Parquet Comparison

| Feature | CSV | Parquet |
|---------|-----|---------|
| Storage format | Row-based | Columnar |
| Compression | None (plain text) | Snappy (~60–90% smaller) |
| Schema | Not stored | Embedded in file |
| Query speed | Reads all columns | Reads only needed columns |
| Human readable | Yes (open in Notepad) | No (binary) |
| Best for | Small data, data sharing | Big data, analytics workloads |

```python
# File size comparison
import os

def dir_size_kb(path):
    if os.path.isfile(path):
        return os.path.getsize(path) / 1024
    return sum(os.path.getsize(os.path.join(dp,f))
               for dp,_,files in os.walk(path) for f in files) / 1024

print(f"CSV    : {dir_size_kb('large_sales_data.csv'):.1f} KB")
print(f"Parquet: {dir_size_kb('sales_bronze.parquet'):.1f} KB")
```

---

## Medallion Architecture

```
SOURCE DATA (raw CSV/JSON/API)
        |
        v
  ┌─────────────────────────────┐
  │  BRONZE LAYER               │  ← Raw data, no modifications
  │  sales_bronze.parquet       │    Append-only, source of truth
  └─────────────────────────────┘
        |
        | Clean + Enrich
        v
  ┌─────────────────────────────┐
  │  SILVER LAYER               │  ← Cleaned, validated, enriched
  │  sales_silver.parquet       │    Nulls fixed, types correct
  └─────────────────────────────┘
        |
        | Aggregate
        v
  ┌─────────────────────────────┐
  │  GOLD LAYER                 │  ← Pre-aggregated for analytics
  │  gold_region_revenue.parquet│    Fast dashboards, BI tools
  │  gold_product_summary.parquet
  │  gold_monthly_trend.parquet │
  └─────────────────────────────┘
```

| Layer | Contains | Format | Users |
|-------|---------|--------|-------|
| Bronze | Raw data as-is | CSV/JSON original | Data Engineers |
| Silver | Cleaned + enriched | Parquet | Data Scientists, ML |
| Gold | Pre-aggregated tables | Parquet | Analysts, Dashboards |

---

## Data Storage Architecture

| | Data Warehouse | Data Lake | Lakehouse |
|--|---------------|-----------|-----------|
| **Schema** | Fixed upfront | Applied at read | Both |
| **Data types** | Structured only | All types | All types |
| **Speed** | Very fast | Slower | Fast |
| **Cost** | High | Low | Medium |
| **Examples** | Snowflake, Redshift | AWS S3, ADLS | Databricks, Iceberg |

---

## Common Errors — Day 4

| Error | Cause | Fix |
|-------|--------|-----|
| `ModuleNotFoundError: pyspark` | Not installed | `!pip install pyspark --quiet` |
| `JAVA_HOME not set` | Java missing | `!apt-get install -y default-jdk` then reinstall |
| `Path does not exist` | CSV not uploaded | Upload `large_sales_data.csv` to Colab Files |
| `Multiple SparkContexts` | Session created twice | Use `.getOrCreate()` — already in Cell 2 |
| `Out of Memory` after `.collect()` | Brought all data to driver | Use `.show(20)` or `.limit(100).toPandas()` |
| `AnalysisException: column not found` | Wrong column name | Check `df.columns` for exact spelling |
| GroupBy result has `sum(revenue)` as name | Missing `.alias()` | Add `.alias("total_revenue")` after `F.sum()` |

---

## Practice Questions

1. Explain the 5 Vs of Big Data using Flipkart as an example.
2. What is the difference between a Transformation and an Action in PySpark?
3. Write PySpark code to find the top 5 cities by total revenue.
4. Why is Parquet preferred over CSV for Big Data? Give three reasons.
5. Explain the Medallion Architecture — what lives in each layer?
6. What is the difference between a Data Lake and a Data Warehouse?

---

## Day 4 Completion Checklist

- [ ] `spark.version` printed successfully (Cell 2)
- [ ] `df_bronze.count()` returns 5000
- [ ] `sales_bronze.parquet` saved
- [ ] `sales_silver.parquet` saved with new columns
- [ ] At least one Gold Parquet table saved
- [ ] `big_data_dashboard.png` visible in output
- [ ] Notebook uploaded to GitHub
- [ ] Commit: `Add Day 4: Big Data PySpark Medallion Pipeline`

---

## Day 5 Preview
**Machine Learning for Data Engineers — Phase 1 Finale**
- `student_performance.csv` from **Day 1** returns as ML training data!
- Supervised learning: Linear Regression to predict student scores
- Complete pipeline: Raw CSV → ETL → Feature Engineering → Train → Predict
- Phase 1 Final Mini Project: Student Analytics OR Weather Intelligence System

---
*Codeboosters Tech | Data Engineering + GenAI Internship | Day 4 of 10*
