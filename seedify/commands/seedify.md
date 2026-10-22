---
description: 种子化全链路入口——七模块与两席串链，把嘈杂口语输入整理成种子提示词
argument-hint: <要优化的原始文本>
allowed-tools: Read, Task, Agent
---

按 seedify 编排器技能（`${CLAUDE_PLUGIN_ROOT}/skills/seedify/SKILL.md`）的运行步骤处理以下文本：先读该技能文件，按其中的边界判断与串链规则调度各模块（需派发时用 Task/Agent），只交付最终提示词（代码块），不输出对照、依据与总结，交付即止。

$ARGUMENTS
