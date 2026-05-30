# Day 3 — ETL Pipelines + Pandas Data Cleaning + APIs
### Codeboosters Tech — Data Engineering + GenAI Internship

---

## Continuity from Days 1 & 2
- Day 1: Loaded `student_performance.csv` with Pandas
- Day 2: Stored it in SQLite, queried with SQL, visualized
- Day 3: Learn WHERE clean data comes from — the ETL pipeline

---

## ETL at a Glance

```
EXTRACT     →  Pull data from source (CSV file, API, database)
TRANSFORM   →  Clean it (fix nulls, duplicates, formats, types)
LOAD        →  Save to destination (CSV, database, warehouse)
```

**Water treatment analogy:** River (raw source) → Treatment plant (transform) → Water tank (load) → Homes (users)

| | ETL | ELT |
|--|-----|-----|
| **Order** | Transform before loading | Load first, transform inside warehouse |
| **Where transform happens** | Python / Pandas | SQL inside cloud DB |
| **When to use** | Relational DBs, Python pipelines | Cloud warehouses (Snowflake, BigQuery) |

---

## Pandas Data Cleaning — Essential Commands

```python
import pandas as pd

# ── ALWAYS start with a copy ──
df = raw_df.copy()          # Never modify raw data directly

# ── DETECT problems ──
df.isnull().sum()           # Count missing values per column
df.duplicated().sum()       # Count duplicate rows
df.dtypes                   # Check data types
df['col'].unique()          # Check unique values for inconsistencies

# ── FIX missing values ──
df['col'].fillna(df['col'].median(), inplace=True)   # Fill numbers with median
df['col'].fillna('Unknown', inplace=True)            # Fill text with label
df.dropna(subset=['order_id'], inplace=True)         # Drop rows where ID is null

# ── REMOVE duplicates ──
df.drop_duplicates(inplace=True)                                      # Remove exact duplicates
df.drop_duplicates(subset=['customer', 'product'], inplace=True)     # Targeted dedup

# ── FIX data types ──
df['qty']   = pd.to_numeric(df['qty'],   errors='coerce')  # String → number
df['price'] = pd.to_numeric(df['price'], errors='coerce')  # errors='coerce' → NaN not crash
df['qty']   = df['qty'].astype(int)                        # Float → integer

# ── PARSE dates ──
df['date'] = pd.to_datetime(df['date'], errors='coerce')   # String → datetime
df['year']  = df['date'].dt.year                           # Extract year
df['month'] = df['date'].dt.month                          # Extract month
df['day']   = df['date'].dt.strftime('%A')                 # Extract day name

# ── FIX text ──
df['name'] = df['name'].str.strip().str.title()   # Remove spaces + Title Case
df['col']  = df['col'].str.upper()                # All caps
df['col']  = df['col'].str.lower()                # All lowercase

# ── FIX specific cells with boolean mask ──
mask = (df['product'] == 'Keyboard') & (df['category'] == 'Electronics')
df.loc[mask, 'category'] = 'Accessories'
# df.loc[row_condition, column] = new_value

# ── CREATE new columns ──
df['revenue'] = df['quantity'] * df['unit_price']

# ── AGGREGATE cleaned data ──
summary = df.groupby('category').agg(
    total_revenue = ('revenue', 'sum'),
    avg_order     = ('revenue', 'mean'),
    num_orders    = ('order_id', 'count')
).round(2).reset_index()

# ── SAVE clean data ──
df.to_csv('clean_data.csv', index=False)
```

---

## Missing Values — Decision Guide

| Situation | Strategy |
|-----------|----------|
| Numeric, few nulls (<5%) | `fillna(column.median())` |
| Numeric, many nulls (>30%) | Investigate — may drop column |
| Text / category column | `fillna('Unknown')` |
| ID / primary key column | `dropna(subset=['id'])` — drop row |
| Date column | `dropna()` or fill forward |

---

## APIs — Quick Reference

