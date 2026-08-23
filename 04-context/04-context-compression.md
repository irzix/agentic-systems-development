# Context Compression

**Context Compression** is the process of reducing the size of existing context while preserving the information needed by the agent.

Common techniques include:

- **Summarization**: Replace long conversation history with a concise summary.
- **Compaction**: Compress accumulated messages, tool calls, and intermediate results into a smaller representation.
- **Pruning**: Remove irrelevant or redundant information.
- **Structured Extraction**: Extract important facts, decisions, and constraints into structured state.

Compression can also be combined with retrieval. Important information can be compressed and stored, then retrieved when needed.

## Practical Examples

### 1. Sliding Window with Summary
```python
def compress_history(history: list[dict], max_recent: int = 3) -> list[dict]:
    if len(history) <= max_recent:
        return history
    
    old_messages = history[:-max_recent]
    recent_messages = history[-max_recent:]
    summary = summarize_with_llm(old_messages)
    
    return [{"role": "system", "content": f"Summary of earlier steps: {summary}"}] + recent_messages
```

### 2. Tool Result Compaction
```python
def compact_tool_output(raw_output: str, max_lines: int = 10) -> str:
    lines = raw_output.splitlines()
    if len(lines) <= max_lines:
        return raw_output
    # Retain the top and bottom of the output
    return "\n".join(lines[:5]) + f"\n... [{len(lines) - 10} lines truncated] ...\n" + "\n".join(lines[-5:])
```

> **The goal is to reduce context size without losing information that is important for the agent's future decisions.**
