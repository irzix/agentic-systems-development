# What is Context Engineering?

**Context Engineering** is the process of designing, selecting, and managing the information provided to an LLM during an agent's execution.

The goal is not to provide as much information as possible, but to provide the **minimum sufficient context** needed to reach the correct result efficiently.

Context can include:

- User input and conversation history
- Relevant memory
- Retrieved knowledge
- Tool results
- Current state
- System instructions

Good context engineering aims to:

- Reduce irrelevant information
- Reduce token usage and latency
- Improve the signal-to-noise ratio
- Give the model the information it needs at the right time

> **Context Engineering is about giving the model the right information, at the right time, in the right amount.**
