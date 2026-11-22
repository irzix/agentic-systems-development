# What is RAG?

**RAG (Retrieval-Augmented Generation)** is the process of retrieving relevant external information and adding it to the context before the LLM generates an answer.

Instead of relying only on what the model learned during training, RAG grounds the model in specific, up-to-date facts.

`User Query → Retrieve Relevant Chunks → Add to Context → LLM → Answer`

---

## Why RAG?

- **Fresh Knowledge**: Overcomes model training cutoff dates.
- **Private Data**: Connects models to internal documents, databases, and APIs.
- **Reduces Hallucinations**: Gives the model facts to reference instead of guessing.
- **Source Grounding**: Allows the model to cite exact sources.

---

## The Basic Flow

1. **Ingest**: Split documents into chunks and store their embeddings in a vector database.
2. **Retrieve**: Find the most relevant chunks for the user's query.
3. **Generate**: Pass the retrieved chunks to the LLM to generate the answer.

---

## Minimal Example

```python
# 1. Retrieve relevant chunks
chunks = vector_db.search(query, top_k=3)

# 2. Augment context and generate
context = "\n".join([c.text for c in chunks])
prompt = f"Answer using this context:\n{context}\n\nQuestion: {query}"

response = llm.generate(prompt)
```

---

> **RAG turns the LLM into a reasoning engine that operates on retrieved, verified facts rather than relying only on its internal memory.**
