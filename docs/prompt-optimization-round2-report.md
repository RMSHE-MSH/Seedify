# Agent 提示词优化调研 · 第二轮报告

**核验日期**：2026-10-17（星标、活跃度、README 均为当日实时数据）
**信息源**：GitHub API（搜仓库、README 端点）为一手来源；本地《Claude 插件全解》手册交叉参照。
**未能在线核对**：anthropic.com、docs.anthropic.com、developers.openai.com、docs.deepseek.com、web.archive.org 本机不可达，涉及这些站点的细节（如官方文档表述、定价页）一律以仓库 README 自述为准并标注。

---

## 〇、结论先行

**三条件（实践驱动 ∧ 输出哲学克制 ∧ 非冷门）交集仍然为空**，与第一轮判断一致，本轮实时数据没有改变格局，但补上了几个此前漏掉的关键拼图：

1. "帮我优化提示词"这一触发形态，**已被猜测派占位**（prompt-alchemist 的触发词表里明文写着它），而哲学合格的成品全都冷门（5–65★）或形态不对（CLI / 沉淀器）；
2. 最接近交集的是 **BayramAnnakov/claude-reflect（1,814★）**——但它做的是 L3 沉淀而非提示词改写；**skillberry-ai/cap-evolve（65★）**是 L2 方法论的教科书实现（held-out 显著性闸门）但极冷门；
3. 高星质量集中在**引擎层**（GEPA/DSPy/promptfoo/Kiln）与**方法论层**（loop-engineering 11.6k★、12-factor-agents 26.7k★、superpowers 301k★）。
4. 可落地路径仍是**组合**：薄触发技能（自建 L1）+ cap-evolve 或 harness-evolver（L2 验证闸门）+ claude-reflect（L3 沉淀）。

---

## 一、分层清单

### A. 实践闭环成品（优化信号来自执行轨迹 / 评测 / 真实纠错）

| 项目 | 星标（10-17） | 形态 | 优化信号来源 | L1–L3 匹配 |
|---|---|---|---|---|
| skillberry-ai/cap-evolve | 65★ | Python CLI（`pip install ./core`） | 自备评测 + 完整轨迹 + per-task 因果反馈；**held-out 显著性闸门（Δ>k·SE）**、test 集封存 | L2 教科书 |
| BayramAnnakov/claude-reflect | 1,814★ | Claude Code 插件 + mod | **真实使用纠错**（hook 捕获）+ 会话史归纳，人工批准入库 | L3 教科书 |
| NousResearch/hermes-agent-self-evolution | 5,512★ | CLI（Hermes 生态） | GEPA 读执行轨迹；评测源可为 **Claude Code/Copilot/Hermes 真实会话史（sessiondb）**；约束闸门（测试/体积上限/基准） | L2 强匹配 |
| raphaelchristi/harness-evolver | 56★ | Claude Code 插件（`/harness:evolve`） | LangSmith 数据集 + 轨迹洞察/失败聚类 + LLM-as-judge；holdout enforcement、回归守护、Pareto 选择 | L2 完整 |
| bergr7/claude-skill-prompt-optimizer | 5★ | **纯 SKILL.md（可被一句话触发）** | 人提供测试用例+打分器做质量闸门；baseline→诊断→改→keep/revert→防回归 | L1–L2 全对齐 |
| gaasher/Agent-Loop-Skills | 175★ | 可嵌入 agentic loop 技能库 | 每步由客观 feedback signal 裁决，verification-gated + 显式终止，回归丢弃 | L2 方法论成品 |
| romiluz13/cc10x | 164★ | Claude Code 插件（1 router·9 agents） | Phase-exit 闸门、test honesty 门、anti-anchored review（"engineer the loop, not the prompt"） | L2 执行引擎 |
| chigwell/Penelopa.ai | 140★ | 桌面应用 + 云仪表盘 | 真实会话事件（Stop/SessionEnd hooks）→ 改进建议 | L3 观察者 |

