# What Is State? (Modeling, Reducers & Checkpointing)

In traditional software, state is simple: it is the variables and database records that track what your application is doing. 

In Agentic Systems, **State is the foundation of autonomy**. Because LLMs are inherently stateless, the surrounding system must maintain an explicit, evolving snapshot of the agent's world: where it is, what it has discovered, and what it needs to do next.

---

## 1. The Core Definition of Agent State

**State is the dynamic data contract representing the agent and its execution environment at a specific step in time.**

Mathematically, an agent is a state machine governed by a transition function:

$$S_{t+1} = \text{Transition}(S_t, \text{Action}_t, \text{Observation}_t)$$

```
┌────────────────────────────────────────────────────────┐
│                      AGENT STATE                       │
│  {                                                     │
│    "task_id": "flight_search_01",                      │
│    "current_step": "filter_prices",                    │
│    "search_results": [...],                            │
│    "selected_option": null,                            │
│    "retry_count": 0,                                   │
│    "is_authorized": false                              │
│  }                                                     │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
               LLM Decision / Tool Call
                           │
                           ▼
                  State Reducer / Update
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│                   NEW STATE (t + 1)                    │
│  {                                                     │
│    "current_step": "awaiting_user_approval",           │
│    "selected_option": {"flight_id": "CY204"},          │
│    ...                                                 │
│  }                                                     │
└────────────────────────────────────────────────────────┘
```

---

## 2. The Critical Distinctions: State vs. History vs. Context vs. Memory

Developers often confuse these four terms. In production architectures, they serve distinct purposes:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         INFORMATION HIERARCHY                            │
├─────────────┬─────────────────────────────────┬──────────────────────────┤
│ Concept     │ What It Represents              │ Lifespan                 │
├─────────────┼─────────────────────────────────┼──────────────────────────┤
│ **State**   │ Where the agent is *right now*  │ Current execution run    │
│ **History** │ Complete log of *what happened* │ Session / Audit Trail    │
│ **Context** │ What is fed into the *LLM prompt*│ Single inference call    │
│ **Memory**  │ What survives *across sessions* │ Long-term persistence    │
└─────────────┴─────────────────────────────────┴──────────────────────────┘
```

- **State:** Structured variables needed to decide the immediate next step (`retry_count = 2`, `status = "RUNNING"`).
- **History:** Chronological ledger of every message, tool call, and raw response.
- **Context:** The curated subset of State, History, and Knowledge passed into the model's active token window.
- **Memory:** Cross-session persistent knowledge (e.g., *"The user prefers window seats"*), covered in [Chapter 6](./06-what-is-memory.md).

![State in Agentic Systems](../assets/agentic-state-flow.jpeg)

---

## 3. Modeling State in Python: TypedDict vs. Pydantic

In early prototypes, developers often use raw dictionaries (`state = {}`). In production, this leads to runtime `KeyError` exceptions and untrackable schema drift.

Modern agent frameworks model state using **TypedDict** (fast, native dictionary compatibility) or **Pydantic** (strict runtime validation):

```python
from typing import Annotated, List, Optional
from typing_extensions import TypedDict
from pydantic import BaseModel, Field

# Option A: TypedDict (Standard for LangGraph)
class AgentStateDict(TypedDict):
    task: str
    messages: List[dict]
    tool_results: List[dict]
    step_count: int
    is_completed: bool


# Option B: Pydantic Model (Strict runtime validation & type casting)
class ProductionAgentState(BaseModel):
    task_id: str
    query: str
    current_step: str = "init"
    retry_count: int = Field(default=0, ge=0, le=5)
    selected_flight: Optional[str] = None
    is_completed: bool = False
```

---

## 4. State Reducers: Managing Concurrent & Incremental Updates

When an agent step finishes, how does its output update the existing State?

### Default Behavior: Overwrite (Replacement)
If field `current_step` was `"search"` and a node returns `{"current_step": "filter"}`, the old string is overwritten.

### Reducer Behavior: Accumulation & Merging
What happens when you have a list of messages or when multiple tool nodes execute in parallel? If they both overwrite the state, data is lost!

A **Reducer** is a function that dictates how new updates are merged into the existing state:

```python
from typing import Annotated, List
from typing_extensions import TypedDict

# A custom reducer that appends new items and removes duplicates
def append_unique_logs(existing: List[str], incoming: List[str]) -> List[str]:
    return list(dict.fromkeys(existing + incoming))

class ReducerState(TypedDict):
    # Overwrites on update:
    current_stage: str

    # Reducer: new logs are merged via our custom function
    execution_logs: Annotated[List[str], append_unique_logs]
