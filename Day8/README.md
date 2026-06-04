# Day 8 Student Reference — RAG Systems
## Retrieval-Augmented Generation
### Codeboosters Tech | Data Engineering + GenAI Internship | Phase 2

---

## Quick Navigation

1. [Key Concepts Cheat Sheet](#key-concepts)
2. [RAG Pipeline — 7 Steps](#rag-pipeline)
3. [Code Templates](#code-templates)
4. [API and Function Reference](#api-reference)
5. [Common Errors and Fixes](#common-errors)
6. [Practice Questions](#practice-questions)
7. [Mini Project Checklist](#mini-project-checklist)
8. [Day 8 Completion Checklist](#completion-checklist)

---

## Key Concepts Cheat Sheet

| Term | Definition |
|------|-----------|
| **Hallucination** | When an LLM generates confident-sounding text that is factually wrong or made up |
| **RAG** | Retrieval-Augmented Generation — grounding LLM answers in retrieved real documents |
| **Chunk** | A small, focused piece of a document (typically 200–500 words) |
| **Embedding** | A vector (list of numbers) representing the semantic meaning of text |
| **Vector Database** | A database that stores and searches embeddings by similarity (e.g. ChromaDB) |
| **Retrieval** | Finding the most similar document chunks for a given query |
| **Context Injection** | Adding retrieved chunks to the LLM prompt before the question |
| **Grounding** | Constraining the LLM to answer only from provided context |
| **top_k / n_results** | How many similar chunks to retrieve (typically 3–5) |
| **Indexing Phase** | Done once: chunk → embed → store in ChromaDB |
| **Querying Phase** | Per question: embed query → retrieve → inject → generate |
| **Cosine Similarity** | Similarity metric: 1.0 = identical meaning, 0.0 = unrelated |
| **Knowledge Base** | The collection of documents the RAG system retrieves from |
| **Temperature (0.1)** | Low randomness setting — LLM sticks closely to provided context |

---

## RAG Pipeline — 7 Steps

### Phase A: Indexing (Done Once)

```
Step 1: DOCUMENTS     — Your PDFs, CSVs, text files
         ↓
Step 2: CHUNK         — Split into small focused pieces (200-500 words each)
         ↓
Step 3: EMBED         — Convert each chunk to a 384-dim vector
                         using all-MiniLM-L6-v2 model
         ↓
Step 4: STORE         — Save vectors + text + metadata in ChromaDB
```

### Phase B: Querying (Every User Question)

```
Step 5: USER QUESTION — "What is ETL?"
         ↓
Step 6: EMBED QUERY   — Convert question to vector (SAME model as step 3!)
         ↓
Step 7: RETRIEVE      — ChromaDB finds top-3 most similar chunks
         ↓
Step 8: INJECT        — Add chunks to LLM prompt as context
         ↓
Step 9: GENERATE      — LLM answers using ONLY the provided context
         ↓
        ANSWER with source citations
```

**Critical rule:** Use the SAME embedding model for both documents and queries.

---

## Code Templates

### Template 1: Complete Setup

```python
import pandas as pd
import chromadb
from sentence_transformers import SentenceTransformer
from groq import Groq
import os

# Configuration
GROQ_API_KEY = "gsk_your_key_here"   # Replace with your key
EMBEDDING_MODEL = "all-MiniLM-L6-v2"
LLM_MODEL = "llama-3.1-8b-instant"
TOP_K = 3
TEMPERATURE = 0.1

# Initialize clients
groq_client = Groq(api_key=GROQ_API_KEY)
embed_model = SentenceTransformer(EMBEDDING_MODEL)
chroma_client = chromadb.Client()
```

---

### Template 2: Index Knowledge Base

```python
# Load your documents
df = pd.read_csv("college_notes.csv")
docs = df["content"].tolist()
ids  = [f"note_{nid}" for nid in df["note_id"].tolist()]
meta = [{"subject": r["subject"], "topic": r["topic"]}
        for r in df.to_dict("records")]

# Create ChromaDB collection
collection = chroma_client.get_or_create_collection("my_knowledge_base")

# Generate embeddings — ALWAYS .tolist() to convert from numpy
embeddings = embed_model.encode(docs, show_progress_bar=True).tolist()

# Add to ChromaDB
collection.add(
    documents=docs,
    embeddings=embeddings,
    ids=ids,
    metadatas=meta
)

print(f"Indexed {collection.count()} documents.")
```

---

### Template 3: Retrieve Relevant Chunks

```python
def retrieve(question, top_k=3):
    # Embed the question with the SAME model
    q_vec = embed_model.encode(question).tolist()
    
    # Search ChromaDB
    results = collection.query(
        query_embeddings=[q_vec],
        n_results=top_k
    )
    return results

# With subject filter
def retrieve_filtered(question, subject, top_k=2):
    q_vec = embed_model.encode(question).tolist()
    results = collection.query(
        query_embeddings=[q_vec],
        n_results=top_k,
        where={"subject": subject}    # Metadata filter
    )
    return results
```

---

### Template 4: Build Context String

```python
def build_context(results):
    parts = []
    for doc, meta in zip(results["documents"][0],
                         results["metadatas"][0]):
        parts.append(f"[{meta['subject']} - {meta['topic']}]\n{doc}")
    return "\n\n---\n\n".join(parts)
```

---

### Template 5: Generate RAG Answer

```python
def generate_answer(question, context):
    system_prompt = """You are a helpful academic assistant.
Answer ONLY using the information provided in the context below.
If the answer is not in the context, say:
'I don't have enough information in my knowledge base to answer this.'
Do not use your general training knowledge.
Cite the source in brackets at the end of your answer."""

    response = groq_client.chat.completions.create(
        model=LLM_MODEL,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user",   "content": f"Context:\n{context}\n\nQuestion: {question}"}
        ],
        temperature=TEMPERATURE,
        max_tokens=500
    )
    return response.choices[0].message.content
```

---

### Template 6: Complete RAG Pipeline (One Function)

```python
def ask(question, top_k=3, verbose=True):
    """Complete RAG pipeline — one function call."""
    
    # 1. Retrieve
    results = retrieve(question, top_k)
    
    if verbose:
        print("Retrieved sources:")
        for i, (m, d) in enumerate(zip(results["metadatas"][0],
                                       results["distances"][0])):
            print(f"  {i+1}. [{m['subject']}] {m['topic']} (dist: {d:.4f})")
    
    # 2. Build context
    context = build_context(results)
    
    # 3. Generate
    answer = generate_answer(question, context)
    
    if verbose:
        print(f"\nANSWER:\n{answer}")
    
    return answer

# Usage
ask("What is ETL and what are its three stages?")
```

---

## API Reference

### ChromaDB

| Operation | Code | Notes |
|-----------|------|-------|
| Create client (in-memory) | `chroma = chromadb.Client()` | Data lost on restart |
| Create client (persistent) | `chroma = chromadb.PersistentClient(path="./db")` | Saves to disk |
| Get/create collection | `col = chroma.get_or_create_collection("name")` | Safe to call multiple times |
| Add documents | `col.add(documents, embeddings, ids, metadatas)` | IDs must be unique strings |
| Query | `col.query(query_embeddings=[vec], n_results=3)` | vec must be a Python list |
| Count documents | `col.count()` | Returns integer |
| Delete collection | `chroma.delete_collection("name")` | Use to reset before re-adding |
| Filter by metadata | `col.query(..., where={"subject": "GenAI"})` | Use `where` parameter |

### SentenceTransformer

| Operation | Code | Notes |
|-----------|------|-------|
| Load model | `model = SentenceTransformer("all-MiniLM-L6-v2")` | Downloads ~80MB on first run |
| Embed text | `vec = model.encode("text").tolist()` | Returns numpy → convert to list |
| Embed list | `vecs = model.encode(list_of_texts).tolist()` | Returns 2D array |
| Output shape | `(384,)` for single text | `(n, 384)` for a list |

### Groq API (RAG-specific)

```python
response = groq_client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[
        {"role": "system", "content": system_prompt},
        {"role": "user",   "content": user_prompt_with_context}
    ],
    temperature=0.1,      # Low = deterministic, follows context
    max_tokens=500        # Limit response length
)
answer = response.choices[0].message.content
```

### ChromaDB Results Object

```python
results = collection.query(...)

# Access retrieved data:
results["documents"][0]    # List of text strings  (top-k docs)
results["distances"][0]    # List of floats         (lower = more similar)
results["metadatas"][0]    # List of dicts          (subject, topic, etc.)
results["ids"][0]          # List of strings        (note_1, note_2, etc.)

# [0] because we sent one query — results supports batched queries
```

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `AuthenticationError` | Wrong/expired API key | Verify key starts with `gsk_`, no spaces |
| `IDAlreadyExistsError` | `collection.add()` run twice | Run `chroma.delete_collection("name")` or restart runtime |
| `FileNotFoundError: college_notes.csv` | File not uploaded to Colab | Upload via Files panel (folder icon, left sidebar) |
| `ValueError: n_results > count` | top_k larger than document count | Use `n_results <= collection.count()` |
| `TypeError` on embeddings | Forgot `.tolist()` | Add `.tolist()` after `embed_model.encode()` |
| LLM hallucinates despite RAG | System prompt not strict | Add: "Do not use any information outside the provided context." |
| Slow first run | Model downloading (~80MB) | Normal — wait 1-2 minutes |

---

## RAG vs Without RAG

| Feature | Without RAG | With RAG |
|---------|------------|----------|
| Knowledge source | LLM training data (fixed cutoff) | Your custom documents |
| Hallucination risk | High | Low |
| Private data | No | Yes |
| Knowledge cutoff | Yes | No (add new docs anytime) |
| Cites sources | No | Yes |
| Works offline | Yes | Embedding + ChromaDB yes; LLM call no |

---

## When to Use RAG

**Use RAG when:**
- Answering questions from private company documents
- Need up-to-date information beyond LLM training cutoff
- Cannot tolerate hallucination (medical, legal, financial)
- Need to cite sources for answers

**Do NOT use RAG when:**
- Simple general knowledge questions
- Creative writing tasks
- Basic conversational interactions

---

## Real-World Data Engineering RAG Use Cases

1. **Data Catalog Assistants** — "What does the `customer_churn` table contain?"
2. **Pipeline Debugging** — "Why did ETL job fail with error 504?" (retrieves from incident logs)
3. **SQL Generation** — "Write a query for revenue by region" (retrieves schema docs)
4. **Data Governance Q&A** — "What is our PII retention policy?" (retrieves policy docs)

---

## Practice Questions

### Beginner
1. What is hallucination in LLMs? Give one real-world example of harm it could cause.
2. What does RAG stand for? Explain it in 2 sentences.
3. What is the role of the vector database in the RAG pipeline?

### Intermediate
4. What is the difference between the Indexing phase and the Querying phase?
5. Why must you use the same embedding model for both documents AND queries?
6. Why is `temperature=0.1` preferred for LLM calls in RAG systems?

### Coding
7. Modify the retrieval function to also print distance scores.
8. Update the system prompt to make the LLM always answer in bullet points.
9. Add subject-based filtering to retrieve only from the "GenAI" subject.

---

### Answers

**Q1:** Hallucination = LLM generates confident but wrong text. Harm example: Medical AI states wrong drug dosage; patient harmed.

**Q2:** RAG = Retrieval-Augmented Generation. Before answering, the system retrieves relevant documents and gives them to the LLM as context. The LLM answers only from that context.

**Q3:** The vector database stores document embeddings and retrieves the most semantically similar chunks for a given query using distance calculations.

**Q4:** Indexing is done once to build the knowledge base (chunk, embed, store). Querying happens every time a user asks a question (embed query, retrieve, inject, generate).

**Q5:** Both must live in the same vector space for distance calculations to be meaningful. Different models produce incompatible vector spaces.

**Q6:** Low temperature makes the LLM more deterministic and more likely to follow the context strictly, rather than introducing creative variations from training data.

**Q7:**
```python
for i, (doc, dist, meta) in enumerate(zip(
    results["documents"][0],
    results["distances"][0],
    results["metadatas"][0]
)):
    print(f"Result {i+1}: {meta['topic']}  —  distance: {dist:.4f}")
```

**Q8:**
```python
system_prompt = """You are a helpful assistant.
Answer ONLY using the provided context.
Always format your answer as bullet points.
If not found, say: 'This topic is not in my knowledge base.'"""
```

**Q9:**
```python
results = collection.query(
    query_embeddings=[q_vec],
    n_results=2,
    where={"subject": "GenAI"}
)
```

---

## Mini Project Checklist

College Knowledge Assistant — Complete all steps:

- [ ] Load `college_notes.csv` into a pandas DataFrame
- [ ] Prepare `documents`, `ids`, and `metadatas` lists
- [ ] Load the `all-MiniLM-L6-v2` embedding model
- [ ] Create ChromaDB client and collection
- [ ] Generate embeddings for all 15 notes using `.encode(...).tolist()`
- [ ] Add all documents to ChromaDB using `collection.add()`
- [ ] Verify with `collection.count()` — should return 15
- [ ] Write a `retrieve(question)` function using `collection.query()`
- [ ] Write a `build_context(results)` function
- [ ] Write a `generate_answer(question, context)` function with grounding system prompt
- [ ] Combine into one `ask(question)` function
- [ ] Test with an in-scope question — verify correct answer with citation
- [ ] Test with an out-of-scope question — verify "I don't know" response
- [ ] Test with a cross-subject question — verify correct subject retrieved

---

## Completion Checklist

Mark each item when complete:

- [ ] I can explain hallucination in my own words
- [ ] I can draw the 7-step RAG pipeline from memory
- [ ] I ran all notebook cells without errors
- [ ] I successfully indexed 15 notes into ChromaDB
- [ ] I tested retrieval and saw relevant topics returned at rank 1
- [ ] I tested the full pipeline: question → retrieve → inject → answer
- [ ] I verified out-of-scope questions return "I don't know"
- [ ] I completed the College Knowledge Assistant mini project
- [ ] I know at least 2 real-world Data Engineering RAG applications
- [ ] I can explain why temperature=0.1 is used for RAG

---

## Day 9 Preview — AI Agents and Tool Use

Tomorrow you will learn how to build AI systems that **take actions** — not just answer questions.

Topics: What are AI agents, function calling, tool use, multi-step reasoning, building an agent with Groq.

**Homework:** Test your College Knowledge Assistant with 5 of your own questions. Try adding a new row to `college_notes.csv` and re-indexing to verify the system learns new knowledge instantly.

---

*Codeboosters Tech — Data Engineering + GenAI Internship | Day 8 of 10*
