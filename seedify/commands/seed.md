---
description: 用 seedify 优化提示词（直接入口，不依赖触发短语匹配）
argument-hint: <要优化的原始文本>
allowed-tools: Read
---

按 seedify 技能（`${CLAUDE_PLUGIN_ROOT}/skills/seedify/SKILL.md`）的运行步骤处理以下文本：先读该技能文件与其 references/charter.md，严格遵守其中全部规则——有疑必问、先问后写；种子自主成文，三要素仅作提示；只交付优化后的提示词（代码块），不输出改动对照、依据与总结。

$ARGUMENTS
