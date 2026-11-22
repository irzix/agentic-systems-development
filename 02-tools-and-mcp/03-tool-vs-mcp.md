# Tool Calling vs. MCP

**Tool Calling** is an LLM capability for requesting a tool or function, while **MCP** is a protocol for standardizing how AI applications connect to external capabilities.

| Tool Calling | MCP |
|---|---|
| LLM capability | Communication protocol |
| Requests a tool execution | Standardizes tool/resource integration |
| Usually defined by the application | Exposed through an MCP server |
| Focuses on invocation | Focuses on interoperability |

For example:

```text
LLM → Tool Call → Application → Tool
```

With MCP:

```text
LLM → Tool Call → MCP Client → MCP Server → Tool
```

> **Tool Calling is about invoking tools; MCP is about standardizing how those tools are exposed and connected.**
