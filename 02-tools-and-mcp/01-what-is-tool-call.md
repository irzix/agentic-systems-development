# How does Tool Calling work?

**Tool Calling** allows an LLM to request an external tool or function to perform an action.

The LLM **does not execute the tool itself**. It generates a structured request, and the application executes it.

```text
User → LLM → Tool Call → Tool Execution → Result → LLM
```

For example, if the agent has:

```text
get_weather(city: string)
```

The LLM might request:

```json
{
  "name": "get_weather",
  "arguments": {
    "city": "Nicosia"
  }
}
```

The application executes the function and sends the result back to the LLM.

This loop can happen multiple times until the agent has enough information to complete the task.

## Tool Calling vs. Function Calling

You will often see both terms:

- **Function Calling** — Usually refers to calling a specific function defined by the application.
- **Tool Calling** — A broader term that can include functions, APIs, search, code execution, and other capabilities.

Modern LLM APIs generally use the term **tool calling**.

## Concept

The model doesn't magically gain access to your systems.

The application defines the available tools, executes the requested tool, and controls what happens with the result.
