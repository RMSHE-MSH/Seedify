# M2 两席制改造（内置 grill/to-spec）Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 把 M2 撤编重组为两席——追问席（内置 mattpocock grill 全家）与方案席（内置 to-spec 换皮），流程体检能力吸收，编排器/M3/M5 接口联动，v0.3.0 发布就绪。

**Architecture:** 五个上游技能（grilling、grill-me、grill-with-docs、domain-modeling、to-spec）从本地克隆 vendored 进 `seedify/skills/`，按「机制保真、域换皮、口径对齐」改写：grilling 只做口径微改，domain-modeling 文件机制换皮为对话内语境词典，to-spec 模板换皮为提示词构建方案；`seed-clarify` 改薄壳串两席；编排器链位与批准门规则改写；全程 charter 入包、零外置依赖。

**Tech Stack:** Markdown 技能文件（Claude Code plugin / Cowork zip 双分发）；验证用 `claude plugin validate`、python yaml、grep/sha256。

**Spec:** `docs/superpowers/specs/2026-10-22-m2-grill-tospec-design.md`（十二项裁定已定稿）

## Global Constraints

- 七项用户裁定全生效；总纲 v4 原文一字不动（各席 `references/charter.md` 逐字节副本，sha256 全一致；技能正文含「运行前先读」）。
- 机制保真红线：grilling 的设计树/frontier 收敛/推荐答案/「事实归 agent」不动；to-spec 的「不再访谈、只综合已知」不动；壳的 `Call the Skill tool with "grilling"` 行原样保留。
- 交付物只有提示词一件（裁定#5）；方案与语境词典是对话内工件，不落盘、不进交付代码块。
- 内置件自带归属行「内置自 mattpocock/skills（MIT，Copyright (c) Matt Pocock）；改动声明见 THIRD-PARTY-NOTICES.md」；全量改动登记根目录 `THIRD-PARTY-NOTICES.md`。
- 可分发文件无本机绝对路径（`${CLAUDE_PLUGIN_ROOT}` 不含盘符）；frontmatter 一律块标量 description（`description: |`），改后 `python -c "import yaml; yaml.safe_load"` 自测。
- 版本双处一致：`plugin.json` 与**全部** SKILL.md 均 `0.3.0`（8 现役 + 5 内置 = 13 处）。
- 提问口径：开放问法（裁定#2）；grilling 题干「including multiple choices」删除；「推荐答案、yes=采纳」保留（提案非模板）。
- 每任务以验证命令收尾并独立成 commit；不引入任何外部依赖/MCP/Hook。

## Review Focus

- **换皮把机制改坏**（grilling 的 frontier/推荐答案、to-spec 的不访谈约束被顺手「优化」）→ 各任务含「机制点对照」步骤（grep 上游关键句在位）；钉在 Task 1/4。
- **方案渗漏成蓝图**（构建方案混进交付提示词、或写成提示词逐句）→ T28 护栏 + to-spec/assemble 正文双处声明；钉在 Task 4/7。
- **Cowork 无 Skill 派发时壳断线**（grill-me 那行 `Call the Skill tool` 在 Cowork 跑不通）→ 探测-降级条款 grep + 发布门禁实测项；钉在 Task 3/9。
- **内置件缺中文触发/错字容错**导致路由失败 → description 增补句 grep + T7 注记；钉在 Task 1/4。
- **charter 副本漂移或忘读** → 每任务 sha256 唯一值校验 + 「运行前先读」在位 grep；钉在每个引入任务。

---

### Task 1: 引入 grilling 引擎（追问席核心）

**Files:**
- Create: `seedify/skills/grilling/SKILL.md`（源：`D:/AIGC/research/mattpocock-skills/skills/productivity/grilling/SKILL.md`）
- Create: `seedify/skills/grilling/references/charter.md`（源：`seedify/skills/seed-express/references/charter.md` 逐字节拷贝）
- Test: 同文件 grep/yaml 校验（见步骤）

**Interfaces:**
- Produces: 技能 `grilling`（model-invoked），供 grill-me/grill-with-docs 壳与编排器/seed-clarify 以 Skill 名 `grilling` 或 Read 其 SKILL.md 调用。

- [ ] **Step 1: 拷贝上游原文**

```bash
mkdir -p seedify/skills/grilling/references
cp /d/AIGC/research/mattpocock-skills/skills/productivity/grilling/SKILL.md seedify/skills/grilling/SKILL.md
cp seedify/skills/seed-express/references/charter.md seedify/skills/grilling/references/charter.md
```

