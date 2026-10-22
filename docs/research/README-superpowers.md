# obra/superpowers 调研留档（round5 · 2026-10-22）

> 对象：https://github.com/obra/superpowers （MIT，Jesse Vincent / Prime Radiant）
> 本地克隆：`D:/AIGC/research/superpowers`（2026-10-22，HEAD 2ce8aa1，v7.0.1）
> 研究动机：与 mattpocock/skills（round4）并列的第二个参照系；本会话就装着 Superpowers 技能（SessionStart hook 注入 using-superpowers），有第一手体感可对照。
> 定位一句话（其 README）："A complete software development methodology for your coding agents" —— 不是技能杂烩，是**强制工作流的完整开发方法论**："The agent checks for relevant skills before any task. Mandatory workflows, not suggestions."

---

## 1. 仓库是什么

Superpowers 是一套「让 coding agent 走完整开发方法论」的技能体系：brainstorming 弄清意图 → 写设计文档 → 写实施计划 → 子代理逐任务执行+审查 → TDD 实现 → 收尾分支。哲学四条：TDD、Systematic over ad-hoc、Complexity reduction、**Evidence over claims**。核心承诺：「agent 可以自主工作几小时而不偏离计划」。

与 mattpocock/skills 的气质对比（两者互补）：

| 维度 | mattpocock/skills | obra/superpowers |
|---|---|---|
| 形态 | 可组合工具箱（"hack around with them"） | **强制方法论**（mandatory workflows） |
| 触发 | user/model-invoked 二分法，人是索引 | SessionStart hook 注入 using-superpowers，1% 可能就必须调用 |
| 技能写作验证 | SCOPE 观察失败制 + changeset | **TDD 化技能写作**：无失败测试不写技能 + 压力场景 + 措辞微测 |
| 分发 | Claude 插件 + skills.sh | **16+ harness 全适配**（Claude/Codex/Cursor/Devin/Gemini/Grok/Kimi/OpenCode/Pi/Hermes/Muse…） |
| 共同点 | 开放问法、事实归 agent、证据主义、反 slop、零/低依赖、不收新技能提案 | 同左 |

## 2. 仓库分层

```
skills/            15 技能（brainstorming / writing-plans / SDD / executing-plans / TDD /
                   systematic-debugging / verification-before-completion / writing-skills /
                   using-superpowers / diagnosing-superpowers / 请求与接收 code-review /
                   worktrees / finishing-branch / dispatching-parallel-agents）
hooks/session-start   每会话注入 using-superpowers 引导（Windows PATH 坑有工程防护）
.claude-plugin/ .codex-plugin/ .cursor-plugin/ .devin-plugin/ .antigravity-plugin/
.hermes-plugin/ .kimi-plugin/ .muse-plugin/ .opencode/ .pi/ gemini-extension.json
                   多 harness 打包清单（同一批技能）
tests/             基础设施测试 + explicit-skill-requests 行为测试（10 种用户话术
                   测技能触发）+ skill-size 测试 + brainstorm-server 协议测试
docs/superpowers/specs/   设计规格存档（SDD 风格，YYYY-MM-DD-*-design.md）
docs/plans/        实施计划存档
AGENTS.md          贡献者守则（94% PR 拒收率的反 slop 政策）
```

- **零依赖方针**："Superpowers is a zero-dependency plugin by design"——要外部工具的改动另立插件；
- **贡献纪律**：不收新技能提案；改技能必须跨所有 harness 有效；**人写 PR 摘要**（agent 代写即关）；**强制自报** model/harness/插件清单；PR 一律打 `dev` 分支；「agent 绕开人直接修别人的 issue」的 PR 直接关。

## 3. 核心方法论（对 Seedify 最有料的部分）

### 3.1 引导层：using-superpowers（每会话强制注入）

