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

**Script file**: `.security-power/scan-scripts.json` — key = step_id, value = `{ "tool": "...", "scripts": ["..."] }`. Record scripts for every step, including skipped ones (write to file, do not run).

## Plan selection (before run)

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

1. Read **last selected plan** from `.security-power/last-plan.json` if it exists. Format: `{ "steps": ["compile", "secret-scan", ...] }` (order preserved). If missing or invalid, default = all 7 steps.
2. **Show user** the optional plans (example wording):
   - **当前默认（上次执行）**: 展示 last-plan 中的步骤对应序号，如 `1+2+3+4`；若无 last-plan 则默认 `1+2+3+4+5+6+7`（全部 7 步）。
   - **可选**: 全部 7 步、或子集如 1+2+3、2+3+4、1+3+5 等（按序号组合，保持顺序）。
3. **Ask user** to either:
   - **确认执行**: 回答「执行」「执行计划」「确认」或 "run" → 按**当前默认**（上次计划）执行。
   - **选择新计划**: 以序号组合回答，如 `1+2+3`、`1+2+3+4`（可用 `+` 或 `,`，无需空格）。解析为 step_id 有序列表，写入 `.security-power/last-plan.json`，再执行该计划。
4. When parsing `1+2+3`, map indices to step_id (1→compile, 2→secret-scan, …), keep order, then run only those steps (and record scripts for them; skipped steps still get tool+scripts written if not yet in scan-scripts.json).

## Run flow

0. **First**: If the project is under **git** (`.git` exists), add the report path to **`.gitignore`** so reports are not committed and privacy is protected. Add: `.security-power/.output/` (and optionally `.security-power/` if you want to ignore scripts/plan files too).
1. **Plan selection** (see above): read `.security-power/last-plan.json` as default; show optional plans; user confirms or replies with e.g. `1+2+3`; resolve to ordered step list and persist to `last-plan.json` when user chooses a new plan.
2. Read `.security-power/scan-scripts.json`. Run only **steps in the selected plan** (subset of the 7).
3. **Step has non-empty `scripts`** → run those scripts in the container (no tool choice).
4. **Step has no scripts** → call **scan-tool-choice** then **scan-script-record** to set `tool` and `scripts`, then run (or only write if step is skipped).
5. **Step skipped** (not in selected plan) → if no scripts yet, still set tool + scripts and write (do not run).
6. After run, **update** `.security-power/last-plan.json` with the steps that were actually executed (so next time this plan is the default).
7. **Review execution**: Review the execution process (logs, actual commands run, failures, or user corrections). If any step’s **scripts** differed from what is in `scan-scripts.json` (e.g. command was adjusted during run, or a better script was used), **update** `.security-power/scan-scripts.json` with the actual/improved scripts so the next run uses them.

Execution: start one container, copy code in, run each selected step’s `scripts` in order in that container; write reports to `.security-power/.output/`. On step failure: record and continue by default; finally summarize. **Review** (step 7) then hand off results to **executing-agent** for report and optional PR.

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
