---

title: "看 AgentRun 如何玩转记忆存储，最佳实践来了！"
type: entity
tags: [agent, cloud]
created: 2026-05-21
updated: 2026-11-17
review_value: 6
review_confidence: 8
sources: [raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了]
score_validated: 2026-09-05
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: pr
review_note: "judged pr-0.75: 产品文档; retained as hub (in-links>=20); MOC rewrite candidate"
moc_rebuilt: 2026-09-07
---# 看 AgentRun 如何玩转记忆存储，最佳实践来了！

> 本页原内容在 2026-09-07 质量闭环中判定为 **pr-0.75**，已按导航页（MOC）重建；
> 原文备份见 `_archive/hub-rewrite-2026-09-07/看-agentrun-如何玩转记忆存储最佳实践来了.md`，一手来源仍见下方 sources。

## 机制与论文
- [[entities/agentscope-java-harness-framework-enterprise-distributed|AgentScope Java Harness Framework 2.0 — 企业级 Agent 分布式场景的 Harness 实现 (Java 2.0 重大升级)]] — AgentScope Java全版
- [[entities/agent-oriented-infra-intent-driven-code-sedimentation|晓斌：从 People-Oriented 到 Agent-Oriented Infra —— 意图驱动 + 代码沉淀的进化体]] — Agent-Oriented Infra长文
- [[entities/skill-os-learning-skill-curation-self-evolving-agents|SkillOS: Learning Skill Curation for Self-Evolving Agents]] — SkillOS策展RL架构清晰rv10
- [[entities/muse-autoskill-bytebrain-self-evolving-agent-arxiv-2605-27366|MUSE-Autoskill：字节 ByteBrain 自进化 Agent 五阶段技能生命周期，arXiv 2605.27366]] — 五阶段技能生命周期，自生成87.94%超人类68.40%
- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering: A Survey]] — harness工程survey
- [[entities/agent-harness-architecture-design-production-guide|Agent Harness 架构设计与实现：生产级 Agent 系统落地指南]] — 七层金字塔生产指南
- [[entities/17-agent-architectures-evolution|17种Agent架构演进：控制流设计的完整演化史]] — 17架构系统拆解高价值
- [[entities/kimi-work-codex-vibe-working-paradigm-shift|Kimi Work：通用 Agent 战场从云端迁移到本地]] — Vibe Working开启+本地Harness 20356字rv9
- [[entities/从-anthropic-到-googleagent-skills-正在进入设计模式阶段|从 Anthropic 到 Google：Agent Skills 正在进入\]] — 过程资产五类设计模式
- [[entities/agent-从能用到管好中间差了什么|Agent 从\]] — agent治理分析

## 工程实践
- [[entities/long-running-agent-ralph-loop-handover-harness-ruofei|'长周期 Agent 详解：从 Ralph Loop 到可接管 Harness']] — 三类漂移+5张卡治理12390字rv10全版
- [[entities/boris-cherny-新访谈开发工具正在从-ide-变成-agent-控制台|Boris Cherny 新访谈：开发工具正在从 IDE 变成 Agent 控制台]] — Boris访谈rv10全版
- [[entities/karpathy-vibe-coding-agentic-engineering-v4|Karpathy 最新访谈：从 Vibe Coding 到 Agentic Engineering]] — v4 8090字rv10：可验证性上限+MenuGen警示
- [[entities/yumanju-ai-full-flow-efficiency|柚漫剧 AI 全流程提效拆解]] — rv10全流程提效规则基建
- [[entities/claude-code-core-internals|Claude Code 源码核心机制详解]] — 源码机制18k主版
- [[entities/harness-engineering-long-term-agent-tasks|Harness Engineering：让 Coding Agent 可靠完成长程任务]] — 长程任务四原则+3000行粒度公式rv9
- [[entities/claude-code-deep-architecture-analysis|Claude Code 架构深度解析]] — 并发与延迟加载深析
- [[entities/harness-generator-evaluator-anthropic|Claude Harness 设计：Generator-Evaluator 架构与 Context Reset 演进]] — Generator-Evaluator+context reset 10329字rv9全版
- [[entities/wow-harness-v3-governance-protocol|wow-harness v3：AI 开发的治理协议]] — 事件溯源跨session治理协议
- [[entities/claude-code-openclaw-memory-comparison|Claude Code Openclaw Memory Comparison]] — 记忆系统对比rv9
- [[entities/building-multi-tenant-agents-with-amazon-bedrock-agentcore|Bedrock AgentCore 多租户 Agent 构建实践]] — 十组件多租户
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南 + Agent Skill 迭代式编写 — AI 按你的标准执行]] — 菜单菜谱比喻+三级渐进披露18168字rv9全版
- [[entities/alibaba-agentic-cloud|阿里云 Agentic Cloud]] — 云战略有架构层borderline
- [[entities/阿里云刚发布的-agentloop-是什么|阿里云刚发布的 AgentLoop 是什么？]] — MVP五环ATIF标准

