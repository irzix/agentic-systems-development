# Reflection & Experience

**Reflection** is the process by which an agent actively evaluates its own past actions, reasoning, and outcomes to generate higher-level insights and improve future performance.

**Experience** is the distilled, durable lesson that emerges from reflection — a new rule, belief, or strategy the agent carries forward.

```text
Episodic Memory ──(Reflect)──► Critique & Insight ──(Distill)──► Experience / New Rule
```

---

## Reflection vs. Episodic Memory

Reflection is not simply retrieving past events. It is the **active reasoning step** where the agent looks at those events and asks:

- *What went wrong and why?*
- *What could I have done better?*
- *What pattern keeps repeating across my past actions?*

| Concept | Episodic Memory | Reflection | Experience |
|---|---|---|---|
| **What it is** | Raw event log | Active self-critique process | Distilled lesson |
| **When it happens** | During task execution | After task completion | After reflection |
| **Output** | Timeline of actions | Insight & critique | New rule or strategy |

---

## The Reflexion Pattern

**Reflexion** (introduced by Shinn et al., 2023) is a framework where an agent runs a task, stores the trajectory in episodic memory, then explicitly reflects on failures and stores a verbal critique in its long-term memory before retrying.

```text
1. Attempt task ──► Store trajectory in episodic memory.
2. Reflect: LLM critiques the trajectory → generates a "lesson learned".
3. Store the lesson in long-term memory.
4. On retry → inject the lesson into the next attempt's context.
```

This allows the agent to improve across attempts **without retraining**.

---

## Higher-Level Reflection

In Stanford's **Generative Agents**, reflection works at two levels:

1. **Single-event reflection:** After one task ("Today's code review went well because I checked type safety first").
2. **Pattern reflection:** Across many events ("Over the past week, I find that user Anette prefers concise bullet-point answers over long explanations").

Pattern-level reflection synthesizes higher-order insights and stores them as semantic memories, gradually shaping the agent's long-term behavior.

---

## Minimal Example (Reflexion Loop)

```python
# 1. Agent attempts the task
result = agent.run(task)

if not result.success:
    # 2. Reflect on the failed trajectory
    reflection_prompt = f"""
    Task: {task}
    What happened: {result.trajectory}
    Why did it fail and what should I do differently next time?
    """
    lesson = llm.generate(reflection_prompt)

    # 3. Store the lesson as a long-term memory
    memory_store.save({"type": "reflection", "lesson": lesson, "task": task})

    # 4. Retry with the lesson injected into context
    result = agent.run(task, extra_context=lesson)
```

---

> **Reflection transforms raw experience into intelligence — it is how agents grow smarter without changing their weights.**

---

## References & Further Reading

- [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366)
  - Introduces the Reflexion framework: verbal self-reflection stored in long-term memory to guide future attempts without gradient updates.
- [Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442)
  - Demonstrates multi-level reflection (individual event → pattern synthesis) in autonomous social agents.
