---
name: implement
description: Implement the plan at docs/specs/<index>_<name>/plan.md, spawning developer sub-agents per phase, writing tests test-first, and marking off completed steps.
---

Implement the plan at `docs/specs/<index>_<name>/plan.md`.

## Plan Discovery

**With `$ARGUMENTS`:** resolve against `docs/specs/` — look for `docs/specs/<arg>/plan.md` or a folder whose name starts with `<arg>`.

**Without `$ARGUMENTS`:**
1. List all `*/plan.md` files under `docs/specs/`
2. Extract the leading numeric prefix from each
3. Use the file with the highest numeric prefix

## Orchestration

Read the plan file first. Then decide based on plan size:

- **1–2 steps or trivial changes:** implement inline (no sub-agent needed)
- **3+ steps, or steps spanning distinct areas of the codebase:** spawn a **developer sub-agent** per major phase

When spawning a developer sub-agent, brief it with:
- The full plan text
- Which step(s) it is responsible for
- Relevant project rules (`CLAUDE.md`, any project rules directory) — code style and any domain-specific conventions
- Current file state / context it needs (read key files first and include excerpts)
- Whether any external tool the plan depends on (an editor, engine, or service with its own connection state) is available and connected
- For steps touching source code: write the test for the new behavior first so it fails against the current code, then implement until it passes — never disable or weaken an existing test to force a pass, fix the underlying code instead

After each sub-agent phase:
1. Relay its results and any errors to the user
2. Verify the work (check for errors, read changed files) before moving to the next phase
3. If a phase fails, diagnose and re-brief the sub-agent with the fix context — do not skip ahead
4. Mark each completed agent step in the plan file by changing `- [ ]` to `- [x]`

## Pre-flight Checks

- Follow all project standards in `CLAUDE.md` and any project rules directory
- If the plan depends on an external tool or editor with its own connection/session state, verify that connection is live before starting. If it isn't, stop and ask the user to establish it first.

## Completion

After all steps are done:
- If any changes touch source code: confirm tests were written test-first (red before green) for the affected logic and all pass
- Run the `code-review` skill (or `/code-review` if unavailable as a skill) on the changed files — present any concerns one by one and ask the user to approve each fix before applying it