```python
import requests

# ── Setup ──
API_KEY  = 'your_key_here'
BASE_URL = 'https://api.openweathermap.org/data/2.5/weather'

# ── Make a GET request ──
params   = {'q': 'Mumbai', 'appid': API_KEY, 'units': 'metric'}
response = requests.get(BASE_URL, params=params, timeout=10)

# ── Check status ──
print(response.status_code)
# 200 = OK    401 = Bad key    404 = Not found    429 = Rate limit

# ── Parse JSON ──
if response.status_code == 200:
    data = response.json()          # JSON text → Python dict
    temp = data['main']['temp']     # Navigate nested dict
    desc = data['weather'][0]['description']   # Navigate list inside dict
    hum  = data['main'].get('humidity', 0)     # Safe access with default

# ── Convert API results to DataFrame ──
records = [{'city': 'Mumbai', 'temp': 32.5}, {'city': 'Delhi', 'temp': 38.2}]
df = pd.DataFrame(records)   # List of dicts → DataFrame
```

**HTTP Status Codes:**
| Code | Meaning | Action |
|------|---------|--------|
| 200 | Success | Proceed |
| 401 | Unauthorized | Check API key |
| 404 | Not Found | Check city name / URL |
| 429 | Rate Limit | Wait 1 min, retry |
| 500 | Server Error | Retry later |

---

## ETL Best Practices

1. **Always check shape before and after** — `print(df.shape)` at start and end
2. **Never modify raw data** — `df = raw_df.copy()`
3. **Log every step** — `print()` what changed and how many rows affected
4. **Validate after each fix** — `isnull().sum()` should be 0 after fillna
5. **Handle errors gracefully** — `errors='coerce'` on type conversions
6. **Document your choices** — comment WHY you chose median vs 0 for fillna

---

## Common Errors — Day 3

| Error | Cause | Fix |
|-------|--------|-----|
| `KeyError: 'main'` | API returned error (wrong city name) | Print `response.json()` to see error message |
| `ConnectionError` | No internet | Check connectivity; use fallback data (Cell 15) |
| `ValueError: mixed date formats` | to_datetime can't parse inconsistent dates | Add `errors='coerce'` |
| `TypeError: str * float` | Multiplying string column | Convert first: `pd.to_numeric(df['col'], errors='coerce')` |
| `AttributeError: 'DataFrame' has no attribute 'append'` | Using old Pandas API | Use `pd.concat([df1, df2], ignore_index=True)` |
| Duplicates not removed | subset= columns too broad/narrow | Check `df[df.duplicated(keep=False)]` to see what's flagged |

---

## Day 3 Datasets

| File | Description |
|------|-------------|
| `messy_sales_data.csv` | 30 rows, 9 columns — 5 data quality issues to fix |
| `clean_sales_data.csv` | Output after Activity 1 cleaning |
| `weather_data.csv` | Output after Activity 2 API pipeline |

---

## Practice Questions

1. Explain ETL using a hospital example (patient records).
2. A DataFrame has 500 rows. After `df.dropna()` it has 412. What does this tell you?
3. Write code to remove rows where `customer_name` AND `product` are both the same.
4. What is the difference between `fillna(0)` and `fillna(df['col'].median())`?
5. Write code to call the weather API for 'Delhi' and print the temperature.
6. What does `response.status_code == 401` mean? How do you fix it?

---

## Day 3 Completion Checklist

- [ ] All notebook cells run without errors
- [ ] `df.isnull().sum().sum() == 0` after cleaning sales data
- [ ] `df.duplicated().sum() == 0` after deduplication
- [ ] `weather_data.csv` saved (8 cities)
- [ ] `clean_sales_data.csv` saved
- [ ] Weather dashboard (3-panel chart) visible
- [ ] Upload: `Add Day 3: ETL Pipeline + Weather API`

---

## Day 4 Preview
**Big Data + Apache Spark + Modern Data Architecture**
- Why Python fails at 300 million rows
- Apache Spark — distributed computing engine
- Medallion Architecture: Bronze → Silver → Gold
- Run PySpark in Google Colab!

---
*Codeboosters Tech | Data Engineering + GenAI Internship | Day 3 of 10*