**逐项说明与局限：**

- **cap-evolve**：README 原话——"evaluate → diagnose the failures → propose an edit → keep it only if it beats a held-out split by a significant margin → commit"，"Every iteration reads full trajectories and per-task causal feedback (which task ids a prior edit broke and fixed)"，"Acceptance is a val-only significance gate (Δ > k·SE)"。优化对象不止提示词：系统提示、工具代码、MCP 面、整个技能包可联合优化。**局限**：65★ 冷门；CLI 形态，需自备评测与 agent 接口；不能直接被"帮我优化提示词"触发（需薄技能包装调用）。当日仍在推送（10-17）。
- **claude-reflect**：hook 捕获你对 Claude 的纠正（任意语言），`/reflect` 沉淀为规则同步 CLAUDE.md，`/reflect-skills` 从会话史（示例：68 个会话）归纳可复用命令/技能并**经你批准**；"Corrections during /deploy improve the deploy skill itself"（纠错直接改进技能本体）。支持 Codex/Cursor（经 AGENTS.md）。**局限**：是沉淀器不是评测闸门式优化器；模式发现是 AI 归纳，靠人工批准把关。
- **hermes-agent-self-evolution**：GEPA 读轨迹"理解失败原因而不只是失败结果"（ICLR 2026 Oral）；约束闸门含**体积上限**（防膨胀）；无 GPU、约 $2–10/次优化。**局限**：面向 Hermes Agent 的技能/提示词/代码进化；最后推送 2026-06-17（约 4 个月未动）；跨生态复用需自行适配。
- **harness-evolver**：`/harness:setup|health|evolve|status|deploy` 全套命令；进化对象是整个 agent 代码库（prompt/路由/工具/架构）；"Smart Gating: constraint gates, efficiency gate (cost/latency pre-merge), regression guards, Pareto selection, holdout enforcement"。**局限**：56★；依赖 LangSmith；LLM-as-judge 属弱信号；2026-04 后未推送。
- **claude-skill-prompt-optimizer**：**形态上是最贴题的成品**——"The human provides the quality gates (test cases + scorers). The machine iterates toward them"；带预算分档（先便宜后全量）与 revert-on-regression 纪律。**局限**：5★ 极冷门、单人项目、需自备 eval runner。
- **Agent-Loop-Skills**：核心主张"get most of their power not from one clever prompt but from iterating against feedback"；五要素 "program · artifact · feedback signal · run ledger · termination"。**局限**：给任意任务的循环套件（含 optimize-loop、alpha-evolve），不是提示词专项优化器。
- **cc10x**：与你们哲学同源（确定性放循环不放提示词），状态落盘抗压缩。**局限**：它是执行循环引擎，不改提示词；重型工作流。
- **Penelopa.ai**：信号是真实会话（实践驱动成立），但**会话数据上传其云端服务**（隐私考量），建议只进仪表盘不自动改提示词。

**哲学正确但极冷门的碎片**（据仓库简介归类，未逐份读 README）：davismartens/ev（4★，CLI 评测+自动提示词优化）、abhishekgahlot2/agent-optimization（3★，"without benchmark hacks"）、studiomeyer-io/darwin-agents（10★，A/B 自进化）、xiaoshiyilangzhao1996-droid/evolution-engine（8★，五层：评测→规则沉淀→Prompt→Skill 进化，复刻 Hermes）、lendtrain/autoresearch-for-skills（1★）、angboontiong79/claude-skill-self-improvement（1★）、bfernandez31/prompt-eval（0★）。共同局限：无社区量级，仅作方法参照。

### B. 引擎层（提供进化算法 / 测量能力，需编程或产品形态接入）

