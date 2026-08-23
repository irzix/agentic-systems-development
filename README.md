# Agentic Systems Development Course

In this repository, I'm going to explain the fundamentals of agent development using general-purpose programming languages and real-world examples based on my own projects.
My goal is to create easy-to-follow learning content and videos based on practical examples, covering a complete series on agent development. The goal is to help programmers understand how to build agentic systems and make the right engineering trade-offs.
Agentic development is still a relatively new field, but many useful articles and resources have already been published. Throughout this repository, I'll try to reference relevant resources and research as I build and explain each topic.

## 1. Foundations

- [How do LLMs work?](./01-foundation/01-how-llm-works.md)
- [What is an Agent?](./01-foundation/02-what-is-agent.md)
- [What is State?](./01-foundation/03-what-is-state.md)
- [What is Memory?](./01-foundation/04-what-is-memory.md)
- [What is Knowledge?](./01-foundation/05-what-is-knowledge.md)
- [What is a Chunk?](./01-foundation/06-what-is-chunk.md)
- [What is the Agent Lifecycle?](./01-foundation/07-what-is-agent-lifecycle.md)

## 2. Tools & Actions

- [How does Tool Calling work?](./02-tools/01-what-is-tool-call.md)
- [What is MCP and how does it work?](./02-tools/02-how-mcp-works.md)
- [Tool Calling vs. MCP](./02-tools/03-tool-vs-mcp.md)
- [Tool Design & Schemas](./02-tools/04-tool-design.md)
- [MCP Discovery](./02-tools/05-mcp-discovery.md)


## 3. Agent Architecture

- [What is Agent Architecture?](./03-agent-architecture/01-what-is-agent-architecture.md)
- [Agent State](./03-agent-architecture/02-agent-state.md)
- [Agent Context](./03-agent-architecture/03-agent-context.md)
- [Agent Memory](./03-agent-architecture/04-agent-memory.md)
- [Agent Orchestration](./03-agent-architecture/05-agent-orchestration.md)
- [Single-Agent vs. Multi-Agent](./03-agent-architecture/06-what-is-multi-agent.md)
- [Common Agent Architectures](./03-agent-architecture/07-common-architectures.md)

## 4. Context Engineering

- [What is Context Engineering?](./04-context/01-what-is-context-engineering.md)
- [Context Window](./04-context/02-context-window.md)
- [Context Selection](./04-context/03-context-selection.md)
- [Context Compression](./04-context/04-context-compression.md)
- [Context Isolation](./04-context/05-context-isolation.md)
- [Context vs. Memory vs. Knowledge](./04-context/06-context-memory-knowledge.md)

## 5. Knowledge & RAG

- [What is RAG and How It Works?](./05-rag/01-what-is-rag.md)
- [Chunking Strategies](./05-rag/02-chunking-strategies.md)
- [Query Transformations](./05-rag/03-query-transformations.md)
- [Retrieval & Hybrid Search](./05-rag/04-retrieval-and-search.md)
- [Reranking & Compression](./05-rag/05-reranking-and-compression.md)
- [Agentic RAG](./05-rag/06-agentic-rag.md)
- [RAG Trade-offs & Optimization](./05-rag/07-rag-tradeoffs.md)

## 6. Memory & Experience

- Short-term Memory
- Long-term Memory
- Semantic Memory
- Episodic Memory
- Procedural Memory
- Experience
- Reflection
- Human Feedback

## 7. Reliability

- Agent Failure Modes
- Retry
- Idempotency
- Timeout
- Caching
- Agent Failure Recovery
- Deterministic vs. Probabilistic Behavior
- Human-in-the-Loop (HITL)

## 8. Multi-Agent Systems

- Why Multiple Agents?
- Agent Delegation
- Agent Communication
- Supervisor Pattern
- Specialist Agents
- Parallel Agents
- Multi-Agent Workflows
- Trade-offs of Multi-Agent Systems

## 9. Evaluation

- Why is Agent Evaluation Different?
- Step-level Evaluation
- Final Evaluation
- Tool-call Evaluation
- Trajectory Evaluation
- LLM-as-a-Judge
- Evaluation Datasets
- Human Evaluation
- Regression Testing

## 10. Observability

- Agent Observability
- Tracing
- Logging
- Metrics
- Token Monitoring
- Cost Monitoring
- Agent Performance Monitoring

## 11. Security & Guardrails

- Prompt Injection
- Tool Permissions
- Agent Authorization
- Data Leakage
- Untrusted Tool Output
- Guardrails
- Input Validation
- Output Validation

## 12. Governance

- Agent Governance
- Permissions
- Policies
- Auditability
- Human Oversight
- Compliance
- Safe Agent Execution

## 13. Performance & Optimization

- Token Optimization
- Context Optimization
- Model Selection
- Caching Strategies
- Parallel Tool Calling
- Concurrency
- Latency Optimization
- Cost vs. Quality Trade-offs

## 14. Production Agent Systems

- Designing Production-ready Agents
- Agent Architecture Trade-offs
- Failure Recovery
- Scalability
- Reliability
- Observability
- Security
- Evaluation
- Cost Optimization
- Continuous Improvement
