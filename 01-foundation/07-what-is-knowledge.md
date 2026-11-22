# What Is Knowledge? (Grounding, Formats & Information Chunks)

In the previous chapters, we covered **State** (what is happening *now*) and **Memory** (what was learned *from past user sessions*). 

This chapter tackles the third pillar of agentic systems: **Knowledge**. 

Knowledge represents the verifiable facts, domain rules, enterprise data, and documentation an agent needs to understand its operational environment and produce reliable results.

---

## 1. Parametric vs. Non-Parametric Knowledge

To build production agents, you must understand the difference between what an LLM *knows in its weights* versus what it *retrieves from the outside world*:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        KNOWLEDGE DUALITY                               │
├────────────────────────────┬───────────────────────────────────────────┤
│ Parametric Knowledge       │ Non-Parametric (External) Knowledge       │
├────────────────────────────┼───────────────────────────────────────────┤
│ • Stored inside neural     │ • Stored outside the model                │
│   network weights (θ)      │   (Databases, PDFs, APIs, Vector Stores)  │
│                            │                                           │
│ • Frozen at training time  │ • Dynamic and updated in real time        │
│   (Knowledge Cutoff)       │                                           │
│                            │                                           │
│ • Prone to hallucinations  │ • Verifiable ground truth with citations  │
│                            │                                           │
│ • Expensive to update      │ • Cheap to update                         │
│   (Requires retraining/FT) │   (Just update a file or database row)    │
└────────────────────────────┴───────────────────────────────────────────┘
```

> **The Grounding Principle:**  
> Never rely on an LLM's parametric memory for proprietary business rules, live prices, or private customer records. External knowledge provides **Grounding**—anchoring the model’s answers to authoritative facts.

---

## 2. The Three Formats of Knowledge

In enterprise systems, an agent must interact with knowledge stored in three distinct formats:

```
┌─────────────────────────────────────────────────────────────┐
│                 ENTERPRISE KNOWLEDGE TYPES                  │
├─────────────────┬────────────────────┬──────────────────────┤
│ Structured      │ Semi-Structured    │ Unstructured         │
├─────────────────┼────────────────────┼──────────────────────┤
│ • PostgreSQL/SQL│ • JSON REST APIs   │ • Technical PDFs     │
│ • Knowledge Graph│ • CSV Spreadsheets │ • Notion / Markdown  │
│   (Neo4j triplets)• YAML / XML       │ • Customer transcripts│
├─────────────────┼────────────────────┼──────────────────────┤
│ Access Method:  │ Access Method:     │ Access Method:       │
│ Text-to-SQL /   │ Schema Parsing /   │ Vector Search /      │
│ Graph Cypher    │ Tool Calling       │ RAG Pipelines        │
└─────────────────┴────────────────────┴──────────────────────┘
```

![Knowledge Flow](../assets/agentic-knowledge-flow.jpeg)

---

## 3. Information Units: Why We Need Chunks

A common rookie mistake is attempting to feed an entire 200-page PDF or an entire software repository into the context window. 

Even with modern million-token context windows, this causes three severe problems:
1. **Inference Latency & Cost:** Processing hundreds of thousands of input tokens on every turn is slow and expensive.
2. **"Lost in the Middle" Degradation:** Models pay less attention to facts buried deep inside massive prompts.
3. **Context Noise:** Irrelevant paragraphs distract the agent's reasoning engine.

The solution is breaking knowledge down into atomic units called **Chunks**.

### What Is a Chunk?
A **Chunk** is a focused, self-contained piece of a larger document designed to be indexed and retrieved independently:

```
Original Document (100 Pages)
  │
  ├──► Chunk 1: [Authentication Overview & Token Expiration] (350 tokens)
  ├──► Chunk 2: [OAuth2 Refresh Flow & Code Examples]         (420 tokens)
  └──► Chunk 3: [Role-Based Access Control Policies]          (300 tokens)
```

### Chunking Strategies:
- **Fixed-Size Chunking:** Splitting text every $N$ characters or tokens (e.g., 500 tokens with a 50-token overlap). Simple, but can cut sentences or code functions in half.
- **Semantic Chunking:** Splitting text along natural boundaries (markdown headers, paragraphs, or semantic topic shifts) so that closely related concepts remain intact.

> [!TIP]
> **The Golden Rule of Chunking:** A chunk should be large enough to contain complete context, but small enough to contain only one coherent topic. (We explore this deeply in [Chapter 5: RAG](../05-rag/02-chunking-strategies.md)).

---

## 4. Python Mental Model: Chunking and Grounded Answering

Here is a pure Python demonstration showing how raw documentation is chunked, indexed, and used to ground an LLM prompt:

```python
from typing import List, Dict

