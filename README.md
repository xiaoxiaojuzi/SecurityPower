# SecurityPower

SecurityPower 是一个面向编码智能体的安全代码工作流，基于一组可组合的 [Skills](https://cursor.com/cn/docs/context/skills) 及初始指令，确保智能体正确使用这些能力。

## 依赖

- [trailofbits/skills](https://github.com/trailofbits/skills)

## Plan Skill

执行计划包含以下步骤（默认全部执行，也可按需选择部分）：

- 编译代码
- 密码泄露扫描
- 依赖扫描 (dependency)
- 静态代码扫描
- 包扫描
- 动态代码扫描
- Fuzzing 扫描

当所选执行计划全部通过时，该 Skill 视为完成。

## Executing-Agent Skill

- 每个执行步骤完成后需产出 **Report**，包含问题描述与修复建议。
- 若修复建议可落为代码，则生成 **Pull Request**。
