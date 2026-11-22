# Logits, Softmax & Decoding Strategies (How LLMs Choose Tokens)

In [Chapter 1](./01-how-llm-works.md), we learned that Large Language Models predict the next token. But how does a network of billions of neural weights actually transform internal activation states into human-readable text, structured JSON, or tool calls?

This chapter explores the final layer of an LLM: the mathematics of **Logits**, the **Softmax Layer**, and the decoding strategies (**Temperature**, **Top-K**, **Top-P**, and **Constrained Decoding**) that control whether an agent behaves like a precise, deterministic software controller or a creative, hallucinating poet.

---

## 1. The Output Pipeline: From Matrix to Token

Inside a Transformer, layers of multi-head self-attention process input tokens and produce a final dense vector called the **Hidden State** ($\vec{h} \in \mathbb{R}^{d_{\text{model}}}$).

Before we see words on our screen, that vector travels through an output pipeline:

```
┌────────────────────────────────────────────────────────┐
│  Final Transformer Layer (Hidden State Vector)         │
│  Vector size: [1 x 4,096]                              │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│  Linear Unembedding Head (Language Model Head)         │
│  Matrix size: [4,096 x Vocabulary_Size (128,000)]      │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
┌────────────────────────────────────────────────────────┐
│  LOGITS (Raw, unnormalized scores for every token)     │
│  [ "def": 14.2, "class": 12.1, "apple": -3.8, ... ]    │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼  ◄── Temperature Scaling (z / T)
┌────────────────────────────────────────────────────────┐
│  SOFTMAX LAYER (Converts logits into probabilities)    │
│  [ "def": 0.72, "class": 0.21, "apple": 0.00001, ... ] │
│  (Total sum of probabilities = 1.0)                    │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼  ◄── Top-K / Top-P Filtering
┌────────────────────────────────────────────────────────┐
│  DECODING STRATEGY (Greedy or Stochastic Sampling)     │
└──────────────────────────┬─────────────────────────────┘
                           │
                           ▼
                     Chosen Token: "def"
```

---

## 2. What Are Logits?

**Logits** are the raw, unnormalized prediction scores produced by the final linear projection layer for every single token in the model's vocabulary (typically 32,000 to 128,000 tokens).

### Key Characteristics of Logits:
- **Unbounded:** A logit can be any real number from $-\infty$ to $+\infty$ (e.g., `-12.5`, `0.0`, `18.3`).
- **Relative Ranking:** A higher logit indicates that the model's neural network finds that token more plausible in the current context.
- **Not Probabilities:** Logits do not sum to 1.0 and cannot be directly interpreted as percentages.

---

## 3. The Softmax Layer: Converting Scores into Probabilities

To convert arbitrary real-valued logits into a valid probability distribution where every value is between $0$ and $1$, and the sum of all values equals exactly $1$, deep learning uses the **Softmax function**:

$$\text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{|V|} e^{z_j}}$$

Where:
- $z_i$ is the logit for token $i$.
- $|V|$ is the total vocabulary size.
- $e^{z_i}$ is Euler's exponential function.

### Why Exponential ($e^z$)?
1. **Positivity:** For any real number $z$, $e^z$ is strictly positive ($> 0$).
2. **Difference Amplification:** Small differences in logits become significant differences in probabilities. If logit $A = 10$ and logit $B = 8$, $e^{10} \approx 22,026$ while $e^8 \approx 2,980$ (a 7.4x difference in probability!).

### Python Implementation of Softmax

```python
import math
from typing import Dict, List

def softmax(logits: Dict[str, float]) -> Dict[str, float]:
    """
    Numerically stable Softmax implementation in pure Python.
    Subtracting max(logits) prevents exponential overflow (float inf).
    """
    max_logit = max(logits.values())
    
    # Calculate e^(z - max_z) for numerical stability
    exp_scores = {token: math.exp(score - max_logit) for token, score in logits.items()}
    sum_exp = sum(exp_scores.values())
    
    # Normalize to probabilities summing to 1.0
    return {token: exp_score / sum_exp for token, exp_score in exp_scores.items()}

# Example usage:
raw_logits = {"def": 12.0, "class": 10.5, "import": 8.0, "banana": -2.0}
probabilities = softmax(raw_logits)

for token, prob in probabilities.items():
    print(f"Token: {token:<8} | Prob: {prob:.4f} ({prob * 100:.2f}%)")
```

---

## 4. Temperature ($T$): Controlling Entropy and Determinism

**Temperature** is a hyperparameter that scales the logits *before* they enter the Softmax function:

$$P(\text{token}_i) = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}$$

