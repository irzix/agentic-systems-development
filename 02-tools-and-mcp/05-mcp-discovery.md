## MCP Discovery

MCP servers can expose many capabilities that clients can discover and use dynamically.

For larger systems, capabilities can be organized hierarchically instead of exposing everything through one large server.

For example:

```text
Company MCP
├── Customer
│   ├── Search
│   └── Profile
├── Compliance
│   ├── KYC
│   └── Risk
└── Support
    ├── Tickets
    └── Conversations
```

This can make large tool ecosystems easier to discover, govern, and manage.

For very large tool ecosystems, a **search or discovery layer** can also help the agent find only the relevant tools instead of exposing thousands of tools in every context.

> **MCP discovery becomes increasingly important as the number of available tools grows.**
