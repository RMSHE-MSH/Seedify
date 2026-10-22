# mattpocock/skills 调研留档（round4 · 2026-10-22）

> 对象：https://github.com/mattpocock/skills （MIT，Matt Pocock / AI Hero）
> 本地克隆：`D:/AIGC/research/mattpocock-skills`（2026-10-22，HEAD 5beb35b，v1.3.1）
> 研究动机：mod 设计 §9 已借鉴其 grill 系列追问方法论（M2）；本次全面研究其架构与写作方法论，供 Seedify v0.3+ 优化讨论。
> 定位一句话（其 README）："Skills For Real Engineers - real engineering, not vibe coding." 反 GSD/BMAD/Spec-Kit 的「框架接管流程」路线，主张技能要 **small, easy to adapt, composable**。

---

## 1. 仓库是什么

Matt Pocock（Total TypeScript / AI Hero 作者）日常使用的 agent 技能集，约 37 个技能、191 个文件。每条技能都对应他观察到的一种 agent 失败模式。四大失败模式与对应解法（README 主叙事）：

| 失败模式 | 解法 | 技能 |
|---|---|---|
| ① Agent 没做我想要的（错位） | **grilling 追问会**：开工前把设计树每个分支问清 | grill-me / grill-with-docs |
| ② Agent 啰嗦（术语漂移） | **共享语言**：GLOSSARY.md + ADR，项目行话词典 | grill-with-docs / domain-modeling |
| ③ 代码不能跑（盲飞） | **反馈回路**：红绿 TDD、纪律化 debug | tdd / diagnosing-bugs |
| ④ 泥球架构（熵加速） | **设计纪律**：deep module、测试缝、架构巡检 | to-spec / codebase-design / improve-codebase-architecture |

## 2. 仓库分层

```
skills/
├── engineering/   20 技能（日常代码工作，promoted）
├── productivity/   7 技能（非代码工作流，promoted）
├── misc/           冻结不维护（frozen，issue 全关）
├── in-progress/    beta：公开求反馈，不进插件包
└── deprecated/     不再使用
docs/<bucket>/    每个 promoted 技能一页人类文档（发布到 aihero.dev/skills-<name>）
.agents/          仓库方法论：invocation.md（调用二分法）、writing-docs.md（文档页写作）、ADR×2
.out-of-scope/    9 份「已否决」档案——每份记一个被拒概念与完整理由
.changeset/       变更集（发版纪律）
CLAUDE.md         仓库约定（桶规则、plugin.json 同步、禁 em-dash 等）
SCOPE.md          收录门槛：①真实会话里观察到的失败 ②契合哲学；假设性改进不收
```

- **晋升制**：`engineering/`+`productivity/` = promoted（进插件、有文档页、被路由）；`misc/` 冻结的理由是「没在日常用就无法判断修复对不对，维护成本与 promoted 相同、收益却小」。
- **安装**：Claude Code 走官方插件（自更新）；Codex/Copilot 走 marketplace；skills.sh 分发**可编辑副本**（「装一种就好，都装会装两遍」）。双 harness 适配：`disable-model-invocation: true`（Claude Code）+ `agents/openai.yaml` 的 `policy.allow_implicit_invocation: false`（Codex），两处必须同步。
- **发布口径**：插件只装 promoted 集合；`claude plugin validate . --strict`。

## 3. 核心方法论（最有价值的部分）

### 3.1 调用二分法（.agents/invocation.md）

技能只按**一个轴**分类——谁能调用：

- **user-invoked**（用户独占）：只有人打字能触发（`disable-model-invocation: true`）；description 是**给人看的**一行摘要、剥掉触发短语；职责是**编排**。零 context 负载，但吃认知负载（「你就是索引」）。
- **model-invoked**（模型可达）：默认；description 是**给模型看的**、带丰富触发短语；承载**可复用纪律**。description 常驻上下文 = 永久 context 负载换可发现性。
- **不变量**：user-invoked 可以调用 model-invoked，**绝不**调用另一个 user-invoked。派发用显式指令 `Call the Skill tool with "grilling"`（比散文里提 `/skill` 命中率高——这是他们踩过的坑，grill-with-docs 最高频 bug）；一次调一个技能（"Call the Skill tool twice, for X and Y"）；前置是 user-invoked 技能时只能对人说「请运行 /setup-xxx」。
- **共享参考**：两个 user-invoked 都需要的参考放不进任何技能（谁也点不着谁）→ 推到技能系统之外的普通文件。
- **router 技能**：user-invoked 多到记不住时，一个路由技能（ask-matt）收拢认知负载——只能提示，不能代发。

