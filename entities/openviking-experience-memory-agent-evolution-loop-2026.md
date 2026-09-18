---
title: "OpenViking 经验记忆：Session→Trajectory→Experience 的 Agent 进化闭环"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [agent-memory, experience-memory, context-database, trajectory, openviking, bytedance, mcp, skill, self-evolving-agent, tau2-bench]
sources: [raw/articles/openviking-experience-memory-agent-evolution-loop-2026, raw/articles/openviking-experience-memory-content-creation-agent-2026]
confidence: 0.8
provenance_state: merged
---

# OpenViking 经验记忆：Session→Trajectory→Experience 的 Agent 进化闭环

OpenViking 是面向 AI Agent 的开源上下文数据库，其「经验记忆」子系统把 Agent 的历史执行过程加工成可跨会话复用的经验知识：Session 保留原始证据，Trajectory 还原执行过程，Experience 给出可复用的方法与风险边界，并在下一次相似任务开始前按需召回、核对后使用。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

## 问题定义：「做过」不等于「学会」

一个 Agent 能调用工具、完成长链路任务，不等于它会从过去的任务中学习：刚排查完一次部署失败（检查环境、定位权限、修正配置、重新验证），换一个会话面对相似问题仍可能从第一步重新试错。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

客服场景同样存在落差——Agent 偶尔能正确处理换货或改签，但面对相似而不完全相同的请求，依然可能漏掉身份校验、选错工具顺序或违反业务政策。企业积累了越来越多的对话、日志与工具调用记录，Agent 却没有因此自然获得更稳定的执行能力。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

作者把根因归结为记录类型错位：聊天记录说明「上一次说了什么」，日志说明「某个工具返回了什么」，但 Agent 再次面对类似任务时需要的另一类信息——这类任务应先确认哪些条件、哪套执行顺序更可靠、哪些路径曾经失败及其原因、最后检查什么才算完成。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

## 三对象加工链路：Session → Trajectory → Experience

OpenViking 用三个对象把「过去」转成「下一次可调用的方法」：^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

- **Session**：记录一次任务中真实发生的用户消息、Agent 回复、工具调用与工具结果（原始证据层）。
- **Trajectory**：从 Session 中还原任务目标、执行路径与最终结果，回答「这件事上次是怎么做的」。
- **Experience**：从相关轨迹中提炼可迁移的方法、检查项与风险边界，回答「下次遇到类似问题应该怎么做」。

随着同类任务样本积累，系统把相似轨迹归类、比较，提取反复出现的成功模式、失败模式与边界条件；第一条经验可以来自较少的轨迹，后续样本继续为它提供验证与更新依据。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

## 两阶段闭环：离线沉淀 + 在线复用

第一阶段发生在任务执行之后：Agent 持续记录过程，当一个目标完成、一次失败被修复、用户给出关键纠错、或会话即将压缩结束时提交完整任务；OpenViking 归档原始过程、分析目标与工具结果、形成 Trajectory 并生成或更新 Experience。作者强调要保留「可判断结果的完整任务片段」——只有目标、过程与结果能对应，系统才有机会区分真正有效的方法与偶然成功，单纯增加记录量达不到目的。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

第二阶段发生在下一次任务开始时：接入在线检索后，Agent 遇到编码、配置、调试、故障恢复等多步骤任务时先检索相关经验、读取少量最匹配的原文，再核对环境、前置条件与风险边界，确认适用后才把步骤与检查项用于当前执行。新的执行结果又成为下一轮经验更新的证据，闭环为「执行任务 → 记录过程 → 提炼经验 → 相似任务召回 → 核对并应用 → 用新结果继续更新」。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

## 评测证据（τ²-bench）

文章给出公开评测 τ²-bench 上的对比：先用训练集在零售、航空领域执行多轮对话任务并后台沉淀经验记忆，测试阶段先检索题目相关经验注入上下文再执行。相对同一大模型不使用经验记忆的基线，Retail 任务成功率从 70.94% 提升到 77.81%（+6.87 个百分点），Airline 从 54.38% 提升到 66.25%（+11.87 个百分点）；在「已送达商品换货」单题测试中，无经验时 8 次运行只成功 2 次，召回含身份验证、状态检查与工具顺序的流程经验后 8 次全部成功。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

