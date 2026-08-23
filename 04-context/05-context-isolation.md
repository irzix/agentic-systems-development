# Context Isolation

**Context Isolation** means keeping different agents, tasks, or execution steps from unnecessarily sharing the same context.

This can prevent irrelevant information from leaking between tasks and helps each agent focus on the information it actually needs.

For example, a research agent does not necessarily need access to the full context of a coding agent.

Isolation can help with:

- Reducing unnecessary token usage
- Improving focus and relevance
- Preventing context pollution
- Limiting sensitive information exposure
- Giving specialized agents their own working context

## Subagent Context Forking Example

Instead of passing the entire main conversation to a subagent, spawn an isolated sub-context and only merge back the concise result:

```python
# 1. Main agent spawns an isolated context for a specialist
subagent_context = {
    "task": "Extract price list from vendor documentation",
    "relevant_doc": doc_snippet,  # Only pass necessary data
    "history": []                 # No parent conversation history
}

result = run_specialist_agent(subagent_context)

# 2. Main agent receives only the final distilled summary
main_context["history"].append({
    "role": "tool",
    "content": f"Subtask result: {result.extracted_prices}"
})
```

> **Context isolation means giving each agent or task only the context it actually needs.**
