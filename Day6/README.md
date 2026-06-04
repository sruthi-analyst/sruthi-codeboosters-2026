# Day 6 — Introduction to GenAI + Prompt Engineering
### Codeboosters Tech | Phase 2 Begins

---

## Phase 1 → Phase 2 Bridge

| Phase 1 (Days 1–5) | Phase 2 (Days 6–10) |
|---------------------|----------------------|
| Pandas, SQL, ETL | LLMs, Prompt Engineering |
| PySpark, Medallion | Embeddings, ChromaDB |
| Scikit-learn, ML | RAG, AI Agents |
| Data Pipelines | AI-Powered Pipelines |

**Your Phase 1 pipelines feed directly into Phase 2 AI systems.**

---

## LLM Key Concepts

| Term | Definition | Value |
|------|-----------|-------|
| **LLM** | Large Language Model — predicts next token | Powers ChatGPT, Copilot etc. |
| **Token** | Basic processing unit (~0.75 words) | Context window = max tokens |
| **Context Window** | Max tokens per request | Llama-3.1-8b: 8192 tokens |
| **Temperature** | Randomness control (0=consistent, 1=creative) | Use 0.0 for data tasks |
| **Hallucination** | Confident but false output | Mitigate with structured prompts |

---

## Groq API Setup

```python
# Install
!pip install groq --quiet

# Import and configure
from groq import Groq

API_KEY = "gsk_your_key_here"   # Get from console.groq.com (free)
client  = Groq(api_key=API_KEY)
MODEL   = "llama-3.1-8b-instant"

# Basic call function
def ask_llm(user_msg, system_msg="You are a helpful assistant.",
            temperature=0.7, max_tokens=500):
    response = client.chat.completions.create(
        model=MODEL,
        messages=[
            {"role": "system", "content": system_msg},
            {"role": "user",   "content": user_msg},
        ],
        temperature=temperature,
        max_tokens=max_tokens,
    )
    return response.choices[0].message.content

# Test
print(ask_llm("What is ETL? Answer in 2 sentences."))
```

---

## Four Prompting Techniques

### 1. Zero-Shot — Ask Directly
```python
response = ask_llm(
    "Extract the city from: 456 Brigade Road, Bangalore 560025"
)
# Best for: simple, well-defined tasks
```

### 2. Few-Shot — Provide Examples
```python
prompt = """Convert employee text to JSON.

Input: RAMESH KUMAR, 45000
Output: {"name": "Ramesh Kumar", "salary": 45000}

Input: priya nair, 52000
Output: {"name": "Priya Nair", "salary": 52000}

Input: ANANYA DAS, 38000
Output:"""

response = ask_llm(prompt, temperature=0.0)
# Best for: custom formats, consistent output
```

### 3. Role Prompting — Set Expert Identity
```python
response = ask_llm(
    "Review this ETL code for production issues:\ndf['rev'] = df['qty'] * df['price']",
    system_msg="You are a senior data engineer with 10 years of "
               "production experience. Be critical and specific."
)
# Best for: domain expertise, professional tone
```

### 4. Chain-of-Thought — Force Reasoning
```python
response = ask_llm(
    """Analyse this sales data step by step:
    1. First identify data quality issues
    2. Then calculate key metrics
    3. Finally state the business insight
    
    Data: [your data here]"""
)
# Best for: complex reasoning, multi-step analysis
```

---

## Structured JSON Extraction Template

```python
# Production-quality extraction prompt
SYSTEM_PROMPT = """You are a data extraction specialist.

Extract information and return ONLY valid JSON.
No explanation. No markdown. No code fences.
Start with { and end with }.

JSON schema:
{
  "field_1": type (or null if missing),
  "field_2": type (or null if missing)
}

Rules:
- Names: Title Case
- Amounts: numeric only (remove Rs./commas)
- Dates: YYYY-MM-DD format
"""

response = ask_llm(
    f"Extract from: {your_text}",
    system_msg=SYSTEM_PROMPT,
    temperature=0.0    # Always 0 for data extraction
)
```