| 项目 | 星标 | 角色 | 信号机制 | 局限 |
|---|---|---|---|---|
| stanfordnlp/dspy | 38,667★ | 优化框架（含 MIPRO 等） | 指标 + 反射式进化 | 库形态，须自建评测 |
| gepa-ai/gepa | 7,021★ | 反射式进化引擎（ICLR 2026 Oral） | "Optimize any text parameter…LLM-based reflection and Pareto-efficient evolutionary search"；"If you can measure it, you can optimize it" | 信号质量取决于你给的 metric；不天然保证种子式克制（可加体积/约束目标对冲） |
| promptfoo/promptfoo | 26,084★ | 评测/红队引擎 | 测试用例 + 攻防扫描 → 提供 L2 闸门信号 | 本身不进化提示词，是"测量"层 |
| Kiln-AI/Kiln | 5,195★ | 一体化工作台 | Eval Builder（judge+合成数据）、Auto-Optimize（搜数百候选）、人类偏好评分 | **Auto-Optimize/AI Assistant 属 Kiln Pro 付费档**；面向 AI 产品开发的桌面工作台 |
| mshumer/gpt-prompt-engineer | 9,689★ | 经典进化式提示工程（Colab） | 用户测试用例 → 生成候选 → 全量测试 → ELO 排序 | 2023 时代的单提示词搜索，非 agent 循环；候选生成靠 LLM 构想 |
| SylphAI-Inc/AdalFlow | 4,219★ | 自动优化库 | 库内建 auto-prompt/auto-ICL | 库形态 |
| zou-group/textgrad | 3,762★ | 文本反传 | "textual gradients" 回传 | 库形态；2025-07 后未推送 |
| OmidZamani/dspy-skills | 124★ | DSPy 使用技能包（22 个技能） | 教 agent 用 DSPy 优化 | 引擎包装，非优化器本身 |
| hinthornw/promptimizer / preset-io/promptimize / shobrook/promptimal | 897/497/300★ | 小型优化器/评测工具箱 | 各异（前者自述 "scratch"） | 量级小或成熟度低 |
| Piebald-AI/tweakcc | 2,548★ | 改 Claude Code 本体系统提示词/工具集 | —（改写工具，非优化器） | 只解决"改哪里"，不解决"怎么验" |

### C. 沉淀类（L3）

- **BayramAnnakov/claude-reflect（1,814★）**：主力，见上。只把真实纠错沉淀为规则/技能，天然不沉淀猜测——与"L3 只固化被实践证实的经验"完全同构。
- **本地插件手册记载：ruflo-metaharness**（ruflo 生态，本地市场）——13 个技能含 **harness-learn（GEPA，默认 $0 干跑）**、harness-evolve（Darwin 策略进化）、harness-drift-from-history（漂移检测）、flywheel promote/evidence-reset 需确认。属"沉淀+进化"混合体。*说明：此条来自本机《Claude 插件全解》手册记载，上游 MetaHarness 包未能在 GitHub 实时核验星标。*
- **obra/superpowers（301,036★）** 的 writing-skills / consolidate-memory 等技能族提供"技能化沉淀"的方法骨架（如何把经验写成可进化技能）。

### D. 明确排除的猜测派（附排除理由）

