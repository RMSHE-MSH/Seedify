# Third-Party Notices

本项目内置（vendored）了来自开源社区的第三方组件，以下列出其来源、许可证与改动声明。

## mattpocock/skills

- 来源：https://github.com/mattpocock/skills
- 许可证：MIT License
- 版权：Copyright (c) Matt Pocock

```
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

### 内置清单

以下文件内置自 mattpocock/skills，随本插件与 cowork zip 分发：

| 内置路径 | 上游路径 |
|---|---|
| `seedify/skills/grilling/SKILL.md` | `skills/productivity/grilling/SKILL.md` |
| `seedify/skills/grill-me/SKILL.md` | `skills/productivity/grill-me/SKILL.md` |
| `seedify/skills/grill-with-docs/SKILL.md` | `skills/engineering/grill-with-docs/SKILL.md` |
| `seedify/skills/domain-modeling/SKILL.md` | `skills/engineering/domain-modeling/SKILL.md` |
| `seedify/skills/domain-modeling/GLOSSARY-FORMAT.md` | `skills/engineering/domain-modeling/GLOSSARY-FORMAT.md` |
| `seedify/skills/domain-modeling/ADR-FORMAT.md` | `skills/engineering/domain-modeling/ADR-FORMAT.md` |
| `seedify/skills/to-spec/SKILL.md` | `skills/engineering/to-spec/SKILL.md` |

各技能目录下的 `references/charter.md` 为本项目思想纲领（总纲 v4）的逐字节副本，非上游内容，不属第三方组件。

### 改动声明

在保留上游核心机制（机制保真、域换皮、口径对齐）的前提下，相对上游做了以下改动：

**`grilling/SKILL.md`**

- 题干示例删除「including multiple choices」（Q1、Q2 两处）——对齐本项目「开放问法、禁填空模板」的裁定口径；设计树、frontier、推荐答案、yes=采纳等机制不变。
- 新增：中文触发短语、`version`、`allowed-tools`、归属行、「先读思想纲领」步骤。

**`grill-me/SKILL.md`、`grill-with-docs/SKILL.md`**

- 新增：中文触发短语、`version`、`allowed-tools`、归属行。
- 新增：探测-降级条款（当前环境无 Skill 派发能力时，改读对应引擎技能的 SKILL.md 按其执行，不报错、不阻塞）；「Call the Skill tool...」派发行原样保留。

**`domain-modeling/SKILL.md`**

- 文件机制换皮：上游的落盘机制（`GLOSSARY.md`、`docs/adr/`、`GLOSSARY-MAP.md`、懒创建规则）换皮为「对话内语境词典」——词条与决策记录只活在对话内，不写任何文件。
- 「Cross-reference with code」（对拍代码）换皮为「对拍已确认事实」（以用户已确认的原话与结论为对照源，而非代码）。
- 「Offer ADRs sparingly」的三个条件（含条件 2）逐字节保留，未作改动；仅该节开头引入句换为中文（「三个条件全部成立，才在对话内记一条决策记录……」）。
- 新增：中文触发短语、`version`、`allowed-tools`、归属行、「先读思想纲领」步骤。

**`domain-modeling/GLOSSARY-FORMAT.md`、`domain-modeling/ADR-FORMAT.md`**

- 删除落盘相关段落（文件路径、`docs/adr/` 扫描、`GLOSSARY-MAP.md` 多语境映射等），换皮为对话内编号与对话内维护规则；词条结构、ADR 三条件、可选段落等主体内容保留。
- `ADR-FORMAT.md` 的三个条件（含条件 2「Surprising without context: a future reader will look at the code and wonder "why on earth did they do it this way?"」）与上游逐字节一致，未作改动。

**`to-spec/SKILL.md`**

- 模板换皮为「提示词构建方案」：Problem Statement → 意图与问题；Solution → 构建路线与取舍；Implementation Decisions → 构建决策；Out of Scope → 范围外；Further Notes → 备注。
- 删除：User Stories 节、Testing Decisions 节（提示词域无对位）、issue tracker 发布步骤、测试 seam 勾勒步骤、代码库探索步骤。
- `disable-model-invocation: true` 删除——改为 model-invoked，使编排器/seed-clarify 能在链内派发。
- 新增：中文触发短语、`version`、`allowed-tools`、归属行、「先读思想纲领」步骤、「方案是蓝图、批准前不写提示词」的门规则说明。

**通用**

- 五个 `SKILL.md`（`grilling`、`grill-me`、`grill-with-docs`、`domain-modeling`、`to-spec`）均新增归属行：「内置自 mattpocock/skills（MIT，Copyright (c) Matt Pocock）；改动声明见 THIRD-PARTY-NOTICES.md。」（两个格式文件 `GLOSSARY-FORMAT.md`/`ADR-FORMAT.md` 不带此行。）
- 五个技能目录均新增 `references/charter.md`（本项目思想纲领副本，非上游内容）；`grilling`、`domain-modeling`、`to-spec` 另新增「先读思想纲领」步骤，`grill-me`/`grill-with-docs`（派发行壳，内容仅一行派发指令+降级条款）不重复此步骤。
