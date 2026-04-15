# Kilo Code Gateway — Model Tiers for Akordium

Akordium uses [Kilo Code](https://kilocode.ai) as the AI gateway. This file defines which model tier each agent uses, balancing quality vs. cost.

> Kilo acts as a unified API gateway — you get access to 300+ models under one subscription/billing. No need to manage individual API keys per provider.

---

## Model Tier Definitions

### 🟢 Free Tier — Zero API cost
Use for: high-volume, low-stakes tasks (logging, formatting, simple grep-style analysis)

```yaml
free:
  primary: google/gemini-2.0-flash-exp       # Fast, capable, free via Kilo
  fallback: deepseek/deepseek-chat           # Strong reasoning, very low cost
  use_for:
    - cost-dashboard agent (token counting, IDR formatting)
    - Simple file reads and grep
    - PR description templating (boilerplate sections)
```

### 🟡 Balanced Tier — Low cost, high throughput
Use for: most coding and review tasks where quality matters but budget is a concern

```yaml
balanced:
  primary: anthropic/claude-3-5-haiku-latest   # Best balanced model via Kilo
  fallback: qwen/qwen2.5-coder-32b-instruct    # Strong at code, cheap
  alternative: deepseek/deepseek-r1-0528       # Good reasoning, low cost
  use_for:
    - coder agent (implementation)
    - reviewer agent (code quality checks)
    - akordium-scout (codebase-only research, no web)
    - verifier agent (running test scripts)
```

### 🔴 Frontier Tier — Higher cost, maximum quality
Use for: architecture, complex planning, ambiguous problems where errors compound

```yaml
frontier:
  primary: anthropic/claude-sonnet-4-5         # Best reasoning via Kilo
  fallback: google/gemini-2.5-pro              # Strong at long context
  alternative: openai/gpt-4o                   # Reliable for structured output
  use_for:
    - architect agent (design docs, tactical plans)
    - akordium-scout (web research + tradeoff analysis)
    - akordium-router (task classification on ambiguous input)
```

---

## Per-Agent Model Assignment

| Agent | Tier | Primary Model | Rationale |
|---|---|---|---|
| `akordium-router` | 🔴 Frontier | claude-sonnet-4-5 | Task misclassification is expensive |
| `architect` | 🔴 Frontier | claude-sonnet-4-5 | Planning errors compound |
| `akordium-scout` (web) | 🔴 Frontier | gemini-2.5-pro | Long doc reading + web search |
| `akordium-scout` (codebase) | 🟡 Balanced | qwen2.5-coder-32b | Pure code grep, no web needed |
| `coder` | 🟡 Balanced | claude-3-5-haiku | High volume, good code quality |
| `reviewer` | 🟡 Balanced | deepseek-r1-0528 | Different model from coder = less bias |
| `verifier` | 🟡 Balanced | claude-3-5-haiku | Script execution, structured output |
| `cost-dashboard` | 🟢 Free | gemini-2.0-flash-exp | Math + formatting, no reasoning needed |

---

## Cost Estimation (USD → IDR)

Reference rates for `cost-dashboard` calculations:

```
gemini-2.0-flash-exp : ~$0.00/1M tokens (free tier)
deepseek-chat        : ~$0.14/1M input, $0.28/1M output
qwen2.5-coder-32b    : ~$0.30/1M input, $0.30/1M output
claude-3-5-haiku     : ~$0.80/1M input, $4.00/1M output
claude-sonnet-4-5    : ~$3.00/1M input, $15.00/1M output
gemini-2.5-pro       : ~$1.25/1M input, $10.00/1M output

USD to IDR: ~16,000 (update monthly in cost-dashboard)
```

---

## OpenCode Config Reference

To wire these models in OpenCode's config, use the provider format:

```json
// .opencode/config.json
{
  "agents": {
    "akordium-router": { "model": "anthropic/claude-sonnet-4-5" },
    "architect":       { "model": "anthropic/claude-sonnet-4-5" },
    "scout":           { "model": "google/gemini-2.5-pro" },
    "akordium-scout":  { "model": "google/gemini-2.5-pro" },
    "coder":           { "model": "anthropic/claude-3-5-haiku-latest" },
    "reviewer":        { "model": "deepseek/deepseek-r1-0528" },
    "verifier":        { "model": "anthropic/claude-3-5-haiku-latest" },
    "cost-dashboard":  { "model": "google/gemini-2.0-flash-exp" }
  },
  "provider": "kilocode"
}
```

All model IDs follow Kilo's routing format: `provider/model-name`.
