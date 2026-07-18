# ClaudeTools

Claude Code plugin marketplace with skills shared across projects.

## Contents

- **`k` plugin** — common skills, invoked with the `k:` prefix:
  - `/k:commit` — commit staged changes, creating a `feature/*` branch first when on the default branch (`main`/`master`)
  - `/k:pr` — create a pull request for the current branch
  - `/k:specify` — capture feature intent and acceptance criteria as `docs/specs/<YY_MM_DD_HH>_<name>/spec.md`, stops for approval
  - `/k:plan` — turn an approved spec (or a purely technical task) into `docs/specs/<YY_MM_DD_HH>_<name>/plan.md`, gated by a required `docs/constitution.md` check
  - `/k:plan-review` — independent review of the latest plan for missing steps, wrong assumptions, or guideline violations
  - `/k:implement` — implement a plan phase by phase, test-first, marking off completed steps

  The `specify → plan → plan-review → implement` skills assume a `docs/specs/` and `docs/constitution.md` layout by default; a project can override these paths via its own `CLAUDE.md`. Projects with extra project-specific steps (engine/tool connection checks, domain-specific rule files, etc.) keep their own thin command that performs those steps and then delegates to the shared skill — the same pattern used for `/commit` below.

## Usage

One-off install (user scope, available in all projects):

```
/plugin marketplace add KonH/ClaudeTools
/plugin install k@claude-tools
```

Or reference it from a project's `.claude/settings.json` so it is offered automatically to anyone opening the repo:

```json
{
  "extraKnownMarketplaces": {
    "claude-tools": {
      "source": {
        "source": "github",
        "repo": "KonH/ClaudeTools"
      }
    }
  },
  "enabledPlugins": {
    "k@claude-tools": true
  }
}
```

Projects that need extra pre-commit steps (version bumps, etc.) keep their own `/commit` command that performs the project-specific steps and then delegates to `k:commit`.

## Versioning

The plugin declares no `version`, so every commit to this repo counts as a new version — consumers pick up changes via `/plugin marketplace update claude-tools` (or marketplace auto-update).