| 项目 | 星标 | 排除理由（证据） |
|---|---|---|
| Jakeschincariol/promptmaster-skill | 39★ | **病灶 1+3 实锤**。README 自带反面示例：用户说 "write me something for instagram, idk"，它改写成 "Write a 30-day Instagram plan for a local coffee shop with **8k followers**. One table: week, theme, 3 post ideas. Under 400 words."——咖啡店、8k 粉丝、30 天、400 词全是**臆造**；格式与字数是**前置硬约束**。信号来源为"35 条 credit 浪费模式"静态清单+对话推断，无执行/评测。 |
| Cherish133/prompt-alchemist | 15★ | **病灶 2 原型**："融合四位大师方法论"的四步蒸馏、诊断报告+成品一次性输出。值得记一笔：它自带 10 题测试集、12 轮优化历史、失效模式手册（FM-1~7），是猜测派里最自省的；但优化信号仍是方法论迭代+对话诊断，无用户任务评测闸门，且**其触发词表正占用"帮我优化提示词"**。 |
| tolimarchuk/promptify-skill | 46★ | `/promptify [+ask+deep+web]` 前置重写提示词：信号是"最佳实践+澄清问题"（`+deep` 翻代码库、`+web` 搜惯例），无评测无轨迹；产物倾向结构化长提示词（流程/格式前置），压制 agent 在循环中的自主发现。 |
| fish2018/YPrompt | 1,723★ | 中文生态里星标最高者，但"通过对话挖掘用户需求生成提示词"为主信号（有 A/B 效果对比、版本管理等弱实证功能）；且是 Web 端绘图/写作提示词工具，**形态不匹配** agent 编码工具的技能/插件。不入合格交集。 |
| Rtur2003/Claude-Code-Promts-Skills | 53★ | 静态提示词库（"APEI methodology, ~35 specs"），是资源库不是优化器。 |

（promptmaster、prompt-alchemist、promptify 三份 README 已逐句读过并留档；YPrompt、Rtur2003 按仓库简介归类。）

### E. 背景层：harness 提示词体系与 Loop 工程方法论（供坐标参照）

| 项目 | 星标 | 一句话 |
|---|---|---|
| cobusgreyling/loop-engineering | 11,568★ | "Loop Engineering" 方法论+starters+CLI：设计系统去发现工作、交给 agent、验证结果、持久化状态；"L1 report → L2 assisted → L3 unattended，**verifier 对了一周再放开下一级**"——与你们 L1–L3 路线同名同构，是现成的参照系 |
| humanlayer/12-factor-agents | 26,742★ | 构建 LLM 软件的原则集（含提示词与可观测性因子） |
| HKUDS/DeepCode | 16,724★ | 开源 agentic coding（harness + 循环工程 + 多智能体编排），实现层参考 |
| wquguru/harness-books | 3,189★ | harness 工程两本书：约束、查询循环、上下文治理 |
| obra/superpowers | 301,036★ | 技能框架+软件开发方法论（TDD、验证后交付、技能写作法） |
| deepseek-ai/deepseek-harness | 251,880★ | "Everything is a Plugin"，dsh-system-prompt 插件化注册表（第一轮已核） |
| openai/codex | 129,615★ | 开源按模型系统提示词（第一轮已核） |
| michaelshimeles/ralphy | 2,976★ | Ralph Wiggum 裸循环跑法代表（多 harness 编排脚本）——"循环即一切"的极简对照组 |
| valkor-ai/loom | 1,251★ | 面向 agentic 交付的循环工程（状态/验证/修复记录） |

---

## 二、诚实结论：三条件交集评估

1. **严格交集 = 空。** 具体地：
   - "实践驱动 ∧ 克制"的成品（cap-evolve 65★、claude-skill-prompt-optimizer 5★、harness-evolver 56★）全部**冷门**；
   - "实践驱动 ∧ 非冷门"的两个（claude-reflect 1,814★、hermes-agent-self-evolution 5,512★）**形态不完全对题**：前者是 L3 沉淀器不是提示词优化器；后者是 Hermes 生态 CLI 且 4 个月未推送；
   - "可被'帮我优化提示词'触发 ∧ 非冷门"的（promptify 46★ 一族）全是**猜测派**，直接违反核心立场。
2. **高星的都是引擎层与方法论层**——DSPy/GEPA（进化算法）、promptfoo/Kiln（测量）、loop-engineering/12-factor/superpowers（方法论）。这印证第一轮判断："提示词优化"作为可安装成品在 agent 编码工具生态里尚未出现三条件齐备的头部项目；头部玩家都把确定性放在**评测与循环**（L2），而不是提示词文本本身。
3. **哲学正确性与流行度呈负相关**，当前窗口内不可兼得——如实报告，不降标准凑数。

---

## 三、建议

