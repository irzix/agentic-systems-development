# How Do LLMs Work? (From Machine Learning to Reasoning & Agents)

Before understanding AI Agents, we must understand the core engine that powers them: the **Large Language Model (LLM)**.

To build reliable agentic systems, knowing that *"an LLM predicts the next token"* is not enough. You also need to understand:
1. Where LLMs sit in the broader world of **Machine Learning** and **NLP**.
2. How data enters the model (**Ingestion**) and how responses are generated (**Inference**).
3. How modern models perform multi-step **Reasoning**.
4. The technical boundary between a pure **LLM** and an **Agent**.

---

## 1. The Big Picture: AI → ML → Deep Learning → LLMs

To understand LLMs, let's look at how natural language processing evolved:

```
┌────────────────────────────────────────────────────────┐
│ Artificial Intelligence (AI)                           │
│  ┌──────────────────────────────────────────────────┐  │
│  │ Machine Learning (ML)                            │  │
│  │   ┌───────────────────────────────────────────┐  │  │
│  │   │ Deep Learning (DL - Neural Networks)     │  │  │
│  │   │   ┌────────────────────────────────────┐  │  │  │
│  │   │   │ Large Language Models (LLMs)       │  │  │  │
│  │   │   └────────────────────────────────────┘  │  │  │
│  │   └───────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

### The Three Eras of NLP (Natural Language Processing)

1. **Rule-Based & Statistical NLP (Pre-2010s):**
   - Relied on regular expressions, dictionaries, and statistical counts (e.g., N-grams, TF-IDF).
   - Extremely rigid: if a user phrased something slightly differently, the system broke.

2. **Classical Deep Learning NLP (2013 – 2018):**
   - Introduced neural networks (RNNs, LSTMs, and early Encoders like BERT).
   - Text was converted to dense vectors (**Word2Vec**).
   - **Limitation:** Models were *task-specific*. You trained one model for sentiment analysis, another for translation, and another for named entity recognition (NER). They also struggled with long-range context.

3. **Modern Autoregressive LLMs (2018 – Present):**
   - Built on the **Transformer** architecture (specifically Decoder-only models like GPT, Claude, LLaMA).
   - **General-Purpose:** A single model can translate, write code, summarize, reason, and follow instructions.
   - **In-Context Learning:** Instead of retraining or fine-tuning weights for every new task, we guide the model purely through *prompts* and *context*.

### Python Mental Model: Rule-Based vs. ML vs. LLM

```python
# 1. Rule-Based Approach: Hardcoded logic
def classify_sentiment_rule(text: str) -> str:
    positive_words = {"good", "great", "excellent", "love"}
    words = set(text.lower().split())
    return "POSITIVE" if words & positive_words else "NEGATIVE"


# 2. Classical Machine Learning: Feature Extraction + Trained Classifier
# Input Text -> TF-IDF Vectorizer -> Logistic Regression / SVM -> Label
# (Requires labeled training datasets and model training for each task)


# 3. Modern LLM: In-Context Instruction Following (Zero-Shot)
def classify_sentiment_llm(text: str, llm_client) -> str:
    prompt = f"Analyze sentiment as POSITIVE or NEGATIVE:\nText: {text}\nSentiment:"
    return llm_client.generate(prompt)  # No retraining needed!
```

---

## 2. Core Mechanics: How LLMs Actually Work

At its mathematical core, an LLM is a probabilistic function trained to predict the most likely **next token** given a sequence of previous tokens.

$$\text{Next Token} = \arg\max P(\text{token} \mid \text{context})$$

### A. Tokens: The Vocabulary of LLMs
LLMs do not process raw text or full words directly. Text is converted into chunks called **tokens** using an algorithm such as **Byte-Pair Encoding (BPE)**.

```
"Agent development is exciting!" 
  ↓ (Tokenizer)
["Agent", " development", " is", " exciting", "!"]
  ↓ (Token IDs)
