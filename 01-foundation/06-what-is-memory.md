# What Is Memory? (Taxonomy, Lifecycle & Cross-Session Persistence)

In [Chapter 5](./05-what-is-state.md), we explored **State**—the temporary data an agent tracks during a single execution run. But what happens when an agent completes a task, the server shuts down, and the user returns tomorrow?

Without a dedicated persistence layer, the agent suffers from **digital amnesia**. It forgets user preferences, repeats past mistakes, and asks the same onboarding questions every session.

**Memory** is the mechanism that allows an agent to **intentionally distill, store, and retrieve information across independent sessions and executions**.

---

## 1. The Definitive Information Matrix

To build reliable systems, an engineer must never confuse Memory with State, History, Context, or Knowledge:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         INFORMATION HIERARCHY                            │
├─────────────┬───────────────────────────┬───────────────┬────────────────┤
│ Concept     │ Primary Question          │ Lifespan      │ Storage Type   │
├─────────────┼───────────────────────────┼───────────────┼────────────────┤
│ **State**   │ Where are we *right now*? │ Current run   │ RAM / Checkpoint│
│ **History** │ What happened *step-by-step*? Session    │ Message Ledger │
│ **Context** │ What does the LLM see *now*? Single prompt │ Token Window   │
│ **Memory**  │ What was *learned for later*? Cross-session │ Key-Value / Vec│
│ **Knowledge**│ What are the *domain facts*? Permanent     │ RAG / Docs / DB│
└─────────────┴───────────────────────────┴───────────────┴────────────────┘
```

### Concrete Example:
- **History:** User wrote: *"Can you change the chart background to dark? White hurts my eyes."*
- **State:** `{"current_task": "update_css", "theme": "dark", "step": 2}`
- **Memory:** `{"user_id": "u42", "ui_preference": "dark_mode"}` (saved for future visits).
- **Knowledge:** Web documentation: *"Company branding guidelines require dark theme contrast $\ge 4.5:1$."*
- **Context:** The prompt sent to the LLM containing the memory + the immediate task.

---

## 2. The Cognitive Taxonomy of Agent Memory

Borrowing from cognitive psychology, modern agent architectures divide memory into distinct functional types:

```
┌────────────────────────────────────────────────────────┐
│                      AGENT MEMORY                      │
├──────────────────────────┬─────────────────────────────┤
│ 1. SHORT-TERM MEMORY     │ 2. LONG-TERM MEMORY         │
│    (In-Session)          │    (Cross-Session)          │
│                          │                             │
│ • Active State           │ • Semantic Memory (Facts)   │
│ • Message History        │ • Episodic Memory (Events)  │
│ • Scratchpad Buffers     │ • Procedural Memory (Rules) │
└──────────────────────────┴─────────────────────────────┘
```

![Agent Memory Architecture](../assets/agentic-memory-architecture.jpeg)

### The Three Branches of Long-Term Memory:

1. **Semantic Memory (Facts & Preferences):**
   - Discrete, declarative knowledge about entities (users, systems, organizations).
   - *Example:* *"The user lives in London and prefers outputs formatted in Markdown tables."*
   - *Implementation:* Structured Key-Value stores (Redis, SQLite) or Entity-Relationship Graphs.

2. **Episodic Memory (Past Events & Experiences):**
   - Autobiographical records of past trajectories, problem-solving episodes, and tool outcomes.
   - *Example:* *"Last Friday, attempting to deploy the container without flag `--no-cache` failed with exit code 137."*
   - *Implementation:* Vector databases storing embeddings of summarized past sessions.

3. **Procedural Memory (Learned Skills & Workflows):**
   - Rules, instructions, and operational procedures the agent learns through reflection.
   - *Example:* *"When modifying the authentication service, always run unit tests in `tests/auth/` before committing."*
   - *Implementation:* Dynamically updated System Prompts or few-shot example repositories.

---

## 3. The 4-Stage Memory Lifecycle

Memory is not a mindless database dump of every conversation. Storing raw chat logs pollutes the context window with conversational noise.

A production memory system follows a 4-stage lifecycle:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  1. EXTRACT  │ ──► │   2. STORE   │ ──► │  3. RETRIEVE │ ──► │  4. INJECT   │
│  (Distill)   │     │   (Index)    │     │   (Search)   │     │  (Augment)   │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
```

1. **Extract (Distillation):** During or at the end of a session, a background LLM prompt analyzes the conversation to extract high-value facts (ignoring pleasantries, typos, and transient errors).
2. **Store (Indexing):** Extracted facts are stored in a database organized by **Namespaces** (e.g., `(user_id, "preferences")` or `(project_id, "learned_rules")`).
3. **Retrieve (Search):** When a new user request arrives, relevant memories are fetched via exact key matching or semantic vector similarity.
4. **Inject (Context Augmentation):** Retrieved memories are prepended into the System Prompt so the model immediately acts on past context.

---

## 4. The Engineering Hazards: Pollution & Poisoning

When designing memory systems, you must guard against two critical failure modes:

### Hazard A: Context Pollution
- **The Problem:** If an agent retrieves 50 past memories on every turn, the context window fills with irrelevant baggage. This increases token costs, slows inference latency, and degrades the model's focus (the *"Lost in the Middle"* problem).
- **The Solution:** Set a strict limit (e.g., top 3–5 most relevant memories) and use high semantic similarity thresholds ($> 0.82$).

### Hazard B: Memory Poisoning & Contradictions
- **The Problem:** In January, the user says: *"I only use Python 3.10."* In November, the user says: *"We upgraded everything to Python 3.12."* If both facts persist, the agent acts on contradictory instructions.
- **The Solution:** Implement memory **recency weighting** and upsert logic (updating existing entities rather than blindly appending duplicates).

