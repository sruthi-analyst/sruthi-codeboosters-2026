# Day 7 — Embeddings + Semantic Search
**Codeboosters Tech | Data Engineering + GenAI Internship | Phase 2 | Day 7 of 10**

---

## What You Learn Today

| Unit | Topic | Key Outcome |
|------|-------|-------------|
| 1 | Why Keyword Search Fails | Understand the vocabulary mismatch problem |
| 2 | Embeddings: Text as Numbers | Generate 384-dimensional vectors with sentence-transformers |
| 3 | ChromaDB Vector Database | Store, query, and filter documents by meaning |
| Mini Project | Smart Notes Search Engine | Build a working semantic search system |

---

## Core Concept: The Problem and Solution

**Problem — Keyword Search:**
Searches for the exact word you typed. If you search `"vehicle"` but a document says `"car"` — it is missed.

**Solution — Semantic Search:**
Converts text to numbers (embeddings) that capture meaning. Similar meanings → similar numbers → found by the system.

**One-line summary:**
> Keyword search matches characters. Semantic search matches meaning.

---

## Key Vocabulary

| Term | Simple Definition |
|------|-------------------|
| **Embedding** | A piece of text converted into a list of 384 numbers that captures its meaning |
| **Vector** | A list of numbers. Our embeddings are 384-dimensional vectors. |
| **Dimension** | One number in the list (384 dimensions = 384 numbers) |
| **Cosine Similarity** | A score 0.0–1.0 showing how similar two vectors are. Higher = more similar. |
| **ChromaDB** | A vector database — stores text + embeddings + metadata, finds similar docs fast |
| **Collection** | A named group of documents in ChromaDB (like a table in SQL) |
| **Distance** | How far apart two vectors are. **Lower = more similar.** (Opposite of similarity score) |
| **Metadata** | Extra info stored with each document (subject, topic, author, etc.) for filtering |
| **In-Memory** | Data stored in RAM only — lost when the notebook restarts |

---

## Install Commands

```python
# Run this ONCE at the top of your notebook
!pip install chromadb sentence-transformers -q
```

---

## Complete Code Templates

### Template 1 — Generate Embeddings

```python
from sentence_transformers import SentenceTransformer

# Load the model (downloads ~80MB on first run)
model = SentenceTransformer('all-MiniLM-L6-v2')

# Encode one sentence → shape (384,)
embedding = model.encode("Your text here")
print(embedding.shape)        # (384,)

# Encode multiple sentences → shape (N, 384)
sentences = ["sentence one", "sentence two", "sentence three"]
embeddings = model.encode(sentences)
print(embeddings.shape)       # (3, 384)
```

---

### Template 2 — Cosine Similarity (Manual)

```python
import numpy as np

def cosine_similarity(vec_a, vec_b):
    dot_product = np.dot(vec_a, vec_b)
    norm_a = np.linalg.norm(vec_a)
    norm_b = np.linalg.norm(vec_b)
    return dot_product / (norm_a * norm_b)

# Score ranges:
# 0.90 – 1.00 → Almost identical meaning
# 0.70 – 0.89 → Very related
# 0.50 – 0.69 → Somewhat related
# 0.00 – 0.49 → Unrelated
```

---

### Template 3 — ChromaDB Full Workflow

```python
import chromadb

# Step 1: Create client (in-memory — resets on kernel restart)
client = chromadb.EphemeralClient()

# Step 2: Create or open a collection (like a SQL table)
collection = client.get_or_create_collection("my_collection")

# Step 3: Add documents
collection.add(
    documents=["ETL cleans data", "SQL queries databases", "ML trains models"],
    ids=["doc001", "doc002", "doc003"],                  # Must be unique strings
    metadatas=[
        {"subject": "DE",  "topic": "ETL"},
        {"subject": "DE",  "topic": "SQL"},
        {"subject": "ML",  "topic": "Supervised"},
    ]
)

# Verify
print(collection.count())    # Should print 3

# Step 4: Query by meaning
results = collection.query(
    query_texts=["How is data prepared?"],   # List — can include multiple queries
    n_results=2                              # Return top 2 matches
)

# Step 5: Read results
docs      = results['documents'][0]     # List of matched texts
ids       = results['ids'][0]           # List of matched IDs
distances = results['distances'][0]     # List of distances (LOWER = more similar)
metadatas = results['metadatas'][0]     # List of metadata dicts

for doc, dist, meta in zip(docs, distances, metadatas):
    print(f"Distance: {dist:.3f} | Subject: {meta['subject']} | {doc}")
```

---

### Template 4 — Filter by Metadata

```python
# Only search within a specific subject
results = collection.query(
    query_texts=["your question here"],
    n_results=3,
    where={"subject": "ML"}     # Filter: only ML documents
)
```

---

### Template 5 — Mini Project Pattern (college_notes.csv)

```python
import pandas as pd
import chromadb

# Load dataset
notes_df = pd.read_csv('college_notes.csv')

# Prepare lists for ChromaDB
all_documents = notes_df['content'].tolist()
all_ids       = notes_df['note_id'].tolist()
all_metadata  = [
    {"subject": row['subject'], "topic": row['topic']}
    for _, row in notes_df.iterrows()
]

# Create client and collection
client = chromadb.EphemeralClient()
notes_collection = client.get_or_create_collection("college_notes")

# Index all notes
notes_collection.add(
    documents=all_documents,
    ids=all_ids,
    metadatas=all_metadata
)

print(f"Indexed: {notes_collection.count()} notes")

# Search
results = notes_collection.query(
    query_texts=["How do I clean messy data?"],
    n_results=3
)

# Display
for doc, dist, meta in zip(
    results['documents'][0],
    results['distances'][0],
    results['metadatas'][0]
):
    print(f"[{dist:.3f}] {meta['subject']} — {meta['topic']}")
    print(f"         {doc[:100]}...")
```