### 方案 A（推荐，一天内可落地）：三件套组合

| 层 | 选用 | 接法 |
|---|---|---|
| L1 种子化 + 触发 | **自建薄技能**（规格见下） | SKILL.md frontmatter description 写入触发短语："帮我优化提示词"、"优化这个 prompt/skill"、"种子化"——Claude Code 技能匹配自动加载 |
| L2 循环进化 | **cap-evolve CLI**（held-out 显著性闸门） | 技能内用 Bash 调用 `cap-evolve`；无评测时技能降级为纯 L1 并明确告知"未经验证" |
| L3 沉淀 | **claude-reflect 插件**（1.8k★） | `claude plugin install claude-reflect@claude-reflect-marketplace`；其 hook 自动捕获真实纠错，`/reflect` 写回 CLAUDE.md/技能 |

要点：触发短语目前被 prompt-alchemist 等猜测派占用，自建技能的 description 要写得更具体（含"只做压缩不新增""评测闸门"等判别词），让技能匹配在竞争条目中选中你自己的。

### 方案 B（重仓进化整个 agent）：harness-evolver 插件

愿意接受 LangSmith 依赖时：`/plugin marketplace add raphaelchristi/harness-evolver-marketplace` 后 `/harness:evolve`。优化对象是提示词+路由+工具+架构全家桶，holdout/回归闸门齐全。注意 4 个月未维护，先在分支上试。

### 方案 C（引擎直用）：GEPA/DSPy + promptfoo

自建评测（promptfoo 出测量信号）→ GEPA 反射进化（可加体积上限等约束目标对冲"越优化越长"）→ 闸门通过才合入。适合把提示词优化做成 CI 的团队。

### 自建技能规格：`seed-loop-optimizer`（可选后续，仅规格）

- **目标**：把草稿提示词压缩为 L1 种子（目标 + 硬边界 + 验收标准）；有评测/轨迹时跑 L2 进化；只把闸门通过的经验写回 L3。
- **触发词**："帮我优化提示词"、"优化这个 prompt/skill"、"optimize this prompt"、"种子化"。
- **硬规则**（违反即为 bug）：
  1. **只压缩不新增**：禁止写入用户未说过的角色、业务背景、流程、示例（病灶 3 清零）；
  2. 输出只含**目标 / 硬边界 / 验收标准**三要素；软细节（步骤、格式、示例）删除，或降级为标注"非约束参考"的附录；
  3. **禁止角色堆砌与前言**：产物中出现"你是一位资深…"即违规（病灶 2 清零）；
  4. 任何**增补**必须标注信号来源（轨迹 ID / 失败用例 / 用户纠错原文），无来源不许写入；
  5. 改动须过**验证闸门**才算数：held-out 显著（Δ>k·SE），或全量用例 + 防回归（原有通过用例不得转坏）；
  6. 无评测信号时**只做 L1 种子化**并明说"未经验证"，不虚构优化建议；
  7. L3 写回仅限被 ≥2 次实践证实且过闸门的规则，写入 memory/CLAUDE.md 前列 diff 供人审。
- **验证方式**：优化前后在同一评测集（或 3–5 个真实任务）对比通过率；改动 diff 逐行附信号来源供人审；防回归清单；预算上限（迭代轮数/费用）。

---

## 附：核验与留档

- 星标与活跃度均为 2026-10-17 经 GitHub API 实时取得（search/repositories）；README 经 api.github.com/repos/*/readme 逐份读取，留存于 `scratch/prompt-research/README-*.md`（14 份）。
- 第一轮旧数据本轮全部复核，无实质变化（hermes 5.5k、dspy 38.7k、gepa 7k 等同量级）。
- 未能在线核对的站点：anthropic.com、docs.anthropic.com、developers.openai.com、docs.deepseek.com、web.archive.org（本机不可达）；ruflo-metaharness/MetaHarness 上游未在 GitHub 核验。
