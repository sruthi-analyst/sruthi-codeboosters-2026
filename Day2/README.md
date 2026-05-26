# Day 2 — Student Reference Guide
## Database Fundamentals + SQL + Data Visualization
### Codeboosters Tech — Data Engineering + GenAI Internship

---

## Continuity from Day 1
The `student_performance.csv` from Day 1 is **loaded into a real SQLite database** today.
Pipeline: `CSV → SQLite DB → SQL Queries → DataFrames → Charts`

---

## Database Quick Concepts

| Term | Meaning | Example |
|------|---------|---------|
| **Database** | Organized storage for structured data | `college.db` |
| **Table** | Rows and columns inside a database | `students` table |
| **Row / Record** | One complete entry | One student's data |
| **Column / Field** | One attribute per record | `math_score` |
| **Primary Key** | Unique ID for each row — cannot repeat | `student_id` |
| **Foreign Key** | Column linking to another table's PK | `dept_code` |
| **SQL** | Language to query databases | `SELECT * FROM students` |
| **SQLite** | Lightweight DB stored in one file | `college.db` |

---

## Python Database Setup — Essential Code

```python
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt

# 1. Connect to (or create) database
conn = sqlite3.connect('college.db')
cursor = conn.cursor()

# 2. Load CSV into database table
df = pd.read_csv('student_performance.csv')
df.to_sql('students', conn, if_exists='replace', index=False)

# 3. Run SQL → get DataFrame
result = pd.read_sql_query("SELECT * FROM students LIMIT 5", conn)

# 4. Close when done
conn.close()
```

---

## SQL Cheat Sheet

```sql
-- Basic SELECT
SELECT name, math_score FROM students;
SELECT * FROM students;                        -- all columns
SELECT * FROM students LIMIT 10;              -- first 10 rows

-- Filter with WHERE
SELECT name FROM students WHERE math_score > 80;
SELECT name FROM students WHERE department = 'Computer Science';
SELECT name FROM students WHERE math_score > 70 AND attendance_percentage > 85;
SELECT name FROM students WHERE department IN ('Computer Science', 'Electronics');
SELECT name FROM students WHERE math_score BETWEEN 70 AND 90;

-- Sort and limit
SELECT name, math_score FROM students ORDER BY math_score DESC LIMIT 5;
SELECT DISTINCT department FROM students;      -- unique values

-- Aggregate functions
SELECT COUNT(*)                 FROM students;             -- total rows
SELECT AVG(math_score)          FROM students;             -- average
SELECT SUM(attendance_percentage) FROM students;           -- sum
SELECT MIN(math_score)          FROM students;             -- minimum
SELECT MAX(math_score)          FROM students;             -- maximum
SELECT ROUND(AVG(math_score),2) FROM students;             -- rounded avg

-- GROUP BY
SELECT department, COUNT(*) AS num_students
FROM students GROUP BY department;

SELECT department, ROUND(AVG(math_score),2) AS avg_math
FROM students GROUP BY department ORDER BY avg_math DESC;

-- HAVING (filter groups)
SELECT department, AVG(math_score) AS avg_math
FROM students GROUP BY department HAVING AVG(math_score) > 70;

-- Calculated column
SELECT name, math_score + science_score + english_score + programming_score AS total_score
FROM students ORDER BY total_score DESC;

-- INNER JOIN
SELECT s.name, s.math_score, d.hod_name
FROM students AS s
INNER JOIN departments AS d ON s.dept_code = d.dept_code;
```

**SQL Execution Order:** `FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT`

---

## WHERE vs HAVING

| | WHERE | HAVING |
|--|-------|--------|
| **Filters** | Individual rows | Groups |
| **When applied** | Before GROUP BY | After GROUP BY |
| **Can use aggregates?** | No | Yes |
| **Example** | `WHERE math_score > 80` | `HAVING AVG(math_score) > 80` |

---

## Matplotlib Chart Templates

```python
import matplotlib.pyplot as plt

# ── Template (works for all chart types) ──
fig, ax = plt.subplots(figsize=(10, 6))   # width x height in inches
# ... draw chart here ...
ax.set_title('Title Here', fontsize=16, fontweight='bold')
ax.set_xlabel('X Label', fontsize=13)
ax.set_ylabel('Y Label', fontsize=13)
plt.tight_layout()
plt.show()

# ── Bar Chart ──
ax.bar(x_categories, y_values, color='#4FC3F7', edgecolor='white')

# ── Histogram ──
ax.hist(data_list, bins=10, color='#4FC3F7', edgecolor='white')
ax.axvline(x=mean_val, color='red', linestyle='--', label='Mean')
ax.legend()

# ── Horizontal Bar ──
ax.barh(y_labels, x_values, color='#4FC3F7')
ax.invert_yaxis()                          # highest value at top

# ── Pie Chart ──
ax.pie(sizes, labels=labels, autopct='%1.1f%%', startangle=90,
       wedgeprops={'edgecolor':'white', 'linewidth':2})

# ── 4-Panel Dashboard ──
fig, axes = plt.subplots(2, 2, figsize=(18, 13))
fig.suptitle('Dashboard Title', fontsize=20, fontweight='bold')
ax1 = axes[0][0]   # top-left
ax2 = axes[0][1]   # top-right
ax3 = axes[1][0]   # bottom-left
ax4 = axes[1][1]   # bottom-right
```

---

## pd.read_sql_query — The Bridge

```python
# SQL → DataFrame in one line
result_df = pd.read_sql_query("""
    SELECT department, ROUND(AVG(math_score), 2) AS avg_math
    FROM students
    GROUP BY department
    ORDER BY avg_math DESC
""", conn)

# Now use result_df like any other DataFrame
print(result_df)
ax.bar(result_df['department'], result_df['avg_math'])
```

---

## Common Errors — Day 2

| Error | Cause | Fix |
|-------|--------|-----|
| `no such table: students` | DB setup cell not run | Run the `df.to_sql()` cell first |
| `no such column: name` | Column name typo | Run `PRAGMA table_info(students)` |
| `ProgrammingError: closed connection` | conn was closed | Re-run `conn = sqlite3.connect('college.db')` |
| Chart appears blank | DataFrame is empty | `print(result_df)` before charting |
| Labels cut off | `tight_layout()` missing | Add `plt.tight_layout()` before `plt.show()` |

---

## Practice Questions

1. What is the difference between a Primary Key and a Foreign Key?
2. Write SQL to find the average programming score for female students only.
3. What is the difference between WHERE and HAVING? Give one example of each.
4. Write SQL to find departments where average attendance is above 85%.
5. What does `pd.read_sql_query()` return and what arguments does it need?
6. You want to show how math scores are distributed. Which chart type is best and why?

---

## Day 2 Completion Checklist

- [ ] `college.db` created from `student_performance.csv`
- [ ] All 8 SQL queries run without errors
- [ ] 4 charts created (bar, histogram, grouped bar, pie)
- [ ] Student Performance Dashboard (2×2 grid) complete
- [ ] Notebook saved as `Day_02_Database_SQL_Visualization.ipynb`
- [ ] Notebook uploaded to GitHub
- [ ] Commit message: `Add Day 2: SQL Database + Visualization Dashboard`

---

## Day 3 Preview
**ETL Pipelines + Pandas Data Cleaning + Weather API**
- Call a real API and collect live weather data
- Clean messy data (nulls, duplicates, wrong types)
- Build a complete ETL pipeline
- The student dataset continues!

---
*Codeboosters Tech | Data Engineering + GenAI Internship | Day 2 of 10*
