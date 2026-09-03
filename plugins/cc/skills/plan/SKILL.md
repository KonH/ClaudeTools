---
name: plan
description: Create a dated implementation plan under docs/specs, gated by a required project-constitution check.
---

Create a plan for the requested task and save it to `docs/specs/<YY_MM_DD_HH>_<name>/plan.md`.

## Doc paths

Default convention: `docs/specs/` for spec/plan folders, `docs/constitution.md` for project principles. Check the project's `CLAUDE.md` for overriding locations before falling back to these defaults.

## Constitution Gate (required)

A plan may not be finalized without a constitution. Before anything else:
1. Check whether `docs/constitution.md` exists.
2. If it does not: stop, tell the user no constitution was found, and ask whether they want one created now — and what principles it should capture. Do not invent principles on their behalf. Only continue once a constitution exists (pre-existing or just created with their input).

Once a constitution exists, the architect sub-agent reads it before finalizing the plan and appends a **Constitution Check** section to the plan:
- Either: `No conflicts found — plan aligns with all principles.`
- Or: a numbered list of principles the plan would violate, each with a one-sentence proposed resolution

The orchestrator surfaces any violations to the user before presenting the final plan. If there are violations, the user must confirm each resolution before the plan is written. Do not write the plan file if unresolved violations exist.

## Orchestration

Spawn an **architect sub-agent** (general-purpose) to design and write the plan. Brief it with:
- The user's task description
- Relevant project rules (from `CLAUDE.md` and any project rules directory)
- The contents of `docs/constitution.md`
- Existing spec/plan folders in `docs/specs/` (list them to resolve an existing dated spec identifier)
- The output path and format rules below

The architect writes the plan file directly. You (orchestrator) then:
1. Present the plan contents to the user
2. Collect feedback and re-brief the architect if changes are needed (iterate until approved)
3. Run the `plan-review` skill as a final check — present any concerns one by one and ask the user to approve each fix
4. Stop and wait for the user to run the `implement` skill

## Spec Detection

If `$ARGUMENTS` contains a spec folder name or index, or if a `docs/specs/<index>_*` folder exists with no `plan.md` yet:
- Read `docs/specs/<YY_MM_DD_HH>_<name>/spec.md`
- Brief the architect with the spec contents
- Include a **Spec** section at the top of the plan (verbatim summary of intent and acceptance criteria from the spec)
- Include a **Technical Mapping** section (see Plan File Rules) — this is where the technical anchors (files, classes, methods, commands, state paths) that the spec deliberately leaves out now live

If no spec folder is found (a purely technical task — migration, refactor, infra), create `docs/specs/<YY_MM_DD_HH>_<name>/plan.md` using the current local timestamp, just without a `spec.md` in that folder.

## Plan File Rules

- The containing spec folder uses `YY_MM_DD_HH_name`, where the timestamp is the local creation time and `name` is a short kebab-case description (e.g. `26_07_18_14_map-prototype`)
- Structure: goal, approach, Technical Mapping (when a spec exists), steps — keep it concise
- **When a spec exists**, include a **Technical Mapping** section mapping each Acceptance Criteria bullet/group from the spec to its concrete implementation:

  ```markdown
  ## Technical Mapping

  Maps each Acceptance Criteria bullet/group from the spec to its concrete implementation.

  - <Acceptance-criteria bullet or group, restated briefly>:
    - <implementation detail: specific files, classes, methods, commands, state paths>
  ```

  One entry per Acceptance Criteria bullet/group that needs one; omit entries for purely product-level bullets that need no technical grounding. This is the only place implementation anchors are tied to spec language — the spec itself stays technology-agnostic.
- If the plan touches source code, include a **Tests** section covering what unit/integration tests should be added or updated
- Do NOT make any code, asset, or file changes — only write the plan document
- End every plan with the line: `Use the implement skill to start working on the plan or request changes.`

## Step Block Structure

Split steps into two sections:

**Section 1 — Agent Steps** (Claude performs autonomously via file edits and tools):
- Use markdown checkboxes: `- [ ] **Step title** — concise description`

**Section 2 — User Steps** (requires manual interaction Claude cannot perform — an external editor/tool, visual inspection, hardware, etc.):
- Use numbered headings (`### N. Title`) with body text — no checkboxes
