# Seedify 插件项目 · 持续开发记忆

> 本目录是 seedify 插件的项目根（兼 marketplace 根）。动手开发前先读 `docs/prompt-optimization-charter.md`（总纲 v4，用户亲手定稿，**我方不得改动**）。

## 项目一句话

把带转写噪声的口语输入，保真整理、问清疑问，自主生成种子提示词——只交付提示词本身。**提示词是种子不是蓝图，优化是进化不是润色。**

## 目录地图

- `seedify/` - 插件本体（`.claude-plugin/plugin.json` + skills/ 13 技能（7 mod + 编排器 + 5 外源席）+ commands/ 8 条 + agents/）
- `docs/prompt-optimization-charter.md` - 总纲 v4（最高指导，原文入包于各技能 references/charter.md）
- `docs/seedify-design-v1.md` - 设计文档 v6（组件规划表 + 裁定记录）
- `docs/prompt-optimization-spec-v1.md` - 需求定稿（R1–R3）
- `docs/seedify-mod-design-v1.md` - 模块化设计 v2（生成族 M1–M5 + 读取族 M6/M7，已审定）
- `docs/superpowers/plans/2026-10-21-seedify-v0.2-mods.md` - v0.2 建造实施计划（七模块改造，Task 1–12）
- `docs/research/` - 调研留档（round2 市场分层 / round3 论文与 L0 设计 / README 摘档；**round4** = `README-mattpocock-skills.md` mattpocock/skills 研究报告 2026-10-22，**长期保留**——含技能全景、调用二分法、grilling 原语、writing-for-agents 元理论、对 Seedify 裁定的独立印证与 13 条可借鉴清单；**round5** = `README-superpowers.md` obra/superpowers 研究报告 2026-10-22，**长期保留**——含强制方法论架构、writing-skills 的 TDD 化技能写作与 Match-the-Form 形态学、brainstorming v7 对照、12 条可借鉴清单与三杠杆五问增量影响；本地全量克隆 `D:/AIGC/research/mattpocock-skills`、`D:/AIGC/research/superpowers`）
- `seedify-skill-cowork.zip` - Cowork 应用内上传包（13 运行时技能的 SKILL.md+references + domain-modeling 根级两格式文件，LF 归一；commands/agents 不随行）
- GitHub 远程：https://github.com/RMSHE-MSH/Seedify（公开仓库，`origin/main`；本地改动后 `git push` 同步，本机需走代理：`export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890`）

## 七项用户裁定（违反即 bug）

1. 种子**自然生成**：三要素（目标/硬边界/验收）只作提示，不设栏位，用户说多少写多少；
2. 提问一律**开放问法**，禁填空模板（模板=过约束）；
3. **正例反例一律不进技能上下文**（少样本锚定固化行为；触发短语/调用示例除外）；
4. 总纲 v4 **原文入包**，运行前先读；
5. **只交付提示词**（代码块）；改动对照/依据/整理过程/总结均为内部检查项，不输出；
6. **有疑必问、先问后写、不限次数**；提问不受输出克制约束；
7. **删一切元指标**（行数/Token/性能预算）——元层面过约束=病灶 1 复发。

## 架构与分工

- `skills/` - 13 个技能目录（7 mod + 编排器 + 5 外源席：grill-me/grill-with-docs/grilling/to-spec/domain-modeling，MIT，内置自 mattpocock/skills，改动声明见 THIRD-PARTY-NOTICES.md），各自独立可调用、可分开改进，每席自带 `references/charter.md` 逐字节副本；生成族 M1 `seed-express` 表达整理 → M2 `seed-clarify` 两席制（追问席=grill 全家内置、方案席=to-spec 换皮，流程体检撤编吸收）→ M3 `seed-gen` 种子生成 → M4 `seed-tools` 工具挑选 → M5 `seed-assemble` 提示词装配（脚手架槽位，缺项整段省略），读取族 M6 `seed-analyze` 意图解读 / M7 `seed-check` 提示词检查（分项通过 + 合格判定 + 纪律符合度总结分）
- `skills/seedify/` - 编排器薄层：触发入口、边界判断、按需串链、交付即止；各模块规则住在各模块 SKILL.md，编排器不复述
- `commands/` - 8 条薄封装：/seed-express /seed-clarify /seed /seed-tools /seed-assemble /seed-analyze /seedify /seed-check；**/seed 语义已收窄**为纯种子生成（v1 准全链路 → v0.2 迁 /seedify）
- `agents/seed-reviewer.md` - M7 独立上下文执行体（适配器探测到可派发才走；新上下文防自我美化；tools 仅 Read）
- Hooks/MCP/Settings = 0，各附判定理由（见设计文档 §2）；依赖方针：**不引 MCP，优先主干**

