# Common Agent Architectures

There are several common ways to structure agentic systems. The right architecture depends on the task, level of autonomy, and required coordination.

### Single Agent

One agent handles the task using its own tools, context, and memory.

```text
User → Agent → Tools
```

### Tool / Plugin-Based

A single agent is extended with external capabilities through tools or plugins.

```text
        Agent
      ↙   ↓   ↘
   Search  DB  GitHub
```

### Router

A router decides which agent or workflow should handle a request.

```text
             ┌→ Agent A
User → Router ├→ Agent B
             └→ Agent C
```

### Supervisor

A central agent coordinates multiple specialized agents.

```text
           Supervisor
          ↙    ↓    ↘
      Agent A Agent B Agent C
```

### Swarm / Peer-to-Peer

Multiple agents operate as peers and can hand off tasks to each other without a central supervisor.

```text
Agent A ↔ Agent B
   ↕          ↕
Agent C ↔ Agent D
```

### Pipeline / Sequential

Agents or steps execute in a predefined sequence, with each step passing its result to the next.

```text
Agent A → Agent B → Agent C
```

### Parallel

Independent agents work on different parts of a task at the same time, and their results are later combined.

```text
             ┌→ Agent A ─┐
User →       ├→ Agent B ─┼→ Aggregator
             └→ Agent C ─┘
```

### Hierarchical

Agents are organized into multiple levels, where higher-level agents coordinate lower-level agents.

```text
          Manager
         ↙       ↘
    Manager A   Manager B
     ↙   ↘       ↙   ↘
   Agent A Agent B Agent C Agent D
```

These architectures can also be combined. For example, a Supervisor can use a Router and delegate work to agents that run tools in parallel.

> **Choose the simplest architecture that provides the coordination and specialization your task actually needs.**
