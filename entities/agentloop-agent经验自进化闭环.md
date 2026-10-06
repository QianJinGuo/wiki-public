---
title: "让 Agent 越用越准、成本越来越低：AgentLoop 的 Agent 经验自进化闭环"
created: 2026-08-15
updated: 2026-10-07
type: entity
tags: [ai, agent, harness, agentloop, 经验自进化, 可观测性, eval, 阿里云, 人工治理, human-in-the-loop, 携程]
sources: [raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环, raw/articles/ctrip-agent-self-evolution-human-governed-2026]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 让 Agent 越用越准、成本越来越低：AgentLoop 的 Agent 经验自进化闭环

阿里云（马云雷）分享的 AgentLoop Agent 经验自进化实践。传统软件追求确定性，AI Agent 则天然具有不确定性：模型采样、上下文变化、任务规划、工具返回和长链路执行都可能影响结果——同一个问题连续运行多次，Agent 可能选择不同的工具、走不同的路径，甚至给出不同的答案。当 Agent 进入运维、研发、数据分析和企业业务流程，企业真正关心的归根到底是两个结果：Agent 现在到底做得准不准（任务成功率、首次完成率、多次执行稳定性）、达到这样的准确率需要付出多少成本（每完成一个成功任务消耗多少 Token、时间、工具调用和人工介入）。Agent 的每次运行不只产生最终答案，也会留下执行轨迹——成功轨迹包含有效路径，失败轨迹记录反复出现的错误和恢复线索。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

原始 Trace 并不天然等于经验：只有经过清洗和组装形成 Trajectory，再结合结果评估进行比较，才能提炼出可验证、可召回、可复用的行动经验。上线前的评测集只能覆盖已知问题，真实业务会不断产生新的表达、工具状态和异常组合，人工数据飞轮高度依赖专家、无法规模化，因此企业需要一条更自动化的进化路径：持续观测真实运行 → 从成功和失败轨迹中自动发现规律 → 生成可复用经验 → 把筛选后的经验带回下一次执行。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

## 经验自进化闭环：从 Trace 到 Trajectory 到召回注入

AgentLoop 在模型之外构建一层可持续更新的经验系统：接收 Agent 的真实 Trace，将高噪音执行数据清洗并组装为标准化 Trajectory（在内部复杂样本中，清洗后的高价值轨迹可降到原始 Trace 约 4%—6% 的数据量级），再从多个轨迹中自动挖掘有效路径、失败模式、工具约束、参数规则和恢复策略，生成结构化经验。当 Agent 再次面对相似任务时，Recall Skill 和 CLI 会根据当前目标、业务对象、工具、执行进度和错误状态召回少量适用经验并注入运行时上下文；任务完成后新的运行结果再次形成 Trace，进入下一轮挖掘，形成「观测 → 轨迹 → 挖掘 → 经验 → 召回 → 运行 → 再观测」的飞轮。这里的「自进化」并不是直接修改模型权重，而是让 Agent 在模型通用能力之外持续获得来自真实业务的行动经验——模型负责推理，工具负责执行，知识库提供事实，经验库帮助 Agent 判断在当前情境下应该优先做什么、哪些路径容易失败、如何恢复、什么才算真正完成。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

## 核心能力与 Bench 验证

广泛接入真实运行数据：提供多种探针、OpenTelemetry、LoongSuite Pilot、eBPF 等接入能力，无论 Agent 是否方便修改代码都能采集模型调用、工具调用、执行结果和运行环境信息。挖掘与召回面向真实轨迹深度优化：经验不是对单条 Trace 做摘要，而是在多个轨迹间比较，识别反复出现的有效动作和高风险路径，生成工具选择与调用顺序、参数规则与前置条件、反模式、错误恢复与绕行策略、结果验证与完成判断规则等结构化经验；召回不仅考虑文本相似度，还结合任务、工具、进度、错误状态和经验适用范围排序过滤。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

多类 Bench 验证质量与成本收益：StarOps 指标查询正确率 7.1%→36.1%（Token -6.8%，平均工具调用次数下降 25.1%、有害事件下降 27.8%）；OpenClaw/PawBench 通过率 24.53%→30.67%（Token -58.16%）；PinchBench 0.2928→0.3464 提升 18.3%（Token +2.9%）；ClawProBench 74.51%→78.43% 提升 3.92pp（Token -11%）；SWE-bench Verified 67.2%→74.4% 提升 7.2pp（Token 362.4M→536M）。实验也说明质量和成本之间可能存在权衡，AgentLoop 不只关注单一分数，而是同时衡量成功率、同类任务多次执行的稳定性、Token/成功任务、工具调用、耗时和超时率，寻找适合具体业务目标的最优点。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

## Skill + CLI 接入与共享进化

