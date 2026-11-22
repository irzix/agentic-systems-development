# Short-term vs. Long-term Memory

**Agent Memory** is the ability of an AI agent to store, update, and recall information from past interactions to guide current and future decisions.

Without memory, an agent starts every session with a blank slate, repeating questions and making the same mistakes.

`Experience ──► Store in Memory ──► Retrieve when Relevant ──► Inject to Context ──► Action`

---

## Short-Term vs. Long-Term Memory

| Feature | Short-Term Memory (Working Memory) | Long-Term Memory (Persistent Memory) |
|---|---|---|
| **Lifespan** | Temporary (Current session / execution only) | Permanent (Persists across sessions and days) |
| **Storage Location** | In-Context prompt & RAM | External Database, Vector DB, or Key-Value store |
| **Purpose** | Track immediate task steps, tools, and variables | Remember user preferences, past executions, and facts |
| **Capacity** | Limited by context window size | Virtually unlimited |

---

## 1. Short-Term Memory (Working Memory)
Short-term memory holds the information the agent needs **right now** to complete its current task:
- Current user messages and conversation turn.
- Intermediate tool outputs and scratchpad reasoning.
- Active variables and workflow state.

*Once the session ends or the agent resets, short-term memory is cleared.*

---

## 2. Long-Term Memory (Persistent Memory)
Long-term memory stores distilled experiences outside the context window and retrieves them when relevant:
- **Semantic Memory:** Facts and user profile (e.g., "User uses macOS and Python 3.12").
- **Episodic Memory:** Specific past experiences (e.g., "Last week we fixed a Docker build error in module X").
- **Procedural Memory:** Rules and instructions (e.g., "Always run linter before committing").

---

![Agent Memory Architecture](../assets/agentic-memory-architecture.jpeg)

---

## Minimal Example (Working vs. Persistent Memory)

```python
class AgentMemory:
    def __init__(self, user_id: str, db_client):
        # Short-term memory (in-memory for current run)
        self.working_memory = []
        
        # Long-term memory (persistent store)
        self.user_id = user_id
        self.db = db_client

    def remember_fact(self, fact: str):
        """Save to persistent long-term storage."""
        self.db.save_memory(user_id=self.user_id, text=fact)

    def load_relevant_memories(self, query: str) -> list[str]:
        """Retrieve relevant past memories into active context."""
        return self.db.semantic_search(user_id=self.user_id, query=query, top_k=2)
```

---

> **Short-term memory is what the agent is thinking about right now; long-term memory is what the agent has learned over time.**

---

## References & Further Reading

- [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442)
  - Landmark Stanford paper introducing memory streams, reflection, and retrieval in autonomous agents.
- [MemGPT: Towards LLMs as Operating Systems](https://arxiv.org/abs/2310.08560)
  - Proposes hierarchical memory management (working context vs. persistent storage) inspired by OS virtual memory.
- [State and Memory is All You Need for Robust and Reliable AI Agents](https://arxiv.org/abs/2507.00081)
  - Explores how structured memory design improves agent reliability in production systems.
