---
name: specify
description: Capture feature intent and acceptance criteria in a dated docs/specs folder, stopping for approval before planning.
---

Capture feature intent and acceptance criteria before planning begins. Writes `docs/specs/<YY_MM_DD_HH>_<name>/spec.md` and stops — the user must approve the spec before the `plan` skill runs.

## Doc paths

Default convention: specs live under `docs/specs/`. Check the project's `CLAUDE.md` for an overriding location (e.g. a "Docs Layout" section or explicit mention of spec paths) before falling back to the default.

## Spec Identifier Derivation

1. Use `Glob` with pattern `docs/specs/*/*.md` to list existing spec folders — do NOT use a trailing-slash pattern (`docs/specs/*/`) or a bare `docs/specs/*`, both silently return nothing on Windows; the nested `*/*.md` form matches files one level down and works
2. Generate the current local timestamp as `YY_MM_DD_HH` (for example, `26_07_18_14`)
3. Combine it with a short kebab-case feature name: `<YY_MM_DD_HH>_<name>`
4. Do not derive or assign a numeric index

## Orchestration

Spawn an **architect sub-agent** (general-purpose) briefed with:
- The user's feature description
- Relevant project rules from `CLAUDE.md` and any project rules directory
- The output path: `docs/specs/<YY_MM_DD_HH>_<name>/spec.md`
- The spec format below

The architect writes the spec file directly. You (orchestrator) then:
1. **Resolve ambiguities interactively.** Extract every `[NEEDS CLARIFICATION: ...]` marker from the written spec. If there are any:
   - Assign each a priority tier — Scope > Security > UX > Tech (highest impact first) — and sort by tier, preserving original order within a tier. Surface every marker; do not cap how many get asked.
   - Number them 1-based in that order (Q1, Q2, Q3, …).
   - For each, draft 2-4 concrete candidate answers reflecting reasonable interpretations.
   - Ask them with the `AskUserQuestion` tool, one question per marker, setting each question's `header` to its priority tier (`Scope`/`Security`/`UX`/`Tech`) so the ordering is visible. Batch across multiple calls if there are more than 4 (that tool caps at 4 questions per call).
   - Re-brief the architect with the resolved answers so it updates the spec, replacing each marker with the chosen (or custom) answer.
2. Present the (now unambiguous) spec contents to the user for approval.
3. Collect feedback and re-brief the architect if changes are needed (iterate until the user approves).
4. **Stop.** Do not run the `plan` skill or write any code. The user must explicitly request the next step.

## Spec Format

```markdown
# Spec: <Feature Name>

## Feature Intent

As a <role>, I want <capability>, so that <benefit>.

## Acceptance Criteria

Legend: `Precondition => Action => Outcome`, grouped under a shared precondition where one applies to several rows.

- <Precondition or scenario shared by the rows below>
  - <action> => <outcome>
  - <action> => <outcome>
- <Next precondition or scenario group>
  - <action> => <outcome>
- (cover the happy path and the most important edge cases; a group can hold a single row if nothing else shares its precondition)

## Success Criteria

Measurable, technology-agnostic outcomes that prove the feature works — no frameworks, APIs, or tools. Mix quantitative (time, rate, volume) and qualitative (satisfaction, completion) measures where they apply.

- <Measurable outcome, e.g. "Player completes a turn in under 10 seconds">
- <Measurable outcome, e.g. "90% of new players successfully build their first structure without help">

## Out of Scope

- (explicit exclusions — things the feature deliberately will not do)

## Ambiguities

- [NEEDS CLARIFICATION: <question the architect cannot resolve from context alone>]
- (omit this section entirely if there are no ambiguities — it should always be empty by the time the spec reaches the user, since Orchestration resolves every marker first)
```

## Rules

- Do NOT write any plan, code, or assets — only the spec document.
- **The spec stays implementation-free, end to end** — no file names, classes, methods, commands, tech stack, or APIs anywhere in the document, Success Criteria included. Anything technical belongs in the `plan` skill's plan.md (its Technical Mapping section), never here.
- **Acceptance Criteria stays in plain product language** — describe what the player/user does and sees (e.g. "Player clicks a province"), never which class, method, or command fires. Group rows that share a precondition under one bullet instead of repeating it per row — this is what keeps the section skimmable instead of a wall of near-duplicate lines.
- **Success Criteria stays measurable and technology-agnostic** — a metric a non-technical stakeholder could verify (time, rate, count, satisfaction), never an implementation detail like an API's response time or a database's throughput.
- Use `[NEEDS CLARIFICATION: …]` markers freely — surfacing unknowns early is the point. Resolve every one via the interactive step in Orchestration before asking the user to approve the spec.
- The spec folder name starts with its creation timestamp and uses a kebab-case name: `docs/specs/26_07_18_14_my-feature/spec.md`.
- Do not create `plan.md` in the spec folder — that is the `plan` skill's job.
