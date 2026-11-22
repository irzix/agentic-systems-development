# Agent Context

**Agent Context** is the information made available to the model at a specific point during an agent's execution.

It can include:

- User input
- Conversation history
- Relevant memory
- Retrieved knowledge
- Tool results
- Current state
- System instructions

Context is not necessarily the same as state or memory.

> **State describes what the agent is doing; context describes what the model can see.**

For example, an agent may have thousands of records in its memory, but only retrieve a few relevant ones into the current context.

> **Good context is the smallest amount of relevant information that enables the agent to make the right decision.**
