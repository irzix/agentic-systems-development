# Agentic Systems Development Course

In this repository, I'm going to explain the fundamentals of agent development using general-purpose programming languages and real-world examples based on my own projects.
My goal is to create easy-to-follow learning content and videos based on practical examples, covering a complete series on agent development. The goal is to help programmers understand how to build agentic systems and make the right engineering trade-offs.
Agentic development is still a relatively new field, but many useful articles and resources have already been published. Throughout this repository, I'll try to reference relevant resources and research as I build and explain each topic.

## Reference Books

[Selected books](./resources/books/README.md) on LLM engineering, planning and multi-agent systems, with suggested chapters and official links.

## 1. Foundations

- [How do LLMs work? (From ML to Reasoning)](./01-foundation/01-how-llm-works.md)
- [Logits, Softmax & Decoding Strategies](./01-foundation/02-logits-softmax-and-sampling.md)
- [What is an Agent?](./01-foundation/03-what-is-agent.md)
- [The Agent-Environment Loop](./01-foundation/04-agent-environment-loop.md)
- [What is State?](./01-foundation/05-what-is-state.md)
- [What is Memory?](./01-foundation/06-what-is-memory.md)
- [What is Knowledge?](./01-foundation/07-what-is-knowledge.md)
- [Deterministic vs. Probabilistic Systems](./01-foundation/08-deterministic-vs-probabilistic.md)
- [What is the Agent Lifecycle?](./01-foundation/09-what-is-agent-lifecycle.md)

## 2. Tools & MCP (Model Context Protocol)

- [How Tool Calling Works (Function Schemas & JSON Output)](./02-tools-and-mcp/01-what-is-tool-call.md)
- [What is MCP and How Does It Work?](./02-tools-and-mcp/02-how-mcp-works.md)
- [Native Tool Calling vs. MCP Architecture](./02-tools-and-mcp/03-tool-vs-mcp.md)
- [Tool Design, Schemas & Idempotency](./02-tools-and-mcp/04-tool-design.md)
- [MCP Discovery & Dynamic Registration](./02-tools-and-mcp/05-mcp-discovery.md)
- [Sandboxed Code Execution (Docker, E2B & Wasm)](./02-tools-and-mcp/06-sandboxed-execution.md)

## 3. Context Engineering

- [What is Context Engineering?](./03-context-engineering/01-what-is-context-engineering.md)
- [Context Window & The "Lost in the Middle" Effect](./03-context-engineering/02-context-window.md)
- [Prompt Caching Mechanics (Anthropic & OpenAI)](./03-context-engineering/03-prompt-caching.md)
- [Context Selection & Filtering](./03-context-engineering/04-context-selection.md)
- [Context Compression & Compaction Strategies](./03-context-engineering/05-context-compression.md)
- [Context Isolation & Security](./03-context-engineering/06-context-isolation.md)

## 4. Knowledge & Agentic RAG

- [What is RAG? (Parametric vs. Non-Parametric)](./04-knowledge-and-rag/01-what-is-rag.md)
- [Chunking Strategies (Fixed vs. Semantic Chunking)](./04-knowledge-and-rag/02-chunking-strategies.md)
- [Query Transformations (Multi-Query, HyDE & Step-Back)](./04-knowledge-and-rag/03-query-transformations.md)
- [Retrieval & Hybrid Search (BM25 + Dense Vectors)](./04-knowledge-and-rag/04-retrieval-and-search.md)
- [Reranking & Context Compression (Cross-Encoders)](./04-knowledge-and-rag/05-reranking-and-compression.md)
- [Agentic RAG & GraphRAG (Dynamic Routing & Knowledge Graphs)](./04-knowledge-and-rag/06-agentic-rag.md)
- [RAG Trade-offs & Production Optimization](./04-knowledge-and-rag/07-rag-tradeoffs.md)

## 5. Memory Systems & Experience

- [Short-term vs. Long-term Memory](./05-memory-systems/01-short-term-vs-long-term-memory.md)
- [Semantic Memory (User Facts & Preferences)](./05-memory-systems/02-semantic-memory.md)
- [Episodic Memory (Task Histories & Past Trajectories)](./05-memory-systems/03-episodic-memory.md)
- [Procedural Memory (Rules, Tools & Instructions)](./05-memory-systems/04-procedural-memory.md)
- [Reflection & Experience (The Reflexion Framework)](./05-memory-systems/05-reflection-and-experience.md)
- [Modern Memory Engines (Mem0, LangGraph Store & Zep)](./05-memory-systems/06-modern-memory-engines.md)

## 6. Agent Architectures & Multi-Agent Systems

- [What is Agent Architecture?](./06-agent-architectures/01-what-is-agent-architecture.md)
- [ReAct Pattern Deep Dive](./06-agent-architectures/02-react-pattern.md)
- [Plan-and-Execute Architecture](./06-agent-architectures/03-plan-and-execute.md)
- [Evaluator-Optimizer & Self-Correction Loops](./06-agent-architectures/04-evaluator-optimizer.md)
- [Agent Orchestration (StateGraphs & Routers)](./06-agent-architectures/05-agent-orchestration.md)
- [Multi-Agent Supervisors & Hierarchical Delegation](./06-agent-architectures/06-multi-agent-supervisors.md)
- [Multi-Agent Swarms & Peer-to-Peer Collaboration](./06-agent-architectures/07-multi-agent-swarms.md)

## 7. Reliability, Security & Guardrails

- [Agent Failure Modes & Resilient Error Recovery](./07-reliability-and-guardrails/01-failure-modes-and-recovery.md)
- [Transactional Safety, Database Locks & Idempotency](./07-reliability-and-guardrails/02-transactional-safety-idempotency.md)
- [Human-in-the-Loop (HITL) Interruption Patterns](./07-reliability-and-guardrails/03-human-in-the-loop.md)
- [Prompt Injection & Untrusted Tool Outputs](./07-reliability-and-guardrails/04-security-and-prompt-injection.md)
- [Production Guardrail Frameworks (NeMo & Guardrails AI)](./07-reliability-and-guardrails/05-guardrail-frameworks.md)

## 8. Evaluation & Operations (AgentOps)

- [Agent Observability & Tracing (Langfuse & OpenTelemetry)](./08-evaluation-and-ops/01-observability-and-tracing.md)
- [Step-Level vs. Trajectory Evaluations](./08-evaluation-and-ops/02-trajectory-evaluations.md)
- [LLM-as-a-Judge & Automated Benchmark Datasets](./08-evaluation-and-ops/03-llm-as-a-judge-benchmarks.md)
- [Cost & Latency Optimization (Cascading, Routing & Caching)](./08-evaluation-and-ops/04-cost-latency-optimization.md)
- [Production Readiness Checklist & Deployment](./08-evaluation-and-ops/05-production-readiness.md)
