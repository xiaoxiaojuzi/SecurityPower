---
name: security-power-plan
description: Run SecurityPower scan plan; steps and scripts in .security-power/scan-scripts.json. Ensure scripts set (scan-tool-choice + scan-script-record if needed), then call scan-in-docker to run all scans in one container. Record scripts even for skipped steps. Use when user asks for security scan or SecurityPower.
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
| threat-modeling | threat model | threat-modeling-expert (see [threat-modeling](threat-modeling/SKILL.md); not in Docker) |

**Script file**: `.security-power/scan-scripts.json` — key = step_id, value = `{ "tool": "...", "scripts": ["..."] }`. Record scripts for every step, including skipped ones (write to file, do not run).

## Plan selection (before run)

**User interaction**: User **only inputs their choice** (options below). **Do not** ask the user to edit `last-plan.json` or `scan-scripts.json`; the **agent** parses user input and **writes** to those files.

**Step index** (for user choice):

| 序号 | step_id        | purpose        |
|------|----------------|----------------|
| 1    | compile        | build          |
| 2    | secret-scan    | secrets/keys   |
| 3    | dependency-scan| deps           |
| 4    | static-scan    | static analysis|
| 5    | package-scan   | packages/images|
| 6    | dynamic-scan   | runtime        |
| 7    | fuzzing        | fuzz           |
| 8    | threat-modeling| threat model   |

1. Read **last selected plan** from `.security-power/last-plan.json` if it exists. If missing or invalid, default = all 8 steps (1–8).
2. **Show user** the options: default = last-plan (e.g. `1+2+3+4`) or full `1+2+3+4+5+6+7+8`; they can pick a subset like `1+2+3`, `1+3+5+8`, etc. (order preserved).
3. **Ask user to enter one of**:
   - **Confirm**: reply "执行" / "执行计划" / "确认" / "run" → use current default plan.
   - **New plan**: reply with step indices, e.g. `1+2+3` or `1+2+3+4` (use `+` or `,`). **Agent** parses input → maps to step_id list → **writes** `.security-power/last-plan.json` → then runs that plan.
4. Agent: when user sends e.g. `1+2+3`, map indices to step_id (1→compile, …, 8→threat-modeling), **write** ordered steps to `last-plan.json`; for steps without scripts yet, agent (via scan-tool-choice / scan-script-record) **writes** to `scan-scripts.json`. User never edits these files.

## Run flow

0. **First**: If the project is under **git** (`.git` exists), add the report path to **`.gitignore`** so reports are not committed and privacy is protected. Add: `.security-power/.output/` (and optionally `.security-power/` if you want to ignore scripts/plan files too).
1. **Plan selection** (see above): read `last-plan.json` as default; show options; **user inputs choice only** (confirm or e.g. `1+2+3`); **agent** resolves and **writes** `last-plan.json` when user picks a new plan (user does not edit the file).
2. Read `scan-scripts.json`. For steps in the selected plan **except threat-modeling**: if a step has no `tool`/`scripts`, **ask user for options** (e.g. which tool); **agent** writes to `scan-scripts.json` via **scan-tool-choice** + **scan-script-record**. User does not edit the file. **threat-modeling**: skip script setup; not run in scan-in-docker.
3. **Execute steps**:
   - **threat-modeling** (if in selected plan): Call **threat-modeling** skill — scope = user-provided diff (show which commits are included) or full code; run per [threat-modeling-expert](https://github.com/sickn33/antigravity-awesome-skills/tree/main/skills/threat-modeling-expert); write report to `.security-power/.output/` with **Threat modeling scope** (git diff + commit records). Do not run threat-modeling inside Docker.
   - **All other steps**: Run **only** via **scan-in-docker**: start one container → copy code in → run each selected step’s `scripts` inside the container → copy reports out. Do **not** run `scripts` on the host; that does not use Docker. On step failure: record and continue; summarize at end.
4. **Agent** writes `last-plan.json` with the steps actually executed (next default). User does not edit.
5. **Review execution**: If scripts changed (e.g. fixed during run), **agent** updates `scan-scripts.json` with the improved scripts. User does not edit the file.

**Summary**: Plan selects steps and ensures scripts are set; **scan-in-docker** performs all scan tool execution in one container. **Review** (step 5) then hand off results to **executing-agent** for report and optional PR.

## Checklist

```
- [ ] compile
- [ ] secret-scan
- [ ] dependency-scan
- [ ] static-scan
- [ ] package-scan
- [ ] dynamic-scan
- [ ] fuzzing
- [ ] threat-modeling
```

## Output

Per-step pass/fail; report paths under `.security-power/.output/`; pass to executing-agent.
