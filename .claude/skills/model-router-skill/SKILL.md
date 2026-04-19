---
name: model-router
description: >
  Dynamic model router for Claude Code — automatically selects the most cost-effective
  Claude model for each task instead of always defaulting to the most expensive model.
  ALWAYS use this skill at the start of any Claude Code session, or whenever the user
  asks Claude to perform a task of any kind (read files, run tests, lint, write code,
  refactor, review architecture, debug, etc.) so the right model is chosen before work
  begins. Triggers on: any coding task, file operations, test runs, linting, code review,
  architecture planning, debugging, or any request where model selection matters for the
  cost/quality tradeoff. When in doubt — use this skill.
---

# Model Router — Claude Code

## Purpose

Select the appropriate Claude model **before starting work**, based on task complexity and
required reasoning depth. Default behavior in Claude Code uses expensive models even for
trivial tasks. This skill fixes that by routing each task to the cheapest model that can
handle it well.

---

## Routing Table

| Tier | Model | Cost Profile | Use When |
|------|-------|-------------|----------|
| **1 — Light** | `claude-haiku-4-5-20251001` | Very low | File reads, grep/search, lint/format runs, test execution, bash commands, config parsing |
| **2 — Standard** | `claude-sonnet-4-6` | Medium | Writing new code, fixing bugs, refactoring, writing tests, explaining code, code review, complex debugging, large-scale refactors, integration work |
| **3 — Heavy** | `claude-opus-4-7` | High | Architecture decisions, system design, security audits, threat modeling, deep investigation with no clear starting point, greenfield design |

> **Default when uncertain:** Use Tier 2 (`claude-sonnet-4-6`) — it covers the vast majority of coding tasks well.

---

## Decision Logic

### Step 1 — Classify the task

**Tier 1 — Execution only (no reasoning needed)**
- Reading / listing / searching files
- Running `ruff`, `black`, `mypy`, `pytest`, `pre-commit`
- `grep`, `ripgrep`, `find`, `awk`
- `git status`, `git log`, `git diff`, `git blame`
- Parsing a config file and extracting a value
- Summarizing output already on screen
- Checking file existence, counting lines, diffing two files
- Adding a trivial docstring or one-liner comment

**Tier 2 — Reasoning required (any scope)**
- Writing a new function or class
- Fixing a bug — simple or multi-file
- Refactoring — single module or across multiple files
- Writing unit tests for an existing function
- Explaining what a code block does
- Adding type hints or docstrings (non-trivial)
- Debugging an issue that spans multiple files or services
- Code review requiring trade-off analysis
- Integrating a new library into an existing system
- Rewriting a component while preserving behavior
- Designing a feature within an existing architecture (not greenfield)
- Large-scale refactors touching multiple files

**Tier 3 — Deep reasoning (ambiguous, architectural, system-wide)**
- Greenfield system or service design
- Security audits and threat modeling
- Evaluating competing architectural approaches
- Diagnosing intermittent/production issues with no clear root cause
- Cross-cutting concerns affecting the whole codebase
- Any task where the user says "I don't know where to start"
- Trade-off decisions with long-term consequences
- Strategic decisions (migrate to GraphQL? CQRS? microservices?)

---

### Step 2 — Apply modifiers

**Upgrade one tier if:**
- The user has already tried a lower-tier approach and it failed
- The task touches production-critical paths (auth, payments, data migrations, security)
- The codebase is large (>10k LOC) and the task is cross-cutting
- User says: "use the best model", "don't rush", "I need this to be solid"

**Downgrade one tier if:**
- User says: "quick check", "just verify", "fast scan", "don't overthink it"
- Output will be reviewed/approved by user before any action is taken
- The change is purely additive with no risk of breaking existing behavior
- It's a one-liner or a template-style change

---

### Step 3 — Set the model

```bash
# Via CLI flag (per-session):
claude --model claude-haiku-4-5-20251001
claude --model claude-sonnet-4-6
claude --model claude-opus-4-7

# Within a session via slash command:
/model claude-haiku-4-5-20251001
/model claude-sonnet-4-6
/model claude-opus-4-7
```

**In `CLAUDE.md` (project-level default):**
```markdown
## Model Selection
Default to the model-router skill for all tasks.
Fallback default if skill is not consulted: claude-sonnet-4-6
```

---

## Quick Reference

```
TIER 1 — claude-haiku-4-5-20251001   → read, run, find, lint, format, check, diff
TIER 2 — claude-sonnet-4-6           → write, fix, test, explain, refactor, debug, review
TIER 3 — claude-opus-4-7             → design, architect, audit, investigate, greenfield
```

---

## Behavior Instructions

When this skill is active, Claude must:

1. **State the selected model and tier before starting work:**
   > "Task: run lint + show failures → Tier 1, using `claude-haiku-4-5-20251001`."
   > "Task: refactor auth module across 4 files → Tier 2, using `claude-sonnet-4-6`."
   > "Task: design event-driven architecture → Tier 3, using `claude-opus-4-7`."

2. **Re-evaluate mid-task** if scope expands unexpectedly. If a "quick fix" reveals a deeper
   architectural issue, stop, announce the escalation, and confirm with the user.

3. **Never silently use a heavier model** than the task requires.

4. **When ambiguous between two tiers**, pick the lower one and state the assumption.
   Ask at most one clarifying question if the stakes are high.

5. **For composite tasks** (e.g., "run tests AND fix failures"), split by sub-task:
   - Run tests → Tier 1
   - Diagnose failures → Tier 2
   - Fix them → Tier 2

---

## Examples

| User Request | Tier | Model | Reason |
|---|---|---|---|
| "Run pytest and show failures" | 1 | `claude-haiku-4-5-20251001` | Pure execution |
| "Read `config.yaml`, tell me the DB host" | 1 | `claude-haiku-4-5-20251001` | File read + extract |
| "Add a docstring to this function" | 1 | `claude-haiku-4-5-20251001` | Trivial, no reasoning |
| "Run `black`, then fix issues it flags" | 1→2 | Haiku→Sonnet | Split by sub-task |
| "Fix this TypeError in `utils.py`" | 2 | `claude-sonnet-4-6` | Single-file bug fix |
| "Write unit tests for `calculate_discount`" | 2 | `claude-sonnet-4-6` | Standard test writing |
| "Refactor the auth module to use DI" | 2 | `claude-sonnet-4-6` | Multi-file, higher stakes |
| "Why does the API return 500 intermittently?" | 2 | `claude-sonnet-4-6` | Complex debug, unclear cause |
| "Review this PR for correctness and architecture" | 2 | `claude-sonnet-4-6` | Nuanced multi-concern review |
| "Should we use event sourcing for orders?" | 3 | `claude-opus-4-7` | Architectural decision |
| "Security audit of our API layer" | 3 | `claude-opus-4-7` | Threat modeling |
| "Design a greenfield service from scratch" | 3 | `claude-opus-4-7` | Greenfield design |

---

## Team Notes

- Override per-session anytime with `/model <model-id>`
- For CI/agentic pipelines — always set model explicitly, don't rely on routing
- To update routing rules for your team: edit the Routing Table and Decision Logic sections above
- Keep this file in version control so routing evolves with the team's needs

---

## See Also

- `task-examples.md` — Extended example bank per tier
- Claude Code docs: https://docs.anthropic.com/en/docs/claude-code