- [ ] **Step 2: 应用四处定点改动**（其余正文一字不动）

frontmatter `description` 末尾追加一行中文触发（沿用现役句式）：`中文触发如「追问我的想法」「把这个需求问透」「grill 一下这个计划」，触发短语可能含语音错字——按读音与语义匹配，不必逐字精确。` 并加 `version: 0.3.0`、`allowed-tools: Read, Task, Agent`。
正文中标题行后插入一行：`> 内置自 mattpocock/skills（MIT，Copyright (c) Matt Pocock）；改动声明见 THIRD-PARTY-NOTICES.md。`
正文第一步前插入：`0. **先读思想纲领**：读取 references/charter.md，领会核心思想后再动手。`（并把原「Interview the user…」段保持原序）
删除题干格式行中的短语 `including multiple choices`（`<question body, might be multiple paragraphs, including multiple choices>` → `<question body, might be multiple paragraphs>`）。

- [ ] **Step 3: 验证**

Run: `python -c "import yaml,sys; t=open('seedify/skills/grilling/SKILL.md',encoding='utf-8').read(); yaml.safe_load(t.split('---',2)[1]); print('yaml ok')"` → `yaml ok`
Run: `grep -c "including multiple choices" seedify/skills/grilling/SKILL.md` → `0`
Run: `grep -c "frontier\|recommended answer\|设计树\|design tree" seedify/skills/grilling/SKILL.md` → `≥3`（机制点在位）
Run: `grep -c "先读思想纲领" seedify/skills/grilling/SKILL.md` → `1`
Run: `sha256sum seedify/skills/grilling/references/charter.md seedify/skills/seed-express/references/charter.md | awk '{print $1}' | sort -u | wc -l` → `1`

- [ ] **Step 4: Commit**

```bash
git add seedify/skills/grilling && git commit -m "feat: 引入 grilling 引擎——内置追问席核心（机制保真+口径对齐+charter 入包）"
```

### Task 2: 引入 domain-modeling 换皮「语境词典」

**Files:**
- Create: `seedify/skills/domain-modeling/SKILL.md`（源：`D:/AIGC/research/mattpocock-skills/skills/engineering/domain-modeling/SKILL.md`）
- Create: `seedify/skills/domain-modeling/references/charter.md`（同 Task 1 拷法）
- Create: `seedify/skills/domain-modeling/GLOSSARY-FORMAT.md`、`ADR-FORMAT.md`（源同目录两格式文件，换皮改写）

**Interfaces:**
- Produces: 技能 `domain-modeling`（model-invoked），提供**对话内语境词典**（术语条目+决策记录）供 grill-with-docs 壳、to-spec（方案席沿用词典语言）与编排器消费。词典**不落盘**（设计裁定#11）。

- [ ] **Step 1: 拷贝上游三文件 + charter**

```bash
mkdir -p seedify/skills/domain-modeling/references
cp /d/AIGC/research/mattpocock-skills/skills/engineering/domain-modeling/{SKILL.md,GLOSSARY-FORMAT.md,ADR-FORMAT.md} seedify/skills/domain-modeling/
cp seedify/skills/seed-express/references/charter.md seedify/skills/domain-modeling/references/charter.md
```

- [ ] **Step 2: 换皮改写 SKILL.md**（机制保真：挑战术语/场景压力测试/ADR 三条件/就地更新四机制不动；仅域与工件形态换皮）

- frontmatter：`description` 追加中文触发句（同 Task 1 句式，触发词含「语境词典」「术语对齐」「这个词怎么定义」）+ `version: 0.3.0` + `allowed-tools: Read`；
- 标题后加归属行（同 Task 1）+ 「0. 先读思想纲领」行（同 Task 1）；
- 「File structure」整节替换为 **「工件形态：对话内语境词典」**——说明：词条与决策记录活在对话内（随对话维护、可回指用户确认），**不写任何文件**（落盘与跨会话沉淀留后续版本）；原 GLOSSARY.md/`docs/adr/` 布局、`GLOSSARY-MAP.md`、懒创建文件段落删除；
- 「Update GLOSSARY.md inline」→ **「就地更新语境词典」**（发现术语即在对话内补词条，不批量攒；格式见 GLOSSARY-FORMAT.md，决策记录见 ADR-FORMAT.md）；
- 「Cross-reference with code」→ **「对拍已确认事实」**（用户已确认的原话与结论是唯一对照源；发现矛盾即指出「你之前说 X，现在说 Y，以哪个为准？」）；
- 「GLOSSARY.md 应完全不含实现细节」→ 「语境词典只收语义与用法，不收实现细节」；
- ADR 三条件（难逆转/无上下文会困惑/真实取舍）原文保留，「创建 ADR 文件」措辞改「在对话内记一条决策记录」。

