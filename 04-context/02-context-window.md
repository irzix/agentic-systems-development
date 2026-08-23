# Context Window

The **Context Window** is the maximum amount of information a model can process in a single request, measured in tokens.

It can contain:

- System instructions
- User messages
- Conversation history
- Retrieved information
- Tool calls and results
- The model's generated output

For example:

```text
Context Window
├── Instructions
├── Conversation
├── Retrieved Knowledge
├── Tool Results
└── Output
```

A larger context window does not mean we should fill it with more information. In long-running agents, context can continuously grow across interactions, tool calls, and execution steps.

As the context grows, **history summarization, compaction, and other context management techniques** can be used to keep the context within a useful size.

> **Keep the context concise from the beginning, because long-running agents can gradually fill the context window and require techniques such as history summarization and compaction.**
