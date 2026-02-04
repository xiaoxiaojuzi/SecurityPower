---
name: scan-in-docker
description: Copy code into one Docker container; run all selected scan steps inside it; copy reports to .security-power/.output/. Use after plan.
---

# Scan in Docker

**Required**: Scan scripts **MUST** run inside a Docker container. Do **not** run them on the host (e.g. `docker run` absent, or running `npm audit` / `gitleaks` / etc. in host shell = wrong). To comply: start container → copy code in → run scripts in container → copy reports out.

Run all scan tool execution in **one** Docker container. Input: selected steps (`.security-power/last-plan.json`), scripts (`.security-power/scan-scripts.json`). Output: per-step pass/fail; reports in `.security-power/.output/`.

## Paths

- **Host**: project root, `.security-power/`, `.security-power/.output/` (reports).
- **Container**: same layout (e.g. `/workspace` = project root); write reports to `.security-power/.output/` inside container, then copy out to host.

## Flow

Concretely: use **Docker CLI** — `docker run` (or `docker compose up`) to start one container; `docker cp` (or bind-mount) to copy project into container; `docker exec` to run each step’s scripts inside the container; `docker cp` to copy `.security-power/.output/` from container to host.

1. **Start one container** (e.g. `docker run -d --name security-power-scan <image>`; image configurable; keep running for all steps).
2. **Copy code + config** into container workdir (e.g. `docker cp . <container>:/workspace/`; include `.security-power/`, exclude `.security-power/.output/` if present).
3. **Run steps** in container: for each step in `last-plan.json` order, `docker exec` to run its `scripts[]` from `scan-scripts.json` in array order; record stdout/stderr/exit; reports go to container `.security-power/.output/`. On step failure: record and continue; summarize at end.
4. **Copy reports**: `docker cp <container>:/workspace/.security-power/.output/ .security-power/.output/`.
5. **Stop/remove** container (e.g. `docker stop` / `docker rm`) or keep per config.

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
