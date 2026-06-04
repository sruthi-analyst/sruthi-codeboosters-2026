# Day 9: AI Agents + Text-to-SQL
## Student Reference Guide
### Codeboosters Tech — Data Engineering + GenAI Internship

---

## Quick Revision: Key Concepts

| Concept | One-Line Definition |
|---------|---------------------|
| AI Agent | An LLM connected to tools that can take multi-step actions autonomously |
| Chatbot | An LLM that answers questions from training data only — no tools, no actions |
| Tool Calling | The AI decides which Python functions to call and when |
| ReAct Pattern | Think → Plan → Act → Respond — the reasoning loop used by AI Agents |
| Text-to-SQL | Converting a plain English question into a SQL database query using an LLM |
| Schema Injection | Including database structure in the system prompt so the AI knows the database |
| temperature=0.0 | Fully deterministic LLM output — same input always produces same SQL |

---

## The Chatbot vs Agent Distinction

```
CHATBOT                          AI AGENT
-------                          --------
Answers from memory only         Can use tools (database, search, APIs)
One step: question -> answer     Multi-step: think -> plan -> act -> respond
Static knowledge                 Can access live / real-time data
No autonomous actions            Can take actions in the real world
Example: FAQ bot                 Example: Text-to-SQL system
```

---

## The ReAct Pattern (Step-by-Step)

```
User Question: "Show top 5 students in Programming"
      |
      v
[1. THINK]   What does the user want?
             -> Top 5 students, filtered by subject = Programming, sorted by marks

[2. PLAN]    What SQL do I need to write?
             -> SELECT name, marks FROM students
                WHERE subject = 'Programming'
                ORDER BY marks DESC
                LIMIT 5

[3. ACT]     Execute the SQL on the real database
             -> Run on college.db, fetch rows

[4. RESPOND] Format and return results
             -> Table + natural language answer
```

---

## Text-to-SQL Pipeline Architecture

```
User Question (plain English)
         +
Database Schema (table structure, column names, sample rows)
         |
         v
    [Groq LLM]
    System prompt = "You are an SQL expert. Here is the schema: ..."
    User message  = "Show top 5 students in Programming"
         |
         v
    Generated SQL query
         |
         v
    [SQLite Database] -- college.db
         |
         v
    Query Results (pandas DataFrame)
         |
         v
    [Optional: Groq LLM again]
    Writes natural language answer from the data
         |
         v
    Final Answer to User
```

---

## Syntax Cheat Sheet

### 1. Create Database and Load CSV

```python
import sqlite3
import pandas as pd

# Load CSV
df = pd.read_csv("student_performance.csv")

# Connect to (or create) SQLite database
conn = sqlite3.connect("college.db")

# Write DataFrame to database as table
df.to_sql("students", conn, if_exists="replace", index=False)
# if_exists="replace"  -> Delete and recreate table if it exists
# index=False          -> Do not write row numbers as a column
```

---

### 2. Extract Database Schema

```python
def get_schema(conn, table_name="students"):
    cursor = conn.cursor()
    cursor.execute(f"PRAGMA table_info({table_name})")
    # PRAGMA table_info() returns: (id, name, type, notnull, default, pk)
    
    columns = cursor.fetchall()
    schema_lines = [f"Table: {table_name}", "Columns:"]
    
    for col in columns:
        schema_lines.append(f"  - {col[1]} ({col[2]})")
        # col[1] = column name
        # col[2] = data type
    
    # Add sample rows
    cursor.execute(f"SELECT * FROM {table_name} LIMIT 3")
    sample_rows = cursor.fetchall()
    schema_lines.append("\nSample rows (first 3):")
    for row in sample_rows:
        schema_lines.append(f"  {row}")
    
    return "\n".join(schema_lines)
```

---

### 3. Generate SQL from Natural Language

```python
from groq import Groq
import os

client = Groq(api_key=os.environ["GROQ_API_KEY"])
MODEL  = "llama-3.1-8b-instant"

def generate_sql(user_question, schema_text, client, model):
    system_prompt = f"""You are an expert SQL assistant.
Database structure:
{schema_text}

Rules:
1. Return ONLY a valid SQLite SQL query.
2. No explanation. No markdown. Raw SQL only.
3. Table name is: students
4. Use single quotes for strings: WHERE subject = 'Programming'
5. For top N: ORDER BY marks DESC LIMIT N
"""
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user",   "content": user_question}
        ],
        temperature=0.0   # 0.0 = exact, deterministic output
    )
    return response.choices[0].message.content.strip()
```

---

### 4. Execute the Generated SQL

```python
import re

def execute_sql(sql_query, conn):
    # Clean: remove markdown code fences if present
    clean_sql = sql_query.strip()
    clean_sql = re.sub(r'```sql', '', clean_sql)  # Remove opening fence
    clean_sql = re.sub(r'```',   '', clean_sql)  # Remove closing fence
    clean_sql = clean_sql.strip()
    
    try:
        result_df = pd.read_sql_query(clean_sql, conn)
        return result_df, None     # (result, no_error)
    except Exception as e:
        return None, str(e)        # (no_result, error_message)

