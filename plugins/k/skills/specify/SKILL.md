---
name: specify
description: Capture feature intent and acceptance criteria before planning begins, writing docs/specs/<index>_<name>/spec.md. Stops for user approval before the plan skill runs.
---

Capture feature intent and acceptance criteria before planning begins. Writes `docs/specs/<index>_<name>/spec.md` and stops — the user must approve the spec before the `plan` skill runs.

## Doc paths

Default convention: specs live under `docs/specs/`. Check the project's `CLAUDE.md` for an overriding location (e.g. a "Docs Layout" section or explicit mention of spec paths) before falling back to the default.

## Index Derivation

1. Use `Glob` with pattern `docs/specs/*/*.md` to list existing spec folders — do NOT use a trailing-slash pattern (`docs/specs/*/`) or a bare `docs/specs/*`, both silently return nothing on Windows; the nested `*/*.md` form matches files one level down and works
2. Extract the leading numeric prefix from each folder segment in the returned paths
3. Take the highest number found and add 1
4. Zero-pad to two digits (e.g. `35`)

## Orchestration

Spawn an **architect sub-agent** (general-purpose) briefed with:
- The user's feature description
- Relevant project rules from `CLAUDE.md` and any project rules directory
- The output path: `docs/specs/<index>_<name>/spec.md`
- The spec format below

The architect writes the spec file directly. You (orchestrator) then:
1. Present the spec contents to the user
2. Collect feedback and re-brief the architect if changes are needed (iterate until the user approves)
3. **Stop.** Do not run the `plan` skill or write any code. The user must explicitly request the next step.

## Spec Format

```markdown
# Spec: <Feature Name>

## Feature Intent

As a <role>, I want <capability>, so that <benefit>.

## Acceptance Criteria

- **Given** <precondition> **When** <action> **Then** <outcome>
- (one bullet per observable behaviour; cover the happy path and the most important edge cases)

## Out of Scope

- (explicit exclusions — things the feature deliberately will not do)

## Ambiguities

- [NEEDS CLARIFICATION: <question the architect cannot resolve from context alone>]
- (omit this section entirely if there are no ambiguities)
```

## Rules

- Do NOT write any plan, code, or assets — only the spec document.
- Use `[NEEDS CLARIFICATION: …]` markers freely — surfacing unknowns early is the point.
- The spec folder name uses kebab-case: `docs/specs/32_my-feature/spec.md`.
- Do not create `plan.md` in the spec folder — that is the `plan` skill's job.
