---
name: scan-tool-choice
description: 为每类扫描步骤确定使用的扫描工具。仅当某步骤在脚本文件中尚无脚本时调用；若该步骤已有脚本则直接执行，不调用本技能。每类可配置一个或多个候选工具；首次使用该步骤时询问用户选用哪个并记录，再次运行时询问是否采用上次使用的工具。当执行 SecurityPower 计划、安全扫描步骤或需要「选择/确认扫描工具」时使用。
---

# 扫描工具选择

每类扫描步骤可配置**一个或多个扫描工具**。本技能负责在项目内确定「每个步骤使用哪个工具」：首次使用该步骤时询问用户选用哪个工具并记录，再次运行时询问用户是否采用上次使用的工具。

**仅当某步骤尚无脚本时使用**：每次运行 plan 时，若某步骤在 `.security-power/scan-scripts.json` 中**已有非空 scripts**，则**直接执行**该步骤，不调用本技能；仅当某步骤**没有脚本**时，才调用本技能确定工具并写入 `tool`，再由 scan-command-record 确定并写入 `scripts` 后执行。若某步骤当次被 skip 但尚无脚本，仍调用本技能与 scan-command-record 确定并写入（不执行）。

## 每类扫描可配置的候选工具

每一类扫描步骤对应**一个或多个可配置的扫描工具**；执行前需在候选工具中确定本次使用的工具。

| 步骤 ID | 说明 | 候选工具（可配置一个或多个） |
|---------|------|------------------------------|
| compile | 编译/构建 | npm、yarn、pnpm、cargo、make、cmake、go 等（按项目类型） |
| secret-scan | 密码/密钥泄露 | gitleaks、trufflehog、detect-secrets 等 |
| dependency-scan | 依赖漏洞 | npm audit、yarn audit、pip audit、cargo audit、trivy、OSV 等 |
| static-scan | 静态分析 | semgrep、codeql、SARIF 等 |
| package-scan | 包/制品扫描 | trivy、syft、grype 等 |
| dynamic-scan | 动态分析 | 按项目类型配置 DAST 等 |
| fuzzing | 模糊测试 | libFuzzer、AFL、go-fuzz、Jazzer 等 |

具体候选列表可根据项目或团队在配置中扩展；上表为常见参考。

## 存储位置与约定

- **存储位置**：项目根目录下的 **`.security-power/`** 目录；脚本与计划相关文件（如 `scan-scripts.json`）均放在此目录（若目录不存在则创建）。报告文件不放在此目录，见 plan 技能中 `.security-power/.output/` 约定。
- **约定**：本技能只负责确定并写入/更新每步的 **tool** 字段；同一文件中的 **scripts** 由 **scan-command-record** 技能管理。每个步骤的格式为 `{ "tool": "选中的工具名", "scripts": [...] }`。扫描实际在**同一个 Docker 容器内**执行（与 plan 其他步骤共用同一容器），本技能只负责工具选择与记录。

## 使用流程

1. **执行某步前（是否已有该步骤的记录）**

   - **无记录（项目首次使用该步骤）**
     - **询问用户**：列出该步骤的候选工具（见上表或项目配置），询问用户「需要使用哪个工具？」。
     - 用户选择后，将**所选 tool** 写入 `.security-power/scan-scripts.json` 中该步骤的 `tool` 字段（`scripts` 可先为空数组 `[]`，由 scan-command-record 后续填充）。
     - 输出本次选定的工具名，供后续 scan-command-record 使用。
   - **有记录（非首次）**
     - **询问用户**：「是否采用上次使用的工具 [记录中的 tool]？」。
     - 若用户选择**采用**：不修改文件，输出记录中的 tool，供后续使用。
     - 若用户选择**不采用**：再次列出该步骤的候选工具，询问用户选用哪个，更新 `scan-scripts.json` 中该步骤的 `tool` 字段（必要时清空或由 scan-command-record 后续更新 `scripts`），输出新选定的工具名。

2. **何时重新选工具**
   - 用户明确要求「更换扫描工具」时，按「有记录但不采用」流程重新询问并更新 `tool`。
   - 记录中的工具执行失败且需换方案时，可再次询问用户是否更换工具并更新记录。

## 与 plan、scan-command-record 的配合

- **每次运行 plan 时**：对每个要执行的步骤，**先查脚本文件**。若该步骤已有非空 `scripts`，则**直接执行**，不调用本技能；若该步骤**没有脚本**，才先按本技能（**scan-tool-choice**）确定工具并写入 `tool`，再按 **scan-command-record** 确定并写入 `scripts` 后执行。若某步骤当次被 skip 但尚无脚本，仍调用本技能与 scan-command-record 确定并写入（不执行）。
- 本技能只负责「选哪个工具」；脚本集合的生成、执行与持久化由 scan-command-record 负责。
