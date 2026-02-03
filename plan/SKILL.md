---
name: security-power-plan
description: 定义并执行 SecurityPower 安全代码工作流计划。默认步骤为在 Docker 中执行的 7 步，每步脚本记录在 .security-power/scan-scripts.json；每次运行先查该文件，有脚本则直接执行，无则选工具并记录后再执行。即使某步被 skip 也记录其脚本。当用户要求安全扫描、执行安全计划或运行 SecurityPower 工作流时使用。
---

# SecurityPower 执行计划

SecurityPower 是基于 [Trail of Bits Skills](https://github.com/trailofbits/skills) 的安全代码工作流。本技能用于规划与执行安全扫描步骤。

## 执行环境与目录约定

- **Docker 执行**：所有扫描工作均在 **同一个 Docker 容器内**完成。启动一个容器，将项目代码复制进该容器后，在该容器内依次执行各步骤的扫描/构建脚本；执行完毕后从该容器内取出报告等产出。不每步起新容器，全程使用同一容器。
- **脚本与计划文件**：生成的脚本记录、计划文件等放在项目根目录下的 **`.security-power/`** 目录（若不存在则创建）。例如：
  - 脚本集合：`.security-power/scan-scripts.json`
  - 当次执行计划/配置（可选）：`.security-power/` 下其他计划或配置文件
- **报告文件**：各步骤产出的报告（SARIF、JSON、日志等）统一放在 **`.security-power/.output/`** 下，便于 executing-agent 读取与汇总。

## 依赖

- 需依赖并复用 [trailofbits/skills](https://github.com/trailofbits/skills) 中的相关技能（如 static-analysis、insecure-defaults、testing-handbook-skills 等）。

## 默认步骤（在 Docker 中执行的步骤）

以下步骤即为 **plan 的默认步骤**，均在 **同一个 Docker 容器内**依次执行；每步所需脚本记录在 **`.security-power/scan-scripts.json`** 中。

| 步骤 ID | 说明 | 对应能力/工具参考 |
|---------|------|-------------------|
| compile | 编译代码 | 项目构建系统（make、cmake、cargo、npm 等） |
| secret-scan | 密码泄露扫描 | insecure-defaults、gitleaks、trufflehog |
| dependency-scan | 依赖扫描 | npm audit、pip audit、cargo audit、trivy、OSV |
| static-scan | 静态代码扫描 | static-analysis（CodeQL、Semgrep）、SARIF |
| package-scan | 包扫描 | trivy image、syft、容器/包扫描 |
| dynamic-scan | 动态代码扫描 | 动态分析、DAST 工具 |
| fuzzing | Fuzzing 扫描 | testing-handbook-skills、libFuzzer、AFL 等 |

- **默认执行范围**：若用户未指定，默认执行上述全部 7 步（即「在 Docker 中执行」的完整步骤）。
- **脚本文件**：每步在 Docker 内要执行的脚本集合记录在 `.security-power/scan-scripts.json` 中，键为步骤 ID，值为 `tool` + `scripts`（该步骤所需的一条或多条脚本）。**即使某步骤当次被 skip，也需记录该步骤使用到的脚本**（确定工具与脚本后写入文件，仅不执行），以便后续运行可直接复用。

## 每次运行：先查脚本，有则直接执行

**每次运行本 skill 时**：

1. **先读取** `.security-power/scan-scripts.json`，对本次要执行的每个步骤检查是否**已有该步骤的脚本**（即该步骤存在且 `scripts` 为非空数组）。
2. **若某步已有脚本**：**直接执行**该步骤——在**同一 Docker 容器内**按记录中的 `scripts` 顺序逐条执行，不再询问工具、不再重新确定脚本。执行后报告写入 `.security-power/.output/`。
3. **若某步没有脚本**（无该键、或 `scripts` 为空）：先走 **scan-tool-choice** 确定工具并写入 `tool`，再走 **scan-command-record** 确定该工具对应的脚本并写入 `scripts`，然后在该**同一容器内**执行；执行后更新脚本文件。
4. **若某步当次被 skip**：若该步骤尚无脚本，仍按 scan-tool-choice 与 scan-command-record 确定工具与脚本并写入文件（仅不执行），以便后续运行可直接复用；若已有脚本则保留不动。

即：**有脚本则直接执行，无脚本才选工具并记录脚本再执行；被 skip 的步骤也记录其脚本**。

## 扫描工具选择与脚本记录（仅当无脚本时）

仅当某步骤在脚本文件中**尚无脚本**时，才依次使用：

1. **scan-tool-choice**：确定该步骤使用的工具并写入 `tool`（首次询问用户选用哪个并记录，再次询问是否采用上次使用的工具）。
2. **scan-command-record**：根据已选定的 tool 确定脚本集合并写入 `scripts`，再按顺序逐条执行。

若某步骤当次被 skip 但尚无脚本，仍调用上述两技能确定并写入（不执行）。详见同项目下的 `scan-tool-choice` 与 `scan-command-record` 技能。

## 使用方式

1. **确认执行范围**
   - 若用户未指定，默认执行上述全部 7 步（Docker 中的默认步骤）。
   - 若用户指定「只做依赖扫描和静态扫描」等，则仅执行对应步骤并保持顺序（跳过未选步骤）。

2. **执行前**
   - **先查脚本文件**：对每个要执行的步骤，读取 `.security-power/scan-scripts.json`。若该步骤已有非空 `scripts`，则本步**不**调用 scan-tool-choice / scan-command-record，仅列出将直接执行的脚本。
   - 若某步无脚本：再按 **scan-tool-choice** 与 **scan-command-record** 确定工具与脚本并写入文件。若某步被 skip 但尚无脚本，仍确定并写入（不执行）。
   - 列出本次将执行的步骤清单及每步将使用的脚本（来自文件或新确定），便于用户确认。

3. **执行中**
   - 在 **同一个 Docker 容器内**执行：先启动一个容器并将代码复制进该容器，再在该容器内按步骤顺序依次执行。每步使用脚本文件中该步骤的 `scripts`（若已有则直接使用，若无则用本次刚确定并写入的脚本），**在同一容器内按顺序逐条执行**。
   - 每步产出的报告、日志、SARIF 等写入 **`.security-power/.output/`**，供 executing-agent 使用。
   - 某步失败时记录结果并决定是否继续（默认继续执行后续步骤，最后汇总失败项）。

4. **完成条件**
   - **全部所选步骤均通过**（无阻塞性错误）→ 本 SKILL 视为完成，可进入报告与修复阶段。
   - 若有步骤未通过，在总结中明确列出未通过步骤及原因，仍可标记「计划执行完毕，存在失败项」，由 executing-agent 生成报告与修复建议。

## 步骤检查清单

执行时可按此清单勾选（与默认步骤 ID 对应）：

```
计划执行进度:
- [ ] compile        编译代码
- [ ] secret-scan    密码泄露扫描
- [ ] dependency-scan 依赖扫描
- [ ] static-scan    静态代码扫描
- [ ] package-scan   包扫描
- [ ] dynamic-scan   动态代码扫描
- [ ] fuzzing         Fuzzing 扫描
```

## 输出

- 每步的执行结果（成功/失败、关键输出或错误信息）。
- 最终汇总：哪些步骤通过、哪些失败；报告文件路径统一为 `.security-power/.output/` 下对应文件。
- 将上述结果交给 executing-agent 技能，用于生成 report 与修复建议（含可选 PR）。
