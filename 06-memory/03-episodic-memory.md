# Episodic Memory

**Episodic Memory** is an agent's historical record of specific past experiences, events, and task executions, anchored in time and context.

While semantic memory remembers timeless facts (*"Postgres runs on port 5432"*), episodic memory remembers past actions and their outcomes (*"Yesterday at 4 PM, running the migration failed because port 5432 was already in use"*).

`Task Execution ──(Record Trajectory)──► Episodic Store ──(Retrieve Best Runs)──► In-Context Guidance`

---

## Anatomy of an Episode

A complete episode usually contains:

```text
{
  "episode_id": "ep_982",
  "timestamp": "2026-08-20T14:30:00Z",
  "task": "Fix Docker build error in auth service",
  "trajectory": [
    "1. Read Dockerfile",
    "2. Ran build -> failed (missing libssl-dev)",
    "3. Added libssl-dev -> build passed"
  ],
  "outcome": "success",
  "summary": "Resolved auth build by installing libssl-dev package."
}
```

---

## How Agents Use Episodic Memory

### 1. Dynamic Few-Shot Learning
Instead of hardcoding static few-shot examples into prompts, the agent dynamically retrieves its own **past successful trajectories** when solving a new, similar task.

### 2. Failure Prevention (Negative Examples)
The agent checks past failed episodes to avoid repeating the same mistaken tool calls or invalid arguments.

### 3. State Continuity Across Sessions
When a user asks: *"What were we working on yesterday?"*, the agent queries its episodic memory to recall the exact steps and state.

---

## Retrieval Scoring for Episodes

In production architectures (such as Stanford's *Generative Agents*), past episodes are scored using three combined factors:

$$\text{Score} = \alpha \cdot \text{Relevance} + \beta \cdot \text{Recency} + \gamma \cdot \text{Importance}$$

- **Relevance:** Semantic similarity to the current task.
- **Recency:** Time decay factor (recent episodes score higher).
- **Importance:** How critical the event was (e.g., successful deployment vs. casual greeting).

---

## Minimal Example (Dynamic Few-Shot Retrieval)

```python
# 1. User gives a new task
new_task = "Deploy payment service to staging"

# 2. Retrieve past successful episodes for this task
past_episodes = episodic_db.search(
    query=new_task,
    filter={"outcome": "success"},
    top_k=1
)

# 3. Inject past successful run as few-shot guidance
prompt = f"""
Current Task: {new_task}

Here is how you successfully solved a similar task previously:
{past_episodes[0].trajectory}

Proceed with the current task:"""
```

---

> **Episodic memory allows an agent to learn from its own history, turning past execution logs into dynamic few-shot examples.**

---

## References & Further Reading

- [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442)
  - Introduces the memory stream architecture combining recency, importance, and relevance for episodic memory retrieval.
- [Voyager: An Open-Ended Embodied Agent with Large Language Models](https://arxiv.org/abs/2305.16291)
  - NVIDIA research demonstrating how agents build an iterative library of past execution skills and episodes to master complex environments.