## 环境坑（已踩过）

- **YAML frontmatter 含 `user: "` 冒号空格必须用块标量 `description: |`** - 普通标量会解析失败；`validate-agent.sh` 用 sed 抽字段会漏掉这类问题，**改 frontmatter 后用 `python -c "import yaml; yaml.safe_load"` 严格自测**
- agent 内路径一律 `${CLAUDE_PLUGIN_ROOT}/...` 开头（本机绝对路径不进可分发文件）
- `allowed-tools: Read, Task, Agent` 兼容两代派发工具名
- 插件描述已被中文化（i18n 方针，见全局记忆 plugin-i18n）；托管技能被应用还原英文时跑 reapply.py
- 安装布局：marketplace.json 在**项目根** `.claude-plugin/`，`source: ./seedify`；`claude plugin validate D:/AIGC/Seedify` 可体检
- 安装命令：`claude plugin marketplace add "D:/AIGC/Seedify"` → `claude plugin install seedify@seedify -y`
- Cowork 是独立体系（不读 ~/.claude），只能应用内上传 zip；commands/agents 不随行
- **Windows autocrlf 换行符假象**：git 检出会给工作区副本套 CRLF，`sha256sum` 逐字节校验会误报双哈希（2026-10-22 合并后踩过）——先比 blob（`git show HEAD:<path> | sha256sum`）或 `tr -d '\r'` 后再比；内容一致即合规
- **盘符正则会误伤 URL**：发布核验的 `grep -rnE "[A-Za-z]:[/\\]"` 会命中 `https://` 的 `s:/`——**seedify/ 下零 URL 是有意基线**，外链只放仓库根 THIRD-PARTY-NOTICES.md（2026-10-23 Task 8 踩过）
- **改外源席必同步 THIRD-PARTY-NOTICES.md 改动声明**（5 席 MIT 内置件：grill-me/grill-with-docs/grilling/to-spec/domain-modeling）——改文不改声明=分发事故

## 验证基线（改动后回归）

- `claude plugin validate <项目根>` 过；
- agents/commands/skills 的 frontmatter 严格 YAML 解析过；
- 行为验收 T1–T30（清单在 `seedify/skills/seedify/references/anti-patterns.md`，开发期资产；四列 T编号|输入特征|通过标准|归属模块，原「测什么」标签列已移除，编号↔行为名对照）：T1 禁增生、T2 零丢失、T3 改口、T4 宁问不猜、T5 输出纯净、T6 缺项必问、T7 触发容错（语音错字变体）、T8 交付即止、T9 转写纠错、T10 无长诊断、T11 可回指、T12 自然生成、T13 提问充分（不限轮次）、T14 装配缺料整段省略、T15 来源/把握度不进交付、T16 用户流程只体检不写死、T17 候选方向不直接入稿、T18 单模块不夹带、T19 全链路交付纯净、T20 脚手架合法段不误报、T21 偏好槽不装步骤、T22 解读可回指不改写、T23 评分红线（只量纪律符合度/不预测效果/不作门槛）、T24 不因详细/结构/示例加分、T25 意图理解产物不进交付、T26 交付不自动质检、T27 分析与检查互不越界、T28 方案/词典不进交付、T29 内置件机制在位、T30 方案批准前不成文；
- 体检类：/seed-check 对病灶提示词逐项指出，对干净提示词明确"未发现病灶"且不改写；v0.2 增合格判定与纪律符合度总结分（脚手架段合法化，越界仍旗标）；
- 发布核验：版本双处一致（plugin.json 与 13 个 SKILL.md 均 0.3.0）；`sha256sum seedify/skills/*/references/charter.md` 全一致；可分发文件无本机绝对路径——`grep -rnE "[A-Za-z]:[/\\]" seedify/` 零命中（`${CLAUDE_PLUGIN_ROOT}` 不含盘符，天然不命中）。

