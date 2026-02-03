# SecurityPower

Security code workflow for coding agents: composable [Skills](https://cursor.com/docs/context/skills) and instructions so the agent runs scans correctly.

**Dependencies**: [trailofbits/skills](https://github.com/trailofbits/skills)

**Paths**:
- Scripts/plan: `.security-power/` (e.g. `scan-scripts.json`)
- Reports: `.security-power/.output/`

**Execution**: One Docker container; copy code in, run all steps there, pull reports out. Default steps: compile, secret-scan, dependency-scan, static-scan, package-scan, dynamic-scan, fuzzing. If a step has scripts in `scan-scripts.json` → run them; else choose tool and record scripts (even when step is skipped).

**Executing-agent**: After steps, produce report (issues + fix suggestions) under `.security-power/.output/`; create PR when fixes are code.
