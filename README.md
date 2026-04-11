# CRISPY Agents

Multi-agent coding system built on [OpenCode](https://opencode.ai) using the [CRISPY framework](https://github.com/nicekid1/CRISPY-Agents) by Dex from HumanLayer.

Five specialized agents, each with a distinct role and a different model, orchestrating through seven phases with a 40-instruction budget per phase.

## What You Get

- **5 agent configs** (`~/.config/opencode/agents/`) — architect, coder, reviewer, scout, verifier
- **1 build command** (`~/.config/opencode/commands/`) — `/build-workflow` orchestrates all agents through the CRISPY phases
- **Model diversity by default** — coder and reviewer use different models so they catch each other's blind spots

## The Agents

| Agent | Role | Model | Why |
|-------|------|-------|-----|
| Architect | Context, Structure, Plan phases | GLM 5.1 | SWE-bench Pro #1, sustained execution |
| Coder | Vertical slice implementation | GLM 5.1 | Same context as architect for alignment |
| Reviewer | Validates against structure.md | Qwen 3.6 Plus | Different model catches different bugs |
| Scout | Pure research, no implementation intent | Kimi K2.5 | Fresh context, pure facts |
| Verifier | Generates and runs validation scripts | GLM 4.7 | Execution, not reasoning |

## The Build Command

`/build-workflow` runs the full CRISPY pipeline:

1. **Context** — Load relevant codebase areas
2. **Research** — Pure facts from scout (no implementation intent)
3. **Investigate** — Dig deeper on specific areas
4. **Structure** — High-level outline with testing checkpoints (produces `structure.md`)
5. **Plan** — Tactical execution plan (produces `plan.md`)
6. **Human Review Gate** — You approve before any code is written
7. **Yield** — Implement in vertical slices with TDD mandatory
8. **PR** — Ship it

Each phase gets a max of 40 instructions. No 1,000-line monolithic plans.

## Setup

1. Install [OpenCode](https://opencode.ai): `npm install -g opencode`
2. Copy agents:
   ```bash
   cp agents/*.md ~/.config/opencode/agents/
   ```
3. Copy the build command:
   ```bash
   cp commands/build-workflow.md ~/.config/opencode/commands/
   ```
4. Configure model providers in `~/.config/opencode/opencode.json` (see below)
5. Run `/build-workflow "your feature description"` in any project directory

## Model Provider Config

You'll need API access to the models used by each agent. The cheapest way:

| Provider | Cost | What You Get |
|----------|------|-------------|
| [Ollama Cloud Pro](https://ollama.com) | ~$20/month | GLM 5.1, GLM 4.7, Kimi K2.5 |
| [Fireworks AI](https://fireworks.ai) | ~$15-25/month | Qwen 3.6 Plus (pay-per-token) |

Total: ~$35-45/month.

Create `~/.config/opencode/opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "your-provider-name": {
      "name": "Your Provider",
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "https://your-provider-endpoint/v1"
      },
      "models": {
        "your-model-id": {
          "name": "Model Name",
          "cost": {
            "input": 0.52,
            "output": 4.40,
            "cache_read": 0.05
          }
        }
      }
    }
  }
}
```

Map the model IDs in each agent's frontmatter to match your provider config. The default agent files use a Bifrost gateway setup — adjust the model IDs to match whatever provider you use.

## Why Different Models for Coder and Reviewer?

When the same model writes code and reviews it, the reviewer shares the same blind spots. It wrote the code. Of course it thinks the code is fine.

GLM 5.1 and Qwen 3.6 Plus have different training data, different architectural biases, different failure modes. Qwen catches things GLM is blind to, and vice versa. The cost difference is negligible. The quality difference is significant.

## Credits

- **CRISPY framework** by Dex from HumanLayer — the instruction budget principle, phase decomposition, and vertical slice methodology
- **OpenCode** — the open-source terminal-based coding agent this runs on
- Model providers: Zhipu AI (GLM), Alibaba (Qwen), Moonshot AI (Kimi)

## License

MIT