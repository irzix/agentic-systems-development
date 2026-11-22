# What Is the Agent Lifecycle? (Execution, Orchestration & Termination)

In the preceding chapters, we explored the core building blocks of AI systems:
- [LLMs and Inference](./01-how-llm-works.md) & [Sampling Mechanics](./02-logits-softmax-and-sampling.md)
- [Agents vs. Workflows](./03-what-is-agent.md)
- [The Environment Loop](./04-agent-environment-loop.md)
- [State](./05-what-is-state.md), [Memory](./06-what-is-memory.md), and [Knowledge](./07-what-is-knowledge.md)
- [Deterministic Guardrails](./08-deterministic-vs-probabilistic.md)

The **Agent Lifecycle** is the temporal dimension where all these components come together. It describes the complete journey from the moment a user submits a goal until the agent verifies completion, updates its memory, and halts execution.

---

## 1. The Canonical 9-Stage Lifecycle

A production agent does not simply answer a question in a single forward pass. It operates across an iterative, monitored execution pipeline:

```
                   ┌────────────────────────────────────────┐
                   │          🎯 1. USER GOAL / INPUT       │
                   └───────────────────┬────────────────────┘
                                       │
                                       ▼
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║                         🧠 AGENT CONTROLLER (State & Context)           ║
    ║                                                                         ║
    ║   ┌─────────────────────────┐             ┌─────────────────────────┐   ║
    ║   │  2. CONTEXT ASSEMBLY    │             │  3. REASONING & PLAN    │   ║
    ║   │  • State + Memory + RAG ├────────────►│  • LLM Thinking / CoT   │   ║
    ║   └─────────────────────────┘             └────────────┬────────────┘   ║
    ╚════════════════════════════════════════════════════════╪════════════════╝
                                                             │ Tool Decision
                                                             ▼
                                                ┌─────────────────────────┐
                                                │ 4. DETERMINISTIC GUARD  │
                                                │ • RBAC / Schema / Budget│
                                                └────────────┬────────────┘
                                                             │ Verified Payload
                                                             ▼
    ┌─────────────────────────────────────────────────────────────────────────┐
    │                        ⚡ 5. ENVIRONMENT & TOOLS (MCP / APIs)            │
    │                                                                         │
    │         [Database Query]       [Terminal Command]       [Web API]       │
    └────────────────────────────────────────┬────────────────────────────────┘
                                             │ Observation / Return Code
                                             ▼
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║   ┌─────────────────────────┐             ┌─────────────────────────┐   ║
    ║   │  7. STATE UPDATE        │             │  6. OBSERVATION PARSE   │   ║
    ║   │  • Reducers apply delta │◄────────────┤  • Extract tool output  │   ║
    ║   └────────────┬────────────┘             └─────────────────────────┘   ║
    ╚════════════════╪═════════════════════════════════════════════════════════╝
                     │
                     ▼
          /────────────────────\
         <   8. STOPPING CHECK? >
          \────────────────────/
             │              │
      [No: Iterate]   [Yes: Goal Met / Budget Cap]
             │              │
             │              ▼
             │   ┌─────────────────────────────────────────┐
             │   │  9. TERMINATION & MEMORY DISTILLATION   │
             │   │  • Save learned insights to database    │
             │   └──────────────────┬──────────────────────┘
             │                      │
             │                      ▼
             │              [🏁 FINAL RESPONSE]
             │
             └───► (Loops back to Step 2 with new state)
```

### Breakdown of Key Stages:

1. **Ingestion & Goal Formulation:** The agent receives unstructured user input, binds it with the agent's persona, and establishes clear success criteria.
2. **Context Assembly:** The harness gathers active state from the checkpointer, fetches relevant user preferences from long-term memory, and retrieves relevant chunks from knowledge bases (RAG).
3. **Reasoning & Deliberation:** The LLM inspects the prompt, reasons about next steps (e.g., using Chain-of-Thought), and emits either a tool invocation request or a final answer.
4. **Deterministic Guardrails:** Before executing the tool, deterministic code validates argument types (Pydantic), checks authorization (RBAC), and checks token/cost budgets.
5. **Action & Tool Execution:** The tool runs inside an isolated sandbox, executing an API call, database query, or bash script.
6. **Observation & Environment Feedback:** The environment returns output (e.g., JSON payload, exit code, or error message).
7. **Progress Evaluation:** The system evaluates if the action moved the agent closer to the goal. State is updated using reducers.
8. **Termination Check:** If stopping criteria are met, the agent exits. Otherwise, it loops back to Stage 2 with updated context.
9. **Post-Execution Reflection:** High-value learnings and experiences are distilled into long-term memory for future runs.

---

## 2. Termination Conditions: Preventing Runaway Loops

The most dangerous failure mode in autonomous agents is **Infinite Looping** (an agent repeatedly calling a failing tool, burning tokens and budget indefinitely).

Production harnesses enforce five strict **Stopping Criteria**:

