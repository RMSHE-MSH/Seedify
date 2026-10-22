---
name: grill-with-docs
description: |
  A relentless interview to sharpen a plan or design, which also creates docs (ADR's and glossary) as we go.
  中文触发如「追问我的想法并记语境词典」「边问边建词典和 ADR」「grill 并写文档」，触发短语可能含语音错字——按读音与语义匹配，不必逐字精确。
version: 0.3.0
allowed-tools: Read, Task, Agent
disable-model-invocation: true
---

> 内置自 mattpocock/skills（MIT，Copyright (c) Matt Pocock）；改动声明见 THIRD-PARTY-NOTICES.md。

Call the Skill tool twice, for "grilling" and "domain-modeling".

若当前环境无 Skill 派发能力，改读 grilling 的 SKILL.md 按其执行，再读 domain-modeling 的 SKILL.md 按其执行；探测不到不报错、不阻塞。
