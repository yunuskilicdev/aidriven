# AI-Driven Lifecycle Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a pure `.md`-file lifecycle orchestration system at `~/Documents/Development/aidriven/` that chains planner → executor → reviewer agents automatically via a single `/lifecycle` command.

**Architecture:** An orchestrator subagent receives the task via `/lifecycle`, delegates sequentially to three specialized subagents (planner, executor, reviewer), and retries execution up to 3 times on reviewer FAIL before reporting the final outcome. Everything is `.md` agent/command files — no plugin code.

**Tech Stack:** OpenCode agent markdown files, OpenCode command markdown files, ECC skills (used natively by reviewer agent).

---

### Task 1: Scaffold the project directory

**Files:**
- Create: `~/Documents/Development/aidriven/`
- Create: `~/Documents/Development/aidriven/opencode.json`
- Create: `~/Documents/Development/aidriven/README.md`
- Create: `~/Documents/Development/aidriven/.opencode/agents/` (directory)
- Create: `~/Documents/Development/aidriven/.opencode/commands/` (directory)
- Create: `~/Documents/Development/aidriven/docs/plans/` (directory)

**Step 1:** Create the directory tree
```bash
mkdir -p ~/Documents/Development/aidriven/.opencode/agents
mkdir -p ~/Documents/Development/aidriven/.opencode/commands
mkdir -p ~/Documents/Development/aidriven/docs/plans
```

**Step 2:** Write `opencode.json`
```json
{
  "$schema": "https://opencode.ai/config.json"
}
```

**Step 3:** Write `README.md` — what it is, how to use, how to install in another project

**Step 4:** Verify directories exist
```bash
ls ~/Documents/Development/aidriven/
ls ~/Documents/Development/aidriven/.opencode/
```

**Step 5:** Commit
```bash
git init ~/Documents/Development/aidriven
cd ~/Documents/Development/aidriven
git add .
git commit -m "chore: scaffold aidriven project structure"
```

---

### Task 2: Create `lifecycle-planner` agent

**Files:**
- Create: `~/Documents/Development/aidriven/.opencode/agents/lifecycle-planner.md`

**Step 1:** Write the file with this exact content:

```markdown
---
description: Creates a detailed step-by-step execution plan for a given task. Read-only — no file changes.
mode: subagent
tools:
  write: false
  edit: false
  bash: false
---

You are a planning specialist. Your job is to produce a clear, numbered, step-by-step plan for implementing the task you are given.

Your plan must include:
- Exact files to create or modify
- What each change should accomplish
- Any dependencies or ordering constraints between steps

Output ONLY the plan. Do not implement anything. Do not ask questions. Do not add commentary outside the plan.

Format:
## Plan
1. [Step description]
2. [Step description]
...
```

**Step 2:** Commit
```bash
git add .opencode/agents/lifecycle-planner.md
git commit -m "feat: add lifecycle-planner subagent"
```

---

### Task 3: Create `lifecycle-executor` agent

**Files:**
- Create: `~/Documents/Development/aidriven/.opencode/agents/lifecycle-executor.md`

**Step 1:** Write the file with this exact content:

```markdown
---
description: Implements a given plan by writing code, editing files, and running commands. Reports exactly what was done.
mode: subagent
---

You are an implementation specialist. You will be given a task description and a numbered plan.

Execute every step of the plan precisely. Use your tools (write, edit, bash) to make the actual changes.

When done, produce an execution report:

## Execution Report
- **Task:** [task description]
- **Steps completed:** [list]
- **Files created/modified:** [list with paths]
- **Commands run:** [list]
- **Outcome:** SUCCESS or PARTIAL (with explanation if partial)
```

**Step 2:** Commit
```bash
git add .opencode/agents/lifecycle-executor.md
git commit -m "feat: add lifecycle-executor subagent"
```

---

### Task 4: Create `lifecycle-reviewer` agent

**Files:**
- Create: `~/Documents/Development/aidriven/.opencode/agents/lifecycle-reviewer.md`

**Step 1:** Write the file with this exact content:

```markdown
---
description: Reviews implementation output and returns a structured PASS or FAIL verdict with notes. Uses skills for domain-specific review.
mode: subagent
tools:
  write: false
  edit: false
permission:
  bash:
    "*": deny
    "git diff*": allow
    "git log*": allow
    "git status*": allow
---

You are a code review specialist. You will be given a task description and an execution report.

First, use your Skill tool to load any relevant skills for this task. Examples:
- Always load: coding-standards
- If the task involves APIs: api-design
- If the task involves auth or secrets: security-review
- If the task involves tests: tdd-workflow

Review the implementation against the task requirements and the loaded skills.

Respond with ONLY this structure:

## Review Verdict

**Result:** PASS or FAIL

**Summary:** [1-2 sentences]

**Issues:** (only if FAIL)
- [specific issue 1]
- [specific issue 2]

**Required fixes:** (only if FAIL)
- [concrete fix 1]
- [concrete fix 2]
```

**Step 2:** Commit
```bash
git add .opencode/agents/lifecycle-reviewer.md
git commit -m "feat: add lifecycle-reviewer subagent"
```

---

