---
name: threat-modeling
description: Threat model per threat-modeling-expert; scope = 1 (entire) | 2+hash | 3+start+end. Report to .security-power/.output/ with scan info (branch, SCOPE_START/END_COMMIT, SCOPE_END_BRANCH, commits). Use when plan includes threat-modeling.
---

# Threat Modeling

Follow [threat-modeling-expert](https://github.com/sickn33/antigravity-awesome-skills/tree/main/skills/threat-modeling-expert): STRIDE, PASTA, attack trees, data flow, risk prioritization, mitigations. Output: report under `.security-power/.output/` with **Scan info** and **Threat modeling scope**.

## Scope: ask user (format `choice+value` to avoid 1/2 vs hash confusion)

| Choice | Input | Resolve |
|--------|-------|---------|
| 1 | `1` or `1+entire` | start=root, end=**branch name + commit hash** (resolve HEAD; do **not** write "HEAD") |
| 2 | `2+<hash>` | start=end=hash |
| 3 | `3+<start_hash>+<end_hash>` | start/end = hashes; commits = `git log start..end` |

Prompt user with these examples. Parse input → resolve start/end/commits → use for modeling.

## Execution

Define scope/trust boundaries → data flow + assets/entry points → STRIDE + attack trees → prioritize threats → mitigations + residual risks. Write `.security-power/.output/threat-modeling-report.md`.

## Report format (required)

**Scan info** — one label per line (no "from X to Y" in one line):
- `Git branch: <name>`
- `SCOPE_START_COMMIT: <hash>`
- `SCOPE_END_COMMIT: <hash>` (Choice 1: use resolved branch’s tip hash, not "HEAD")
- `SCOPE_END_BRANCH: <branch>`
- `Git commits:` then list `hash subject` per line

**Threat modeling scope**: type ("Entire code" | "Single commit" | "Range") + scope-specific. Then STRIDE/attack trees/mitigations per threat-modeling-expert.

## Relations

Plan calls this skill when step=threat-modeling (not in scan-in-docker). executing-agent consumes the report.
