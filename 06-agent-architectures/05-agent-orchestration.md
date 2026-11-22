# Agent Orchestration

**Agent Orchestration** is the logic that controls how an agent or multiple agents coordinate their work to achieve a goal.

It determines things like:

- What should happen next?
- Which tool or agent should handle the task?
- When should the agent continue, retry, delegate, or stop?
- How should results flow between different agents or steps?

For example:

```text
Goal
 ↓
Orchestrator
 ├── Research Agent
 ├── Coding Agent
 └── Review Agent
```

Orchestration can be implemented with simple control flow, an agent loop, workflows, or more complex multi-agent patterns.

> **Agent orchestration is about controlling and coordinating the execution of agents toward a goal.**
