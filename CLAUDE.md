# Seedify 插件项目 · 持续开发记忆

> 本目录是 seedify 插件的项目根（兼 marketplace 根）。动手开发前先读 `docs/prompt-optimization-charter.md`（总纲 v4，用户亲手定稿，**我方不得改动**）。

## 项目一句话

把带转写噪声的口语输入，保真整理、问清疑问，自主生成种子提示词——只交付提示词本身。**提示词是种子不是蓝图，优化是进化不是润色。**

## 目录地图

- `seedify/` - 插件本体（`.claude-plugin/plugin.json` + skills/ 8 技能（7 mod + 编排器）+ commands/ 8 条 + agents/）
- `docs/prompt-optimization-charter.md` - 总纲 v4（最高指导，原文入包于各技能 references/charter.md）
- `docs/seedify-design-v1.md` - 设计文档 v6（组件规划表 + 裁定记录）
- `docs/prompt-optimization-spec-v1.md` - 需求定稿（R1–R3）
- `docs/seedify-mod-design-v1.md` - 模块化设计 v2（生成族 M1–M5 + 读取族 M6/M7，已审定）
- `docs/superpowers/plans/2026-10-21-seedify-v0.2-mods.md` - v0.2 建造实施计划（七模块改造，Task 1–11）
- `docs/research/` - 三轮调研留档（round2 市场分层 / round3 论文与 L0 设计 / README 摘档）
- `seedify-skill-cowork.zip` - Cowork 应用内上传包（8 运行时技能的 SKILL.md+references；commands/agents 不随行）
- GitHub 远程：https://github.com/RMSHE-MSH/Seedify（公开仓库，`origin/main`；本地改动后 `git push` 同步，本机需走代理 127.0.0.1:7890）

## 七项用户裁定（违反即 bug）

1. 种子**自然生成**：三要素（目标/硬边界/验收）只作提示，不设栏位，用户说多少写多少；
2. 提问一律**开放问法**，禁填空模板（模板=过约束）；
3. **正例反例一律不进技能上下文**（少样本锚定固化行为；触发短语/调用示例除外）；
4. 总纲 v4 **原文入包**，运行前先读；
5. **只交付提示词**（代码块）；改动对照/依据/整理过程/总结均为内部检查项，不输出；
6. **有疑必问、先问后写、不限次数**；提问不受输出克制约束；
7. **删一切元指标**（行数/Token/性能预算）——元层面过约束=病灶 1 复发。

## 架构与分工

- `skills/` - 8 个技能目录（7 mod + 编排器），各自独立可调用、可分开改进，每 mod 自带 `references/charter.md` 逐字节副本；生成族 M1 `seed-express` 表达整理 → M2 `seed-clarify` 需求澄清 → M3 `seed-gen` 种子生成 → M4 `seed-tools` 工具挑选 → M5 `seed-assemble` 提示词装配（脚手架槽位，缺项整段省略），读取族 M6 `seed-analyze` 意图解读 / M7 `seed-check` 提示词检查（分项通过 + 合格判定 + 纪律符合度总结分）
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

## 验证基线（改动后回归）

- `claude plugin validate <项目根>` 过；
- agents/commands/skills 的 frontmatter 严格 YAML 解析过；
- 行为验收 T1–T27（清单在 `seedify/skills/seedify/references/anti-patterns.md`，开发期资产；四列 T编号|输入特征|通过标准|归属模块，原「测什么」标签列已移除，编号↔行为名对照）：T1 禁增生、T2 零丢失、T3 改口、T4 宁问不猜、T5 输出纯净、T6 缺项必问、T7 触发容错（语音错字变体）、T8 交付即止、T9 转写纠错、T10 无长诊断、T11 可回指、T12 自然生成、T13 提问充分（不限轮次）、T14 装配缺料整段省略、T15 来源/把握度不进交付、T16 用户流程只体检不写死、T17 候选方向不直接入稿、T18 单模块不夹带、T19 全链路交付纯净、T20 脚手架合法段不误报、T21 偏好槽不装步骤、T22 解读可回指不改写、T23 评分红线（只量纪律符合度/不预测效果/不作门槛）、T24 不因详细/结构/示例加分、T25 意图理解产物不进交付、T26 交付不自动质检、T27 分析与检查互不越界；
- 体检类：/seed-check 对病灶提示词逐项指出，对干净提示词明确"未发现病灶"且不改写；v0.2 增合格判定与纪律符合度总结分（脚手架段合法化，越界仍旗标）；
- 发布核验：版本双处一致（plugin.json 与 8 个 SKILL.md 均 0.2.0）；`sha256sum seedify/skills/*/references/charter.md` 全一致；可分发文件无本机绝对路径——`grep -rnE "[A-Za-z]:[/\\]" seedify/` 零命中（`${CLAUDE_PLUGIN_ROOT}` 不含盘符，天然不命中）。

## 待裁决 / 待办

- /seed-check 体检范围（**已拍板 2026-10-21**）：保持=任意提示词，不收窄；体检员口径随 v0.2 mod 化最小更新（脚手架段合法化，越界仍旗标）。**10-21 晚增裁**：/seed-check 升级为 M7 提示词检查（分项通过+合格判定+纪律符合度总结分，评分只度量纲领纪律符合度、不预测效果、不作门槛）；新增 M6 提示词分析（意图解读，输入任意表达/现成提示词）；链内不自动质检（T19 不动）——详见 mod 设计文档 §3.1；
- 实装真实语音输入试用，按使用反馈迭代（skill-development Step 6）；
- 分发时补 license/LICENSE（版本号双处已同步 0.2.0：plugin.json 与 8 个 SKILL.md）；
- **v0.2 建造状态（2026-10-21）**：七模块 + 编排器 + 8 命令全部建成并通过任务级审阅；README 重写、cowork 包重打包、validate 过——发布工程完成（marketplace.json 插件列表描述已同步 v0.2 口径）。
- **发布门禁（v0.2.0）**：桌面已登录环境新会话人工补测三点——错字触发（如「帮我优化提词」）真实路由到全链路、/seed-check 真实派发 seed-reviewer、/seed-analyze 实跑；跳过=发布带未验证交互面。