## 待裁决 / 待办

- /seed-check 体检范围（**已拍板 2026-10-21**）：保持=任意提示词，不收窄；体检员口径随 v0.2 mod 化最小更新（脚手架段合法化，越界仍旗标）。**10-21 晚增裁**：/seed-check 升级为 M7 提示词检查（分项通过+合格判定+纪律符合度总结分，评分只度量纲领纪律符合度、不预测效果、不作门槛）；新增 M6 提示词分析（意图解读，输入任意表达/现成提示词）；链内不自动质检（T19 不动）——详见 mod 设计文档 §3.1；
- 实装真实语音输入试用，按使用反馈迭代（skill-development Step 6）；
- 分发时补 license/LICENSE（版本号双处已同步 0.3.0：plugin.json 与 13 个 SKILL.md）；
- **v0.2.0 已落地（2026-10-22，9dd2ac8）**：12 任务过任务级审阅＋全分支终审闭环，顺带修复 v0.1 打包 charter 脱漏「步骤、」；
- **M2 两席制 v0.3.0 已落地（2026-10-23）**：5 外源席内置（grill-me/grill-with-docs/grilling/to-spec/domain-modeling，MIT，THIRD-PARTY-NOTICES 随包）+ seed-clarify 薄壳串两席 + 编排器方案席批准门 + T28–T30 + Cowork 包 13 席重打（LF 归一，28 条目）；
- **发布门禁（v0.2.0）**：桌面已登录环境新会话人工补测三点——错字触发（如「帮我优化提词」）真实路由到全链路、/seed-check 真实派发 seed-reviewer、/seed-analyze 实跑；跳过=发布带未验证交互面；
- **发布门禁（v0.3.0 追加三点，桌面已登录新会话人工实测）**：①Cowork 实测两壳（grill-me/grill-with-docs）探测-降级路径；②grill frontier 批问在中文语音输入的真实节奏体验；③新触发真实路由（/seed-clarify 薄壳串两席、直接调 grilling/to-spec、错字变体）；跳过=发布带未验证交互面；
- ~~顺手修~~（**已随 v0.2.1 措辞波落地 2026-10-22**）：seed-gen「删除即愈则删、存疑才问」、seed-analyze「不自行决断」「不自行编写实施方法」、seed-assemble「骨架顺序照抄不重排」、mod 设计 §5.2 /seedify 链补意图解读——均已完成；
- **三杠杆吸收五问已拍板并落地（2026-10-22 晚，v0.2.1 措辞波）**：①M2 保守版——只加收敛条件「不限次数≠无收敛：每主题问到『再问只能靠猜』即收束」，一次一主题不动（superpowers 一问一消息佐证）；frontier 轮询/推荐答案已随 grilling 内置实现（原语即设计树+frontier+推荐答案）；**M2 议题包** 剩余：Play It Back 分块回放、builder-check 三堆分诊、裁定#2 可见护栏文本（终审停放项）。②「过程痕迹」铸词（=改动对照/依据/整理过程/总结，随文定义）。③「最保守」→「零添补」（seed-tools 处用「宁缺毋滥」）。④按 Match-the-Form 分形态：裁定硬护栏保留禁止+正向配对；形状规则改正向配方（seed-tools 交付版恰好四字段）；豁免从句重构为独立条件句（seed-gen 实施方法默认不写、用户要求时按要求写）；重复否定收敛为一处（T21、候选边界、最丰富解释）。⑤版本 0.2.1 双处已同步。不照搬：模板内嵌 example（违裁定#3）、issue-tracker 硬依赖（违零依赖方针）、Rulings-not-stalls 执行期不问人（违有疑必问）；
- 发布说明要点：桌面 `anthropic-skills:seedify` 旧镜像可能劫持触发短语（先 reapply/清镜像再试）；`/seed` 语义收窄（纯种子生成）在发布说明再提一次。