---

## Critical Rules — Memorise These

| Rule | What It Means |
|------|---------------|
| **Same model always** | Use the same embedding model for add() AND query(). Never mix models. |
| **Lower distance = better** | Distance 0.05 is excellent. Distance 0.90 is poor. |
| **Unique IDs** | Every document must have a unique ID string. Duplicates cause errors. |
| **In-memory resets** | After kernel restart, re-run your collection.add() cells. |

---

## Distance Score Interpretation

```
Distance 0.00 – 0.25  →  Excellent match  (very similar meaning)
Distance 0.25 – 0.50  →  Good match       (related meaning)
Distance 0.50 – 0.75  →  Weak match       (somewhat related)
Distance 0.75 – 1.00+ →  Poor match       (unrelated)
```

**Remember:** `similarity = 1 - distance` if you need the similarity score.

---

## Common Errors Cheat Sheet

| Error Message | Likely Cause | Fix |
|---------------|--------------|-----|
| `IDAlreadyExistsError` | Ran `collection.add()` twice with same IDs | Restart kernel, re-run from top |
| `OSError: Can't load model` | No internet or model name typo | Check internet; name must be `all-MiniLM-L6-v2` |
| `KeyError: 'documents'` | Accessing results before running query | Run `collection.query()` first |
| Results look wrong | Different models used for add vs query | Use same model throughout |
| Collection empty after restart | In-memory client was reset | Re-run `collection.add()` cells |

---

## Results Dictionary Structure

```python
results = collection.query(query_texts=["question"], n_results=3)

# results is a dict with these keys:
results['documents'][0]     # → list of 3 matched document texts
results['ids'][0]           # → list of 3 matched IDs
results['distances'][0]     # → list of 3 distance scores (lower = better)
results['metadatas'][0]     # → list of 3 metadata dicts

# The [0] is because results support batch queries
# results['documents'][0] = results for FIRST query
# results['documents'][1] = results for SECOND query (if you sent two)
```

---

## Today's Dataset — college_notes.csv

| Column | Type | Example |
|--------|------|---------|
| `note_id` | String | N001, N002 … N015 |
| `subject` | String | Data Engineering, Machine Learning, Generative AI, Python Programming |
| `topic` | String | ETL Pipelines, Supervised Learning, Prompt Engineering, Pandas Library |
| `content` | String | Full text of the note |

**15 notes total across 4 subjects.**
This file is also used in **Day 8 (RAG Systems)** — do not delete it.

---

## Practice Questions

**Q1.** What is the difference between keyword search and semantic search? Give one real-world example of each.

**Q2.** What does `model.encode(['hello world'])` return? What is the shape?

**Q3.** A student adds documents using Model A but queries using Model B. Will results be correct? Why?

**Q4.** ChromaDB returns distances `[0.12, 0.45, 0.87]`. Which result is most relevant? Which is least?

**Q5.** Write the `collection.add()` call to store one document with `subject='ML'` and `topic='Regression'`.

**Q6.** Modify a query call to return only `'Python Programming'` notes.

---

## Architecture Overview

```
KEYWORD SEARCH                    SEMANTIC SEARCH (Today)
──────────────                    ───────────────────────
User Query                        User Query
    ↓                                 ↓
Tokenize Words                    Convert to Embedding (384 numbers)
    ↓                                 ↓
Match exact words in index        Compare vectors in ChromaDB
    ↓                                 ↓
Rank by word frequency            Rank by distance (lower = better)
    ↓                                 ↓
Return Results                    Return Results

FAILS for synonyms/paraphrasing   WORKS for meaning regardless of words
```

---

## What Comes Next — Day 8

**RAG (Retrieval Augmented Generation)**

Day 8 combines everything from Day 7 with the Groq LLM from Day 6:

```
Your Question
    ↓
ChromaDB retrieves top 3 relevant notes   ← Day 7 skill
    ↓
Groq LLM reads those notes and generates answer  ← Day 6 skill
    ↓
Accurate, grounded answer from YOUR data
```

**Keep your Groq API key ready. Keep `college_notes.csv` saved.**

---

## Day 7 Completion Checklist

- [ ] Ran all notebook cells from top to bottom without errors
- [ ] Observed keyword search missing relevant results (cells 007–008)
- [ ] Generated embeddings and checked shape is (N, 384) (cells 010–012)
- [ ] Calculated cosine similarity — similar pair > 0.6, different pair < 0.3 (cell 013)
- [ ] Created ChromaDB collection and added documents (cells 017–018)
- [ ] Queried by meaning and interpreted distance scores (cells 019–022)
- [ ] Completed mini project: indexed 15 notes, ran 5 queries, used where= filter (cells 030–041)
- [ ] Pushed notebook to GitHub repo `codeboosters-internship-2024`

---

*Codeboosters Tech | Data Engineering + GenAI Internship | Day 7 of 10*