---

## Parsing LLM JSON Output

```python
import json, re

raw = response  # LLM output string

# Method 1: Direct parse (ideal)
try:
    data = json.loads(raw.strip())

# Method 2: Regex fallback (handles markdown fences)
except json.JSONDecodeError:
    match = re.search(r'\{.*?\}', raw, re.DOTALL)
    if match:
        data = json.loads(match.group())
    else:
        data = None   # Extraction failed

# Safe field access
name   = data.get("name")     if data else None
amount = data.get("amount")   if data else None
```

---

## Smart Data Cleaner — Complete Pipeline

```python
# 1. Define messy inputs
invoices = [
    "INV-001 TECHWORLD SOLUTIONS Jan 15 2024 Rs.45000 Laptop",
    "Invoice priya enterprises 07-02-2024 amt 12500 services",
]

# 2. Process each with LLM
records = []
for invoice in invoices:
    result = extract_invoice_data(invoice, SYSTEM_PROMPT, client, MODEL)
    if result:
        records.append(result)
    time.sleep(0.5)   # Rate limit respect

# 3. Build DataFrame (Day 1 skill!)
df = pd.DataFrame(records)
df['amount'] = pd.to_numeric(df['amount'], errors='coerce')
df['invoice_date'] = pd.to_datetime(df['invoice_date'], errors='coerce')

# 4. Analyse (Day 2 skill!)
print(df.groupby('category')['amount'].sum())

# 5. Save (Day 3 skill!)
df.to_csv('cleaned_invoices.csv', index=False)
```

---

## Temperature Guide

| Task Type | Temperature | Why |
|-----------|------------|-----|
| Data extraction / JSON | 0.0 | Need identical output every run |
| Factual Q&A | 0.1–0.2 | Slight variation acceptable |
| Summarization | 0.3–0.5 | Some rephrasing is fine |
| Creative writing | 0.7–1.0 | Variety is desired |

---

## Common Errors — Day 6

| Error | Cause | Fix |
|-------|--------|-----|
| `AuthenticationError` | Wrong API key | Copy full key from console.groq.com including `gsk_` |
| `RateLimitError` | Too many calls | Add `time.sleep(2)` between API calls |
| `JSONDecodeError` | LLM added markdown fences | Use `re.search(r'\{.*?\}', raw, re.DOTALL)` |
| `KeyError: 'field'` | LLM used different key name | Use `dict.get('field')` not `dict['field']` |
| `ModuleNotFoundError: groq` | Library not installed | Run `!pip install groq` |
| Inconsistent output | Temperature too high | Set `temperature=0.0` for data tasks |

---

## Practice Questions

1. What is the difference between ML (Day 5) and Generative AI?
2. What does `temperature=0.0` do? When would you use it?
3. Write a few-shot prompt to extract name and salary from text as JSON.
4. What is LLM hallucination? How does prompt engineering reduce it?
5. LLM returns ` ```json\n{"name":"Ramesh"}\n``` ` and `json.loads()` crashes. Write the fix.
6. How does today's Smart Data Cleaner differ from manual Day 3 ETL?

---

## Day 6 Completion Checklist

- [ ] Groq API key obtained from console.groq.com
- [ ] First API call successful (Cell 3)
- [ ] Weak vs strong prompt comparison done (Cell 9)
- [ ] All 5 invoices processed (Cell 12)
- [ ] `cleaned_invoices.csv` saved
- [ ] Notebook uploaded to GitHub
- [ ] Commit: `Add Day 6: GenAI Prompt Engineering + Smart Data Cleaner`

---

## Day 7 Preview
**Embeddings + Semantic Search**
- Search by meaning, not keywords
- "Find notes about databases" finds documents about SQL even without the word "database"
- Tools: sentence-transformers, ChromaDB vector database
- Same Groq API key works for Day 7

---
*Codeboosters Tech | Data Engineering + GenAI Internship | Day 6 of 10 | Phase 2*
