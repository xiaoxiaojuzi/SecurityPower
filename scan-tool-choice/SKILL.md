---
name: scan-tool-choice
description: 为每类扫描步骤确定使用的扫描工具。每类可配置一个或多个候选工具；项目首次使用某步骤时询问用户选用哪个工具并记录，再次运行时询问是否采用上次使用的工具。当执行 SecurityPower 计划、安全扫描步骤或需要「选择/确认扫描工具」时使用。
---

# 扫描工具选择

每类扫描步骤可配置**一个或多个扫描工具**。本技能负责在项目内确定「每个步骤使用哪个工具」：首次使用该步骤时询问用户选用哪个工具并记录，再次运行时询问用户是否采用上次使用的工具。

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

- **存储位置**：项目根目录下的 `.security-power/scan-commands.json`（若目录不存在则创建）。
- **约定**：本技能只负责确定并写入/更新每步的 **tool** 字段；同一文件中的 **commands** 由 **scan-command-record** 技能管理。每个步骤的格式为 `{ "tool": "选中的工具名", "commands": [...] }`。

## 使用流程

1. **执行某步前（是否已有该步骤的记录）**

   - **无记录（项目首次使用该步骤）**
     - **询问用户**：列出该步骤的候选工具（见上表或项目配置），询问用户「需要使用哪个工具？」。
     - 用户选择后，将**所选 tool** 写入 `.security-power/scan-commands.json` 中该步骤的 `tool` 字段（`commands` 可先为空数组 `[]`，由 scan-command-record 后续填充）。
     - 输出本次选定的工具名，供后续 scan-command-record 使用。
   - **有记录（非首次）**
     - **询问用户**：「是否采用上次使用的工具 [记录中的 tool]？」。
     - 若用户选择**采用**：不修改文件，输出记录中的 tool，供后续使用。
     - 若用户选择**不采用**：再次列出该步骤的候选工具，询问用户选用哪个，更新 `scan-commands.json` 中该步骤的 `tool` 字段（必要时清空或由 scan-command-record 后续更新 `commands`），输出新选定的工具名。

2. **何时重新选工具**
   - 用户明确要求「更换扫描工具」时，按「有记录但不采用」流程重新询问并更新 `tool`。
   - 记录中的工具执行失败且需换方案时，可再次询问用户是否更换工具并更新记录。

## 与 plan、scan-command-record 的配合

- **security-power-plan** 执行某步骤时：先按本技能（**scan-tool-choice**）确定该步骤使用的工具（询问用户并写入/更新 `tool`），再按 **scan-command-record** 技能根据已选定的 tool 读取或确定命令集合，按顺序逐条执行并写入/更新 `commands`。
- 本技能只负责「选哪个工具」；命令集合的生成、执行与持久化由 scan-command-record 负责。