### 3.2 grilling 追问原语（31 行，M2 的方法论源头）

- **设计树**：每个决策分叉出挂在它下面的决策。
- **轮次 + 边界（frontier）**：一轮问完整个 frontier——所有前置已定、**现在问不靠猜**的问题；编号 + 每题附**推荐答案**；「把问题措辞成『是』= 接受推荐答案」。答案重塑树，重算 frontier 进下一轮；依赖本轮未决问题的问题排到下一轮。
- **事实归 agent，决策归用户**：问题需要环境事实就派 sub-agent 去查，不问用户能自查的事；查探不阻塞提问（只有下游问题等它）。
- **收敛条件**：frontier 清空——每个分支都访问过、没有静默假设——才算完；用户确认共识前不行动。
- **提问形态**（.out-of-scope/native-question-tool.md）：**纯聊天文本**，明确拒绝 AskUserQuestion 类选择器——「结构化选择器把模型推向带预置选项的多选题，恰恰与 grilling 的目的相反：用自己的话回答的开放问题，一次决策树的一个分支」。如需某 harness 的提问 UI，写进用户自己的 CLAUDE.md，不进共享技能。
- **无提问上限**（.out-of-scope/question-limits.md）：「有的计划要 3 问，有的要 50 问」；上限会混淆两种失败——真欠规格（该问）vs 冗余低质问（提示词质量问题，修提示词不设计数器）。逃生口本来就自然语言：用户随时叫停、随时说「收尾总结」。

### 3.3 writing-for-agents：为 agent 写文档的元理论

「包装不同，写法相同：同一批杠杆让文档可预测，因为 agent 每次走同样的**过程**，而非产出同样的输出。」核心杠杆：

- **context pointer**：引出上下文外材料的引用 + 触发条件；**措辞决定到达率**（弱措辞 = variance bug，先磨措辞，磨不动再内联）。pointer 每词常驻付费，比正文更狠地剪：一枝一触发词、砍正文已带的身份信息。
- **两种负载**：context load（agent 窗口，常驻税）vs **cognitive load**（人类索引——「不是要最小化的成本，是人类能动性的价格；花在需要人类判断处，删掉不需要处」）。
- **信息层级阶梯**：in-file step → in-file reference → disclosed reference（外置 + pointer）。渐进披露保护层级（**按分支测**：每条分支都要的内联，只有部分分支到的外置）；co-location（一概念的定义/规则/告示聚在一处）；sprawl（每行都活着但太长）为失败态。
- **完成判据**：每步落在一个判据上。**清晰度**（分得清 done 与否，防「premature completion」）+ **要求度**（驱动 legwork，不阶于步骤、也绑得住扁平参考的穷尽性）。最强判据可检查且穷尽。
- **leading words**：借用预训练里已有的紧凑概念（_seam_、_red_、_tracer bullets_、_fog of war_、_relentless_、_tight_）当思考锚——「作为 token 重复，绝不作为句子重复」，分布式累积定义。自造词要付定义税；优先找已有词。例：「fast, deterministic, low-overhead」→ _tight_；「a loop you believe in」→ _red_。
- **否定失效律**：「用禁止导向会把被禁行为拖进上下文、让它更可用——不想大象，大象就是全部」。**正向表述优先**（「写单行注释」而非「别写长注释」）；禁止只配硬护栏，且必须配对正向目标。
- **剪枝**：单一事实源（重复 = 维护税 + 阶梯虚高）；**环境也是事实源**（文档重述环境 = cache，只缓存查不到的东西：未成文约定、选择理由、坑）；逐行查相关性；防 **sediment**（加安全删危险的默认沉淀）；猎 **no-op**（模型默认就会做的指令 = 说空话；判定是**模型相对**的，跑一遍裁定而非辩论；弱词 no-op 的修法是换强词 _relentless_，不是换技术）。
- **技能专属**（SKILL-MECHANICS.md）：分不分成两个技能看**调用切**（独立触发词/别的技能要够得着才值得付 description 税）或**序列切**（后步诱惑当前步抢跑时外置）；router 技能收拢 user-invoked 记忆负担。
- **文档页**（.agents/writing-docs.md）：页面 ≠ 技能 ≠ SKILL.md 副本；给人卸认知负载。固定框架：What it does（必写 **defining constraint**——「一条事实说清这技能为何不同于显然默认」，例：to-spec 不再访谈、只综合已知）/ When to reach for it（调用方式 + 触发边界，易混时给兄弟对句）/ Where it fits（角色四选一：链步/一次配置/周期维护/随时独立 + 邻居 + 指回 router）。**Common questions 只收真实问题**（wiki/issues/CHANGELOG 里挖，「观察到的问题永远胜过编造的」，数量对证据诚实）；「It's working if」每条不打开 SKILL.md 就能自查；不提作者名；多路分支一律表格/列表不进段落。

