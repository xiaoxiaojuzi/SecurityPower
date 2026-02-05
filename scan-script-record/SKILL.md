---
name: scan-script-record
description: Persist and run per-step scripts in .security-power/scan-scripts.json. If step has scripts → run in Docker; else set tool+scripts with scan-tool-choice. Always record scripts (even when step skipped). Use with plan / scan-tool-choice.
---

# Scan script record

File: `.security-power/scan-scripts.json`. Format: `{ "<step_id>": { "tool": "...", "scripts": ["cmd1", "cmd2"] } }`. Step IDs: compile, secret-scan, dependency-scan, static-scan, package-scan, dynamic-scan, fuzzing.

**Rule**: All steps run in the **same** Docker container. Reports → `.security-power/.output/`. **Agent** always persists each step’s `tool` and `scripts` to `.security-power/scan-scripts.json` (even if step skipped). User does **not** edit this file; user only provides choices when asked (e.g. tool choice via scan-tool-choice).

## Flow

1. Read file for step. **Has non-empty `scripts`** → run them in container (no scan-tool-choice).
2. **No scripts** → scan-tool-choice sets `tool`; this skill sets `scripts` (from tool), write to file, then run (or only write if skipped).
3. **Step skipped** → if no scripts yet, still set and write; if already have scripts, keep.

Run scripts in array order in container. On script failure: mark step failed; continue or stop per policy. Re-determine scripts when user changes tool or when recorded scripts fail.

**After execution (review)**: When plan finishes a run, review the execution process (logs, actual commands run, failures, user corrections). If any step’s scripts **changed** compared to `scan-scripts.json` (e.g. command was fixed during run, or a different/better script was used), **agent** updates `.security-power/scan-scripts.json` with the improved `scripts`/`tool`. User does not edit the file.

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