- [ ] **Step 3: 换皮 GLOSSARY-FORMAT.md / ADR-FORMAT.md**

格式本体保留（词条字段、决策记录字段不变），标题与引言改为「语境词典词条格式 / 决策记录格式（对话内工件）」；删除「写入文件/路径」类措辞。

- [ ] **Step 4: 验证**

Run: yaml 解析（同 Task 1 形式）→ `yaml ok`
Run: `grep -cE "GLOSSARY\.md|docs/adr|写入文件|落盘" seedify/skills/domain-modeling/SKILL.md` → `0`
Run: `grep -c "语境词典" seedify/skills/domain-modeling/SKILL.md` → `≥3`
Run: `grep -c "难逆转\|hard to reverse\|真实取舍\|trade-off" seedify/skills/domain-modeling/SKILL.md` → `≥2`（ADR 三条件机制在位）
Run: charter sha256 对照 → `1`（同 Task 1 形式）

- [ ] **Step 5: Commit**

```bash
git add seedify/skills/domain-modeling && git commit -m "feat: 引入 domain-modeling 换皮——语境词典（对话内工件，机制保真域换皮）"
```

### Task 3: 引入两入口壳 grill-me / grill-with-docs

**Files:**
- Create: `seedify/skills/grill-me/SKILL.md`（源：`…/productivity/grill-me/SKILL.md`）
- Create: `seedify/skills/grill-with-docs/SKILL.md`（源：`…/engineering/grill-with-docs/SKILL.md`）
- Create: 两目录各 `references/charter.md`（同拷法）

**Interfaces:**
- Produces: 用户触发入口 `grill-me`、`grill-with-docs`（保留 `disable-model-invocation: true` 上游机制：只有人能点，技能间不可调——追问的技能间调用一律走 `grilling`）。

- [ ] **Step 1: 拷贝两壳 + charter**（源路径见 Files）

- [ ] **Step 2: 每壳三处改动**（`Call the Skill tool…` 行**原样不动**）

frontmatter：`description` 保留原有人类摘要，追加中文触发句 + `version: 0.3.0` + `allowed-tools: Read, Task, Agent`；
标题后加归属行；
派发行后追加探测-降级条款（M7 先例措辞）：`若当前环境无 Skill 派发能力，改读 grilling 的 SKILL.md 按其执行（grill-with-docs 再加读 domain-modeling 的 SKILL.md）；探测不到不报错、不阻塞。`

- [ ] **Step 3: 验证**

Run: 两文件 yaml ok；`grep -c "Call the Skill tool" seedify/skills/grill-me/SKILL.md seedify/skills/grill-with-docs/SKILL.md` → 各 `1`
Run: `grep -c "探测不到不报错" …` → 各 `1`
Run: `grep -c "disable-model-invocation: true" …` → 各 `1`
Run: charter sha256 → `1`

- [ ] **Step 4: Commit**

```bash
git add seedify/skills/grill-me seedify/skills/grill-with-docs && git commit -m "feat: 引入 grill-me/grill-with-docs 入口壳——派发行保真+探测-降级条款"
```

### Task 4: 引入 to-spec 换皮「方案席」

**Files:**
- Create: `seedify/skills/to-spec/SKILL.md`（源：`D:/AIGC/research/mattpocock-skills/skills/engineering/to-spec/SKILL.md`）
- Create: `seedify/skills/to-spec/references/charter.md`（同拷法）

**Interfaces:**
- Produces: 技能 `to-spec`（user-invoked 保持 `disable-model-invocation: true`？——**改为 model-invoked**：方案席是链内环节，编排器/seed-clarify 要能派发它；删除该 frontmatter 行，description 改模型面向）。产出=**提示词构建方案**（对话内），供批准门与 seed-gen 消费。

- [ ] **Step 1: 拷贝上游 + charter**

- [ ] **Step 2: 换皮改写**（机制保真：「Do NOT interview the user; just synthesize what you already know」原句保留；其余按下列映射）