SessionStart hook 把技能全文注入上下文。规则：**1% 可能相关就必须调用技能**（先于任何回应、澄清问题、读文件）；process skills 优先于 implementation skills；「User instructions (CLAUDE.md…) take precedence over skills」；配 Red Flags 合理化对照表（「这只是个小问题/我先收集信息/我记得这个技能」→ 全是逃逸借口）。子代理豁免：被派发执行具体任务时无视本技能。

### 3.2 writing-skills：**把 TDD 用在写技能上**（元方法论核心）

- **铁律**："NO SKILL WITHOUT A FAILING TEST FIRST"——新技能与**改动**皆然；先写技能再补测试？删掉重来。
- **TDD 映射**：测试=带 subagent 的压力场景；RED=**不给技能跑基线**，把 agent 的失败与合理化说辞逐字记下；GREEN=只针对观察到的失败写最小技能；REFACTOR=堵新漏洞直到「bulletproof」。
- **"If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing."**
- **Match the Form to the Failure**（形态-失败匹配表，**附实测数据**）：
  | 基线失败类型 | 正确形态 | 错误形态 |
  |---|---|---|
  | 知道规则却在压力下违反 | 禁止 + 合理化表 + 红旗清单 | 软指导（"prefer…"） |
  | 产出形状不对（臃肿/埋没/复述） | **正向配方/契约**（说清产出是什么、部件与顺序） | 禁止清单 |
  | 已有产出里漏元素 | 结构化：模板里 REQUIRED 槽位 | 模板旁散文提醒 |
  | 行为依赖条件 | 可观察谓词的条件句 | 无条件规则+豁免从句 |
  - 实测：「禁止」用在**形状问题**上会**反噬**——对照测试里禁止组的 unwanted content 比不指导组还多（分布完全分离）；配方组无从讨价还价。
  - **No nuance clauses**：「不要 X，除非重要」一条除非从句就能把稳定配方打成噪声——真例外写成独立条件句。
  - **Exemption clauses don't scope**：「此限制不适用于代码块」仍然压制代码块——要豁免就重构规则让它够不着，而不是口头豁免。
- **措辞微测**（Micro-Test）：每变体 5+ 次新上下文单发、**必带无指导对照组**（对照组没这毛病就别写这条指导）、**方差即指标**（5 次 5 种解读=措辞没绑住）、命中的模板回声要人眼复核。
- **bulletproofing（只用于纪律失败）**：逐条封堵漏洞（「不准留作『参考』、不准『边测边改』」）；「违反字面即违反精神」斩断 spirit-vs-letter 话术；合理化对照表（借口→现实）；红旗自检清单；description 加违 pre-症状。
- **persuasion-principles**：引 Meincke et al. 2025（N=28,000，说服技法使合规率 33%→72%）：Authority（"YOU MUST/No exceptions"）/ Commitment（宣布技能使用、强制三选一）/ …——「EXTREMELY-IMPORTANT」块和红旗表的科学依据。
- **discovery（description 写法，实测结论）**：description **只写何时用、绝不写流程摘要**——「描述里写了流程的技能，agent 会照描述做而不读正文」（实例：description 写"review between tasks"导致只审一次，删掉流程摘要后正确走双段审查）。命名只用字母数字连字符；第三方描述（第三人称）；问题用领域语言描述不用特定技术症状。

### 3.3 brainstorming v7（2026-10 重建，与 Seedify M2 直接对照）

