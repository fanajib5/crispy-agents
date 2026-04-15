---
description: Akordium Agent — single entry point. Routes task to correct agent pipeline automatically. Usage: /akordium-agent "describe your task here"
---

# /akordium-agent

This is the single entry command for all Akordium development tasks. It invokes the **Akordium Router** which classifies your task and dispatches the correct agent pipeline.

## Usage

```bash
/akordium-agent "fix the login button not submitting on mobile"
/akordium-agent "add Google OAuth login"
/akordium-agent "refactor the workspace service to use repository pattern"
/akordium-agent "research: should we use tRPC or REST for the new API layer?"
/akordium-agent "bump Supabase JS SDK from v1 to v2"
```

## What Happens Next

1. **Router** reads your task and classifies it (bug-fix / new-feature / refactor / research-spike / dependency-update)
2. **Router** shows you the planned pipeline + cost estimate
3. **You approve** ("yes" / "go") or revise
4. **Pipeline executes** with the right agents in the right order
5. **Cost Dashboard** logs the session to `.akordium/cost-log.jsonl`

## Model Tiers Used

See `config/kilo-models.md` for the full breakdown. In short:

| Task Phase | Tier | Reason |
|---|---|---|
| Routing + Architecture | 🔴 Frontier | Errors here compound downstream |
| Coding + Review + Verify | 🟡 Balanced | High volume, good quality |
| Cost Logging | 🟢 Free | Math only |

## Output Files

This command produces files in `.akordium/`:
```
.akordium/
├── cost-log.jsonl          ← append-only cost log, every session
├── pr-description.md       ← latest PR description (overwritten each run)
├── last-test-run.log       ← last test output from verifier
├── plan.md                 ← tactical plan from architect (if new-feature route)
└── structure.md            ← design doc from architect (if new-feature route)
```

## Monthly Cost Review

```bash
/akordium-agent --summary   ← shows monthly spending breakdown
```

## Tips

- Be specific in your task description — the more context, the better the routing
- If Router asks a clarifying question, answer it (1 sentence is enough)
- You can force a specific route by prefixing: `bug: ...`, `feature: ...`, `research: ...`
- The Human Gate in new-feature and refactor routes is mandatory — you must approve the plan
