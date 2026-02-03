---
name: scan-command-record
description: 将项目内使用的扫描/构建命令按步骤持久化为「命令集合」到 .security-power/scan-commands.json，再次执行时按集合内顺序逐条执行并优先复用。使用哪个工具由 scan-tool-choice 技能确定。当执行 SecurityPower 计划、安全扫描步骤或需要「记录/复用扫描命令」时使用。
---

# 扫描命令记录与复用

同一项目使用的扫描命令在项目内持久化为**命令集合**，再次执行时按集合内顺序逐条执行。**使用哪个工具**由 **scan-tool-choice** 技能确定（首次询问用户选用哪个并记录，再次询问是否采用上次使用的工具）；本技能只负责**命令集合**的读取、执行与持久化。

## 命令集合

每个扫描步骤在确定工具后，对应一个**命名命令集合**：多条命令按顺序一步一步执行。

- **执行方式**：按 `commands` 数组顺序逐条执行；上一条执行完成后再执行下一条。
- **失败处理**：某条命令失败时，该步骤可标记为失败；是否继续执行该集合内后续命令由策略或用户决定（默认可中止该步骤）。

## 存储位置与格式

- **存储位置**：项目根目录下的 `.security-power/scan-commands.json`（若目录不存在则创建）。
- **存储格式**：按步骤 ID 记录「本步骤所选工具及命令集合」；键为步骤标识，值为 `tool`（本次使用的工具名）+ `commands`（命令数组）。

```json
{
  "compile": {
    "tool": "npm",
    "commands": ["npm ci", "npm run build"]
  },
  "secret-scan": {
    "tool": "gitleaks",
    "commands": ["gitleaks detect --no-git"]
  },
  "dependency-scan": {
    "tool": "npm",
    "commands": ["npm audit --audit-level=moderate"]
  },
  "static-scan": {
    "tool": "semgrep",
    "commands": ["semgrep download", "semgrep scan --config auto"]
  },
  "package-scan": {
    "tool": "trivy",
    "commands": ["trivy fs ."]
  },
  "dynamic-scan": { "tool": "...", "commands": ["..."] },
  "fuzzing": { "tool": "...", "commands": ["..."] }
}
```

- **步骤 ID**：与 SecurityPower 执行计划对应：`compile`、`secret-scan`、`dependency-scan`、`static-scan`、`package-scan`、`dynamic-scan`、`fuzzing`。
- **tool**：由 **scan-tool-choice** 技能确定并记录的扫描工具名称。
- **commands**：该工具对应的命令序列；执行时按数组顺序依次执行。

## 使用流程

1. **执行某步前**
   - **先由 scan-tool-choice 确定该步骤使用的 tool**（首次询问用户选用哪个并记录，再次询问是否采用上次使用的工具；结果已写入 `scan-commands.json` 的 `tool` 字段）。
   - 读取 `.security-power/scan-commands.json` 中该步骤的 `tool` 与 `commands`。
   - 若**已有 commands** 且 tool 未变更：直接使用记录中的命令集合，按 `commands` 顺序逐条执行。
   - 若**无 commands 或 tool 已变更**：根据当前 `tool` 确定命令序列（可搜索或按本技能约定），将 `commands` 写入/更新到 `scan-commands.json`，再按 `commands` 顺序逐条执行。

2. **执行某步时**
   - 按本次确定的 `commands` 数组**从第一条到最后一条依次执行**；每条执行完成后再执行下一条。
   - 某条失败时：该步骤标记为失败；是否继续执行该集合内后续命令由策略或用户决定。

3. **执行某步后**
   - 若本次确定或更新了命令序列，将 **commands**（及已由 scan-tool-choice 确定的 **tool**）写入或更新到 `scan-commands.json`。
   - 确保文件为合法 JSON，避免覆盖其他步骤已有记录。

4. **何时重新确定命令**
   - 用户通过 scan-tool-choice 更换工具后，本技能根据新 tool 重新确定 commands 并更新记录。
   - 记录中的命令集合执行失败且需换方案时，可重新确定命令序列并更新记录。

## 与 plan、scan-tool-choice 的配合

- **security-power-plan** 执行某步骤时：先按 **scan-tool-choice** 确定该步骤使用的工具（询问用户并写入 `tool`），再按本技能（**scan-command-record**）根据 `tool` 读取或确定命令集合，按顺序逐条执行并写入/更新 `commands`。
- 本技能不负责「选哪个工具」，只负责命令集合的读取、执行与持久化。
