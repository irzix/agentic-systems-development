# Agent Memory

**Agent Memory** allows an agent to retain information beyond its current execution and use it in future interactions.

For example, an agent might remember:

- User preferences
- Previous decisions
- Past interactions
- Learned facts
- Previous experiences or outcomes

A useful distinction is:

> **State represents what is happening now; memory represents what can be retained for later.**

Memory can be stored in different forms, such as:

- Conversation history
- Databases
- Vector stores
- Structured user profiles
- Experience or event logs

The important part is that memory does not mean putting everything into the context. Relevant memories should be **retrieved when needed** and added to the current context.

> **Memory stores information; retrieval decides what becomes context.**
