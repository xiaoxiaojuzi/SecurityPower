# SecurityPower

Security code workflow for coding agents: composable [Skills](https://cursor.com/docs/context/skills) and instructions so the agent runs scans correctly.

**Dependencies**: [trailofbits/skills](https://github.com/trailofbits/skills)

**Paths**:
- Scripts/plan: `.security-power/` (e.g. `scan-scripts.json`)
- Reports: `.security-power/.output/`

**Execution**: **First**: if project is under git, add `.security-power/.output/` to `.gitignore` to avoid leaking report data. Then: plan (select steps, ensure scripts in `scan-scripts.json`) → **scan-in-docker** (copy code into one Docker container, run all selected scan steps there, copy reports to `.security-power/.output/`). Default steps: compile, secret-scan, dependency-scan, static-scan, package-scan, dynamic-scan, fuzzing. If a step has no scripts → choose tool and record scripts (even when step is skipped).

**Executing-agent**: After steps, produce report (issues + fix suggestions) under `.security-power/.output/`; create PR when fixes are code.
