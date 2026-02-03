---
name: scan-command-record
description: 将各步骤在 Docker 内所需的脚本持久化到 .security-power/scan-scripts.json。每次运行 skill 时先查该文件：若某步骤已有脚本则直接执行，若无则与 scan-tool-choice 配合确定工具与脚本并写入后执行。即使某步骤被 skip，也需记录该步骤使用到的脚本。当执行 SecurityPower 计划、安全扫描步骤或需要「记录/复用扫描脚本」时使用。
---

# 扫描脚本记录与复用

同一项目在 Docker 中执行各步骤所需的脚本在项目内持久化为**脚本集合**，存放在 `.security-power/scan-scripts.json`。**每次运行 skill 时**：若某步骤**已有脚本**，则**直接执行**该脚本集合（不再询问工具、不再重新确定脚本）；若某步骤**没有脚本**，才由 **scan-tool-choice** 确定工具、本技能确定并写入脚本后再执行。

**即使某步骤被 skip**：当次运行若跳过某步骤不执行，该步骤已配置的 **tool** 与 **scripts** 仍应保留在文件中；若该步骤尚未有脚本，应在确定工具与脚本后写入文件（仅不执行），以便后续运行可直接复用。即：**脚本文件始终记录各步骤所使用到的脚本，与当次是否执行该步骤无关**。

## 脚本集合

每个默认步骤（compile、secret-scan、dependency-scan 等）在脚本文件中对应一个**脚本集合**：该步骤在**同一 Docker 容器内**需要执行的一条或多条脚本，按顺序逐条执行。

- **记录内容**：`scripts` 即该步骤在同一 Docker 容器内所需执行的脚本序列。
- **执行方式**：按 `scripts` 数组顺序在同一容器内逐条执行；上一条执行完成后再执行下一条。
- **有则直接执行**：若该步骤已存在且 `scripts` 非空，则**直接使用并执行**，不重新确定脚本。
- **失败处理**：某条脚本失败时，该步骤可标记为失败；是否继续执行该集合内后续脚本由策略或用户决定（默认可中止该步骤）。

## 存储位置与格式

- **存储位置**：项目根目录下的 **`.security-power/scan-scripts.json`**（若目录不存在则创建）。脚本与计划相关文件均放在 `.security-power/` 下；各步骤产出的报告、日志、SARIF 等写入 **`.security-power/.output/`**，不放在 `scan-scripts.json` 同目录。
- **执行环境**：脚本在 **同一个 Docker 容器内**执行（与 plan 中其他步骤共用同一容器；代码已在该容器内，按 `scripts` 顺序在该容器内逐条执行）；报告等产出从该容器写出到宿主机 `.security-power/.output/`。
- **存储格式**：按步骤 ID 记录「本步骤所选工具及脚本集合」；键为步骤标识，值为 `tool`（本次使用的工具名）+ `scripts`（脚本数组）。**即使某步骤当次被 skip，只要已确定过工具与脚本，也应写入/保留该步骤的 `tool` 与 `scripts`**。

```json
{
  "compile": {
    "tool": "npm",
    "scripts": ["npm ci", "npm run build"]
  },
  "secret-scan": {
    "tool": "gitleaks",
    "scripts": ["gitleaks detect --no-git"]
  },
  "dependency-scan": {
    "tool": "npm",
    "scripts": ["npm audit --audit-level=moderate"]
  },
  "static-scan": {
    "tool": "semgrep",
    "scripts": ["semgrep download", "semgrep scan --config auto"]
  },
  "package-scan": {
    "tool": "trivy",
    "scripts": ["trivy fs ."]
  },
  "dynamic-scan": { "tool": "...", "scripts": ["..."] },
  "fuzzing": { "tool": "...", "scripts": ["..."] }
}
```

- **步骤 ID**：与 SecurityPower 执行计划对应：`compile`、`secret-scan`、`dependency-scan`、`static-scan`、`package-scan`、`dynamic-scan`、`fuzzing`。
- **tool**：由 **scan-tool-choice** 技能确定并记录的扫描工具名称。
- **scripts**：该工具对应的脚本序列；执行时按数组顺序依次执行。

## 使用流程

1. **执行某步前（每次运行 skill 时优先检查脚本文件）**
   - **先读取** `.security-power/scan-scripts.json` 中该步骤的 `tool` 与 `scripts`。
   - 若该步骤**已有非空 scripts**：**直接执行**——不调用 scan-tool-choice、不重新确定脚本，在 Docker 内按记录中的 `scripts` 顺序逐条执行。
   - 若该步骤**无 scripts 或 scripts 为空**：先由 **scan-tool-choice** 确定该步骤使用的 tool 并写入，再根据当前 `tool` 确定脚本序列（可搜索或按本技能约定），将 `scripts` 写入/更新到 `scan-scripts.json`，然后按 `scripts` 顺序在**同一 Docker 容器内**逐条执行。
   - **若该步骤当次被 skip**：若尚无脚本则先确定并写入 `tool` 与 `scripts`（仅写入不执行）；若已有脚本则保留不动。确保文件中始终记录该步骤所使用到的脚本。

2. **执行某步时**
   - 在 **同一个 Docker 容器内**（与 plan 其他步骤共用）按本次确定的 `scripts` 数组**从第一条到最后一条依次执行**；每条执行完成后再执行下一条。
   - 每步产出的报告、日志、SARIF 等写入宿主机 **`.security-power/.output/`**。
   - 某条失败时：该步骤标记为失败；是否继续执行该集合内后续脚本由策略或用户决定。

3. **执行某步后**
   - 若本次确定或更新了脚本序列，将 **scripts**（及已由 scan-tool-choice 确定的 **tool**）写入或更新到 `scan-scripts.json`。
   - 确保文件为合法 JSON，避免覆盖其他步骤已有记录。

4. **何时重新确定脚本**
   - 用户通过 scan-tool-choice 更换工具后，本技能根据新 tool 重新确定 scripts 并更新记录。
   - 记录中的脚本集合执行失败且需换方案时，可重新确定脚本序列并更新记录。

## 与 plan、scan-tool-choice 的配合

- **每次运行 plan 时**：对每个要执行的步骤，**先查脚本文件**。若该步骤已有非空 `scripts`，则**直接执行**，不调用 scan-tool-choice 与本技能的「确定脚本」流程；若没有脚本，再先 **scan-tool-choice** 确定工具，再本技能确定并写入 `scripts` 后执行。**被 skip 的步骤**：若尚无脚本则仍确定并写入（不执行），若已有则保留。
- 本技能负责：脚本集合的**读取**（有则直接执行）、**执行**（在 Docker 内）与**持久化**（无则确定后写入；被 skip 时也写入以便后续复用）；不负责「选哪个工具」。
