---
name: security-power
description: SecurityPower entry. Plan → tool choice → script record → run in one Docker container → executing-agent report/PR. Use when user asks for security scan or SecurityPower.
---

# SecurityPower

Security code workflow for agents. [Trail of Bits Skills](https://github.com/trailofbits/skills) (static-analysis, insecure-defaults, testing-handbook-skills, etc.).

**Flow**: **First** — if repo is git-managed, add `.security-power/.output/` to `.gitignore` (avoid leaking report data). Then: plan (steps, scripts in `.security-power/scan-scripts.json`) → scan-tool-choice (if no scripts) → scan-script-record (persist scripts) → **scan-in-docker** (copy code to one container, run all scans there, copy reports out) → executing-agent (report + optional PR).

| Skill | Role |
|-------|------|
| [plan](plan/SKILL.md) | Default 7 steps; read/write scan-scripts.json; ensure scripts set; call scan-in-docker |
| [scan-tool-choice](scan-tool-choice/SKILL.md) | Pick tool per step when step has no scripts |
| [scan-script-record](scan-script-record/SKILL.md) | Persist scripts; record even when step skipped |
| [scan-in-docker](scan-in-docker/SKILL.md) | Copy code to one Docker container; run all selected scan steps inside it; copy reports to .output/ |
| [executing-agent](executing-agent/SKILL.md) | Report + fix suggestions; PR when code fix |

**Done**: All chosen steps pass and executing-agent produced report (and optional PR).