### 3.4 反目标文化（.out-of-scope/）

每份否决档案 = 概念 + 完整理由 + 曾被请求记录（附 issue 号）。代表性裁定：

| 被拒概念 | 理由精华 |
|---|---|
| 新技能提案一律不收 | 「能从现有技能组合出的行为不配新技能」；每个技能都是永久维护面 |
| 用 harness 原生提问 UI | 破坏开放问法（见 3.2） |
| 提问数量上限 | 破坏穷尽追问（见 3.2） |
| Jira/Linear 等更多工单后端 | 每后端 = 永久 CLI 维护面；「只维护自己能保持正确的」；其余走「Other：一段散文描述你的流程」 |
| 因 harness 内建撞名而改技能名 | 追移动靶会断肌肉记忆；用命名空间 `/mattpocock-skills:research` |
| 技能内防 sub-agent 递归护栏 | harness 的职责；散文护栏费 token 还靠模型听话 |
| setup 技能加 verify 模式 | prompt-driven 本来就支持「只查不改」口令；再加开关 = 两份漂移 |

### 3.5 主链与技能全景

**主链（idea → ship）**：`grill-with-docs → to-spec → to-tickets → implement(×tdd) → code-review → retro`；两支 on-ramp：prototype 岔路（handoff 双向桥接）、multi-session 岔路（wayfinder / implement-spec）。

关键技能速记（完整清单见其 README）：