```
High Temperature (T = 1.5): Flat, creative, high entropy
Probability
 │   ___   ___       ___
 └───┴─┴───┴─┴───────┴─┴──────► Tokens (Rare tokens get a chance)

Standard Temperature (T = 0.7): Balanced, conversational
Probability
 │     █
 │     █   ▄
 └─────█───█───▂──────────────► Tokens

Low Temperature / Greedy (T → 0.0): Winner-takes-all, deterministic
Probability
 │     █
 │     █
 └─────█──────────────────────► Tokens (Only highest logit is picked)
```

### The Three Operating Regimes:

1. **$T \to 0$ (Greedy / Argmax):**
   - The highest logit is scaled toward infinity relative to others.
   - The top token gets $\approx 100\%$ probability.
   - **Crucial for Agents:** Tool calling, JSON schema compliance, and math problems require $T = 0.0$ so the model always picks the most logically sound next token.

2. **$T = 0.6 - 0.8$ (Standard Conversational):**
   - Retains fluency while introducing modest diversity.
   - Ideal for general chat, copywriting, and summarization.

3. **$T \ge 1.2$ (High Entropy / Creative):**
   - Flattens the distribution; differences between likely and unlikely tokens shrink.
   - High risk of syntax errors, hallucinations, and incoherent rambling.

---

## 5. Decoding Strategies: Filtering the Distribution

Even with Softmax and Temperature, picking tokens from a vocabulary of 100,000+ words requires a **decoding strategy**.

```
┌─────────────────────────────────────────────────────────────────┐
│                    DECODING STRATEGIES                          │
├────────────────────────────┬────────────────────────────────────┤
│ Deterministic              │ Stochastic (Probabilistic)         │
├────────────────────────────┼────────────────────────────────────┤
│ • Greedy Search            │ • Top-K Sampling                   │
│   (Always pick argmax)     │   (Keep only top K items)          │
│                            │                                    │
│ • Beam Search              │ • Top-P / Nucleus Sampling         │
│   (Track top B sequences)  │   (Keep smallest sum ≥ P)          │
│                            │                                    │
│ • Constrained Decoding     │ • Min-P Sampling                   │
│   (Grammar/JSON masks)     │   (Filter by percentage of max)    │
└────────────────────────────┴────────────────────────────────────┘
```

### A. Greedy Decoding
Always select the token with the highest probability:
$$\text{token} = \arg\max P(\text{token} \mid \text{context})$$
- **Pros:** Completely deterministic and fast.
- **Cons:** Can fall into repetitive loops (e.g., *"and then, and then, and then"*).

### B. Top-K Sampling
Sort tokens by probability and only sample from the top $K$ candidates (e.g., $K = 50$). All other tokens are zeroed out.
- **Limitation:** In easy contexts (e.g., `"The capital of France is"`), only 1 token makes sense, so taking 50 tokens includes irrelevant noise. In complex contexts, 50 tokens might not be enough.

### C. Top-P (Nucleus Sampling)
Instead of a fixed number of tokens, Top-P dynamically chooses the smallest set of tokens whose cumulative probability exceeds a threshold $P$ (e.g., $P = 0.90$):

```
Sorted Probabilities:
"Paris":   0.85  ──┐ Cumulative Sum = 0.85 (< 0.90) -> KEEP
"Lyon":    0.08  ──┴ Cumulative Sum = 0.93 (≥ 0.90) -> KEEP
─────────────────────────────────────────────────────────────
"Marseille": 0.04  ── Cumulative Sum = 0.97 -> DISCARD (Zero out)
"Bordeaux":  0.03  ── Cumulative Sum = 1.00 -> DISCARD (Zero out)
```

- When the model is confident, the nucleus contains only 1 or 2 tokens.
- When the model is uncertain, the nucleus expands dynamically.

---

## 6. Python Mental Model: The Complete Sampling Engine

Here is a self-contained Python script demonstrating how Logits, Temperature, Top-K, and Top-P work together:

