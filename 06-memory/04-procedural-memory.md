# Procedural Memory

**Procedural Memory** is an agent's stored knowledge of *how to do things* — step-by-step workflows, verified procedures, reusable skills, and learned best practices.

While episodic memory records *what happened*, procedural memory captures *how to do it correctly*, turning successful one-time actions into repeatable skills.

`Successful Execution ──(Distill)──► Procedure / Skill ──(Retrieve)──► Apply to New Task`

---

## What Procedural Memory Stores

- **Reusable Workflows:** Step-by-step guides the agent can follow (e.g., "How to deploy a Docker container to AWS ECS").
- **Tool Recipes:** Sequences of tool calls that reliably achieve a result (e.g., "Run tests → Lint → Build → Merge").
- **Coding Conventions:** Team-specific rules (e.g., "Use `snake_case` for Python functions, `camelCase` for TypeScript").
- **Agent Instructions:** Operational guidelines (e.g., "Always confirm destructive actions with the user before executing").

---

## Episodic vs. Procedural Memory

| Dimension | Episodic Memory | Procedural Memory |
|---|---|---|
| **Records** | What happened (timestamped events) | How to do it (reusable steps) |
| **Format** | Event log / conversation trajectory | Skill definition / workflow template |
| **Lifespan** | Tied to specific past events | General and reusable across tasks |
| **Source** | Automatically recorded during execution | Distilled from repeated successful episodes |

---

## Procedural Memory in Practice (The Voyager Pattern)

In NVIDIA's **Voyager** agent, procedural memory is implemented as a **Skill Library** — a continuously growing collection of verified, runnable code programs that the agent reuses and builds on:

```text
1. Agent attempts a new task.
2. On success → distill the solution into a verified, reusable skill.
3. Store the skill in the Skill Library (procedural memory).
4. On future similar tasks → retrieve and re-execute the stored skill.
```

This allows the agent to become progressively more capable without repeating discovery steps.

---

## Minimal Example (Skill Library)

```python
# Storing a successful procedure as a reusable skill
new_skill = {
    "name": "deploy_docker_service",
    "description": "Deploys a Docker service to AWS ECS using a task definition.",
    "steps": [
        "1. Build Docker image: docker build -t {service_name} .",
        "2. Push to ECR: docker push {ecr_repo}/{service_name}:latest",
        "3. Update ECS task definition and deploy via AWS CLI.",
    ],
    "verified": True
}

skill_library.save(new_skill)

# Retrieving the right skill for a new task
task = "Deploy billing service to production"
matching_skill = skill_library.search(task, top_k=1)
```

---

> **Procedural memory turns repeated discoveries into standing skills, enabling agents to grow more capable with every successful task.**

---

## References & Further Reading

- [Voyager: An Open-Ended Embodied Agent with Large Language Models](https://arxiv.org/abs/2305.16291)
  - NVIDIA research demonstrating a self-improving agent with an automatic skill library (procedural memory store).
- [Cognitive Architectures for Language Agents (CoALA)](https://arxiv.org/abs/2309.02427)
  - Formal classification of procedural memory as action policies and learned skills in LLM-based agents.