[12845, 4120, 374, 9821, 0]
```

* 1 token is roughly $\sim 0.75$ words in English (or $\sim 4$ characters).
* Tokenization varies across languages and models.

### B. Embeddings: Giving Tokens Semantic Meaning
Computers cannot compute meaning from arbitrary integer IDs. Each token ID is mapped to a high-dimensional vector called an **Embedding** (e.g., a vector of 4,096 numbers).

Tokens with similar semantic meanings end up closer to each other in this high-dimensional vector space.

$$\text{Token ID} \xrightarrow{\text{Embedding Matrix}} \vec{v} \in \mathbb{R}^{d_{\text{model}}}$$

> [!NOTE]
> Embeddings are also the foundation of **Vector Search** and **RAG (Retrieval-Augmented Generation)**, which we explore in Chapter 5.

### C. The Transformer & Self-Attention
Why can LLMs understand long sentences and pronouns? The secret is **Self-Attention** (introduced in *Attention Is All You Need*, 2017).

Consider this sentence:
> *"The engineer tested the agent because **it** was ready."*

How does the model know what **"it"** refers to?
- Self-attention calculates a mathematical affinity score between every token and every other token in the prompt.
- The token `"it"` pays high attention to `"agent"`, rather than `"engineer"`.

```
"The"      ───────► [Attention Weights]
"engineer" ───────► [Attention Weights]
"tested"   ───────► [Attention Weights]
"the"      ───────► [Attention Weights]
"agent"    ◄─────── (Strong Attention Link) ──┐
"because"  ───────► [Attention Weights]        │
"it"       ───────────────────────────────────┘
```

### D. Sampling & Temperature: Determinism vs. Creativity
Once the model calculates probabilities for the next token across its entire vocabulary, how does it choose one?

```python
import math
import random

def sample_next_token(logits: dict[str, float], temperature: float = 1.0) -> str:
    """
    Simulates how LLM sampling works under the hood.
    Higher temperature = more diverse / creative.
    Temperature -> 0.0 = greedy (always picks the highest probability).
    """
    if temperature == 0.0:
        return max(logits, key=logits.get)

    # Apply temperature to raw logits and calculate softmax probabilities
    scaled = {token: math.exp(score / temperature) for token, score in logits.items()}
    total = sum(scaled.values())
    probs = {token: score / total for token, score in scaled.items()}

    # Sample randomly according to the probabilities
    tokens, weights = list(probs.keys()), list(probs.values())
    return random.choices(tokens, weights=weights, k=1)[0]
```

---

## 3. Ingestion vs. Inference: The Data & Execution Lifecycle

When working with LLMs in software engineering, you will often hear about **Ingestion** and **Inference**. Understanding the distinction is vital for optimizing speed and cost in agent loops.

```
┌────────────────────────────────────────────────────────┐
│                   DATA INGESTION                       │
│  1. Pre-training Ingestion (Trillions of tokens)       │
│  2. RAG / Knowledge Ingestion (Docs -> Embeddings)     │
│  3. Context Ingestion (Prompt + History into Window)   │
└──────────────────────────┬─────────────────────────────┘
                           │ Loaded Context
                           ▼
