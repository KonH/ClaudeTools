# ClaudeTools

Claude Code plugin marketplace with skills shared across projects.

## Contents

- **`k` plugin** — common skills, invoked with the `k:` prefix:
  - `/k:commit` — commit staged changes, creating a `feature/*` branch first when on the default branch (`main`/`master`)

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