---

## 5. Python Mental Model: A Complete Memory Engine

Here is a self-contained Python demonstration showing how to distill, persist, and inject long-term user memories:

```python
import sqlite3
import json
from typing import Dict, List, Optional

# 1. DATABASE STORE: Persists memories across independent runs
class MemoryStore:
    def __init__(self, db_path: str = ":memory:"):
        self.conn = sqlite3.connect(db_path)
        self.create_table()

    def create_table(self):
        with self.conn:
            self.conn.execute("""
                CREATE TABLE IF NOT EXISTS user_memories (
                    user_id TEXT,
                    memory_key TEXT,
                    memory_value TEXT,
                    PRIMARY KEY (user_id, memory_key)
                )
            """)

    def upsert_memory(self, user_id: str, key: str, value: str):
        with self.conn:
            self.conn.execute("""
                INSERT INTO user_memories (user_id, memory_key, memory_value)
                VALUES (?, ?, ?)
                ON CONFLICT(user_id, memory_key) DO UPDATE SET memory_value = excluded.memory_value
            """, (user_id, key, value))

    def get_memories(self, user_id: str) -> Dict[str, str]:
        cursor = self.conn.cursor()
        cursor.execute("SELECT memory_key, memory_value FROM user_memories WHERE user_id = ?", (user_id,))
        return {row[0]: row[1] for row in cursor.fetchall()}


# 2. DISTILLATION: Simulates an LLM extracting long-term facts
def distill_memories(raw_user_message: str) -> List[tuple]:
    """
    In production, this is an LLM call with a prompt:
    'Extract any persistent user preferences or facts as (key, value) pairs.'
    """
    extracted = []
    if "dark mode" in raw_user_message.lower():
        extracted.append(("theme_preference", "dark_mode"))
    if "typescript" in raw_user_message.lower():
        extracted.append(("preferred_language", "TypeScript"))
    return extracted


# 3. RUNTIME EXECUTION: Session 1 (Learning) vs Session 2 (Recalling)
store = MemoryStore()

# --- SESSION 1: User gives instructions ---
user_id = "user_dev_99"
message_turn_1 = "Please write a landing page in TypeScript. Also, I always prefer dark mode."

# Agent extracts and persists facts
for key, val in distill_memories(message_turn_1):
    store.upsert_memory(user_id, key, val)
print("Session 1: Extracted and saved memories:", store.get_memories(user_id))

# --- SESSION 2: Days later, a brand new session starts ---
print("\n--- Days Later: Session 2 Starts ---")
# 1. Retrieve persistent memories for this user
active_memories = store.get_memories(user_id)

# 2. Inject into the System Prompt
system_prompt = f"""You are an expert AI assistant.
[USER MEMORIES & PREFERENCES]
{json.dumps(active_memories, indent=2)}
"""

print("System Prompt dynamically generated for Session 2:")
print(system_prompt)
```

---

## 6. Ecosystem Implementations

<details>
<summary><b>🔍 Code Example: LangGraph Cross-Thread Memory with BaseStore (Python)</b></summary>

LangGraph differentiates between **Checkpointers** (in-thread state) and **Stores** (cross-thread long-term memory):

```python
from langgraph.store.memory import InMemoryStore
from langgraph.graph import StateGraph, START, END
from typing import TypedDict

# 1. Define Store (Persists across all threads and users)
store = InMemoryStore()

# 2. Put a memory in a dedicated namespace: (user_id, "profile")
store.put(
    namespace=("user_42", "preferences"),
    key="coding_style",
    value={"language": "Python", "framework": "FastAPI"}
)

# 3. In any LangGraph node, access the store dynamically:
def greeting_node(state: dict, *, store: InMemoryStore):
    # Retrieve memories for the specific user across sessions
    user_prefs = store.get(namespace=("user_42", "preferences"), key="coding_style")
    lang = user_prefs.value.get("language") if user_prefs else "Generic"
    return {"message": f"Welcome back! Ready to write some {lang}?"}
```
</details>

<details>
<summary><b>🔍 Code Example: Managed Memory with Mem0 (Python)</b></summary>

[Mem0](https://github.com/mem0ai/mem0) is a dedicated open-source memory engine for AI agents that handles automatic extraction, vector search, and conflict resolution:

```python
from mem0 import Memory

# Initialize memory client
m = Memory()

# 1. Add conversation turns (Mem0 automatically distills and deduplicates facts)
m.add(
    "I am vegetarian and allergic to mushrooms.",
    user_id="alice",
    metadata={"category": "dietary_restrictions"}
)

# 2. Query memories semantically on a new interaction
relevant_memories = m.search("What can I cook for dinner?", user_id="alice")
for mem in relevant_memories.get("results", []):
    print("Recalled Memory:", mem["memory"])
```
</details>

---

## 7. Summary & Rules of Thumb

1. **Memory is Distilled, Not Dumped:** Never dump raw interaction histories directly into long-term memory. Always pass them through an extraction filter.
2. **Namespace Everything:** Structure memories by user ID, organization ID, and project domain to prevent cross-tenant data leakage.
3. **Handle Updates Gracefully:** Provide mechanisms for users to inspect, edit, or delete their stored memories (GDPR compliance and conflict resolution).

We will explore advanced memory architectures (Semantic, Episodic, and Procedural memory) in depth in **[Chapter 5: Memory Systems](../05-memory-systems/01-short-term-vs-long-term-memory.md)**.
