---
name: scan-tool-choice
description: Choose which tool to use per scan step. Only used when step has no scripts in scan-scripts.json; if scripts exist, plan runs them and skips this skill. Record scripts even for skipped steps. Use with plan / scan-script-record.
---

# Scan tool choice

**When**: Step has no (or empty) `scripts` in `.security-power/scan-scripts.json`. **Ask user to choose** a tool (present options); **agent** writes `tool` to file. User **only inputs their choice**; user does **not** edit `scan-scripts.json`. scan-script-record then fills `scripts` and agent writes. If step skipped but no scripts, agent still sets tool and writes — do not run.

## Per-step candidate tools

| step_id | candidates |
|---------|-------------|
| compile | npm, yarn, pnpm, cargo, make, cmake, go |
| secret-scan | gitleaks, trufflehog, detect-secrets |
| dependency-scan | npm/yarn/pip/cargo audit, trivy, OSV |
| static-scan | semgrep, codeql, SARIF |
| package-scan | trivy, syft, grype |
| dynamic-scan | DAST (per project) |
| fuzzing | libFuzzer, AFL, go-fuzz, Jazzer |

## Flow

- **No record for step**: List candidates, **ask user to pick one** (user inputs choice only). **Agent** writes `tool` to `.security-power/scan-scripts.json`. User does not edit the file.
- **Has record**: Ask “Use same tool [tool]?” — user replies yes/no; if no, agent updates `tool` in file.
- Re-ask when user says “change tool” or tool run failed.

Agent writes `tool`; scan-script-record writes `scripts` and runs. User never edits scan-scripts.json.