```python
import math
import random
from typing import Dict, List, Tuple

def sample_token(
    logits: Dict[str, float],
    temperature: float = 0.7,
    top_k: int = 50,
    top_p: float = 0.9
) -> str:
    """
    A complete LLM sampling engine in pure Python.
    """
    # 1. Greedy shortcut for agents
    if temperature <= 0.001:
        return max(logits, key=logits.get)

    # 2. Temperature scaling
    scaled_logits = {tok: score / temperature for tok, score in logits.items()}

    # 3. Sort by logit score descending
    sorted_items = sorted(scaled_logits.items(), key=lambda x: x[1], reverse=True)

    # 4. Top-K Truncation
    if top_k > 0:
        sorted_items = sorted_items[:top_k]

    # 5. Compute Softmax probabilities on truncated candidates
    max_score = sorted_items[0][1]
    exp_scores = [(tok, math.exp(score - max_score)) for tok, score in sorted_items]
    sum_exp = sum(score for _, score in exp_scores)
    probs = [(tok, score / sum_exp) for tok, score in exp_scores]

    # 6. Top-P (Nucleus) Filtering
    cumulative_prob = 0.0
    nucleus_candidates: List[Tuple[str, float]] = []
    
    for tok, prob in probs:
        nucleus_candidates.append((tok, prob))
        cumulative_prob += prob
        if cumulative_prob >= top_p:
            break

    # 7. Normalize nucleus candidates and sample
    nucleus_total = sum(p for _, p in nucleus_candidates)
    tokens = [tok for tok, _ in nucleus_candidates]
    weights = [p / nucleus_total for _, p in nucleus_candidates]

    return random.choices(tokens, weights=weights, k=1)[0]


# Test the engine with simulated logits:
simulated_logits = {
    "get_weather": 14.5,
    "search_database": 14.1,
    "send_email": 8.0,
    "delete_file": -5.0,
    "sing_a_song": -12.0
}

# In an agent tool-selection turn:
deterministic_choice = sample_token(simulated_logits, temperature=0.0)
print(f"Deterministic Agent Choice (T=0.0): {deterministic_choice}")

creative_choice = sample_token(simulated_logits, temperature=1.2, top_p=0.95)
print(f"Stochastic Choice (T=1.2):          {creative_choice}")
```

---

## 7. The Agent Perspective: Why Decoding Strategy Is Critical

In an Agentic System, an LLM is not writing a poem—it is generating **structured code, API calls, and tool parameters**.

```
PROMPT: "Call the database tool to fetch user #42."

WITH HIGH TEMPERATURE (T = 0.9):
Result: {"tool": "fetch_user", "id": 42,}  <── Trailing comma breaks JSON parser!

WITH DETERMINISTIC DECODING (T = 0.0):
Result: {"tool": "fetch_user", "id": 42}   <── 100% valid JSON payload.
```

### Why Agents Require $T = 0.0$:
1. **JSON Schema Validity:** Tool parameters must adhere to strict schemas. High temperature introduces syntax corruption (unmatched brackets, invalid keys).
2. **Reproducibility:** When debugging agent workflows, you want the agent to take the same path for the same input to diagnose errors.
3. **Tool Hallucination:** At $T > 0$, the model might invent non-existent tool names from its training memory instead of using the tools provided in the prompt.

---

## 8. Modern Breakthrough: Constrained Decoding (Structured Outputs)

In late 2024 and 2025, modern inference engines (such as OpenAI Structured Outputs, vLLM, and llama.cpp) solved the JSON failure problem permanently using **Constrained Decoding** (Grammar-guided generation):

```
At Step t:
Model wants to emit next token for JSON:
Valid next tokens: ['"', ':', '{', 'number']
Invalid next tokens: ['while', 'def', 'SELECT', 'apple']

──► Linear Layer produces logits for all 128,000 tokens
──► LOGIT MASKING: Set logits of all INVALID tokens to -∞
──► Softmax(invalid) = 0.000%
──► The model is mathematically FORCED to generate 100% valid JSON!
```

<details>
<summary><b>🔍 Code Example: Structured Outputs with Pydantic in Python</b></summary>

Modern libraries enforce schema conformance at the decoding level rather than relying on regex retries:

```python
from pydantic import BaseModel, Field
from typing import Literal

# 1. Define strict tool parameter schema
class DatabaseQueryTool(BaseModel):
    action: Literal["SELECT", "COUNT"]
    table_name: str = Field(description="Target SQL table")
    limit: int = Field(default=10, ge=1, le=100)

# In modern SDKs (OpenAI / Instructor / Outlines):
# The engine compiles this Pydantic schema into a context-free grammar (CFG)
# and masks out invalid token logits on every step of token generation.
```
</details>

---

## 9. Summary & Engineering Rules of Thumb

| Use Case | Recommended Temperature | Recommended Top-P | Why? |
| :--- | :--- | :--- | :--- |
| **Agent Tool Calling & JSON** | `0.0` (Greedy) | `1.0` (Off) | Maximum determinism, zero syntax errors. |
| **Code Generation & Refactoring**| `0.0 - 0.2` | `0.9` | Prevents subtle logic bugs and typos. |
| **Multi-Agent Brainstorming** | `0.6 - 0.7` | `0.9` | Encourages varied perspectives among agents. |
| **Creative Writing & Persona** | `0.8 - 1.0` | `0.95` | Maximizes lexical richness and natural variety. |
