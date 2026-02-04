---
name: threat-modeling
description: Run threat modeling per threat-modeling-expert; scope = user-provided diff (show commits) or full code. Report to .security-power/.output/ with scope (git diff + commits). Use when plan includes threat-modeling step.
---

# Threat Modeling

Run threat modeling following [threat-modeling-expert](https://github.com/sickn33/antigravity-awesome-skills/tree/main/skills/threat-modeling-expert): STRIDE, PASTA, attack trees, data flow, risk prioritization, mitigations. **Scope**: either a **diff** (user must provide; show which commits are included) or **full codebase** if no diff.

**Output**: Threat modeling report under `.security-power/.output/` that **must** include a **Threat modeling scope** section with: (1) the scope description (diff vs full code), (2) **git diff** (when diff is used), (3) **commit records** (list of commits in scope; when diff is used, clearly list which commits the diff contains).

## Scope

- **With diff**: The **diff** (exact value) must be **manually provided by the user** (e.g. paste or path). **Explicitly tell the user** which commits are included — e.g. display "包含以下提交记录:" / "Scope includes the following commits:" then list each commit (hash + subject). Use that diff as the modeling scope.
- **Without diff**: If no diff is provided, use the **entire codebase** for modeling. State "Full codebase" in the report scope section and **write the last commit’s git hash** (e.g. `git rev-parse HEAD`) in the scope section so the codebase state is pinned.

## Execution (align with threat-modeling-expert)

1. Define scope and trust boundaries (from diff or full code).
2. Create data flow diagrams; identify assets and entry points.
3. Apply STRIDE to each component; build attack trees for critical paths.
4. Score and prioritize threats; design mitigations; document residual risks.
5. Write report to `.security-power/.output/threat-modeling-report.md` (or agreed path).

## Report format

Report **must** include at the top (or in a dedicated section):

- **Threat modeling scope**
  - **Scope type**: "Diff" or "Full codebase".
  - **Git diff** (when scope is diff): the full diff text or path; if summarized, link to full diff.
  - **Commit records** (when scope is diff): list of commits included, e.g. `commit_hash short_subject` per line. If user did not provide commit list, derive from diff or ask user to confirm.
  - **Last commit hash** (when scope is Full codebase): the git hash of the latest commit (e.g. `git rev-parse HEAD`). Must be written so the full-codebase scope is pinned to a specific revision.

Then: methodology findings (STRIDE, attack trees, mitigations, etc.) per threat-modeling-expert.

## Relations

- **Plan**: if selected plan includes `threat-modeling` step, call this skill (do not run threat-modeling inside scan-in-docker; this step is expert-driven, not script-in-container).
- **executing-agent**: may consume `.security-power/.output/threat-modeling-report.md`.

## Output

Report path: `.security-power/.output/threat-modeling-report.md` (or as configured). Report includes scope (git diff + commit records when diff; last commit hash when Full codebase) and full threat model.
