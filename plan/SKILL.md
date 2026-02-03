---
name: security-power-plan
description: Run SecurityPower scan plan in one Docker container. Steps and scripts in .security-power/scan-scripts.json; if scripts exist run them, else choose tool and record. Record scripts even for skipped steps. Use when user asks for security scan or SecurityPower.
---

# SecurityPower Plan

Run all scan steps inside **one Docker container**. Paths: scripts/plan → `.security-power/` (create if missing); reports → `.security-power/.output/`.

## Default steps (step ID → tools reference)

| step_id | purpose | tools |
|---------|---------|--------|
| compile | build | npm, cargo, make, cmake, go |
| secret-scan | secrets/keys | gitleaks, trufflehog, detect-secrets |
| dependency-scan | deps | npm audit, pip audit, cargo audit, trivy, OSV |
| static-scan | static analysis | semgrep, codeql, SARIF |
| package-scan | packages/images | trivy, syft, grype |
| dynamic-scan | runtime | DAST per project |
| fuzzing | fuzz | libFuzzer, AFL, go-fuzz, Jazzer |

Default: run all 7. User may choose a subset (keep order). **Script file**: `.security-power/scan-scripts.json` — key = step_id, value = `{ "tool": "...", "scripts": ["..."] }`. Record scripts for every step, including skipped ones (write to file, do not run).

## Run flow

0. **First**: If the project is under **git** (`.git` exists), add the report path to **`.gitignore`** so reports are not committed and privacy is protected. Add: `.security-power/.output/` (and optionally `.security-power/` if you want to ignore scripts/plan files too).
1. Read `.security-power/scan-scripts.json`.
2. **Step has non-empty `scripts`** → run those scripts in the container (no tool choice).
3. **Step has no scripts** → call **scan-tool-choice** then **scan-script-record** to set `tool` and `scripts`, then run (or only write if step is skipped).
4. **Step skipped** → if no scripts yet, still set tool + scripts and write (do not run).

Execution: start one container, copy code in, run each step’s `scripts` in order in that container; write reports to `.security-power/.output/`. On step failure: record and continue by default; finally summarize. Hand off results to **executing-agent** for report and optional PR.

## Checklist

```
- [ ] compile
- [ ] secret-scan
- [ ] dependency-scan
- [ ] static-scan
- [ ] package-scan
- [ ] dynamic-scan
- [ ] fuzzing
```

## Output

Per-step pass/fail; report paths under `.security-power/.output/`; pass to executing-agent.