# Usage:
result_df, error = execute_sql(generated_sql, conn)
if error:
    print(f"Error: {error}")
else:
    print(result_df)
```

---

### 5. Generate Natural Language Answer

```python
def generate_answer(user_question, result_df, client, model):
    if result_df is None or len(result_df) == 0:
        return "No results found."
    
    results_text = result_df.to_string(index=False)
    
    prompt = f"""The user asked: '{user_question}'
The database returned:
{results_text}

Write a clear 2-3 sentence answer. Be specific. Mention actual names and numbers."""
    
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        temperature=0.3    # 0.3 = slightly expressive but still accurate
    )
    return response.choices[0].message.content.strip()
```

---

### 6. Complete Agent in One Function

```python
def text_to_sql_agent(user_question, conn, client, model):
    # Step 1: Get schema
    schema_text = get_schema(conn)
    
    # Step 2: Generate SQL
    generated_sql = generate_sql(user_question, schema_text, client, model)
    print(f"Generated SQL: {generated_sql}")
    
    # Step 3: Execute SQL
    result_df, error = execute_sql(generated_sql, conn)
    if error:
        print(f"Error: {error}")
        return None
    
    # Step 4: Natural language answer
    answer = generate_answer(user_question, result_df, client, model)
    
    print(result_df)
    print("\nAnswer:", answer)
    return result_df
```

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `OperationalError: no such column` | LLM used wrong column name | Check schema output. List all columns explicitly in system prompt |
| SQL wrapped in ` ``` ` fences | LLM added markdown formatting | execute_sql() strips these with re.sub(). Or add 'No markdown' to prompt |
| `AuthenticationError` | Wrong or missing API key | Check `os.environ["GROQ_API_KEY"]`. Keys start with `gsk_` |
| Empty results (0 rows) | Case mismatch in WHERE clause | Values are case-sensitive. Use 'Programming' not 'programming' |
| Chart not rendering | Result has no numeric column | Check your query returns numeric data (marks, attendance, count) |

---

## SQL Patterns Reference

```sql
-- Top N students in a subject
SELECT name, marks FROM students
WHERE subject = 'Programming'
ORDER BY marks DESC
LIMIT 5;

-- Average marks per subject
SELECT subject, AVG(marks) as avg_marks
FROM students
GROUP BY subject
ORDER BY avg_marks DESC;

-- Count by gender
SELECT gender, COUNT(*) as count
FROM students
GROUP BY gender;

-- Students above a score threshold
SELECT name, subject, marks
FROM students
WHERE marks > 90
ORDER BY marks DESC;

-- Students in multiple conditions
SELECT name, subject, marks
FROM students
WHERE gender = 'Female'
  AND marks > 85
ORDER BY marks DESC;

-- Highest scorer per subject
SELECT subject, name, MAX(marks) as top_mark
FROM students
GROUP BY subject;
```

---

## Key Parameter Summary

| Parameter | Value | Reason |
|-----------|-------|--------|
| temperature (SQL generation) | 0.0 | Exact, deterministic SQL — no variation wanted |
| temperature (NL answer) | 0.3 | Slightly expressive for natural language |
| model | llama-3.1-8b-instant | Fast, efficient, good SQL generation |
| if_exists | "replace" | Recreate table if it already exists |
| index | False | Do not write row numbers to database |

---

## Mini Project Completion Checklist

- [ ] CSV loaded into SQLite: `len(df) == 30`
- [ ] Schema printed with all 8 columns visible
- [ ] SQL generated correctly for at least one test question
- [ ] execute_sql returns a DataFrame with correct rows
- [ ] Bar chart renders for a query with numeric results
- [ ] Natural language answer mentions actual names and numbers from data
- [ ] Dashboard tested with at least 3 different question types
- [ ] Database connection closed at end: `conn.close()`

---

## Self-Test Questions

**Can you answer these without looking at notes?**

1. What is the difference between a chatbot and an AI Agent?
2. What are the four steps of the ReAct pattern?
3. Why do we use `temperature=0.0` for SQL generation?
4. What information do we include in the schema injected into the system prompt?
5. What does `PRAGMA table_info(students)` return?
6. Why do we include sample rows in the schema? Give a specific example of why they matter.
7. If the AI returns zero results, what is the most likely cause?
8. What does `re.sub()` do in the `execute_sql()` function?

---

## Quick Reference: Day Continuity

| Day | What You Learned | How It Connects to Day 9 |
|-----|-----------------|--------------------------|
| Day 1 | pandas, CSV | pd.read_csv(), DataFrame display |
| Day 2 | SQLite, SQL | sqlite3.connect(), pd.read_sql_query() |
| Day 6 | Groq API, prompts | client.chat.completions.create() |
| Day 7 | Embeddings, ChromaDB | Semantic search concept |
| Day 8 | RAG pipeline | Context injection → schema injection |
| Day 9 | AI Agents, Text-to-SQL | Combines all of the above |

---

*Day 9 Complete — Codeboosters Tech | Data Engineering + GenAI Internship*
