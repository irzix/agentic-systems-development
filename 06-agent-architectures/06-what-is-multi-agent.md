# Single-Agent vs. Multi-Agent

A **single-agent system** uses one autonomous agent to handle a task.

A **multi-agent system** consists of multiple relatively independent agents that can operate with their own state, memory, context, and goals, while communicating or coordinating with each other.

A **subagent** is different: it is usually a component of a parent agent and is delegated a specific task.

| Single-Agent | Multi-Agent |
|---|---|
| One autonomous agent | Multiple autonomous agents |
| Shared execution state | Each agent can have its own state |
| One reasoning process | Multiple reasoning processes |
| Simpler coordination | Requires agent-to-agent coordination |
| Lower overhead | More communication, latency, and token cost |

The key distinction is **independence**.

> **If one agent simply delegates a task to another as part of its own execution flow, that is usually a subagent pattern. Multi-agent systems involve multiple relatively independent agents that coordinate with each other.**
