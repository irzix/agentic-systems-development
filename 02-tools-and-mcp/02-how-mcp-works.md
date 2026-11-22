# What is MCP and How Does It Work?

**MCP (Model Context Protocol)** is an open protocol for connecting AI applications to external **tools, resources, and prompts** in a standardized way.

Instead of building a custom integration for every AI application, an MCP server can expose its capabilities through a common protocol.

A simplified flow:

```text
AI Application
      ↓
   MCP Client
      ↓
   MCP Server
      ↓
Tool / API / Database / System
```

For example, an MCP server could expose tools such as:

```text
search_documents
create_ticket
get_customer
```

The client can discover available tools and their schemas, then request a tool call when needed.

```text
Discover Tools
      ↓
Choose Tool
      ↓
Call Tool
      ↓
Receive Result
```

The important distinction is:

> **Tool Calling defines how an LLM requests an action; MCP defines a standardized way for applications to discover and interact with external capabilities.**
