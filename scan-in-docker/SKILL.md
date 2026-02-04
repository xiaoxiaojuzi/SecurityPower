---
name: scan-in-docker
description: Copy code into one Docker container; run all selected scan steps inside it; copy reports to .security-power/.output/. Use after plan.
---

# Scan in Docker

Run all scan tool execution in **one** Docker container. Input: selected steps (`.security-power/last-plan.json`), scripts (`.security-power/scan-scripts.json`). Output: per-step pass/fail; reports in `.security-power/.output/`.

## Paths

- **Host**: project root, `.security-power/`, `.security-power/.output/` (reports).
- **Container**: same layout (e.g. `/workspace` = project root); write reports to `.security-power/.output/` inside container, then copy out to host.

## Flow

1. **Start one container** (image configurable; keep running for all steps).
2. **Copy code + config** into container workdir (e.g. `/workspace`); include `.security-power/` (exclude `.security-power/.output/` if present).
3. **Run steps** in container: for each step in `last-plan.json` order, run its `scripts[]` from `scan-scripts.json` in array order; record stdout/stderr/exit; write reports to container `.security-power/.output/`. On step failure: record and continue; summarize at end.
4. **Copy reports** from container `.security-power/.output/` to host `.security-power/.output/`.
5. **Stop/remove** container (or keep per config).

## Relations

- **Plan**: calls this skill after steps and scripts are set.
- **scan-script-record**: persists scripts; this skill only runs them, does not edit `scan-scripts.json`.
- **executing-agent**: consumes host `.security-power/.output/`.

## Checklist

- [ ] One container for all selected steps
- [ ] Code + config copied in; reports copied out
- [ ] Steps run in last-plan order; scripts in array order; failures recorded, continue by default

## Output

Per-step pass/fail; report paths under `.security-power/.output/`; for plan review and executing-agent.