frontmatter：删 `disable-model-invocation: true`；`description` 换为中文触发（「把谈透的内容综合成构建方案」「to-spec」「出方案」+ 错字容错句式）+ `version: 0.3.0` + `allowed-tools: Read`；归属行 + 「0. 先读思想纲领」；
- 「publish it to the project issue tracker / ready-for-agent label / setup-matt-pocock-skills」→ 全部删除（tracker 无对位）；产出改为**对话内呈现方案，交批准门**；
- 「Explore the repo…」→ 删除（无对位）；「Use the project's domain glossary vocabulary」→ 「沿用语境词典的术语（若上游建有）」；
- 测试缝 Sketch 步骤（seams）→ 整步删除（无对位）；
- 模板节映射（模板块整替，注释性箭头不进模板正文）：
  - Problem Statement → **意图与问题**（想解决什么、给谁用、为什么——只收用户说过的与已确认结论）；
  - Solution → **构建路线与取舍**（考虑过的路线、各自取舍、选定哪条与为什么；路线须可回指追问席的选定记录）；
  - User Stories → **删除节**（验收期望归种子三要素、方案不代拟）；
  - Implementation Decisions → **构建决策**（造这条提示词的决策：入哪些槽位、留白什么、如何取舍；**用户明确给出的流程原样收录、不升格为约束**（T16）；不写提示词逐句文本——成文归 seed-gen/装配）；
  - Testing Decisions → 删除节（无对位）；
  - Out of Scope → **范围外**；
  - Further Notes → **备注**。
- 文首加边界行：`方案是「造提示词的蓝图」（排练层工件）：不写提示词逐句文本，不进最终交付代码块；方案批准前不写提示词（跳层与单模块直调不受此门约束）。`

- [ ] **Step 3: 验证**

Run: yaml ok；`grep -c "不访谈\|Do NOT interview\|只综合" seedify/skills/to-spec/SKILL.md` → `≥1`（机制在位）
Run: `grep -cE "issue tracker|ready-for-agent|setup-matt|seams" seedify/skills/to-spec/SKILL.md` → `0`
Run: `grep -c "方案批准前不写提示词" seedify/skills/to-spec/SKILL.md` → `1`
Run: `grep -c "不升格" seedify/skills/to-spec/SKILL.md` → `≥1`（T16 在位）
Run: charter sha256 → `1`

- [ ] **Step 4: Commit**

```bash
git add seedify/skills/to-spec && git commit -m "feat: 引入 to-spec 换皮——方案席（提示词构建方案，模板换皮+门规则）"
```

### Task 5: seed-clarify 改薄壳串两席

**Files:**
- Modify: `seedify/skills/seed-clarify/SKILL.md`（全文重写为薄壳，约 30 行）

**Interfaces:**
- Consumes: 技能 `grilling`、`domain-modeling`、`to-spec`（Task 1/2/4）。
- Produces: `/seed-clarify` 与原触发短语继续可用；产物=两席产物（追问共识 → 构建方案），流程体检单点请求兜到追问席。

- [ ] **Step 1: 重写 SKILL.md**（保留 frontmatter name/description 触发语与错字容错句式，`version: 0.3.0`）

正文结构：定位一句话（M2 薄入口：先追问后方案）→ 运行步骤：1 先读 charter；2 取输入；3 **追问**：调用 grilling（有文档沉淀需要时叠加 domain-modeling；无 Skill 派发则 Read 其 SKILL.md 执行——探测-降级同款条款）；4 **方案**：调用 to-spec 把共识综合成构建方案；5 **交批准门**：方案呈现给用户，批准前不写提示词；用户改口从受影响环节重走。流程体检请求=就流程开一轮 grilling（压力测试+建议在问答内），T16 随行。内部纪律：可回指/零丢失/过程痕迹不输出（沿用锚词）。禁止事项：不代行 M3/M5、不落盘、方案不进交付。

- [ ] **Step 2: 验证**

Run: yaml ok；`grep -c "grilling\|to-spec" seedify/skills/seed-clarify/SKILL.md` → `≥2`
Run: `grep -c "流程" seedify/skills/seed-clarify/SKILL.md` → `≥1`（流程体检兜底在位）
Run: `grep -c "先读思想纲领" → 1`；charter sha256 → `1`

- [ ] **Step 3: Commit**

```bash
git add seedify/skills/seed-clarify && git commit -m "feat: seed-clarify 改薄壳——串追问席与方案席，流程体检兜入追问"
```

