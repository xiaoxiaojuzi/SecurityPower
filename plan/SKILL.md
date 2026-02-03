---
name: security-power-plan
description: 定义并执行 SecurityPower 安全代码工作流计划。包含编译、密码泄露扫描、依赖扫描、静态扫描、包扫描、动态扫描、Fuzzing 等步骤。默认执行全部步骤，也可按需选择子集。当用户要求安全扫描、执行安全计划或运行 SecurityPower 工作流时使用。
---

# SecurityPower 执行计划

SecurityPower 是基于 [Trail of Bits Skills](https://github.com/trailofbits/skills) 的安全代码工作流。本技能用于规划与执行安全扫描步骤。

## 依赖

- 需依赖并复用 [trailofbits/skills](https://github.com/trailofbits/skills) 中的相关技能（如 static-analysis、insecure-defaults、testing-handbook-skills 等）。

## 执行计划步骤

按顺序执行以下步骤（默认全部执行，用户可指定只执行其中若干项）：

| 步骤 | 说明 | 对应能力/工具参考 |
|------|------|-------------------|
| 1. 编译代码 | 确保项目可成功编译/构建 | 项目构建系统（make、cmake、cargo、npm 等） |
| 2. 密码泄露扫描 | 检测硬编码密码、密钥、token 等 | insecure-defaults、gitleaks、trufflehog |
| 3. Dependency 扫描 | 依赖漏洞与许可证 | npm audit、pip audit、cargo audit、trivy、OSV |
| 4. 静态代码扫描 | 静态分析漏洞与坏味道 | static-analysis（CodeQL、Semgrep）、SARIF |
| 5. 包扫描 | 制品/包安全与合规 | trivy image、syft、容器/包扫描 |
| 6. 动态代码扫描 | 运行时/交互式检测 | 动态分析、DAST 工具 |
| 7. Fuzzing 扫描 | 模糊测试发现崩溃与漏洞 | testing-handbook-skills、libFuzzer、AFL 等 |

## 扫描工具选择与命令记录

每类扫描可配置**一个或多个扫描工具**；每个步骤对应一个**命令集合**（多条命令），按顺序逐条执行。执行各步骤时，按以下两个技能依次完成：

1. **scan-tool-choice**：确定该步骤使用的工具。
   - **项目首次使用该步骤**：询问用户需要使用哪个工具，并将所选工具记录下来。
   - **再次运行该步骤**：询问用户是否采用上次使用的工具；若采用则用已记录的工具，若不采用则再次让用户选择并更新记录。
2. **scan-command-record**：根据已选定的工具读取或确定命令集合，按顺序逐条执行并写入/更新记录。

详见同项目下的 `scan-tool-choice` 与 `scan-command-record` 技能。

## 使用方式

1. **确认执行范围**
   - 若用户未指定，默认执行上述全部 7 步。
   - 若用户指定「只做依赖扫描和静态扫描」等，则仅执行对应步骤并保持顺序（跳过未选步骤）。

2. **执行前**
   - 每步先按 **scan-tool-choice** 确定使用的工具（首次询问用户选用哪个并记录，再次询问是否采用上次使用的工具），再按 **scan-command-record** 确定该工具对应的命令集合。
   - 列出本次将执行的步骤清单（及每步将使用的工具与命令集合），便于用户确认。

3. **执行中**
   - 按步骤顺序执行；每步先 scan-tool-choice 再 scan-command-record：读取命令集合，**按集合内顺序逐条执行**，执行后写入/更新记录。
   - 某步失败时记录结果并决定是否继续（默认继续执行后续步骤，最后汇总失败项）。
   - 每步产出（日志、报告、SARIF 等）保留供 executing-agent 使用。

4. **完成条件**
   - **全部所选步骤均通过**（无阻塞性错误）→ 本 SKILL 视为完成，可进入报告与修复阶段。
   - 若有步骤未通过，在总结中明确列出未通过步骤及原因，仍可标记「计划执行完毕，存在失败项」，由 executing-agent 生成报告与修复建议。

## 步骤检查清单

执行时可按此清单勾选：

```
计划执行进度:
- [ ] 1. 编译代码
- [ ] 2. 密码泄露扫描
- [ ] 3. Dependency 扫描
- [ ] 4. 静态代码扫描
- [ ] 5. 包扫描
- [ ] 6. 动态代码扫描
- [ ] 7. Fuzzing 扫描
```

## 输出

- 每步的执行结果（成功/失败、关键输出或错误信息）。
- 最终汇总：哪些步骤通过、哪些失败，以及产出的报告/文件路径。
- 将上述结果交给 executing-agent 技能，用于生成 report 与修复建议（含可选 PR）。