| 技能 | 形态 | 精华 |
|---|---|---|
| grilling | model | 设计树/ frontier 轮询/推荐答案（31 行原语） |
| grill-me / grill-with-docs | user | **7 行纯派发壳**："Call the Skill tool with/ twice for..." |
| domain-modeling | model | 术语挑战/场景压力测试/ADR 三条件（难逆转+无上下文会困惑+真实取舍）/GLOSSARY 只收词义不收实现 |
| to-spec | user | **不再访谈只综合**；spec 模板（问题/方案/用户故事长列表/实现决策**不写文件路径**/测试决策/范围外）；测试缝越多越糟、理想是一 |
| to-tickets | user | tracer-bullet 垂直切片（一片=一个新鲜上下文窗口）；阻塞边；宽重构走 expand-contract；切完**考用户**粒度 |
| implement / implement-spec | user | 薄编排：tdd@已议缝→code-review→提交；整 spec 版 = 任务图 frontier + 并行 implementer sub-agent（各自 worktree）+ merger，通信只走 context pointer |
| code-review | model | **双轴并行 sub-agent**（Standards: 仓库标准 + Fowler 12 味基线，判断题不硬违规；Spec: 缺失/超纲/做错）——「不合并重排两轴发现，分轴报告防一轴掩盖另一轴」 |
| tdd | model | 红绿循环；**缝先与用户议定**（一缝一行注：抓住什么/漏掉什么）；反模式：实现耦合/同义反复/横切→竖切 tracer bullet；重构不入环 |
| diagnosing-bugs | model | **Phase 1 造反馈回路就是整个技能**（10 种造法排序）；「收紧回路」当产品做（快/信号尖/确定）；「30 秒抖动的回路≈没有」 |
| research | model | 12 行：后台 agent + **一手来源**（「每个论断追到拥有它的来源」）+ 一份带引用 markdown |
| wayfinder | user | 多会话巨型工作规划：目的地/共享地图/决策工单/边界 frontier/**fog of war**（能说清问题才立票，不看能否回答）/out-of-scope 永不毕业/HITL vs AFK 票型 |
| prototype | model | 「扔掉的代码回答一个问题」；问题定形态（LOGIC/UI 二支）；答案沉淀进主线、原型存一次性分支作一手来源 |
| wizard | model | 生成走人工步骤的 bash 向导（凭证/面板/迁移）；**模板库与 STAGES 作者分界**——UX 库永不手改，agent 只写 stage；「agent 能自己做的步骤不走向导」 |
| to-questionnaire | user | 「**Grill the send, not the subject**」——只问发送对象/要什么答案；问项对准知识缺口，一题一义 + 答案桩 + why-this-matters |
| wait-what | user | 7 行理解断路器：「没听懂，重讲：补上下文、简明技术英语、用 GLOSSARY 词」 |
| handoff | user | 会话→交接文档存临时目录；「suggested skills」节；**不重复已有工件只给指针**；脱敏 |
| writing-fragments/shape/beats | user | 写作 explore/exploit 对：碎片挖掘（leading word 是最值钱碎片）/逐段成形（grounding 理论：先验 vs 内文落地；「原料是采石场不是剧本」）/beat 冒险式成文 |
| chief-of-staff | user | 长目标单会话协调 sub-agent：战术/战略双轨；**wiki 记忆**（index+页+Ingest/Query/Lint，「写历史不写现在」，引用不搬运来源） |
| ask-matt | user | 路由：「你记不住所有技能，那就问」；摘要只为定位，推荐前先读 SKILL.md |

## 4. 与 Seedify 的对照

### 4.1 独立印证（两条核心裁定被第三方用不同路径抵达）

| Seedify 裁定 | mattpocock 对应物 | 理由同构度 |
|---|---|---|
| #2 开放问法、禁填空模板 | native-question-tool 否决案：「结构化选择器推向带预置答案的多选，与 grilling 相反：用自己的话回答的开放问题」 | **几乎逐句同构** |
| #6 有疑必问、不限次数 | question-limits 否决案：「3 问或 50 问都有」；且区分「真欠规格 vs 冗余问」两种失败 | 高度同构，且更进一步（区分两种失败） |
| 提示词是种子不是蓝图 | wayfinder「Plan, don't do」/ prototype「throwaway answers a question」/ to-spec「不写文件路径会过期」 | 哲学同族：交付方向与决策，不预写施工 |
| 总纲 v4 冻结原文入包 | SCOPE + .out-of-scope 一票否决制、changeset 发版 | 同为「我方不得改」的治理意识 |
| T1–T27 行为护栏 | code-review 的 smell 基线、diagnosing-bugs 的阶段门、docs 的「It's working if」 | 同为「行为护栏≠效果验收」 |

### 4.2 差异（各有权衡，非优劣）

| 维度 | Seedify | mattpocock | 备注 |
|---|---|---|---|
| 提问形态 | 一次一主题、纯开放 | frontier 整轮批问、编号、**每题附推荐答案**、yes=接受推荐 | 他们减用户往返；推荐答案有锚定风险（与裁定#3 精神有张力）——**留给 M2 讨论** |
| 收敛条件 | 有疑必问、不限次数（无显式终止） | 无上限 + **frontier 清空即收敛** | 「不限」与「有收敛条件」可并存 |
| 少样本 | 裁定#3 正反例一律不进上下文 | 模板内嵌 example（`<user-story-example>` 等） | 分歧点：防锚定 vs 定输出形状；Seedify 脚手架槽位（不填样例）是中间态 |
| 正向/禁止表述 | 技能含大量「禁止事项」节 | 否定失效律：正向优先，禁止只配硬护栏+正向目标对 | Seedify 禁止项多属硬护栏（合法），但可审计「可正向化的软禁止」 |
| 依赖面 | 零依赖（Hooks/MCP=0，不引 MCP） | 依赖 gh/glab CLI、issue tracker、worktree 工作流 | 他们面向代码工程；Seedify 面向提示词生成，零依赖方针不必动摇 |
| 运行时知识 | charter 原文入包（8 份副本） | 无包内宪法；GLOSSARY/ADR 是**项目运行时**产物 | 两者解决不同问题：一个守纪律，一个守项目语境 |
| 调用架构 | 命令薄封装 → 7 mod + 编排器 | user-invoked（编排）/model-invoked（纪律）+ 显式 Skill tool 派发 + router | **最值得吸收的结构差异**（见 5.1） |

### 4.3 可借鉴清单（供 v0.3+ 讨论，均不动七项裁定）

1. **调用二分法的治理语义**：Seedify 的 7 mod 全部可被模型自主触发+命令触发，没有「谁独占触发」的概念。他们的不变量（user-invoked 不被任何技能调用）给「编排入口唯一性」一个可验证的定义。/seedify 与 /seed-* 命令的边界可借此表述。
2. **显式派发措辞**：「Call the Skill tool with X」而非散文提及——他们实测散文提及命中率差（grill-with-docs 最高频 bug）。Seedify 编排器「按需串链」的措辞可对照审计。
3. **frontier 轮询式追问**（M2 效率升级候选）：依赖已决的才问、一轮一批、编号管理——对语音输入用户可一轮答多题，比一次一题少往返；与「一次一个主题」的兼容性需要裁定。
4. **推荐答案机制**（M2 摩擦降低候选）：每问附推荐、yes=采纳。优点是倒逼 agent 先想清楚；风险是锚定。若引入，需用户裁定是否违反裁定#2/#3 精神。
5. **收敛条件**（M2 补全候选）：「不限次数」之上补「设计树无未访分支即收」的终止语义，成本为零、行为更可控。
6. **完成判据理论**（写 SKILL.md 的通用杠杆）：每步一个可判定的 done 条件 + 要求度（穷尽性），防提前收工。Seedify 各 mod 运行步骤可对照加固。
7. **leading words 纪律**：种子提示词也受益——用预训练里已有的强词（_seed_、_charter_、_frontier_ 之类）当锚，替代长描述；弱词 no-op 换强词。
8. **否定失效律**：审计各 mod「禁止事项」，可正向化的软禁止改正向句；硬护栏保留但配正向目标。
9. **.out-of-scope 否决档案**：Seedify 有「裁定记录」但无「明确拒收的提案档案」。随社区使用增长，一份「我们不做什么+为什么」档案是低成本高价值的边界防护。
10. **文档页的 defining constraint / It's working if**：若未来给 Seedify 做用户文档，这两节格式直接可用（说清「与显然默认的差异」+ 用户可自查的生效信号）。
11. **推荐问题只收真实的**：M2 追问话术库若做评估集，「观察到的问题胜过编造的」是取材纪律。
12. **router 模式**：ask-matt「摘要只为定位，动手前先读原文」——若 Seedify 技能继续增殖，编排器可升级为显式路由器。
13. **技能体积极简**：37 技能平均 73 行、核心原语 31 行、派发壳 7 行。「small and composable」在 Seedify 已有共识（薄封装），可继续以此为修剪标尺。

## 5. 不建议照搬

- **模板内嵌 example**：与裁定#3 冲突，保持 Seedify 现行（脚手架槽位不填样例）。
- **issue-tracker/CLI 硬依赖**：违反零依赖方针；其「Other = 一段散文配置」倒是与 Seedify 气质相合。
- **双 harness 元数据（openai.yaml）**：仅当未来要出 Codex/Cowork 双分发再议（Cowork 包已另行解决）。
- **grilling 的多选题干**：「including multiple choices」与 Seedify 开放问法边界有张力，不引入。

## 6. 来源

- 仓库全量克隆本地研读（README/SCOPE/CLAUDE.md/.agents/*/.out-of-scope/* 九份/grilling 族/writing-for-agents 族/主链各技能/CHANGELOG/plugin.json）
- 克隆路径：`D:/AIGC/research/mattpocock-skills`（如需复查原文）
