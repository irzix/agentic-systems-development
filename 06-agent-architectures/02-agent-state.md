# Agent State

**Agent State** is the information that represents the agent's **current execution state**.

It can include things like:

- Current task or goal
- Conversation history
- Tool calls and results
- Current step or status
- Intermediate results
- Variables needed for the next action

For example:

```json
{
  "task": "Find the customer's latest order",
  "current_step": "search_orders",
  "customer_id": "123",
  "tool_result": null
}
```

The state changes as the agent executes:

```text
State → Reason → Action → Observation → Updated State
```

**State is usually temporary and execution-specific.** It represents where the agent is *right now*, while memory is used to retain information beyond the current execution.

> **State answers: "What is happening in this execution right now?"**
