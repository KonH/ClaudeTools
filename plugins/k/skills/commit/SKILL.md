---
name: commit
description: Create a git commit for the staged changes, branching off the default branch first if needed
---

Create a git commit for the staged changes.

## Pre-commit step: branch selection

Before committing, check the current branch:
1. `git branch --show-current`
2. Determine the repository's default branch (`main` or `master` — e.g. via `git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to whichever of `main`/`master` exists locally).
3. If the current branch is the default branch: create and switch to a new branch via `git checkout -b feature/<short-kebab-description>`, where `<short-kebab-description>` is a meaningful slug derived from the change being committed (e.g. `feature/province-division-config`). Do this before staging/committing anything else.
4. If it is anything other than the default branch (i.e. already on a feature branch or other non-default branch): leave the branch as-is — do not create or switch branches.

## Rules
- Subject line: short, imperative, no period
- Explain *why*, not *what* — the diff already shows what changed
- No bullet-point summaries of changed files
- Always add a `Co-Authored-By` trailer for the model in use, e.g. `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`
