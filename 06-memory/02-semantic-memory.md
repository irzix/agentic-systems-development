# Semantic Memory

**Semantic Memory** is an agent's storage for general facts, concepts, user profiles, and preferences learned over time, independent of specific moments in time.

While episodic memory remembers *when* something happened, semantic memory stores *what is true*.

`Conversation ──(Extract Facts)──► Semantic Memory Store ──(Inject)──► Future Prompts`

---

## What Does Semantic Memory Store?

- **User Preferences:** "Prefers Python over TypeScript", "Likes concise explanations".
- **User Profile & State:** "Uses macOS", "Enterprise tier customer with ID #4092".
- **Domain & Project Facts:** "The production database runs on Postgres 16 on port 5432".
- **Agent Self-Identity:** "My role is senior code reviewer with strict security policies".

---

## Semantic Memory Lifecycle

```text
1. Extract   ──► LLM identifies persistent facts from the conversation.
2. Update    ──► Adds new facts and updates/replaces outdated ones (conflict resolution).
3. Store     ──► Persists in a vector store, key-value table, or user profile database.
4. Retrieve  ──► Injects relevant user facts into the prompt on future runs.
```

---

## Semantic Memory vs. Knowledge Base (RAG)

| Dimension | Semantic Memory | Knowledge Base (RAG) |
|---|---|---|
| **Source** | Learned dynamically from user interactions | Pre-loaded external docs and manuals |
| **Scope** | Personalized to specific users or agents | General domain/company information |
| **Updates** | Continuously modified and updated | Periodic batch ingestion |

---

## Minimal Example (Fact Extraction & Profile Store)

```python
# 1. Fact extraction prompt
extract_prompt = f"""
Extract any permanent user preferences or facts from this conversation.
Conversation: "{user_message}"
Output JSON: {{"preferences": [...], "facts": [...]}}
"""

# 2. Update user semantic memory store
extracted = llm.extract_json(extract_prompt)

for fact in extracted["facts"]:
    user_semantic_db.upsert(
        user_id="user_123",
        key=fact["topic"],
        value=fact["content"]
    )
```

---

> **Semantic memory turns one-off user statements into permanent personalized agent intelligence.**

---

## References & Further Reading

- [Cognitive Architectures for Language Agents (CoALA)](https://arxiv.org/abs/2309.02427)
  - Princeton and DeepMind framework categorizing working, semantic, episodic, and procedural memory in AI agents.
- [Mem0: The Memory Layer for Personalized AI](https://github.com/mem0ai/mem0)
  - Production reference implementation for extracting, updating, and querying user semantic profiles.
