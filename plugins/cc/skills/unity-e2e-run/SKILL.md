---
name: unity-e2e-run
description: Drive a live Unity Editor play session through a filesystem request/response handshake — start a run, poll the report, submit interactive steps, and read evidence. Use when verifying or debugging a game change against the running editor without a human reproducing the flow.
---

# Unity E2E run

Drive the Unity Editor in Play mode through a **file handshake**. Do not wait for a live MCP connection. Do not ask the developer for confirmation before starting a run.

## Protocol

Write a JSON `RunRequest` into the project's request drop folder (the project wrapper names the path). The Editor watcher picks it up, enters Play mode, executes the script, and writes `report.json` / `report.md` plus per-step evidence under that run's folder.

`protocolVersion` must match the runner. On mismatch the watcher writes `outcome: "protocol-mismatch"` naming both versions and runs nothing.

### RunRequest fields

- `protocolVersion` (int, required)
- `runId` (string; defaulted from the filename if omitted)
- `script` (string: committed flow name, or a scratch script name)
- `initiator` (string; default `"agent"`)
- `mode` (`"scripted"` default, or `"interactive"`)
- `inputs` (object of placeholder values, e.g. org / save)
- `failurePolicy` (`"stopOnFirstFailure"` default, or `"continueOnFailure"`)
- `consoleErrors` (`"report"` default, or `"strict"`)
- `timeoutSeconds` (whole-run cap; default 300)
- `idleTimeoutSeconds` (interactive idle; default 120)
- `settings.tutorialsEnabled` (bool; default false in the isolated root)

### Run folder layout

```
<e2e-root>/
  requests/<runId>.json
  current_run.json
  runs/<runId>/
    request.json
    editor_state.json
    report.json
    report.md
    steps/<nnn>_screenshot.png
    steps/<nnn>_console.log
    steps/<nnn>_state.json
    inbox/step_<n>.json
    outbox/step_<n>.json
  scratch/<name>.json
```

## Starting a run

1. Confirm the Editor is open on the project. A run **refuses** if Play mode is already active or an open scene is dirty — that is correct; do not interrupt a human session.
2. Write the `RunRequest` to `requests/<runId>.json`. Starting it needs no confirmation.
3. Poll `runs/<runId>/report.json` until `outcome` is a terminal value (`succeeded`, `failed`, `interrupted`, `abandoned`, `crashed`, `protocol-mismatch`, `refused`) rather than `running` or missing.
4. Read `report.md` for the human summary. Console errors are quoted in the summary.

A refusal report means a session is in progress, a scene is dirty, or a lock exists. Do not retry by killing Play mode.

## Interactive steps

With `mode: "interactive"`, write `inbox/step_<n>.json` (a step object, or `{"kind":"end"}` / `{"kind":"capture"}`). Read `outbox/step_<n>.json` and the matching evidence files. The session stays on the state that step left. After `idleTimeoutSeconds` with no inbox file the run is `abandoned`.

## Rules

- **Do not edit any source file while a run is in progress.** A domain reload kills the run and reports `interrupted (assembly reload)`.
- A hard main-thread hang freezes both the host and the Editor watchdog. The developer must kill the Editor; on the next load, stale-lock recovery finalizes the run as `crashed`. Do not promise a watchdog that can break a frozen main thread.
- Leave the Editor as you found it: the runner restores the previous scene and input settings and deletes the run's isolated saves. Do not clean up by hand unless a crash left Play mode running.

## Reading outcomes

- `Reached map` in the summary tells you whether the session arrived on the map.
- Each completed step has a screenshot, console log, and state snapshot. Missing evidence on a completed step is a runner bug.
- `inputPath: "panel"` on a step means the device-level click path could not address the target and a lower-fidelity panel dispatch was used.
