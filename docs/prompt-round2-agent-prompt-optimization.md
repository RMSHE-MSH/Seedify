# Agent 提示词优化调研 · 第二轮重启提示词

## 任务

调研并找出**真正适合 Agent 编码工具（Claude Code / Codex / DeepSeek Harness）的提示词优化技能、插件与方法论项目**。重点形态：可安装、可在说"帮我优化提示词"时被触发调用的 skill / 插件，以及支撑它们的引擎与工作流。GitHub 与各厂商官方资源为主要信息源。

"真正适合"的判定，按第一轮对话已澄清的真实需求执行。

## 核心立场（验收标准的来源）

我们反对"对话猜测式"提示词优化，其三大病灶：
1. **过度约束 / 过拟合**——把软细节（流程、步骤、示例、格式）当硬约束预先写死；
2. **角色堆砌与铺垫**——"你是一位资深 X 专家……"式包装与冗长前言；
3. **臆造要求和细节**——优化器在对话里猜出用户从未说过的角色、业务背景、流程、示例。

这些病灶会诱导 Agent 照提示词执行，压制它在实践循环中自主研究、自主规划、自我修正的能力。Agent 的价值在于循环中建立理解、发现路径不对就自我修正——提示词不应预先夺走这个空间。

我们主张的路线（Prompt Engineering → Loop Engineering）：
- **L1 种子提示词**：只写目标 + 硬边界 + 验收标准，软细节留给 Agent；
- **L2 循环进化**：提示词优化必须来自真实信号（执行轨迹、评测集、真实使用纠错），改动须过验证闸门（如 held-out 显著性、防回归）才算数；
- **L3 沉淀**：只把被实践证实的经验固化回记忆 / 技能，不沉淀猜测。

## 合格标准

候选项目须同时尽量满足：
1. **实践驱动**——优化信号来自执行轨迹 / 评测 / 真实使用，而非对话内推断。甄别方法：读 README 与文档，确认反馈从哪来；
2. **输出哲学克制**——产物有利于保留 Agent 自主空间（目标式而非流程式），无角色堆砌、臆造细节倾向；
3. **非冷门**——GitHub 星标达到可作统计依据的量级。若三条件交集内无高星成品，分层如实报告（引擎层 / 成品层），不得为凑数降低标准而不加说明。

## 第一轮已确认的事实（从此出发，勿从零重复）

检索环境：
- GitHub API 可用（搜仓库、拉 README 均正常）；内置 WebSearch 可能返回空结果；
- 本机不可达：anthropic.com、docs.anthropic.com、developers.openai.com、docs.deepseek.com、web.archive.org（需注明"未能在线核对"）；api-docs.deepseek.com 可达。

已核实的关键项目（星标为 2026-10-17 数据，需实时复核）：
- **实践派（符合哲学）**：NousResearch/hermes-agent-self-evolution（5.5k★，GEPA 读执行轨迹与真实会话史、约束闸门、出 PR）；stanfordnlp/dspy（38.7k★）+ gepa-ai/gepa（7k★，反射式进化，ICLR 2026）；skillberry-ai/cap-evolve（65★，失败轨迹学习 + held-out 显著性闸门）；BayramAnnakov/claude-reflect（1.8k★，真实纠错沉淀进 CLAUDE.md）；gaasher/Agent-Loop-Skills（175★，"iterating against feedback"）；romiluz13/cc10x（164★，"engineer the loop, not the prompt"）。
- **猜测派（已排除，三大病灶重灾区）**：tolimarchuk/promptify-skill、Jakeschincariol/promptmaster-skill、Cherish133/prompt-alchemist 等对话式重写器。
- **harness 提示词体系（背景参考）**：deepseek-ai/deepseek-harness 的 dsh-system-prompt 插件化注册表；openai/codex 仓库内开源的按模型系统提示词；Piebald-AI/tweakcc（2.5k★，可改 Claude Code 本体提示词）。

第一轮结论：三条件交集内**无高星成品 skill/插件**；高星的是引擎层（DSPy/GEPA/promptfoo 等）；成品技能层要么冷门、要么属猜测派。

## 边界

- 星标数与项目状态实时核验，不照抄本提示词中的旧数据；
- 不臆造未核实的项目、星标、功能或方法论描述；描述项目前先读其 README；
- 对无法访问的网站，明确标注未能在线核对；
- 输出语言：中文。

## 交付物

1. **分层清单**：实践闭环成品 / 引擎层 / 沉淀类 / 明确排除的猜测派（附排除理由）。每项含：星标、优化信号来源（对话推断 or 执行轨迹/评测）、与 L1–L3 的匹配点、局限；
2. **诚实结论**：三条件交集的评估，包括"为空"在内的如实判断；
3. **建议**：现成可组合的落地方案（含"帮我优化提示词"自动触发的接法）；若无合格成品，给出一个符合上述哲学的自建技能规格（目标 / 触发词 / 硬规则 / 验证方式）作为可选后续——只需规格，不必实现。
