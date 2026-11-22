# What Is Knowledge?

Knowledge is information about the **world, a domain, a system, or a specific subject** that an Agent can use to understand a situation and make decisions.

For example, a company might have knowledge about:

- Products
- Pricing
- Documentation
- Policies
- Customers
- Internal processes
- Technical documentation

An Agent can access this knowledge through different sources:

`Knowledge → Database / Documents / APIs / Search / RAG → Context → LLM`

## Knowledge vs. Memory

Knowledge and Memory can look similar, but they have different purposes.

For example:

`Knowledge → "Webito's Pro plan costs €38/month."`

`Memory → "This user prefers the Pro plan."`

The first is a fact about the product.

The second is information about a specific user's preference.

## Knowledge vs. Context

Knowledge is not the same as Context either.

Knowledge can exist outside the current execution and be retrieved when needed.

Context is the information actually provided to the LLM at a specific step.

For example:

`Knowledge Base → Retrieve relevant documents → Context → LLM`

This is why **RAG** is commonly used to provide relevant knowledge to an Agent without putting the entire knowledge base into the context.

## Knowledge Sources

Knowledge can come from many different sources:

- Documents
- Databases
- APIs
- Websites
- Search engines
- Internal systems
- Knowledge bases
- Other Agents

## Knowledge Is Not Always Static

Knowledge can be:

- **Static** — documentation, product information, policies.
- **Dynamic** — stock prices, weather, order status, system state.
- **Personalized** — information specific to a user or organization.

## Summary

A useful mental model is:

`Knowledge → Retrieve → Context → LLM → Decision`

![Knowledge & Retrieval in Agentic Systems](../assets/agentic-knowledge-flow.jpeg)

> **Knowledge describes what is known about a domain or the world, while Context is the information available to the LLM at a specific point in an execution.**