### Task 5: Create `lifecycle-orchestrator` agent

**Files:**
- Create: `~/Documents/Development/aidriven/.opencode/agents/lifecycle-orchestrator.md`

**Step 1:** Write the file with this exact content:

```markdown
---
description: Orchestrates the full task lifecycle — plan, execute, review — with up to 3 retries on failure.
mode: subagent
tools:
  write: false
  edit: false
  bash: false
permission:
  task:
    "*": deny
    "lifecycle-planner": allow
    "lifecycle-executor": allow
    "lifecycle-reviewer": allow
---

You are a lifecycle orchestrator. When given a task, run the full pipeline:

**Pipeline:**
1. Call @lifecycle-planner with the task. Receive the plan.
2. Call @lifecycle-executor with the task + plan. Receive the execution report.
3. Call @lifecycle-reviewer with the task + execution report. Receive PASS or FAIL verdict.
4. If FAIL and retry_count < 3: increment retry_count, go to step 2. Pass the reviewer's required fixes to the executor alongside the original plan.
5. If PASS: report DONE.
6. If retry_count reaches 3 and still FAIL: report FAILED with the last reviewer verdict.

**Status updates:** Before each stage, output a one-line status:
- `[LIFECYCLE] Stage: PLANNING...`
- `[LIFECYCLE] Stage: EXECUTING (attempt N/3)...`
- `[LIFECYCLE] Stage: REVIEWING (attempt N/3)...`
- `[LIFECYCLE] Result: DONE ✓` or `[LIFECYCLE] Result: FAILED after 3 attempts ✗`

Do not skip stages. Do not combine stages. Always run them in order.
```

**Step 2:** Commit
```bash
git add .opencode/agents/lifecycle-orchestrator.md
git commit -m "feat: add lifecycle-orchestrator subagent"
```

---

### Task 6: Create the `/lifecycle` command

**Files:**
- Create: `~/Documents/Development/aidriven/.opencode/commands/lifecycle.md`

**Step 1:** Write the file with this exact content:

```markdown
---
description: Run the full AI-driven lifecycle (plan → execute → review) for a task
agent: lifecycle-orchestrator
subtask: true
---

Run the full lifecycle for this task:

$ARGUMENTS
```

**Step 2:** Verify by listing commands
```bash
ls ~/Documents/Development/aidriven/.opencode/commands/
```

**Step 3:** Commit
```bash
git add .opencode/commands/lifecycle.md
git commit -m "feat: add /lifecycle command"
```

---

### Task 7: Write README

**Files:**
- Modify: `~/Documents/Development/aidriven/README.md`

**Step 1:** Write the README with these sections:
- **What this is** — AI-driven lifecycle orchestration using pure OpenCode agent markdown files
- **How it works** — pipeline diagram: `/lifecycle "task"` → orchestrator → planner → executor → reviewer → retry up to 3x → DONE/FAILED
- **Usage** — open OpenCode in this folder, type `/lifecycle "your task description"`
- **Install in another project** — three options:
  1. Symlink: `ln -s ~/Documents/Development/aidriven/.opencode /your-project/.opencode`
  2. Copy: `cp -r ~/Documents/Development/aidriven/.opencode /your-project/`
  3. Global: copy agents + command into `~/.config/opencode/agents/` and `~/.config/opencode/commands/`
- **Agents** — table listing each agent, its mode, tools, and purpose

**Step 2:** Commit
```bash
git add README.md
git commit -m "docs: write README"
```

---

### Task 8: Smoke test

**Step 1:** Open OpenCode in `~/Documents/Development/aidriven/`

**Step 2:** Verify agents appear — type `@` in the prompt and confirm these four agents are listed:
- `lifecycle-orchestrator`
- `lifecycle-planner`
- `lifecycle-executor`
- `lifecycle-reviewer`

**Step 3:** Verify command appears — type `/lifecycle` and confirm it shows in autocomplete with description "Run the full AI-driven lifecycle (plan → execute → review) for a task"

**Step 4:** Run a minimal test task:
```
/lifecycle "create a hello.txt file with the text Hello World"
```

**Step 5:** Confirm:
- Orchestrator prints `[LIFECYCLE] Stage: PLANNING...`
- Orchestrator prints `[LIFECYCLE] Stage: EXECUTING (attempt 1/3)...`
- Orchestrator prints `[LIFECYCLE] Stage: REVIEWING (attempt 1/3)...`
- Final output is `[LIFECYCLE] Result: DONE ✓`
- `hello.txt` exists in the project directory

---

## Summary

| Task | File | Purpose |
|---|---|---|
| 1 | `opencode.json`, dirs | Scaffold |
| 2 | `lifecycle-planner.md` | Read-only planner subagent |
| 3 | `lifecycle-executor.md` | Full-access executor subagent |
| 4 | `lifecycle-reviewer.md` | Read-only reviewer with skills |
| 5 | `lifecycle-orchestrator.md` | Orchestrator: sequences + retries |
| 6 | `lifecycle.md` (command) | `/lifecycle` entrypoint |
| 7 | `README.md` | Documentation |
| 8 | — | Smoke test |

**Total files: 6 new `.md` files + `opencode.json` + `README.md`**
