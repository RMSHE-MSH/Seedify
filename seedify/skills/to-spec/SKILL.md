---
name: to-spec
description: |
  Turn what the conversation has already settled into a 提示词构建方案 (prompt-construction plan): no interview, just synthesis of what's already been discussed. Use when the user wants to consolidate the clarified discussion into a construction plan, or uses any 'to-spec' trigger phrases.
  中文触发如「把谈透的内容综合成构建方案」「to-spec」「出方案」，触发短语可能含语音错字——按读音与语义匹配，不必逐字精确。
version: 0.3.0
allowed-tools: Read
---

> 内置自 mattpocock/skills（MIT，Copyright (c) Matt Pocock）；改动声明见 THIRD-PARTY-NOTICES.md。

0. **先读思想纲领**：读取 references/charter.md，领会核心思想后再动手。

> 方案是「造提示词的蓝图」（排练层工件）：不写提示词逐句文本，不进最终交付代码块；方案批准前不写提示词（跳层与单模块直调不受此门约束）。

This skill takes what the 追问席 has already extracted and produces a 提示词构建方案. Do NOT interview the user; just synthesize what you already know. 不再访谈，只综合已知。

The 方案 is presented in conversation and handed to the 批准门 for review.

## Process

1. 沿用语境词典的术语（若上游建有）；尊重已有决策记录（若已建）。

2. 按下方模板写成方案，在对话内呈现，交批准门审阅。

<spec-template>

## 意图与问题

想解决什么、给谁用、为什么——只收用户说过的与已确认的结论。

## 构建路线与取舍

考虑过的路线、各自取舍、选定哪条与为什么。路线须可回指追问席的选定记录。

## 构建决策

造这条提示词的决策：入哪些槽位、留白什么、如何取舍。用户明确给出的流程原样收录、不升格为约束。不写提示词逐句文本——成文归 seed-gen/装配。

## 范围外

本方案不覆盖的内容。

## 备注

其他需要留档的事项。

</spec-template>
