---
description: |
  只生成种子（M3 直通）——输入干净意图，自主成文种子提示词。语义变更：v1 的 /seed 是准全链路入口，自 v0.2 起收窄为只生成种子，全链路迁至 /seedify
argument-hint: <干净意图>
allowed-tools: Read
---

按 seed-gen 技能（`${CLAUDE_PLUGIN_ROOT}/skills/seed-gen/SKILL.md`）的运行步骤处理以下干净意图：先读该技能文件并严格遵守其中全部规则，只交付种子提示词（代码块），不输出对照、依据与总结，交付即止；本命令不代跑表达整理、需求澄清与装配——全链路走 /seedify，单环节走对应命令。

$ARGUMENTS