```

> [!NOTE]
> In LangGraph, the most widely used reducer is `add_messages`, which appends new messages to the conversation list while updating existing messages that share the same unique ID.

---

## 5. Checkpointing & Persistence (The Production Lifeline)

In production, an agent cannot simply run as an ephemeral in-memory Python script. If your container restarts, your API times out, or the task requires human approval, all state is lost.

**Checkpointing** is the mechanism of saving a serialized snapshot of the state at the end of every step.

```
Superstep 1: User Prompt ──► [Checkpoint #1 Saved to SQLite / Postgres]
Superstep 2: Tool Call   ──► [Checkpoint #2 Saved to SQLite / Postgres]
Superstep 3: HITL Pause  ──► [Server can shut down safely]
                             [Hours later: Human approves via Webhook]
Superstep 4: Resume      ──► [Load Checkpoint #3 and continue!]
```

### What Checkpointing Unlocks:
1. **Crash Recovery & Fault Tolerance:** If a worker crashes on step 4, the orchestrator reloads Checkpoint #3 and resumes without re-running earlier expensive LLM calls.
2. **Human-in-the-Loop (HITL):** You can set an interrupt gate before dangerous tools (e.g., `interrupt_before=["execute_wire_transfer"]`). The agent safely freezes state until a human reviews and approves.
3. **Time-Travel Debugging:** Developers can rewind an agent execution to Checkpoint #2, edit the prompt or tool output, and observe how the downstream trajectory changes.
4. **Thread Isolation (`thread_id`):** Each user or conversation has a dedicated thread ID, isolating their state completely from others.

---

## 6. Ecosystem Implementations

<details>
<summary><b>🔍 Code Example: Stateful LangGraph Agent with SQLite Persistence (Python)</b></summary>

```python
from typing import Annotated, TypedDict, Literal
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langgraph.checkpoint.sqlite import SqliteSaver

# 1. Define State Schema with Reducers
class OrderState(TypedDict):
    messages: Annotated[list, add_messages]  # Appends messages
    order_id: str
    shipping_status: str

# 2. Define State Machine Nodes
def check_shipping(state: OrderState):
    # Access state fields safely
    order_id = state["order_id"]
    # Return partial update to state
    return {"shipping_status": f"Dispatched via Express (ID: {order_id})"}

# 3. Assemble Graph with SQLite Checkpointer
builder = StateGraph(OrderState)
builder.add_node("check_shipping", check_shipping)
builder.add_edge(START, "check_shipping")
builder.add_edge("check_shipping", END)

# In production, use PostgresSaver for distributed environments
with SqliteSaver.from_conn_string(":memory:") as memory:
    app = builder.compile(checkpointer=memory)

    # 4. Execute with Thread ID (State persists under this thread!)
    config = {"configurable": {"thread_id": "session_user_789"}}
    initial_input = {"order_id": "ORD-9912", "messages": [("user", "Where is my package?")]}
    
    result = app.invoke(initial_input, config=config)
    print("Final State:", result["shipping_status"])
```
</details>

<details>
<summary><b>🔍 Code Example: AWS Bedrock Agent Session State (Python / boto3)</b></summary>

In Amazon Bedrock Agents, state is maintained via the `sessionState` parameter in the `InvokeAgent` API:

```python
import boto3

client = boto3.client("bedrock-agent-runtime", region_name="us-east-1")

# Bedrock maintains state across turns using sessionId and sessionAttributes
response = client.invoke_agent(
    agentId="YOUR_BEDROCK_AGENT_ID",
    agentAliasId="PRODUCTION_ALIAS",
    sessionId="user_session_456",  # Identifies the persistent state thread
    sessionState={
        # sessionAttributes persist for the lifetime of this sessionId
        "sessionAttributes": {
            "user_tier": "Enterprise",
            "account_id": "ACC-5541"
        },
        # promptSessionAttributes apply strictly to the current turn
        "promptSessionAttributes": {
            "current_location": "Frankfurt"
        }
    },
    inputText="What is my enterprise SLA limit?"
)
```
</details>

<details>
<summary><b>🔍 Code Example: Azure AI Agent Service Thread State (Python)</b></summary>

Azure AI Agent Service manages state implicitly through server-side **Threads**:

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

client = AIProjectClient.from_connection_string(
    credential=DefaultAzureCredential(),
    conn_str="<your-connection-string>"
)

# 1. State is persisted in a server-side Thread
thread = client.agents.create_thread()

# 2. Add message to thread state
client.agents.create_message(
    thread_id=thread.id,
    role="user",
    content="Start invoice audit for Q3"
)

# 3. Run agent on this state
run = client.agents.create_and_process_run(
    thread_id=thread.id,
    agent_id="audit_agent_id"
)
```
</details>

---

## 7. Engineering Best Practices for Agent State

1. **Keep State Lean:** Do not store megabytes of raw PDF text or large binary blobs directly in the state. Store identifiers, URLs, or database keys instead.
2. **Explicit Error Fields:** Always include `retry_count: int` and `last_error: Optional[str]` in your state schema so the agent can inspect and recover from tool errors.
3. **Pure Node Functions:** Ensure nodes return updates rather than mutating global objects in-place.
4. **Use Checkpoints Before Destructive Actions:** Always configure an interrupt or checkpoint before calling tools that delete data, charge payments, or send emails.

---

## 8. References & Further Reading

1. **[State and Memory for Robust and Reliable AI Agents](https://arxiv.org/abs/2507.00081)** — Research paper exploring how explicit state machines drastically improve reliability in long-running agent workflows.
2. **[LangGraph State Management & Checkpointing Guide](https://langchain-ai.github.io/langgraph/concepts/persistence/)** — The official technical documentation on threads, checkpoints, and time travel.
3. **[Amazon Bedrock SessionState Runtime Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/agents-session-state.html)** — Comprehensive guide on sessionAttributes, persistence, and state passing in AWS Bedrock.
