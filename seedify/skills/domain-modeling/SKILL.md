---
name: domain-modeling
description: |
  Build and sharpen a project's domain model. Use when discussing project terminology, writing or editing the conversation-internal context dictionary (语境词典), or recording or editing an ADR.
  中文触发如「建个语境词典」「术语对齐一下」「这个词怎么定义」，触发短语可能含语音错字——按读音与语义匹配，不必逐字精确。
version: 0.3.0
allowed-tools: Read
---

# Domain Modeling

> 内置自 mattpocock/skills（MIT，Copyright (c) Matt Pocock）；改动声明见 THIRD-PARTY-NOTICES.md。

0. **先读思想纲领**：读取 references/charter.md，领会核心思想后再动手。

Actively build and sharpen the project's domain model as you design. This is the *active* discipline: challenging terms, inventing edge-case scenarios, and writing the 语境词典 entries and decisions down the moment they crystallise. (Merely *reading* the 语境词典 for vocabulary is not this skill: that's a one-line habit any skill can do. This skill is for when you're changing the model, not just consuming it.)

## 工件形态：对话内语境词典

语境词典的词条与决策记录活在对话内——随对话维护、可回指用户确认。**不写任何文件**（持久化与跨会话沉淀留后续版本）。

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with the existing language in the 语境词典, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account': do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### 对拍已确认事实

用户已确认的原话与结论是唯一对照源。发现矛盾即指出：「你之前说 X，现在说 Y，以哪个为准？」

### 就地更新语境词典

发现术语即在对话内补词条，不批量攒。格式见 [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md)，决策记录见 [ADR-FORMAT.md](./ADR-FORMAT.md)。

语境词典只收语义与用法，不收实现细节。不要把语境词典当作规格书、草稿本或实现决策的存放处——它只是词典，不是别的。

### Offer ADRs sparingly

三个条件全部成立，才在对话内记一条决策记录（难逆转、无上下文会困惑、真实取舍）：

1. **Hard to reverse**: the cost of changing your mind later is meaningful
2. **Surprising without context**: a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off**: there were genuine alternatives and you picked one for specific reasons

If any of the three is missing, skip the ADR. Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md).
