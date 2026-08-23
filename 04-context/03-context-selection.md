# Context Selection

**Context Selection** is the process of deciding which information should be included in the context for a specific model call.

An agent may have access to a large amount of information, but only a small portion may be relevant to the current step.

Common selection strategies include:

- **Relevance**: Select information related to the current task.
- **Recency**: Prefer recent information when it matters.
- **Importance**: Keep information that is important for the task.
- **Semantic Retrieval**: Retrieve relevant information from memory or knowledge stores.

For example:

```text
Available Information
        ↓
   Selection / Retrieval
        ↓
Relevant Context
        ↓
       LLM
```

Context selection can combine multiple signals rather than relying only on semantic similarity.

> **The goal is not to retrieve everything relevant, but to retrieve what is relevant enough to help the model make the right decision.**
