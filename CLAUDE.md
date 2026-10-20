# Seedify 插件项目 · 持续开发记忆

> 本目录是 seedify 插件的项目根（兼 marketplace 根）。动手开发前先读 `docs/prompt-optimization-charter.md`（总纲 v4，用户亲手定稿，**我方不得改动**）。

## 项目一句话

把带转写噪声的口语输入，保真整理、问清疑问，自主生成种子提示词——只交付提示词本身。**提示词是种子不是蓝图，优化是进化不是润色。**

## 目录地图

- `seedify/` - 插件本体（`.claude-plugin/plugin.json` + skills/seedify/ + commands/ + agents/）
- `docs/prompt-optimization-charter.md` - 总纲 v4（最高指导，原文入包于 skills/seedify/references/charter.md）
- `docs/seedify-design-v1.md` - 设计文档 v6（组件规划表 + 裁定记录）
- `docs/prompt-optimization-spec-v1.md` - 需求定稿（R1–R3）
- `docs/research/` - 三轮调研留档（round2 市场分层 / round3 论文与 L0 设计 / README 摘档）
- `seedify-skill-cowork.zip` - Cowork 应用内上传包（SKILL.md+references 三文件）

## 七项用户裁定（违反即 bug）

1. 种子**自然生成**：三要素（目标/硬边界/验收）只作提示，不设栏位，用户说多少写多少；
2. 提问一律**开放问法**，禁填空模板（模板=过约束）；
3. **正例反例一律不进技能上下文**（少样本锚定固化行为；触发短语/调用示例除外）；
4. 总纲 v4 **原文入包**，运行前先读；
5. **只交付提示词**（代码块）；改动对照/依据/整理过程/总结均为内部检查项，不输出；
6. **有疑必问、先问后写、不限次数**；提问不受输出克制约束；
7. **删一切元指标**（行数/Token/性能预算）——元层面过约束=病灶 1 复发。

## 架构与分工

- `skills/seedify/` - 核心：表达整理（可回指+零丢失，内部纪律）→ 有疑必问 → 种子自主成文 → 代码块交付
- `commands/seed.md` - /seed 直接入口（薄封装，$ARGUMENTS）
- `commands/seed-check.md` - /seed-check 体检入口（派发 seed-reviewer，只诊断不改写）
- `agents/seed-reviewer.md` - 独立体检员（新上下文防自我美化；tools 仅 Read）
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
- 行为验收 T1–T13（清单在 `seedify/skills/seedify/references/anti-patterns.md`，开发期资产）：禁增生、零丢失、改口、宁问不猜、输出纯净、缺项必问、触发容错（语音错字变体）、交付即止、自然生成、提问充分等；
- 体检类：/seed-check 对病灶提示词逐项指出，对干净提示词明确"未发现病灶"且不改写。

## 待裁决 / 待办

- /seed-check 体检范围：目前=任意提示词（按总纲"第二步(Agent提示词工程/提示词优化)"）；用户未最终拍板是否收窄到仅 seedify 产出；
- 实装真实语音输入试用，按使用反馈迭代（skill-development Step 6）；
- 分发时补 license/LICENSE、版本号双处同步（plugin.json 与 SKILL.md）。
