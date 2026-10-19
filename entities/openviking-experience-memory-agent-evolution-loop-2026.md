---
title: "OpenViking 经验记忆：Session→Trajectory→Experience 的 Agent 进化闭环"
created: 2026-09-18
updated: 2026-10-20
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

## 深度分析

### 记录错位：数据表示问题，不是提示词问题

对话记录答「上次说了什么」，日志答「某工具返回了什么」，都是**证据视角**；Agent 面对同类任务要的是**决策视角**：先确认哪些条件、哪套顺序更可靠、哪些路径曾失败、最后检查什么才算完成。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

prompt 补不上，因为 SOP 是写死的应然流程，覆盖不到长尾，也不携带「此处曾失败」这类反事实，而它只能从真实轨迹里挖。经验记忆因此不是「更好的 prompt」，而是数据建模：把执行过程当一等公民。

### 三次有损变换：每一层丢掉了什么

Session → Trajectory → Experience 是有损压缩链，损失点比保真点更值得看：

- **Session**：全量证据，保真最高但不可迁移，混着噪音，无法区分必然有效与偶然成功；负面知识只在这一层完整。
- **Trajectory**：还原目标、路径与结果，丢掉噪音与未选中的分支，换来「目标-过程-结果可对应」的可判定结构。原文要求保留能判断结果的完整片段，否则无法区分有效方法与偶然成功。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md] 风险也在此：失败分支若被裁掉，Experience 学不到负面知识。
- **Experience**：丢掉任务参数（数字、订单号），留方法、检查项与风险边界。抽象换来可迁移性，代价是**适用条件被迫放宽**——误用的根源。

首条经验可来自少量轨迹：冷启动快，但早期属低样本过拟合，靠后续样本对冲，等于承认经验有生命周期，与 [[concepts/memory-consolidation-decay|记忆的巩固与衰减]] 同源。

### 两阶段闭环的失效面：过期经验的高置信度重放

离线阶段以「结果可知」为条件——完成、失败修复、用户纠错、会话压缩前，而非固定时间片；在线阶段是「检索 → 读少量原文 → 核对环境、前置条件与风险边界 → 应用」，新结果又成为下轮证据。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

「核对」是唯一防线，三类失效正从这里漏过：**过期**（工具改版、政策更新，经验仍是旧世界的准确描述）、**错配**（抽象时放宽的边界在此报复）、**自证**（上次复用错了但碰巧成功，结果被当作新证据写回）。第三类最险：闭环把「结果」当标签，结果却可能已被旧经验污染，飞轮转成自我确认。故 [[concepts/memory-source-provenance|记忆来源溯源]] 是必需配套。

### τ²-bench 证明到哪里为止

同一模型下 Retail 由 70.94% 升到 77.81%（+6.87pp），Airline 由 54.38% 升到 66.25%（+11.87pp）；「已送达商品换货」单题无经验时 8 次只成 2 次，召回含身份验证、状态检查与工具顺序的经验后 8 次全成。^[raw/articles/openviking-experience-memory-agent-evolution-loop-2026.md]

它证明的是同领域、同类任务、有训练集沉淀条件下的增益，单题案例还说明经验主要压缩方差（2/8 → 8/8），而非抬高上限。不证明跨领域迁移、库膨胀后的检索精度、错误经验的破坏力。两个保留：Airline 基线更低，增益空间更大；训练集与测试集同源，存在「看过题」式泄露。

### 从记忆到运行时策略：MCP、Skill 与 L0-L2 的 token 预算

MCP 是工具面，Experience Skill 是使用协议（何时检索、如何核对、如何应用），Plugin 借宿主生命周期解决「没人主动记」。三者合起来，经验才从「可检索文档」变成**运行时策略**。

与 skill-based 路线重叠之处是都把过程固化为可复用文本资产；差异在治理：Skill 由人编写、版本化、静态，Experience 自动蒸馏、按任务检索、带来源与适用条件、可增量更新。二者是上下游——Experience「发现」模式，Skill「固化」承诺；高频稳定者应上迁为 Skill，省掉每次注入的 token。见 [[entities/how-to-encode-experience-into-skills|把经验编码进 Skill]]。

L0-L2 分层（摘要 → 概览 → 全文）的逻辑是：库随任务数线性增长，每次注入原文则成本与噪音同步增长，分层把「是否值得读全文」变成一次检索决策。^[raw/articles/openviking-experience-memory-content-creation-agent-2026.md] 代价是新失真——L0 摘要最容易被压掉的恰是「适用条件与失效条件」，即决定能否复用的那句，故 L0 应强制保留负面条件。

## 实践启示

1. **以「可判定结果的任务片段」为记录单位，而非会话。** 最小单元含目标、步骤、工具结果与验收结论；在完成、修复、纠错、即将压缩四个边界提交，无结果的轨迹不入库。

2. **蒸馏用事件触发 + 阈值重蒸馏。** 边界触发增量，同类轨迹累积到阈值后重建。缺前者库长不大，缺后者经验僵化。

3. **复用前做条件比对，别相似度命中即注入。** 环境、前置依赖、工具版本、政策边界做成结构化字段，先比对再决定「当流程用」还是「当参考看」。

4. **给每条经验上失效管理。** 绑定来源 Session、生成时间、依赖的版本；变更时批量失效；复用后的失败显式写回负反馈，别静默重试。

5. **评测同时覆盖增益与伤害。** 除同域 train/test 分离，加未见过题型（测迁移）与注入错误经验（测盲从）；单题多次运行看方差。

6. **建经验上迁机制。** 高频稳定、跨任务重复者固化为 Skill 或硬规则，库只留长尾，否则检索成本随库增长而收益递减。

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
