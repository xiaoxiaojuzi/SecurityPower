# SecurityPower

SecurityPower 是一个面向编码智能体的安全代码工作流，基于一组可组合的 [Skills](https://cursor.com/cn/docs/context/skills) 及初始指令，确保智能体正确使用这些能力。

## 依赖

- [trailofbits/skills](https://github.com/trailofbits/skills)

## 执行环境与目录

- **Docker**：所有扫描工作均在 **同一个 Docker 容器内**完成；启动一个容器、将代码复制进该容器后，在该容器内依次执行各步骤，执行完毕后从该容器取出报告。
- **脚本与计划文件**：生成的脚本记录、计划文件等放在项目根目录下的 **`.security-power/`**（如 `scan-scripts.json`）。
- **报告文件**：各步骤产出的报告（SARIF、JSON、日志等）放在 **`.security-power/.output/`**。

## Plan Skill

**默认步骤**即「在 Docker 中执行的步骤」，每步所需脚本记录在 `.security-power/scan-scripts.json`。**每次运行 skill 时**：先查该脚本文件；若某步骤**已有脚本则直接执行**，若无则选工具并记录脚本后再执行。即使某步骤被 skip，也记录该步骤使用到的脚本。

默认步骤（均可按需选择子集执行）：

- compile、secret-scan、dependency-scan、static-scan、package-scan、dynamic-scan、fuzzing

当所选执行计划全部通过时，该 Skill 视为完成。

## Executing-Agent Skill

- 每个执行步骤完成后需产出 **Report**，包含问题描述与修复建议；报告写入 `.security-power/.output/`。
- 若修复建议可落为代码，则生成 **Pull Request**。
