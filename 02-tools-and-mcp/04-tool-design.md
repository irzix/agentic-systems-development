# Tool Design & Schemas

A good tool should be **clear, focused, and easy for the LLM to understand**.

The schema tells the model:

- What the tool does
- When to use it
- What arguments it accepts
- What type each argument has

For example:

```json
{
  "name": "get_customer",
  "description": "Get customer information by ID",
  "parameters": {
    "customer_id": {
      "type": "string",
      "description": "The customer's unique ID"
    }
  }
}
```

Good tool design usually means:

- **Clear purpose** : One tool should perform one well-defined action.
- **Good names and descriptions** : The model should easily understand when to use it.
- **Strong schemas** : Inputs should be typed and validated.
- **Focused interfaces** : Avoid exposing large, generic tools with many unrelated operations.

For example, this is usually better:

```text
get_customer(customer_id)
```

than:

```text
execute_operation(operation, payload)
```

because the model has a clearer understanding of what the tool does and when it should use it.
