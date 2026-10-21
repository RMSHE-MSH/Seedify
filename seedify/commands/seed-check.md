---
description: 提示词检查入口（M7）——按适配器规则派发独立评审，只诊断不改写
argument-hint: <待体检的提示词>
allowed-tools: Read, Task, Agent
---

按 seed-check 技能（`${CLAUDE_PLUGIN_ROOT}/skills/seed-check/SKILL.md`）的运行步骤检查以下提示词：先读该技能文件，按其中的适配器规则办理——探测到可派发子代理则把送检全文原样派发 seed-reviewer 做独立评审、收回报告原样转述；探测不到则自身执行并在报告开头注明独立性弱化。只诊断不改写，不代替改写提示词或补充结论；用户要求改写时另行发起、走生成族。

$ARGUMENTS
