---
name: scan-tool-choice
description: Choose which tool to use per scan step. Only used when step has no scripts in scan-scripts.json; if scripts exist, plan runs them and skips this skill. Record scripts even for skipped steps. Use with plan / scan-script-record.
---

# Scan tool choice

**When**: Step has no (or empty) `scripts` in `.security-power/scan-scripts.json`. Then ask user which tool, write `tool` to file; scan-script-record fills `scripts`. If step is skipped but has no scripts, still set tool (and let scan-script-record set scripts) and write — do not run.

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

- **No record for step**: List candidates, ask user, write `tool` to `.security-power/scan-scripts.json` (scripts `[]` until scan-script-record fills).
- **Has record**: Ask “Use same tool [tool]?” — yes → keep; no → ask again, update `tool`.
- Re-ask when user says “change tool” or tool run failed.

This skill only sets `tool`; scan-script-record sets `scripts` and runs.