### Task 6: 编排器串链与门规则

**Files:**
- Modify: `seedify/skills/seedify/SKILL.md:21-29`（第 4 步按需串链、第 7 步交付即止邻近行）

**Interfaces:**
- Consumes: 两席技能名（Task 1/4）。

- [ ] **Step 1: 改写第 4 步链位**

典型全链改为：`seed-express` → `seed-analyze` → `grilling`（追问，可叠 domain-modeling）→ `to-spec`（构建方案）→ **批准门** → `seed-gen` → `seed-tools` → `seed-assemble`。
跳层条目保留并显式标注：`意图已清晰 → 从 seed-gen 起；已有种子核心 → 直接 seed-assemble；跳层与单模块直调不受批准门约束。`
新增门规则行：`走方案席的链路，构建方案须经用户批准才进生成/装配——方案批准前不写提示词；跳层与单模块直调不受此门约束。`
「中途迷路/提示词翻新」两支的表述中涉及追问的环节改为 grilling。

- [ ] **Step 2: 验证**

Run: `grep -c "方案批准前不写提示词" seedify/skills/seedify/SKILL.md` → `1`
Run: `grep -c "跳层与单模块直调不受" seedify/skills/seedify/SKILL.md` → `≥1`
Run: `grep -c "grilling" seedify/skills/seedify/SKILL.md` → `≥1`；yaml ok

- [ ] **Step 3: Commit**

```bash
git add seedify/skills/seedify/SKILL.md && git commit -m "feat: 编排器串两席链位——批准门只管方案席链路，跳层保留"
```

### Task 7: M3/M5 接口 + T 护栏增补

**Files:**
- Modify: `seedify/skills/seed-gen/SKILL.md`（第 2 步取意图）
- Modify: `seedify/skills/seed-assemble/SKILL.md`（第 2 步取材料）
- Modify: `seedify/skills/seedify/references/anti-patterns.md`

- [ ] **Step 1: seed-gen 第 2 步增输入源**

追加一句：`上游可提供提示词构建方案：按方案的构建路线与取舍成文；方案不写逐句文本，种子仍自然生成。`（不改其余纪律）

- [ ] **Step 2: seed-assemble 第 2 步材料清单增一项**

材料枚举加「提示词构建方案（若有）：其构建决策用于槽位安置与留白判断，仍不代行种子成文」。

- [ ] **Step 3: anti-patterns.md 增补**

T16 归属列 `M2 流程体检` → `方案席（to-spec）`；表尾新增三行（编号续 T28–T30，四列同表）：
`T28 | 交付物含构建方案/语境词典 | 交付代码块只有提示词；方案与词典留在对话 | 方案席/装配`
`T29 | 内置件机制被改写 | 设计树/frontier/推荐答案/不访谈等机制点 grep 在位 | 追问席/方案席`
`T30 | 走方案席链路未经批准即成文 | 方案批准前不写提示词；跳层/单模块直调豁免 | 编排器`
T7 行「通过标准」追加 `内置件中文触发与错字变体同测`。

- [ ] **Step 4: 验证**

Run: `grep -c "T28\|T29\|T30" seedify/skills/seedify/references/anti-patterns.md` → `3`
Run: `grep -c "构建方案" seedify/skills/seed-gen/SKILL.md seedify/skills/seed-assemble/SKILL.md` → 各 `≥1`；yaml ok ×2

- [ ] **Step 5: Commit**

```bash
git add seedify/skills/seed-gen seedify/skills/seed-assemble seedify/skills/seedify/references && git commit -m "feat: M3/M5 纳入方案输入源 + T28–T30 护栏增补（T16 改挂方案席）"
```

### Task 8: 版本 0.3.0、README、THIRD-PARTY-NOTICES、mod-design 勘误

**Files:**
- Modify: `seedify/.claude-plugin/plugin.json`（version）
- Modify: 全部 13 个 SKILL.md 的 `version:` 行（8 现役中未改的 + 核对 Task 1–5 已改）
- Modify: `seedify/README.md`（版本、目录树、命令梯子中 /seed-clarify 说明、新增技能段）
- Create: `THIRD-PARTY-NOTICES.md`（仓库根）
- Modify: `docs/seedify-mod-design-v1.md`（§4.1 M2 行加勘误指针）

- [ ] **Step 1: 版本对齐**