经验生成后不需要重新训练模型：用户在客户端安装 Recall Skill，通过 CLI 配置经验库（Experience Store）和库级访问凭证；Agent 在任务开始、调用关键工具、遇到错误或准备交付时主动检索相关经验，将召回结果作为参考上下文。接入路径：创建经验库（填名称、提取 Agent 应用、经验抽取起始时间）→ 创建 API Key 访问凭证（或 AK/SK）→ 安装 alibabacloud-agentloop-experience Skill 并复制 Recall Endpoint 配置 → 用 `search_context.js search --query "..." --context-type experience` 验证召回链路。团队可以让不同客户端和不同 Agent 使用同一个经验库：一个 Agent 验证过的有效路径可被其他 Agent 召回，一个团队遇到的失败可成为其他团队提前避开的反模式——新 Agent 继承已有方法降低冷启动成本，通用经验跨 Agent 共享，业务专属经验按 AgentSpace 和经验库隔离。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

经验库位于模型之外、更新更快迁移更容易，与服务不同模型和 Agent 框架、按任务动态召回、快速更新、按团队和权限控制作用范围。与 Memory（记得过去）、RAG（找到知识）、Workflow（固定流程）、Skill（获得能力）、微调/RL（改变模型本身）相比，经验自进化让 Agent 在当前任务中复用真实执行验证过的方法——在完整的企业 Agent 系统中，这些能力可以协同工作。^[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环.md]

## 受人工治理的 Agent 自进化（携程实践 SUPP 2026-08-20）

携程技术（Li Yi）提供同一主题下「受人工治理」的可补全新维度：经验自进化闭环之外，强调修改必须经过人工治理与严格验证才能进入生产。^[raw/articles/ctrip-agent-self-evolution-human-governed-2026.md]

**自进化四条件**：反馈来自 Agent 真实运行；系统能据反馈修改 Agent 后续仍会使用的内容；修改经过验证和人工选择；修改后的 Agent 继续产生新反馈。缺任何一环都只是普通优化而非持续循环。^[raw/articles/ctrip-agent-self-evolution-human-governed-2026.md]

**三类错误定位**：信息足够但推理没走完；字段已拿到但不知业务含义（不需加数据源，只需写清字段含义）；现有工具拿不到必要信息（在只读隔离环境验证，未审核不进生产）。^[raw/articles/ctrip-agent-self-evolution-human-governed-2026.md]

**保留修改的三个并列条件**：生效性（修改真的被 Agent 使用）、可归因性（回放时同名同参调用返回当时保存结果，修改前后面对同一份基础事实，避免外部数据变化干扰；完全冻结回放则关闭所有新增查询）、安全性（重跑原本正确处理的问题，不伤害其他场景）。单一模型同时分析和裁判时无效修改误判约三成，拆分职责 +「必须实际生效」条件后降到个位数百分比。^[raw/articles/ctrip-agent-self-evolution-human-governed-2026.md]

**人工治理与权限分离**：分析系统只读历史副本；重新调工具进只读隔离环境；仅修改通过验证和人工审核后发布流程才取得写入权限并记录修改人/版本/回滚点。接入前提三件套：看得见（可观测完整执行）、改得动（工具说明/规则/prompt 可单独更新回滚）、有人负责（明确审核/发布/回滚责任）。^[raw/articles/ctrip-agent-self-evolution-human-governed-2026.md]

**真实业务指标**：携程酒店技术支持两类 Agent 一线采纳率 20%→40%+、40%→60% 左右；协作体系总采纳率 25.9%→51.7%；2026-06 AI 有效覆盖率 48.08%；有效帮助问题解决时长中位数 -40%，及时解决率 +4.9pp。^[raw/articles/ctrip-agent-self-evolution-human-governed-2026.md]

## 深度分析

### Trace 到 Trajectory 到召回注入管线的经济学

这条管线的核心是把原始 Trace（observability 数据）转化为可复用的行动经验，每一步都有明确的成本账：原始 Trace 包含大量基础设施 Span、重复消息和与决策无关的数据，若直接长期存储并送模型分析，存储与 Token 成本会快速增长，有效信号也会被噪音淹没。AgentLoop 将 Trace 清洗、去噪并组装为标准化 Trajectory（保留任务目标、行动步骤、工具调用、观察结果、错误与恢复过程），在内部复杂样本中高价值轨迹可压缩到原始 Trace 约 4%-6% 的数据量级——这不仅是存储压缩，更让挖掘算法从"处理杂乱日志"变成"直接分析决策过程"。召回端同样讲经济性：不只做文本相似度，而是结合任务、工具、执行进度、错误状态和经验适用范围排序过滤，目标是"在真正影响决策的时刻注入少量能改变行动的经验"，而非返回更多历史内容。整体上，管线把两类成本一起降：Agent 运行侧（无效重试、重复查询、延长链路）和平台侧（Trace 存储、传输、重复人工分析）。

### Skill + CLI 共享进化机制

