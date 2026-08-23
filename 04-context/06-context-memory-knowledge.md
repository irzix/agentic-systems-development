# Context vs. Memory vs. Knowledge

These concepts are related, but they serve different purposes in an agentic system.

| Concept | Purpose | Example |
|---|---|---|
| **Context** | Information available to the model right now | Current task, tool results, recent messages |
| **Memory** | Information retained from previous interactions or executions | User preferences, previous decisions |
| **Knowledge** | Information about the world or a domain | Documentation, company policies, product data |

The relationship is often:

**Memory and Knowledge → Retrieval → Context → LLM**

For example, an agent may have thousands of documents in its knowledge base and years of stored memories, but only retrieve the small portion relevant to the current task and place it into the context.

> **Knowledge and memory are sources of information; context is the information currently available to the model.**