`plugin.json` → `"version": "0.3.0"`；`grep -rln "version: 0.2.1" seedify/skills/ | xargs sed -i 's/version: 0.2.1/version: 0.3.0/'`；内置件若缺 version 行则补。

- [ ] **Step 2: THIRD-PARTY-NOTICES.md**

内容：上游 mattpocock/skills（MIT，Copyright (c) Matt Pocock，https://github.com/mattpocock/skills）；内置清单（五个技能文件 + 两格式文件）；**改动声明**（逐条）：删「including multiple choices」；domain-modeling 文件机制换皮对话内语境词典、cross-reference 换皮对拍已确认事实；to-spec 模板换皮为提示词构建方案、删除 tracker/seams/测试决策节、改 model-invoked；两壳加探测-降级条款与中文触发；各文件加归属行与 charter 入包。

- [ ] **Step 3: README 与 mod-design**

README：版本行/目录树加五席；命令梯子 /seed-clarify 注「薄壳串两席」；说明段加一句两席来源与 MIT 归属。mod-design §4.1 表 M2 行末加 `（2026-10-22 起由 docs/superpowers/specs/2026-10-22-m2-grill-tospec-design.md 取代：两席制）`。

- [ ] **Step 4: 验证**

Run: `grep -rn "0.2.1" seedify/ | wc -l` → `0`；`grep -rn "version: 0.3.0" seedify/skills/ | wc -l` → `13`
Run: `grep -c "Copyright (c) Matt Pocock" THIRD-PARTY-NOTICES.md` → `≥1`
Run: `claude plugin validate .` → `Validation passed`；yaml 全量解析 ok

- [ ] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: 发布工程 v0.3.0——版本双处同步/THIRD-PARTY-NOTICES/README 与 mod-design 勘误"
```

### Task 9: Cowork 重打、全量回归、发布门禁与项目记忆

**Files:**
- Modify: `seedify-skill-cowork.zip`（重打，13 技能）
- Modify: `CLAUDE.md`（待办与架构段更新）
- Modify: 持久记忆（如需）

- [ ] **Step 1: Cowork 包重打**（LF 归一，沿用既有 python zipfile 打法：`*/SKILL.md` + `*/references/*` + 根级格式文件 `GLOSSARY-FORMAT.md`/`ADR-FORMAT.md` 一并入包）

- [ ] **Step 2: 全量回归**（命令与期望）

`claude plugin validate .` → 过；13+8 commands+1 agent frontmatter yaml 全过；`sha256sum seedify/skills/*/references/charter.md | awk '{print $1}' | sort -u | wc -l` → `1`；`grep -rnE "[A-Za-z]:[/\\\\]" seedify/ | wc -l` → `0`；`grep -c "including multiple choices" seedify/` → `0`；包内 charter 哈希 `git show HEAD:…` blob 一致；zip 文件数=13×(SKILL+charter)+2 格式文件。

- [ ] **Step 3: 发布门禁清单追加**（CLAUDE.md 待办段）

三项既有 + 新增三项：①Cowork 实测两壳探测-降级路径；②grill frontier 批问在中文语音输入的真实节奏体验；③新触发（/seed-clarify 薄壳、直接调 grilling/to-spec、错字变体）真实路由。

- [ ] **Step 4: CLAUDE.md 架构段同步**

架构与分工：M2 描述改「两席制（追问席=grill 全家内置、方案席=to-spec 换皮），流程体检撤编吸收」；目录地图/验证基线版本号改 0.3.0；T 列表加 T28–T30；待办记「M2 两席制 v0.3.0 已落地」。

- [ ] **Step 5: Commit**

```bash
git add -A && git commit -m "feat: v0.3.0 收尾——cowork 包 13 席重打/全量回归/发布门禁与项目记忆更新"
```

---

## Self-Review 记录

- 覆盖：设计 §2 目标各条→Task 1–7；§3 约束→Global Constraints；§4 裁定 1–12→Task 1–6 步骤点（裁定 10/11→Task 6/2，裁定 12→Task 1/3）；§5 裁量→Task 8。无缺口。
- 步骤扫描：每步一动作一验证；模板节映射等留实施者判断处已给精确映射表。
- 名称一致：`grilling`/`domain-modeling`/`to-spec` 调用名全文一致；`references/charter.md` 路径一致。
- Review Focus 五行各钉到任务步骤（见各 Task 验证步）。
- 篇幅：计划 < 换皮改写点清单总量，代码块均为可执行命令或模板映射，非程序转写。