- **开场分流**：请求里一切都清楚 → 直接做；缺非琐碎信息 → 开始对话。
- **Look Before You Ask**：先读项目与上下文，「Never ask what the project or their context already answers」（事实归 agent——与 grilling 同构）。
- **首条消息协议**：已说清要什么 → ~200 词复述回放+发现的约束/冲突/更便宜路径+**一个**最能改变产出的问题；没说清 → 就最大的未决点问**一个**开放问题+说明「答案会改变什么」。未点名要 brainstorm 时，末尾附一行退出口（"Or if you'd rather I just dive in, say so"）。
- **一问一消息**：「a single sentence with no list of example answers hanging off it」（问题下不挂示例答案——与裁定#3 同精神）；说用户的语言，不甩流程黑话；**先开放问，菜单只在卡壳时给**（"Options steer. Ask them to describe it first."）。
- **Ground to cover = territory, not a script**——「Never read these out as questions」。
- **苏格拉底式追问**：每问搭在上一答/你的发现上，说出想学到什么；「**A proposal they can correct, plus your reason for it, often beats a question**」（可纠正的提案+理由 > 问题——接近 grilling 的推荐答案，但以提案形式）。
- **Play It Back**：理解一块就 200-300 词回放一块，与提问交替；**「Only what they said goes in as theirs. Anything you filled in goes in as your proposal, with your reason」**（=可回指在对话里的形态）；每块末尾问「哪里错了或缺了」。
- **Lay out the approaches**：理解后、动笔前，单条消息摆 2–3 条真实路径+取舍+推荐一条——用户挑（=M2 头脑风暴候选，但强调先杀弱选项）。
- **Spike**：一次性试造可跳过全流程（划为 throwaway）；**Keeping it is a new request**——留存产物要重新过门。
- **设计文档**（小改动=短版；项目=全版）：intent/why、goals/non-goals/**anti-goals**（「技术上成了但商业上败了」）、constraints、approaches、用户拍板的 how、留给 builder 的空白。存 `docs/superpowers/specs/YYYY-MM-DD-*-design.md` 入库。
- **Builder check**：交付前派**新 subagent** 拿 builder-check-prompt 审设计文档，问题分三堆（已在对话里答过 / builder 可自行定 / **只有人能答且改变产出**），只有第三堆回问用户，**仅此一轮**。
- **HARD-GATE**：「Nothing gets built until your human partner approves the design doc」（两个例外：用户退出提问流程；同意的 spike）。
- **Red Flags 表**：「他们说 sounds good」→「Approval covers what you showed them. A design doc you haven't written isn't approved.」

### 3.4 writing-plans / SDD / executing-plans（计划与执行链）

- **writing-plans**：为「没看过这个代码库和这份 spec 的工程师」写计划——**把你做过的决定全部写清**（哪些文件、哪些名字与签名、spec 里哪些值、哪些测试证明），留白处默认执行者会做合理选择。任务粒度=「自带测试循环、值得一道新审查门的最小单元」；每步=一个可检查结果的动作。计划头必带 **Global Constraints**（spec 逐字抄）与 **Review Focus**（「spec 暗示了但没有任务测试覆盖、最可能咬人的五类输入/失败」——先写清单再钉到任务上）：「spec 的沉默不是许可」。
- **subagent-driven-development**：每任务派**全新 subagent**（隔离上下文，「绝不继承本会话的历史」），任务后双维审查（spec 合规+代码质量），末尾全分支大审查。**Continuous execution**：「Should I continue?」是浪费时间。**Rulings, not stalls**：冲突/含糊/计划缺陷自己裁决，记入台账 `Ruling: 决定 — 理由 — 错了的代价`，继续走；「只有四件事停下来」（不可逆、安全敏感、工作区外副作用、计划彻底坏掉）。
- **verification-before-completion**：铁律「NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE」；Gate Function 五步（识别证明命令→跑→读全输出→对照→才许宣称）；Common Failures 表（「Tests pass」需要什么证据、什么不够）；红旗（"should/probably/seems to"、宣称前说 "Great!"）。
- **diagnosing-superpowers**（元改进环）：会话出问题时（技能没触发/重复劳动/跑偏/太贵），读磁盘 transcript **只报告不改设计**（「Whoever triages decides whether superpowers changes」）；「Every finding cites `path:line`. No citation, no finding.」；七个维度并行分析员 subagent（skill-timeline/plan-adherence/repeated-work/stumbles/quality-evidence/request-conflicts/cost-and-time）；脱敏导出 bundle 三级（skeleton/evidence/full）报 issue。

### 3.5 工程文化

- **CREATION-LOG.md**：技能留「创建日志」——从哪抽取、取舍了什么、为何这样结构（溯源文档）。
- **explicit-skill-requests 行为测试**：10 种用户话术（"skip formalities"、"I know what SDD means"…）测触发鲁棒性；drill eval 框架独立仓库（superpowers-evals）。
- **AGENTS.md 反 slop**：94% 拒收率；「This pull request is slop that's made of lies」；人写摘要、自报生成环境、搜过既有 PR、真实问题驱动。

## 4. 与 Seedify 的对照

### 4.1 又一批独立印证

| Seedify 裁定/纪律 | Superpowers 对应物 | 备注 |
|---|---|---|
| 裁定#2 开放问法禁模板 | brainstorming「一问一消息、**问题下不挂示例答案**」「menu only when stuck」「Options steer」 | 三方（Seedify/mattpocock/superpowers）同构 |
| 裁定#3 正反例不入 | 「no list of example answers hanging off it」（提问层）；但**技能内允许好示例**（分歧点） | 部分印证 |
| 裁定#6 有疑必问 | 「Their own words carry intent that your options can't」+ Ground to cover 穷尽领域 | 部分印证 |
| 可回指/不臆造 | 「Only what they said goes in as theirs. Anything you filled in goes in as your proposal, with your reason」 | 对话内可回指的教科书表述 |
| 事实归 agent（M2 可借鉴） | Look Before You Ask：「Never ask what the project or their context already answers」 | 与 grilling 同构 |
| 输出纯净/只交付提示词 | Play It Back 只放用户说过的；「过程不外泄」 | 域不同，纪律同族 |
| 有疑必问的保守退路 | opt-out「just dive in」+ 最保守默认 | 与「用户不愿澄清时作最保守理解」同构 |
| 总纲=评判唯一依据 | 「The spec is the binding authority, the plan is its argument」 | 权威链同构 |
| seed-reviewer 独立上下文 | builder check 派新 subagent 审设计文档 | 同一动机（防自我美化） |
| T1–T27 行为护栏 | 压力场景测试 + 行为测试套件 | 他们更系统（见 4.3） |

### 4.2 真差异

| 维度 | Seedify | Superpowers | 判断 |
|---|---|---|---|
| 产品形态 | 工具集（用不用随你） | 强制方法论（1% 规则+hook 注入） | Seedify 保持工具集定位不动 |
| 示例政策 | 裁定#3 一律不入 | 「One excellent example beats many mediocre ones」 | **保留分歧**：他们域是代码技能（示例=可运行代码），我们域是提示词行为（示例=锚定风险）。但其反面教条可借：禁填空模板、禁造作示例 |
| 技能测试 | T1–T27 人工清单 | TDD 化：基线先测失败→逐字记合理化→最小技能→压力复测→微测措辞 | **最值得吸收的工程差** |
| description 写法 | 触发短语+错字容错+参数说明 | **只写何时用、绝不写流程**（实测：写流程会被照做而跳过正文） | Seedify 描述无流程摘要（已合规）；可吸收「无指导对照组」自检习惯 |
| 提问形态 | 一次一主题、不限次数 | 一问一消息、Play It Back 分块回放、builder check 只回问一轮 | 分块回放值得进 M2 讨论 |
| 否定措辞 | 大量禁止事项（硬护栏为主） | **Match the Form**：形状问题用正向配方、纪律问题才用禁止（实测禁止反噬形状问题） | **直接精化我们三杠杆讨论的否定审计** |
| 豁免从句 | 「不写实施方法…用户明确要求的除外」 | 「Exemption clauses don't scope」——豁免要重构结构而非口头豁免 | 新审计点（见 5.2） |

### 4.3 可借鉴清单（更新并入 round4 的 13 条之后）

1. **Match the Form to the Failure**（三杠杆否定审计的精化器）：逐条归类 Seedify 的「禁止」——纪律失败（裁定背书）保持禁止+可配合理化表；**输出形状规则改为正向配方**（「交付物=且仅=提示词代码块」已是正向；查漏补缺）。
2. **No nuance clauses / Exemption don't scope**（新审计点）：找 Seedify 全文的「除非/除外」从句——「不写实施方法（…），用户明确要求的除外」应重构为独立条件句（「用户明确要求写入实施方法时，按要求写入」），而非挂在禁止句上。
3. **措辞微测法**（落地三杠杆五问的工具）：改「过程痕迹」铸词、「最保守」换强词前，5+ 次新上下文单发+无指导对照组+方差检查——比拍脑袋定稿硬。
4. **基线先测的技能开发法**（T1–T27 升级方向）：新护栏立项前先跑「无技能基线」记合理化说辞，护栏只对症下药；T 清单从「验收清单」进化为「基线对照+压力复测」。
5. **合理化表 + 红旗表**（纪律类护栏的形态升级）：三大病灶自检可配「借口→现实」表（如「只是整理一下应该不算改写吧」→…）——注意裁定#3 边界：这是对策不是示例。
6. **description 自检**：保持「只写何时用」；改 frontmatter 时警惕流程泄漏进 description（实测会劫持行为）。
7. **Play It Back 分块回放**（M2 UX 候选）：理解一块回放一块（200-300 词）、用户说的才是「他们的」、我补的是「我的提案+理由」——比收束时一次性清单更早暴露误解，对语音用户友好。
8. **Builder check 三堆分诊**（M2/M5 交互候选）：问题分「已答过/我可定/只有你能答且改变产出」，只有第三堆占用用户——与「有疑必问」兼容（第三堆才是真疑）。
9. **Review Focus 思想**（M5 装配自检候选）：装配自检可加一问——「这份提示词暗示了但没写明、执行时最可能咬人的五类风险是什么？」——写进【硬边界】或留对话（需裁定，可能越「不写实施方法」线）。
10. **Creation Log**：技能改动留一行溯源（改了什么/为何）——比 changeset 轻，可并入 CLAUDE.md 待办流。
11. **persuasion 权威措辞**：硬护栏用「违反即 bug/无例外」的 Authority 框架已有实证支撑（Meincke 2025）；软偏好避免 MUST 滥用。
12. **行为测试话术库**（发布门禁的正式化）：像 explicit-skill-requests 那样把「错字触发/口语变体」做成固定话术清单回归跑——正对我们的 T7 与发布门禁三点。

## 5. 不建议照搬

- **强制注入+1% 规则**：Seedify 是「排练室工具」不是开发方法论；把 using-superpowers 式强制装进编排器会与「交付即止、按需调用」气质冲突。
- **技能内嵌示例**：与裁定#3 冲突，保持。
- **Rulings, not stalls**（执行期不问人）：与「有疑必问」直接相反——域不同（执行期 vs 排练期），**不要**混入 M2/M3；仅可把「决定+理由+错了的代价」记法留给未来执行类产品。
- **16+ harness 打包**：Cowork 双分发已够，不追移动靶。

## 6. 对「三杠杆五问」的增量影响（讨论衔接）

- **问④（否定审计）被精化**：不是「少用禁止」而是**按失败类型选形态**——裁定类硬护栏保持禁止（正确形态），形状类规则改正向配方，豁免从句重构为独立条件句。建议按此重审后落刀。
- **问②③（铸词/换强词）获得落地工具**：措辞微测（无指导对照组+方差）可小成本验证「过程痕迹」「零添补」是否真绑住行为。
- **问①（M2）新增候选件**：Play It Back 分块回放、builder-check 三堆分诊（与 frontier 轮询、推荐答案一起进 M2 议题包）。

## 7. 来源

- 仓库全量克隆本地研读（README/AGENTS.md/writing-skills 全家/brainstorming v7/using-superpowers/writing-plans/SDD/verification-before-completion/diagnosing-superpowers/persuasion-principles/discovery/bulletproofing/testing-skills-with-subagents/session-start hook/CREATION-LOG）
- 克隆路径：`D:/AIGC/research/superpowers`（复查原文）
- 一手体感：本会话即 Superpowers 注入环境（SessionStart hook + 技能列表），观察记录可与文本互证
