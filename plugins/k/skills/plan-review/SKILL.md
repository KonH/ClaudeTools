---
name: plan-review
description: Review a dated plan under docs/specs/ for missing steps, wrong assumptions, or guideline violations before implementation begins.
---

Review the latest dated plan under `docs/specs/` (or the plan file specified in `$ARGUMENTS`) for potential problems before implementation begins.

## Plan Discovery

**With `$ARGUMENTS`:** resolve against `docs/specs/` — look for `docs/specs/<YY_MM_DD_HH>_<name>/plan.md` using the full dated spec identifier.

**Without `$ARGUMENTS`:**
1. List all `*/plan.md` files under `docs/specs/`
2. Extract the `YY_MM_DD_HH` prefix from each folder
3. Use the file with the latest timestamp prefix

## Rules

- Read the plan file first, then read relevant project rules (`CLAUDE.md` and any project rules directory) that apply to the plan's scope
- Spawn a sub-agent to perform the review independently (it should re-read the plan and rules itself)
- The sub-agent must NOT modify any files — review only
- Present all concerns at once, then ask the user to approve all or indicate which to skip
- Keep each point short: state the problem, why it matters, and a concise fix — no nitpicking on style or naming
- Format proposed fixes as `diff` blocks (lines removed prefixed with `-`, lines added prefixed with `+`) alongside the plain-text explanation
- Focus on: missing steps, wrong assumptions, inconsistency with project guidelines, architectural mismatches, and anything likely to cause rework during implementation
- If no real concerns are found, say so briefly and stop

Sub-agent prompt template:
> Review the plan at `[plan path]`. Read it in full, then read the relevant project rules for the plan's scope (check `CLAUDE.md` and any project rules directory for domain-specific guidance). Identify concerns — missing steps, wrong assumptions, guideline violations, architectural mismatches — that would cause problems during implementation. Return a numbered list of concerns, each with: problem, why it matters, proposed fix. For each fix, include a `diff` block showing the before/after lines (lines removed prefixed with `-`, lines added with `+`). Skip style/naming nitpicks. If there are no real concerns, say so.
