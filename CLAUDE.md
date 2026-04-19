## Model Selection

Before starting ANY task, classify it and announce the tier + model. This is mandatory — never skip it.

### Routing Rules

| Tier | Model | When |
|------|-------|------|
| **1 — Light** | `claude-haiku-4-5-20251001` | Read files, run tools (ruff, black, pytest, mypy, pre-commit), grep, git status/log/diff, parse config, count lines |
| **2 — Standard** | `claude-sonnet-4-6` | Write code, fix bugs, refactor, write tests, explain code, code review, debug (any scope), integration work |
| **3 — Heavy** | `claude-opus-4-7` | Architecture decisions, greenfield design, security audits, threat modeling, strategic decisions, "I don't know where to start" |

### Required Behavior

1. **Before every response**, output exactly this line first:
   `> Task: [one-line description] → Tier [N], using [model-id]`

2. Then switch model if needed:
   `/model claude-haiku-4-5-20251001`   (Tier 1)
   `/model claude-sonnet-4-6`           (Tier 2)
   `/model claude-opus-4-7`             (Tier 3)

3. **Upgrade one tier** if: task touches auth/payments/migrations, user says "don't rush" or "use the best model", previous attempt at lower tier failed.

4. **Downgrade one tier** if: user says "quick check", "fast scan", output will be reviewed before action.

5. **Composite tasks** (e.g. "run tests and fix failures"): split by sub-task, announce tier per step.
   - Run tests → Tier 1
   - Fix failures → Tier 2

### Default
When uncertain: **Tier 2, `claude-sonnet-4-6`**

### Extended Reference
Full examples and edge cases: `.claude/skills/model-router-skill/SKILL.md`
