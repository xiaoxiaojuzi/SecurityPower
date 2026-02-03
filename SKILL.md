---
name: security-power
description: SecurityPower 安全代码工作流入口。基于可组合技能的安全扫描流程，包含执行计划、工具选择、命令记录与执行报告。当用户要求安全扫描、执行 SecurityPower、运行安全计划或需要安全代码工作流时使用本技能作为入口。
---

# SecurityPower 入口

SecurityPower 是基于 [Trail of Bits Skills](https://github.com/trailofbits/skills) 的安全代码工作流，由一组可组合技能构成。本文件为该工作流的**入口**：当用户需要进行安全扫描或运行 SecurityPower 时，由此进入并按下列技能组合执行。

## 依赖

- 需依赖并复用 [trailofbits/skills](https://github.com/trailofbits/skills) 中的相关技能（如 static-analysis、insecure-defaults、testing-handbook-skills 等）。

## 子技能与调用顺序

| 技能 | 路径 | 职责 |
|------|------|------|
| **plan** | [plan/SKILL.md](plan/SKILL.md) | 定义并执行安全扫描计划（编译、密码泄露、依赖、静态、包、动态、Fuzzing 等步骤）；默认全部执行，也可按需选择子集。 |
| **scan-tool-choice** | [scan-tool-choice/SKILL.md](scan-tool-choice/SKILL.md) | 为每类扫描步骤确定使用的工具：首次询问用户选用哪个并记录，再次询问是否采用上次使用的。 |
| **scan-command-record** | [scan-command-record/SKILL.md](scan-command-record/SKILL.md) | 按步骤持久化命令集合，再次执行时按记录逐条执行；与 scan-tool-choice 配合使用。 |
| **executing-agent** | [executing-agent/SKILL.md](executing-agent/SKILL.md) | 每步执行后给出 report（问题与修复建议）；若修复可代码化则生成 Pull Request。 |

## 推荐执行流程

1. **从本入口进入**：用户提出安全扫描或 SecurityPower 需求时，使用本技能作为入口。
2. **执行计划**：按 **plan** 技能确定执行范围（全部或部分步骤），对每个步骤依次：
   - 先按 **scan-tool-choice** 确定该步骤使用的工具（询问用户并记录）；
   - 再按 **scan-command-record** 根据已选工具读取或确定命令集合，按顺序逐条执行并写入/更新记录。
3. **报告与修复**：计划执行完毕后，按 **executing-agent** 技能生成报告（问题与修复建议），若修复可代码化则生成 PR。

## 完成条件

- 当**所有所选计划步骤均通过**且 **executing-agent** 已产出报告（及可选 PR）时，SecurityPower 工作流视为完成。

## 详细说明

- 计划步骤与使用方式详见 [plan/SKILL.md](plan/SKILL.md)。
- 工具选择与命令记录详见 [scan-tool-choice/SKILL.md](scan-tool-choice/SKILL.md) 与 [scan-command-record/SKILL.md](scan-command-record/SKILL.md)。
- 报告与 PR 规范详见 [executing-agent/SKILL.md](executing-agent/SKILL.md)。