# 1. RAW UNSTRUCTURED DOCUMENTATION
ENTERPRISE_DOC = """
# Company Travel Policy 2026

## Hotel Booking Rules
Employees traveling for business must book hotels through our internal portal.
The maximum allowable rate for domestic hotels is $200 per night.
For international travel, the maximum rate is $350 per night.
Any hotel booking exceeding these limits requires VP written approval.

## Meal Allowances
Daily meal allowances are fixed at $75 per day.
Alcoholic beverages are not reimbursable under any circumstances.
Receipts must be submitted within 14 days of travel completion.
"""

# 2. CHUNKING: Split along Markdown headers (Semantic Chunking)
def chunk_markdown_by_headers(text: str) -> List[Dict[str, str]]:
    sections = text.strip().split("## ")
    chunks = []
    
    for section in sections:
        if not section.strip():
            continue
        lines = section.strip().split("\n")
        title = lines[0].replace("#", "").strip()
        body = "\n".join(lines[1:]).strip()
        chunks.append({"title": title, "content": body})
        
    return chunks

# 3. RETRIEVAL & GROUNDING: Find relevant chunk and inject into prompt
def generate_grounded_prompt(user_query: str, chunks: List[Dict[str, str]]) -> str:
    # Simple keyword retrieval (simulating vector search)
    query_words = set(user_query.lower().split())
    best_chunk = max(chunks, key=lambda c: len(set(c["content"].lower().split()) & query_words))

    # Construct the Grounded Prompt (Zero-Hallucination Guardrail)
    return f"""You are a corporate policy assistant.
Answer the question using ONLY the provided verified knowledge. If the answer is not in the text, say 'I cannot find that in the official policy'.

[VERIFIED KNOWLEDGE CHUNK]
Topic: {best_chunk['title']}
Content: {best_chunk['content']}

[USER QUESTION]
{user_query}

[ANSWER]:"""


# Execution
doc_chunks = chunk_markdown_by_headers(ENTERPRISE_DOC)
query = "What is the budget limit for domestic hotels?"
grounded_prompt = generate_grounded_prompt(query, doc_chunks)

print(grounded_prompt)
```

---

## 5. Ecosystem Implementations

<details>
<summary><b>🔍 Code Example: AWS Bedrock Knowledge Bases (Python / boto3)</b></summary>

Amazon Bedrock Knowledge Bases is a fully managed enterprise RAG service that automates document chunking, embeddings, and vector storage (OpenSearch Serverless):

```python
import boto3

# Initialize Bedrock Agent Runtime
client = boto3.client("bedrock-agent-runtime", region_name="us-east-1")

# Retrieve grounded knowledge chunks directly from Bedrock Knowledge Base
response = client.retrieve(
    knowledgeBaseId="KB_ENTERPRISE_DOCS_01",
    retrievalQuery={"text": "domestic hotel travel allowance"},
    retrievalConfiguration={
        "vectorSearchConfiguration": {
            "numberOfResults": 3  # Return top 3 most relevant chunks
        }
    }
)

for result in response.get("retrievalResults", []):
    print("Found Chunk Content:", result["content"]["text"][:150], "...")
    print("Source URI:", result["location"]["s3Location"]["uri"])
```
</details>

<details>
<summary><b>🔍 Code Example: Semantic Vector Search with ChromaDB / LangChain (Python)</b></summary>

```python
from langchain_community.vectorstores import Chroma
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

# 1. Create document chunks
docs = [
    Document(page_content="Domestic hotel rate limit is $200/night.", metadata={"source": "policy.md"}),
    Document(page_content="Daily meal allowance is $75/day.", metadata={"source": "policy.md"})
]

# 2. Embed and store in vector database
vectorstore = Chroma.from_documents(docs, embedding=OpenAIEmbeddings())

# 3. Query relevant knowledge
results = vectorstore.similarity_search("How much can I spend on a hotel in Chicago?", k=1)
print("Retrieved Knowledge Chunk:", results[0].page_content)
```
</details>

---

## 6. Summary: The Knowledge Checklist

1. **Verify Source of Truth:** Never let the model guess domain facts; ground critical operations in external knowledge.
2. **Chunk Before Ingestion:** Transform unwieldy documents into modular, semantically distinct chunks.
3. **Cite Sources:** Ensure your agent quotes chunk metadata (e.g., file name, section, page number) so human users can audit answers.

We explore retrieval strategies, reranking, and Hybrid Search in depth in **[Chapter 5: Knowledge & RAG](../05-rag/01-what-is-rag.md)**.