```
┌─────────────────────────────────────────────────────────────┐
│                 AGENT TERMINATION CONDITIONS                │
├──────────────────────────┬──────────────────────────────────┤
│ Condition                │ Trigger Mechanism                │
├──────────────────────────┼──────────────────────────────────┤
│ **1. Goal Satisfaction** │ The agent produces a final       │
│                          │ answer or confirms success.      │
├──────────────────────────┼──────────────────────────────────┤
│ **2. Turn / Step Limit** │ Reaching maximum allowed turns   │
│                          │ (e.g., `max_steps = 10`).        │
├──────────────────────────┼──────────────────────────────────┤
│ **3. Budget / Token Cap**│ Total token spend or dollar cost │
│                          │ exceeds threshold (e.g. > $0.50).│
├──────────────────────────┼──────────────────────────────────┤
│ **4. Cycle / Stagnation**│ The agent invokes the exact same │
│                          │ tool with identical arguments    │
│                          │ without making progress.         │
├──────────────────────────┼──────────────────────────────────┤
│ **5. User Interruption** │ A human cancels the run or       │
│    (HITL Gate)**         │ rejects a high-risk tool call.   │
└──────────────────────────┴──────────────────────────────────┘
```

> [!IMPORTANT]
> Never deploy an autonomous agent loop without **hard step caps** and **stagnation detectors**. A simple `while True:` loop without budget guards is a financial vulnerability.

---

## 3. Orchestration Patterns: Controlling the Lifecycle

How an agent moves through its lifecycle depends on the **Orchestration Pattern** chosen for the task:

```
A. ReAct (Reason + Act) — Dynamic Micro-Steps:
┌─────────┐      ┌─────────┐      ┌────────┐      ┌─────────────┐
│ Prompt  ├─────►│ Thought ├─────►│ Action ├─────►│ Observation │
└─────────┘      └────▲────┘      └────────┘      └──────┬──────┘
                      │                                  │
                      └──────────── (Loop until done) ───┘

B. Plan-and-Execute — Macro Planning with Step Execution:
┌─────────┐      ┌────────────────────────┐      ┌───────────────┐
│ Prompt  ├─────►│ 1. Generate Plan       ├─────►│ 2. Execute    │
└─────────┘      │    [Step 1, 2, 3, 4]   │      │    Step-by-Step│
                 └───────────▲────────────┘      └───────┬───────┘
                             │                           │
                             └── (Replanned on Failure) ─┘

C. Evaluator-Optimizer (Self-Reflection):
┌─────────┐      ┌────────────────────────┐      ┌───────────────┐
│ Prompt  ├─────►│ Generator (Worker)     ├─────►│ Evaluator     │
└─────────┘      │ Produces draft solution│      │ Scores quality│
                 └───────────▲────────────┘      └───────┬───────┘
                             │                           │
                             └─── [Feedback / Refine] ───┴──► [Pass: Finish]
```

1. **ReAct (Reason + Act):** The standard pattern for dynamic tasks where each step depends unpredictably on the outcome of the previous tool call.
2. **Plan-and-Execute:** The agent breaks a complex problem into a structured checklist first, then executes sub-tasks sequentially, updating the plan only if a sub-task fails.
3. **Evaluator-Optimizer (Reflection):** One model call generates a solution, and a separate validation check or LLM evaluates it against strict criteria, looping until acceptable quality is achieved.
4. **Hierarchical Supervisor:** A central coordinator breaks tasks down and delegates them to specialized sub-agents.

---

## 4. Python Mental Model: A Complete Lifecycle Runner

Here is a production-style Agent Lifecycle Runner in pure Python featuring step limits, token budgeting, tool execution, and stagnation detection:

