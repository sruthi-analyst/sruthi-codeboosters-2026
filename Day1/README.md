# Day 1 - Introduction to Data Engineering + GitHub
### Codeboosters Tech — Data Engineering + GenAI Internship Program

---

## Quick Reference Card

| Topic | Key Point |
|-------|-----------|
| Data Engineering | Building systems to collect, store, clean, and deliver data |
| Structured Data | Rows and columns — CSV, Excel, SQL |
| Semi-Structured | Key-value pairs — JSON, XML |
| Unstructured | No fixed format — images, videos, audio, PDFs |
| Pandas | Python library for working with tabular data |
| DataFrame | In-memory table with rows and columns |
| GitHub | Cloud platform for version control and code portfolio |
| Commit | Saved snapshot of code with a description |

---

## Essential Pandas Commands — Day 1

```python
import pandas as pd                    # Import Pandas library

df = pd.read_csv('filename.csv')       # Load a CSV file

df.head()                              # Show first 5 rows
df.head(10)                            # Show first 10 rows
df.tail()                              # Show last 5 rows

df.shape                               # (number_of_rows, number_of_columns)
df.dtypes                              # Data type of each column
df.describe()                          # Statistical summary
df.isnull().sum()                      # Count missing values per column
df.columns.tolist()                    # List all column names

df['column_name']                      # Select one column

# Boolean Filtering
df[df['column'] == 'value']            # Filter rows equal to value
df[df['column'] > 80]                  # Filter rows greater than 80
df[df['column'] < 75]                  # Filter rows less than 75

# Groupby and Aggregation
df.groupby('column')['other'].mean()   # Average per group
df['column'].value_counts()            # Count each unique value

# Sorting
df.sort_values(by='column', ascending=False)   # Sort highest first
df.sort_values(by='column', ascending=True)    # Sort lowest first

# New Column
df['new_col'] = df['col1'] + df['col2']  # Add columns together

# Save DataFrame
df.to_csv('output.csv', index=False)   # Save as CSV
```

---

## Types of Data — Summary

### Structured Data
- Organized in rows and columns
- Every record has the same fields
- Examples: marksheet, bank transactions, attendance register
- Formats: CSV, Excel (.xlsx), SQL Database

### Semi-Structured Data
- Has labels/tags but not in strict rows and columns
- Can have nested data (data inside data)
- Examples: API responses, WhatsApp message metadata, config files
- Formats: JSON, XML

### Unstructured Data
- No predefined format or structure
- Cannot be stored directly in a spreadsheet without losing information
- Examples: photos, lecture videos, voice notes, PDFs, handwritten notes
- Formats: JPG, MP4, MP3, PDF, TXT

**Key Fact:** 80% of all data created in the world is unstructured.

---

## GitHub Key Concepts

| Term | Meaning | Analogy |
|------|---------|---------|
| Repository (Repo) | Project folder on GitHub | Google Drive folder for your project |
| Commit | Saved snapshot with a message | Ctrl+S in Google Docs with a label |
| Push | Send local code to GitHub | Upload to Google Drive |
| Pull | Download code from GitHub | Download from Google Drive |
| README.md | Project description file | Cover page of your project report |
| Branch | Separate version of code | Making a copy to experiment on |

### GitHub Setup Steps
1. Go to github.com
2. Sign Up → enter email, username, password
3. Verify email
4. Click '+' → 'New repository'
5. Name: `codeboosters-internship-2024`
6. Set to **Public**
7. Check 'Add a README file'
8. Click 'Create repository'

### Uploading Notebook to GitHub
1. Go to your repository
2. Click 'Add file' → 'Upload files'
3. Drag and drop your `.ipynb` file
4. Write a commit message: `Add Day 1 notebook`
5. Click 'Commit changes'

---

## Common Errors and Fixes

| Error | Fix |
|-------|-----|
| `ModuleNotFoundError: No module named 'pandas'` | Run `!pip install pandas` |
| `FileNotFoundError: student_performance.csv` | Upload file via Colab Files panel |
| `KeyError: 'column_name'` | Check `df.columns` for exact spelling |
| `SyntaxError` | Check for missing brackets, quotes, colons |
| `IndentationError` | Use exactly 4 spaces. Never mix tabs and spaces |

---

## Dataset Reference — student_performance.csv

| Column | Type | Description |
|--------|------|-------------|
| student_id | int | Unique student number |
| name | text | Student's full name |
| age | int | Age in years |
| gender | text | Male or Female |
| department | text | Engineering department |
| semester | int | Current semester |
| math_score | int | Math marks (0-100) |
| science_score | int | Science marks (0-100) |
| english_score | int | English marks (0-100) |
| programming_score | int | Programming marks (0-100) |
| attendance_percentage | int | Attendance % (0-100) |
| city | text | Home city |
| admission_year | int | Year of admission |

**This dataset will be used across all 10 days of the internship.**

---

## Practice Questions — Day 1

1. A hospital stores patient X-ray images, doctor's notes, and a table of patient vital signs. Classify each data type.

2. Why do companies need Data Engineers? Give a real-world example.

3. Write Python code to load `sales.csv` and show its first 10 rows.

4. What does `df.shape` return? If it returns `(500, 15)`, what does each number mean?

5. How do you filter a DataFrame called `orders` to show only rows where `city` is `Mumbai`?

6. You accidentally delete your Python script. How would GitHub have saved you?

---

## Day 1 Completion Checklist

- [ ] I can explain what a Data Engineer does
- [ ] I can classify Structured, Semi-Structured, and Unstructured data
- [ ] I have loaded `student_performance.csv` in Google Colab
- [ ] All notebook cells run without errors
- [ ] I have created a GitHub account
- [ ] I have created a repository named `codeboosters-internship-2024`
- [ ] I have uploaded my Day 1 notebook to GitHub
- [ ] I have updated my README.md

---

## What's Coming on Day 2

On Day 2, we take the **same student dataset** and:
- Store it in a real **database** using SQLite
- Write **SQL queries** to find insights
- Create **charts and visualizations** using Matplotlib
- Build a **Student Performance Dashboard**

---

*Codeboosters Tech | Data Engineering + GenAI Internship | Day 1 of 10*