## 深度分析

### 三种记忆类型的边界划分
AgentRun 把 Agent 记忆拆成三类正交能力：会话历史在同一对话内维持上下文，长期记忆跨会话保存用户偏好等结构化信息，会话状态则是可直接读写的运行时状态，三者统一落在 Tablestore 存储底座上^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:23]。会话历史的关联机制藏在请求头里：session_id 通过 `X-AgentRun-Conversation-ID` 传递，服务端用它关联 Tablestore 中的会话记录，且每次运行生成新 session_id 互不干扰^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:45-46]。会话状态的落库结构是两张表——会话元数据进 session 表、消息内容进 message 表，天然适合审计与数据分析这类只读场景^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:70]。三类记忆对应不同 API 与一致性需求，边界清晰是这套设计最值得借鉴的地方。

### 控制平面托管配置，客户端零硬编码
所有示例共享一个模式：客户端不配置任何存储细节，而是从控制平面读取。`MemoryCollection.to_mem0_memory()` 一次调用自动完成读取记忆存储配置（Tablestore 实例、集合、向量维度）并初始化 mem0 客户端^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:58]；ADK 侧的 `SessionStore.from_memory_collection()` 同理，自动获取实例与 endpoint，无需手动拼 OTS 连接参数^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:77]。管理 API 还能反向查询配置详情，包括嵌入模型与长期记忆提取模型的选择^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:83]。这种"存储基础设施托管在控制平面、代码只引用逻辑名称"的做法，把向量库选型与维度调整从代码变更降级为配置变更。相关平台背景见 [[entities/aliyun-agentrun|阿里云 AgentRun]]。

### 三条集成路径：自动提取、显式写入、框架适配
长期记忆提供两条接入方式，代表两种控制权取舍：MCP 集成下 Agent 在每轮对话自动调用工具检索和更新记忆——对话中出现"我喜欢喝咖啡"这类可提取信息时自动存入向量库^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:54]；直接操作 mem0 则需要代码显式调用 `memory.add()`，适合自定义框架或数据管道中由业务逻辑精确控制写入时机，mem0 内部还会用 LLM 把一段描述拆解为多条结构化记忆^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:56-57]。框架侧另有两条适配路径：LangChain 采用"检索-注入-回写"循环，每轮先检索相关记忆注入 prompt，回复后再把本轮内容写回，记忆随对话持续积累^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:63]；Google ADK 通过 OTSSessionService 把持久化完全下沉到框架层，进程重启后会话可续^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:75-76]。

### 工程细节里的隐性约束
几处容易被忽略的细节值得注意。其一，官方推荐流式模式：Agent 处理消息时会调用 MCP 记忆工具读写 Tablestore，流式可避免客户端因等待工具调用而超时^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:47]。其二，函数计算遵循 Serverless 架构，首次请求需创建实例，冷启动等待较长^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:34]。其三，user_id 语义有陷阱：示例从 Endpoint URL 解析账号 ID 当作用户标识，仅适合单用户演示，多用户应用必须改用终端用户 ID^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:46]。此外凭据一律走环境变量，明确禁止硬编码密钥^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:42]。多智能体场景下的平台定位可参考 [[entities/agentrun-multi-agent-a2a-alibaba-cloud|AgentRun 多智能体 A2A]]。

## 实践启示

1. **按三类建模记忆**：把记忆需求拆成"会话内上下文 / 跨会话偏好 / 可直接读写状态"分别选 API，不要用单一方案硬扛所有场景，对应 Tablestore 底座的三类能力^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:23]。
2. **默认走 MCP 自动提取**：让 Agent 在对话中自主检索和维护长期记忆；只有当写入时机需要业务逻辑精确控制时，才降到 SDK 显式 `memory.add()`^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:54-56]。
3. **框架集成用现成适配层**：LangChain 场景复用"检索-注入-回写"循环模式，ADK 场景直接挂 OTSSessionService，把持久化下沉到框架层而非手写存取代码^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:63,75-76]。
4. **客户端默认流式调用**：避免 MCP 记忆工具调用阻塞导致超时，并对 Serverless 冷启动的首次延迟设置预期^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:34,47]。
5. **区分 user_id 语义并托管配置**：多用户系统必须用终端用户 ID 而非开发者账号；存储配置引用逻辑名称、由控制平面解析，凭据一律环境变量^[raw/articles/看-agentrun-如何玩转记忆存储最佳实践来了.md:42,46,58]。入门操作路径可对照 [[entities/aliyun-agentrun-5min-quickstart|AgentRun 5 分钟快速上手]]。