> [!note] 证据边界
> 上述数字由项目方自报，未见第三方独立复现；评测设计（训练集沉淀经验、测试集复用）本身由作者描述，属于 vendor 自评口径。引用时应视为方向性证据而非可迁移的独立基准。

## 工程接入：MCP + Experience Skill + Plugin

接入路径分两类：企业自研 Agent 可在任务运行过程中持续写入完整过程、在可靠任务边界提交，再通过 OpenViking MCP 与 Experience Skill 让 Agent 在高价值任务前检索并读取经验；使用 Codex、Claude Code、OpenClaw、Hermes 等消费级 Agent 产品时，可集成官方 Plugin 利用宿主的生命周期能力自动完成消息捕获、任务提交与经验召回，减少业务代码改造。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

作者对边界的表述是克制的：「更强的模型、更长的上下文和更完整的 SOP 仍然重要，经验记忆并不替代它们」——经验记忆补的是把过程沉淀为可复用方法的一环。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

## 第 2 来源 — 内容创作场景（Resource / Memory / Skill 与 L0-L2 渐进加载）

同一账号同日发布的第二篇把同一记忆底座搬到内容创作场景，构成为互补角度：^[raw/articles/openviking-experience-memory-content-creation-agent-2026.md]

- **对象模型换成 Resource / Memory / Skill**：Resource 是资料库（参考文章、热点链接、账号定位、历史稿件），Memory 是自动沉淀的长期记忆（表达偏好、选题判断、被否标题、复盘经验），Skill 是做事方法（定选题、审稿、复盘报告等工作流）——与开发场景的 Session/Trajectory/Experience 是同一底座的两套切片。
- **L0/L1/L2 渐进式上下文加载**：先读 L0 摘要定位主题，再看 L1 概览（选题方向、目标读者、文章结构），需要细节时才读 L2 完整内容，把上下文空间留给真正重要的信息，降低重复读取的 token 开销与无关信息干扰。
- **跨 Agent 记忆共享**：多个 Agent（Trae 等）切换时共享同一 OpenViking 服务，解决「换工具就得重讲一遍」的上下文断裂；触发点是额度耗尽/换工具导致的中断。
- **发布后复盘回流**：阅读数据、读者评论与转化反馈回到记忆层，作为下一篇内容优化的依据，形成「创作 → 复盘 → 下一篇」的闭环。
- **副作用视角**：该文也暴露了消费级场景的约束——记忆召回依赖账号权限与服务可达性，且内容创作类经验的正确性缺少可验证的评测口径（与开发场景有 τ²-bench 形成对照）。

> [!note] 该篇性质
> 第 2 来源整体是产品使用故事（含开通企业版/个人版引导），其可迁移部分集中在对象模型与 L0-L2 渐进加载设计；具体创作流程细节绑定 OpenViking 产品形态。

## 相关概念与实体

- [[concepts/agent-memory-architecture|Agent 记忆架构]]
- [[concepts/agent-memory-substrate-three-layer|记忆底座三层]]
- [[concepts/episodic-vs-semantic-memory-agent|情节记忆 vs 语义记忆]]
- [[concepts/context-engineering|上下文工程]]
- [[concepts/context-management-agent-systems|Agent 系统的上下文管理]]
- [[concepts/model-context-protocol-mcp|Model Context Protocol (MCP)]]
- [[concepts/skill-engineering-principles|Skill 工程原则]]
- [[entities/如何利用-agentcore-openviking-快速搭建具备高效记忆的-agent|AgentCore + OpenViking 高效记忆 Agent]]
- [[entities/99-元月用-agentplan-openviking-给销售团队配一个不会忘事的-ai-助手-bytedance|AgentPlan + OpenViking 销售助手]]
- [[entities/evoscientist-experience-memory-autoskills-2026|EvoScientist：经验记忆 + AutoSkills]]
- [[entities/how-to-encode-experience-into-skills|如何把经验编码进 Skill]]
- [[entities/emces-icml2026-episodic-memory-controlled-experience-synthesis-rl|EMCES：情节记忆受控经验合成]]
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆工程税]]

→ [[raw/articles/openviking-experience-memory-agent-evolution-loop-2026|第 1 来源原文]]
→ [[raw/articles/openviking-experience-memory-content-creation-agent-2026|第 2 来源原文]]