经验生成后不需要 retraining，也不需要重建 Agent——接入成本被压到客户端安装 Recall Skill、用 CLI 配置 Experience Store 和库级访问凭证（API Key 或 AK/SK），再用 `search_context.js search --query "..." --context-type experience` 验证召回链路。关键设计是经验库位于模型之外：更新比模型快、迁移容易、可服务不同模型和 Agent 框架、按任务动态召回、按团队和权限控制作用范围。这催生一种组织级的共享进化结构：一个 Agent 验证过的有效路径可被其他 Agent 召回；一个团队踩过的坑成为其他团队提前避开的反模式（anti-pattern）；新 Agent 继承已有方法降低冷启动成本。隔离边界按 AgentSpace 和经验库划分——通用经验跨 Agent 共享，业务专属经验按库隔离。与 Memory（记得过去）、RAG（找到知识）、Workflow（固定流程）、Skill（获得能力）、微调/RL（改变模型本身）相比，经验自进化复用的是"真实执行验证过的方法"，在企业 Agent 系统中与上述能力协同而非替代。

### 受人工治理的自进化：质量与速度的权衡

自动挖掘 + 自动召回的飞轮速度快，但"自动修改"若不设防，错误经验会被系统持续放大。携程的实践给出对照设计：修改必须经过验证与人工选择才能进生产，分析系统只读历史副本，重新调用工具进只读隔离环境，只有通过验证和人工审核的修改才取得写入权限，并记录修改人、版本和回滚点。其经验教训有量化价值：单一模型同时承担分析和裁判时，无效修改误判约三成；拆分职责并增加"必须实际生效"（生效性、可归因性、安全性三个并列条件）后，误判降到个位数百分比。代价是用进化速度换可靠性——自动飞轮（AgentLoop 主打）适合经验探索与积累，人工治理闸门（携程模式）适合生产发布，两者构成"探索快、上线稳"的两段式结构。接入前提三件套：看得见（完整可观测性）、改得动（工具说明/规则/prompt 可独立更新与回滚）、有人负责（明确审核/发布/回滚责任）。

### Bench 数字背后的改进声明边界

AgentLoop 的多类 Bench 结果整体积极但不均匀：StarOps 指标查询正确率 7.1% 到 36.1%（Token -6.8%，工具调用次数 -25.1%，有害事件 -27.8%）；OpenClaw/PawBench 通过率 24.53% 到 30.67% 且 Token -58.16%；PinchBench 提升 18.3% 但 Token +2.9%；ClawProBench 提升 3.92pp（Token -11%）；SWE-bench Verified 67.2% 到 74.4%（提升 7.2pp）但 Token 从 362.4M 升至 536M。这说明质量与成本之间确实存在权衡：有些任务为换取更高成功率需要注入更多上下文。合理的解读框架是"单位成功成本"（Token/成功任务、时间、工具调用、人工介入），而非单一分数或单一 Token 数——这也是 AgentLoop 自身的评估口径（成功率、多次执行稳定性、失败模式集中度、人工接管率/返工率并看）。对读者而言，这些数字应视为特定任务形态下的实验结果，其可迁移性取决于自身业务与 Bench 场景的经验属性相似度。

## 实践启示

1. **先把 Trace 变成 Trajectory，再谈经验挖掘**：直接拿原始日志做长期存储和 LLM 分析，成本会随调用量线性膨胀且信号被噪音淹没；先清洗压缩（本例可到 4%-6% 量级），挖掘质量和平台成本同时受益。
2. **经验库放在模型外、用 Skill + CLI 接入**：不需要 retraining 也不需要重建 Agent，接入成本只是装一个 Recall Skill 配一个凭证；这使经验更新速度远快于模型迭代，也天然支持多框架、多 Agent 共用。
3. **把"一个团队的失败"变成"所有团队的反模式"**：按 AgentSpace/经验库设计共享与隔离边界，让通用经验跨 Agent 复用、新 Agent 继承已有方法，冷启动成本由组织分摊而非每个团队重付。
4. **用"每成功任务成本"而不是"单次 Token"做预算口径**：经验注入可能让部分场景 Token 上升（如 SWE-bench Verified 实验中消耗更多 Token），衡量的是单位成功成本与稳定性，而非单次调用最省。
5. **自动飞轮与人工治理分两段跑**：自动挖掘/召回负责探索和积累，生产发布前过验证、人工审核、写入权限分离、回滚点的闸门；分析、裁判、写入三职责必须拆开（携程数据：单模型兼任误判约三成，拆分后降到个位数百分比）。
6. **改造前先确认三件套**：看得见（完整执行可观测）、改得动（规则/prompt 可独立更新回滚）、有人负责（明确的审核与回滚责任）——缺任何一件，自进化循环就退化成一次性优化。

## 相关实体

- [[entities/aliyun-agentloop-enterprise-agent-self-evolution-flywheel|阿里云 AgentLoop 企业自进化飞轮]]
- [[entities/agent-audit-risk-noise-aliyun-agentloop-2026|AgentLoop 审计与风险]]
- [[concepts/agent-self-improvement-loops|Agent 自改进闭环]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[entities/阿里云发布-agentteams-与-agentloop破解企业智能体规模化落地两大难题|AgentTeams 与 AgentLoop]]
- [[entities/ctgan-llm-test-data-generation-ctrip|携程 LLM 测试数据生成]]

→ [[raw/articles/让-agent-越用越准成本越来越低agentloop-的-agent-经验自进化闭环|原文存档]]