┌────────────────────────────────────────────────────────┐
│                   INFERENCE ENGINE                     │
│  Phase 1: PREFILL (Process Prompt in Parallel)         │
│  Phase 2: DECODE  (Generate Tokens Autoregressively)   │
│  Accelerated by:  KV CACHING                           │
└────────────────────────────────────────────────────────┘
```

### 1. Ingestion: How Information Enters the System
Ingestion refers to loading, preparing, and indexing data so the model or agent can access it:

| Ingestion Level | What Happens | When It Happens |
| :--- | :--- | :--- |
| **Pre-training Ingest** | Ingesting massive web-scale corpora to learn model weights ($\theta$). | Once (during model creation). |
| **RAG / Storage Ingest** | Chunking markdown/PDFs, generating embeddings, and storing in vector databases. | Offline or via asynchronous background pipelines. |
| **Runtime Context Ingest** | Feeding system prompts, conversation history, and tool outputs into the model's active context window. | Dynamically on every turn of an agent loop. |

### 2. Inference: How the LLM Generates Output
Inference is running the trained model to process context and produce a completion. Inference consists of two distinct phases:

#### Phase A: The Prefill Phase (Prompt Ingestion)
- The model ingests the entire input prompt (System prompt + History + Tools + User message).
- Because all input tokens are already known, GPUs can process them **in parallel**.
- This phase is **Compute-Bound** (FLOPs-heavy) and relatively fast per token.

#### Phase B: The Decode Phase (Token Generation)
- The model generates tokens **one by one** (autoregressively).
- Each new token requires reading the model weights from GPU memory into the compute cores.
- This phase is **Memory-Bandwidth Bound** and is where latency accumulates.

#### Why KV Cache Matters for Agents
In an Agent loop, the conversation grows with every action:
`User Prompt` → `Tool Call 1` → `Observation 1` → `Tool Call 2` → ...

Without optimization, the model would have to recalculate attention for all previous tokens from scratch on every step!
- **KV Cache (Key-Value Cache):** Stores previously computed Attention Key and Value tensors in GPU RAM.
- **Agent Implication:** When an agent takes multiple steps, KV caching allows the model to only compute attention for the *new* tokens, drastically reducing turn-by-turn latency.

---

## 4. How Reasoning Works: From Intuition to Deliberation

Standard next-token prediction can answer simple questions quickly. But complex tasks (such as writing an algorithm, planning multi-step tool calls, or debugging code) require **Reasoning**.

### System 1 vs. System 2 Thinking in AI

Borrowing terminology from cognitive psychology (Daniel Kahneman):

- **System 1 (Fast, Intuitive, Reflexive):**
  - Standard LLMs generating tokens at high speed without stopping to verify.
  - Good for: Translation, summarization, creative writing.
  - Risk: Can easily hallucinate or pick an incorrect tool because it commits to the first plausible token.

- **System 2 (Slow, Deliberative, Logical):**
  - The model explicitly breaks problems into intermediate reasoning steps, checks hypotheses, and backtracks if an error is detected.
  - Good for: Architecture planning, math, complex code refactoring, agent decision-making.

```
Standard (System 1):
Prompt ───────────────────────────────► Direct Answer (Higher error rate)

