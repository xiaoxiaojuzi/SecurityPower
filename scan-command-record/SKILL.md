---
name: scan-command-record
description: Persist and run per-step scripts in .security-power/scan-scripts.json. If step has scripts → run in Docker; else set tool+scripts with scan-tool-choice. Always record scripts (even when step skipped). Use with plan / scan-tool-choice.
---

# Scan script record

File: `.security-power/scan-scripts.json`. Format: `{ "<step_id>": { "tool": "...", "scripts": ["cmd1", "cmd2"] } }`. Step IDs: compile, secret-scan, dependency-scan, static-scan, package-scan, dynamic-scan, fuzzing.

**Rule**: All steps run in the **same** Docker container. Reports → `.security-power/.output/`. **Always persist** each step’s `tool` and `scripts` (even if step is skipped this run).

## Flow

1. Read file for step. **Has non-empty `scripts`** → run them in container (no scan-tool-choice).
2. **No scripts** → scan-tool-choice sets `tool`; this skill sets `scripts` (from tool), write to file, then run (or only write if skipped).
3. **Step skipped** → if no scripts yet, still set and write; if already have scripts, keep.

Run scripts in array order in container. On script failure: mark step failed; continue or stop per policy. Re-determine scripts when user changes tool or when recorded scripts fail.

## Example

```json
{
  "compile": { "tool": "npm", "scripts": ["npm ci", "npm run build"] },
  "secret-scan": { "tool": "gitleaks", "scripts": ["gitleaks detect --no-git"] },
  "dependency-scan": { "tool": "npm", "scripts": ["npm audit --audit-level=moderate"] }
}
```

## With plan / scan-tool-choice

Plan checks file first; only calls scan-tool-choice + this skill when step has no scripts. This skill: read → execute → persist. Tool choice is scan-tool-choice’s job.
