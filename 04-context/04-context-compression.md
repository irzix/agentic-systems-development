# Context Compression

**Context Compression** is the process of reducing the size of existing context while preserving the information needed by the agent.

Common techniques include:

- **Summarization**: Replace long conversation history with a concise summary.
- **Compaction**: Compress accumulated messages, tool calls, and intermediate results into a smaller representation.
- **Pruning**: Remove irrelevant or redundant information.
- **Structured Extraction**: Extract important facts, decisions, and constraints into structured state.

Compression can also be combined with retrieval. Important information can be compressed and stored, then retrieved when needed.

> **The goal is to reduce context size without losing information that is important for the agent's future decisions.**
