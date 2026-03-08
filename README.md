# aidriven

AI-driven lifecycle orchestration using pure OpenCode agent markdown files. No plugins, no code — just `.md` files.

## What this is

A single `/lifecycle` command that chains three specialized subagents automatically:

```
/lifecycle "your task" → orchestrator → planner → executor → reviewer → retry up to 3x → DONE / FAILED
```

Everything is implemented as OpenCode agent and command markdown files. Drop them into any project and they work immediately.

## How it works

```
/lifecycle "task description"
        │
        ▼
lifecycle-orchestrator
        │
        ├─► lifecycle-planner     (read-only — produces numbered plan)
        │
        ├─► lifecycle-executor    (full access — implements the plan)
        │
        └─► lifecycle-reviewer    (read-only — PASS or FAIL verdict)
                │
                ├── PASS → [LIFECYCLE] Result: DONE ✓
                │
                └── FAIL (retry_count < 3) → back to executor with required fixes
                         FAIL (retry_count = 3) → [LIFECYCLE] Result: FAILED after 3 attempts ✗
```

The orchestrator prints a status line before each stage:

```
[LIFECYCLE] Stage: PLANNING...
[LIFECYCLE] Stage: EXECUTING (attempt 1/3)...
[LIFECYCLE] Stage: REVIEWING (attempt 1/3)...
[LIFECYCLE] Result: DONE ✓
```

## Usage

Open OpenCode in this folder (or any folder where you've installed the agents) and run:

```
/lifecycle "create a REST endpoint for user registration"
```

That's it. The orchestrator handles the rest.

## Install in another project

**Option 1 — Symlink** (changes here apply everywhere):
```bash
ln -s ~/Documents/Development/aidriven/.opencode /your-project/.opencode
```

**Option 2 — Copy** (independent copy per project):
```bash
cp -r ~/Documents/Development/aidriven/.opencode /your-project/
```

**Option 3 — Global** (available in every OpenCode session):
```bash
cp ~/Documents/Development/aidriven/.opencode/agents/* ~/.config/opencode/agents/
cp ~/Documents/Development/aidriven/.opencode/commands/* ~/.config/opencode/commands/
```

## Agents

| Agent | Mode | Tools | Purpose |
|---|---|---|---|
| `lifecycle-orchestrator` | subagent | no write/edit/bash | Sequences planner → executor → reviewer, retries on FAIL |
| `lifecycle-planner` | subagent | no write/edit/bash | Produces a numbered implementation plan |
| `lifecycle-executor` | subagent | full access | Implements the plan, produces execution report |
| `lifecycle-reviewer` | subagent | no write/edit; bash read-only | Reviews output with ECC skills, returns PASS/FAIL verdict |