Reasoning (System 2 / CoT):
Prompt ──► Step 1 ──► Step 2 ──► Verify ──► Self-Correct ──► Final Answer
```

### The Three Layers of Reasoning in LLMs

#### 1. Chain-of-Thought (CoT) Prompting
Prompting the model to *"think step by step"* forces it to generate intermediate tokens. Because transformers condition each new token on all previous tokens, writing down intermediate steps gives the model a computational "scratchpad".

#### 2. ReAct (Reason + Act)
In Agent systems, reasoning is coupled with action. Before invoking a tool, the agent reasons about *why* it needs the tool:
```
Thought: I need to check the user's order status. I should call get_order_status.
Action: get_order_status(order_id="12345")
Observation: Status = "Shipped, out for delivery"
Thought: The package is out for delivery. I have enough information to answer.
Final Answer: Your order #12345 is currently out for delivery!
```

#### 3. Test-Time Compute & Reinforcement Learning (e.g., OpenAI o1/o3, DeepSeek-R1)
Modern **Reasoning Models** are trained using Reinforcement Learning with verifiable rewards (RLVR):
- **Test-Time Compute:** The model spends dynamic compute *during inference* (generating internal hidden thought chains, exploring multiple search paths, and self-correcting) before emitting the visible response.
- **Error Recovery:** If an intermediate thought leads to a contradiction, the model backtracks and explores alternative solutions internally.

> [!TIP]
> **Why this matters for Agents:** Reasoning models significantly decrease tool hallucination and failure rates in complex agent workflows because the model verifies its plan before calling an external tool.

---

## 5. The Context Window & Hallucinations

### What Goes Into the Context Window?
An LLM has no persistent memory; its entire world during inference is bounded by its **Context Window**.

In an agentic system, the context window contains:

```
┌─────────────────────────────────────────────────────────────┐
│ 1. System Prompt     (Identity, instructions, tool schemas) │
│ 2. Few-Shot Examples (Reference patterns, expected behavior)│
│ 3. Memory & State    (Summary of past turns, current goals) │
│ 4. RAG Knowledge     (Chunks retrieved from vector store)   │
│ 5. Tool Output       (Raw API responses, error traces)      │
│ 6. User Message      (The latest instruction)               │
└─────────────────────────────────────────────────────────────┘
```

### Why Do LLMs Hallucinate?
Hallucination is not a bug; it is an inherent property of probabilistic models:
1. **No Source of Absolute Truth:** The model generates what sounds statistically plausible based on training data patterns, not what is factually verified.
2. **Context Compression:** The model cannot hold all world knowledge in its parameters without lossy compression.
3. **Overconfidence:** Autoregressive models are incentivized to produce fluent text even when uncertain.

**How Agentic Systems Fix This:**
We don't rely on the LLM's static memory for critical facts. We augment it with:
- **Tools:** To fetch live data (e.g., databases, search engines, APIs).
- **RAG:** To ground responses in verified documents.
- **Validation Layers:** Programmatic schemas (e.g., Pydantic) to validate outputs.

---

## 6. The Fundamental Rule: LLM ≠ Agent

This distinction is the starting point for everything we build in this course:

```
┌───────────────────────────────────────────────┐
│ LLM (The Brain / Model)                       │
│ Input Context ──► Predict Next Token ──► Text │
└───────────────────────────────────────────────┘

                      VS

┌───────────────────────────────────────────────┐
│ AGENT (The Complete System)                   │
│                                               │
│             ┌──────────────┐                  │
│             │  Controller  │                  │
│             └──────┬───────┘                  │
│                    │                          │
│     ┌──────────────┼──────────────┐           │
│     ▼              ▼              ▼           │
│  ┌─────┐      ┌─────────┐    ┌─────────┐      │
│  │ LLM │◄────►│  Tools  │◄──►│ Memory  │      │
│  └─────┘      └─────────┘    └─────────┘      │
│                    │                          │
│                    ▼                          │
│              Environment                      │
└───────────────────────────────────────────────┘
```

> **An LLM is a stateless prediction engine. An Agent is a stateful system wrapped around an LLM.**

The LLM is responsible for comprehension, reasoning, and deciding what to do next. The surrounding software handles state, memory, tool dispatch, error retries, and execution safety.

---

## 7. Milestone References & Further Reading

To dive deeper into the research that shaped modern LLMs and reasoning:

1. **[Attention Is All You Need (2017)](https://arxiv.org/abs/1706.03762)** — *Vaswani et al.*
   The foundational paper introducing the Transformer architecture and Multi-Head Self-Attention.
2. **[Language Models are Few-Shot Learners (2020)](https://arxiv.org/abs/2005.14165)** — *Brown et al. (GPT-3)*
   Demonstrated that scaling autoregressive models produces remarkable in-context learning capabilities without fine-tuning.
3. **[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models (2022)](https://arxiv.org/abs/2201.11903)** — *Wei et al.*
   Showed that generating step-by-step intermediate thoughts dramatically boosts performance on complex reasoning tasks.
4. **[ReAct: Synergizing Reasoning and Acting in Language Models (2022)](https://arxiv.org/abs/2210.03629)** — *Yao et al.*
   The seminal framework that combined Chain-of-Thought reasoning with external action execution, laying the groundwork for modern agents.
5. **[DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning (2025)](https://arxiv.org/abs/2501.12948)** — *DeepSeek-AI*
   Showcased how large-scale reinforcement learning (without massive human supervised data) enables self-directed reasoning, search, and verification.
