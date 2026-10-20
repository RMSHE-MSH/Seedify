# seedify（种子化）

把带转写噪声的口语输入，保真整理、问清疑问，自主生成种子提示词——只交付提示词本身。

提示词是种子，不是蓝图；优化是进化，不是润色。

## 它做什么

语音转写输入常带错字、冗余、歧义与口语结构。seedify 在"人 → 提示词 → Agent"的交接处工作：

1. **表达整理**：把已说的说清楚，未说的不添加，有歧义的先询问（有疑必问、不限次数）；
2. **种子提炼**：自主生成种子提示词——目标、硬边界、验收标准仅作提示，用户说到多少写多少；不写实施方法（流程、步骤、角色、示例、格式）；
3. **交付即止**：只给出优化后的提示词，不输出改动对照与依据，不自动开工。执行循环与经验沉淀归 Claude Code / 桌面版 / cowork 的平台机制。

## 安装

**Claude Code**（一步安装，2.1.275+）：

```
/plugin install seedify --marketplace D:/AIGC/Seedify
```

或命令行等价：

```bash
claude plugin marketplace add D:/AIGC/Seedify
claude plugin install seedify@seedify
```

临时加载（开发试用）：

```bash
claude --plugin-dir <本目录路径>
```

若要把本插件纳入自己的 marketplace 分发：在市场的 `marketplace.json` 中列入本插件后，`/plugin marketplace add <该市场>` 再 `/plugin install seedify`。

**Claude 桌面版 / cowork**：将 `skills/seedify/` 目录作为技能安装（含 `references/` 子目录，须完整保留）。

## 用法

**命令入口**（确定性，免触发匹配）：
- `/seed <原始文本>` —— 直接进入优化；
- `/seed-check <提示词>` —— 体检一段提示词（派发独立评审代理 seed-reviewer，只诊断不改写）。

**自然语言触发**（含语音转写错字变体）：

- "帮我优化提示词 <原始文本>"
- "优化提示词 <原始文本>"
- "提示词种子化 <原始文本>"
- "整理我说的话 <原始文本>"

## 结构

```
seedify/
├── .claude-plugin/plugin.json        # 插件清单
├── commands/
│   ├── seed.md                       # /seed 直接优化入口（薄封装）
│   └── seed-check.md                 # /seed-check 提示词体检入口
├── agents/
│   └── seed-reviewer.md              # 独立体检员（只读，新上下文）
├── skills/seedify/
│   ├── SKILL.md                      # 运行规则与输出契约
│   └── references/
│       ├── charter.md                # 思想纲领（总纲 v4 原文，运行时先读）
│       └── anti-patterns.md          # 开发期测试清单（T1–T13）
└── README.md
```

## 设计依据

- 思想纲领：`skills/seedify/references/charter.md`（总纲 v4 原文）
- 设计文档：`../docs/seedify-design-v1.md`（v5）——位于插件目录之外，不随插件分发
- 无外部依赖、无脚本、无示例（正例反例不进技能上下文，避免少样本锚定）