```python
import hashlib
from typing import Dict, Any, List, Set

class ProductionLifecycleRunner:
    def __init__(self, max_steps: int = 5, max_tokens: int = 10000):
        self.max_steps = max_steps
        self.max_tokens = max_tokens
        self.total_tokens_used = 0
        self.executed_action_hashes: Set[str] = set()

    def run(self, goal: str, tools: Dict[str, Any], llm_mock) -> Dict[str, Any]:
        print(f"=== Starting Agent Lifecycle for Goal: '{goal}' ===")
        
        # STAGE 1 & 2: Ingest & Context Assembly
        state: Dict[str, Any] = {"goal": goal, "step": 0, "status": "RUNNING", "history": []}

        while state["status"] == "RUNNING":
            state["step"] += 1
            print(f"\n--- Superstep {state['step']} ---")

            # TERMINATION CHECK 1: Max Steps
            if state["step"] > self.max_steps:
                state["status"] = "HALTED_MAX_STEPS"
                break

            # TERMINATION CHECK 2: Token Budget
            if self.total_tokens_used >= self.max_tokens:
                state["status"] = "HALTED_BUDGET_EXCEEDED"
                break

            # STAGE 3: Reasoning & Deliberation (Simulated LLM call)
            # In production: decision = llm.decide(state)
            self.total_tokens_used += 450  # Simulate token consumption
            
            if state["step"] == 1:
                chosen_tool = "search_orders"
                tool_args = {"customer_id": "cust_100"}
            elif state["step"] == 2:
                chosen_tool = "search_orders"
                tool_args = {"customer_id": "cust_100"}  # Deliberate duplicate to test stagnation
            else:
                state["status"] = "SUCCESS"
                state["result"] = "Order #8812 delivered successfully."
                break

            # TERMINATION CHECK 3: Cycle / Stagnation Detection
            action_signature = f"{chosen_tool}:{sorted(tool_args.items())}"
            action_hash = hashlib.md5(action_signature.encode()).hexdigest()
            
            if action_hash in self.executed_action_hashes:
                print(f"[Warning] Stagnation detected! Agent repeated action: {action_signature}")
                state["status"] = "HALTED_CYCLE_DETECTED"
                break
            self.executed_action_hashes.add(action_hash)

            # STAGE 4 & 5: Guardrails & Tool Execution
            print(f"Action: Invoking tool '{chosen_tool}' with args {tool_args}")
            # Mock tool execution
            observation = {"order_id": "8812", "status": "Delivered"}

            # STAGE 6 & 7: Observation & State Update
            state["history"].append({"step": state["step"], "action": chosen_tool, "obs": observation})
            print(f"Observation: {observation}")

        print(f"\n=== Lifecycle Complete: Status = {state['status']} | Total Tokens: {self.total_tokens_used} ===")
        return state


# Test the runner
runner = ProductionLifecycleRunner(max_steps=5, max_tokens=5000)
final_state = runner.run("Check customer order status", tools={}, llm_mock=None)
```

---

## 5. Ecosystem Implementations

<details>
<summary><b>🔍 Code Example: LangGraph Lifecycle with Conditional Routing & HITL (Python)</b></summary>

LangGraph compiles the lifecycle into a formal state graph where edges define lifecycle transitions:

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Literal

class LifecycleState(TypedDict):
    step_count: int
    task_done: bool

def reason_step(state: LifecycleState):
    print(f"Executing lifecycle turn #{state['step_count'] + 1}")
    return {"step_count": state["step_count"] + 1, "task_done": state["step_count"] >= 2}

def termination_router(state: LifecycleState) -> Literal["continue", END]:
    # Deterministic stopping condition check
    if state["task_done"] or state["step_count"] >= 5:
        return END
    return "continue"

workflow = StateGraph(LifecycleState)
workflow.add_node("agent", reason_step)
workflow.add_edge(START, "agent")
workflow.add_conditional_edges("agent", termination_router, {"continue": "agent", END: END})

app = workflow.compile()
output = app.invoke({"step_count": 0, "task_done": False})
print("Final Lifecycle State:", output)
```
</details>

<details>
<summary><b>🔍 Code Example: AWS Bedrock Agent Orchestration Tracing (boto3)</b></summary>

In Amazon Bedrock Agents, the lifecycle emits detailed telemetry events during execution:

```python
import boto3

client = boto3.client("bedrock-agent-runtime")

response = client.invoke_agent(
    agentId="MY_AGENT_ID",
    agentAliasId="PROD",
    sessionId="lifecycle_sess_1",
    inputText="Generate inventory report",
    enableTrace=True  # Enables detailed lifecycle event streaming
)

# Inspect lifecycle stages in real-time
for event in response.get("completion", []):
    trace = event.get("trace", {}).get("trace", {})
    if "orchestrationTrace" in trace:
        orch = trace["orchestrationTrace"]
        if "rationale" in orch:
            print("Reasoning Step:", orch["rationale"]["text"])
        if "invocationInput" in orch:
            print("Action Dispatched:", orch["invocationInput"]["actionGroupInvocationInput"]["actionGroupName"])
```
</details>

---

## 6. Milestone References & Foundational Research

To master agent lifecycles, study these seminal research papers:

1. **[ReAct: Synergizing Reasoning and Acting in Language Models (ICLR 2023)](https://arxiv.org/abs/2210.03629)** — *Yao et al.*  
   The landmark paper establishing the cyclical `Thought → Action → Observation` execution pattern that powers modern agents.
2. **[Reflexion: Language Agents with Verbal Reinforcement Learning (NeurIPS 2023)](https://arxiv.org/abs/2303.11366)** — *Shinn et al.*  
   Pioneered the evaluation and self-reflection stage of the agent lifecycle, showing how agents learn from past errors across trials.
3. **[Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning (ACL 2023)](https://arxiv.org/abs/2305.04091)** — *Wang et al.*  
   Formulated the Plan-and-Execute lifecycle, separating global macro-planning from step-by-step execution.
4. **[Building Effective Agents (Anthropic Research, 2024/2025)](https://www.anthropic.com/research/building-effective-agents)**  
   Industry-standard guide on orchestrating agent lifecycles using composable patterns (Evaluator-Optimizer, Orchestrator-Workers, and Routing).
5. **[A Survey on Large Language Model Based Autonomous Agents (2024)](https://arxiv.org/abs/2308.11432)** — *Wang et al.*  
   Comprehensive academic survey mapping out the 4 universal agent modules: Profiling, Memory, Planning, and Action.
