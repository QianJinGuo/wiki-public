---

title: "端侧模型专用Harness：Qwen3.8-27B + Perplexity Portable Computer"
created: 2026-08-30
updated: 2026-09-07
type: entity
tags: [harness, on-device, local-inference, qwen, agent, privacy]
sources: [raw/articles/on-device-harness-qwen38-27b-portable-computer]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 端侧模型专用Harness：Qwen3.8-27B + Perplexity Portable Computer

## 摘要

Perplexity 推出的 Portable Computer 是首个系统性证明"本地优先 Agent 运行范式"可行的产品化 harness：以 Qwen 3.8 27B 这类可本地部署的开源模型为底座，通过框架-模型协同设计、上下文效率优化、OS 级沙箱执行与可选的顾问升级，在接近零推理成本的前提下完成真实知识工作，同时保证敏感数据不出设备。^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

## 核心要点

- **Local-first 全栈**：模型、框架、对话与轨迹默认全部在本地运行，网络搜索、连接器、云端顾问仅在用户批准时跨设备边界，敏感数据未经许可绝不离开设备 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]
- **框架须适配模型**：通用框架假定模型具备前沿能力（长上下文、大量工具、长链规划），本地小模型达不到；Perplexity 让框架按模型能力量身定制，模型再经后训练以有效使用框架 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]
- **上下文效率是端侧第一设计约束**：Qwen 3.8 27B 标称 26 万 token 窗口，实际超过 10 万 token 性能即衰减；最小系统 harness + 按需技能加载 + 过期上下文压缩是应对手段 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]
- **连接器 CLI 化而非 MCP 化**：MCP 服务器庞大的工具定义消耗上下文，Perplexity 把最常用 MCP 转成简洁命令行工具并辅以自定义技能 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]
- **默认沙箱、不可降级**：OS 级沙箱限制进程、文件系统路径与网络访问；沙箱不可用时禁用自身而非降级到非沙箱执行——与 Pi/Hermes 默认以用户权限直接运行命令形成对比 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]
- **顾问升级是可控的差距弥合器**：本地模型自行决定何时咨询前沿模型，协调器保留工具权限并控制发送的上下文，升级前经 PII 分类器标记敏感信息 ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]
- **后训练闭环**：用合成 RL 环境（Docker 容器 + 验证器）做拒绝采样微调 + 强化学习两阶段训练，PPLX 27B 将本地知识工作台得分从 82.6% 提升至 85.4% ^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

## 深度分析

### 1．"框架适配模型"取代"模型适配框架"：端侧 harness 的设计哲学转向

主流 harness 的隐含假设是模型能力足够强（长上下文、任意多工具、长链自主规划），因此可以围绕"榨取模型能力"来设计。端侧模型打破这一假设后，harness 的设计重心转向"补偿模型短板"。Perplexity 的四项设计选择——上下文效率、连接器 CLI 化、自我验证、沙箱执行——本质上是同一逻辑的不同切面：把稀缺的上下文预算留给核心推理，把不可靠的环节交给确定性代码，把风险隔离在沙箱内。这印证了 Harness Engineering 中"框架适配模型"的设计哲学：不是让小模型去够大模型的框架，而是让框架按模型能力量身定制，模型再经后训练适配框架。^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

### 2．执行循环中"确定性协调器 + 概率性模型"的职责切分

Portable Computer 的执行循环中，**协调器是确定性的框架代码而非大模型**：它维护循环、构建上下文、执行策略；本地模型只提出下一步操作，协调器在沙箱中执行已批准的工具调用并将结果返回给模型。这种切分是端侧 harness 可行性的关键——把安全、策略、上下文管理等高风险职责从 LLM 手中拿回确定性代码，LLM 只负责"决策"，而"行动"交给受控的执行层。相比 Pi/Hermes 默认以用户权限直接运行命令，Portable Computer 的隔离始终启用、无需配置、不可降级，从机制上消除了"提示注入→任意命令执行"的经典链路。^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

### 3．顾问升级的成本-隐私经济学：约 2/3 成本弥合 3/5 差距

Terminal Bench 2.1 上的数据构成一个清晰的取舍模型：完全本地运行成本近乎为零，得分 59.6%；升级到 Claude Opus 5 顾问后得分升至 73.0%，单次部署 API 成本约 0.415 美元；纯前沿模型直跑（Opus 5）得分 82.4%，成本约 0.65 美元。也就是说，用前沿成本的三分之二，弥合了与前沿差距的五分之三，且用户在"何时升级"上有完全决定权。这个权衡结构比"本地 vs 云端"的二元叙事更精细：端侧 harness 不是放弃前沿能力，而是把前沿能力降格为按需、受控、可审计的"顾问服务"——本地模型决定是否需要，PII 分类器把关泄露面，协调器保留工具权限。^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

### 4．从基准看端侧 harness 的真实护城河与短板

BrowseComp（研究型搜索）与 ParseBench-100（多模态文档理解）表明端侧 harness 在知识工作场景具备真实竞争力：搜索准确率 66.7%，对比 Pi 的 50.2% 与 Hermes 的 43.9%；多模态文档理解 65.1%，对比 34.6% 与 13.9%；且运行时间与 token 消耗全面领先（比 Hermes 快 61%、token 少 16%；比 Pi 的 token 少 70%）。护城河来自"本地文件为权威来源 + 公共来源做上下文 + 多模态原生支持"的组合。但短板同样明显：Terminal Bench 2.1 高难度编码任务上本地模型全面落后于前沿模型——这提示端侧 harness 当前的天花板由模型能力决定，顾问升级只能缩小、不能消除差距。^[raw/articles/on-device-harness-qwen38-27b-portable-computer.md]

## 实践启示

1. **先定上下文预算，再定框架复杂度**：即使模型标称窗口很大（26 万 token），实际有效窗口可能小得多（超过 10 万 token 即衰减）——按有效窗口设计最小核心框架，其余能力全部模块化为按需加载的技能。
2. **高频连接器优先 CLI 化**：MCP 服务器虽标准，但庞大的工具定义会挤占稀缺上下文；把最常用的连接器转成简洁命令行工具 + 自定义技能，是低成本的上下文优化手段。
3. **把安全做成默认而非配置项**：沙箱策略（限进程、限文件系统路径、限网络）+ 沙箱不可用时禁用而非降级，从机制上保证隔离永远生效，避免"配置漏配→裸奔"。
4. **用确定性协调器接管高风险职责**：循环维护、上下文构建、策略执行、工具调用审批交给框架代码，LLM 只保留"下一步做什么"的决策权——这是缓解提示注入与失控的根本性架构选择。
5. **顾问升级要设计泄露面控制**：升级前 PII 分类器标记、只发送已批准的上下文、顾问只返回文本指导、协调器保留工具权限——让"借用云端大脑"不变成"把家底交给云端"。
6. **小模型也可走后训练闭环**：用合成 RL 环境（Docker 容器 + 验证器）做拒绝采样 + 强化学习两阶段训练，能在不接触真实数据的前提下显著提升框架内表现（82.6% → 85.4%）。

## 相关实体

- [[entities/agent-harness-production|Agent Harness 生产化]]
- [[entities/harness-engineering-core-patterns-claude-code|Harness Engineering 核心模式]]
- [[concepts/harness-engineering-framework|Harness Engineering 框架]]
- [[entities/claude-code-agent-memory-four-levels-analysis|Claude Code Agent 记忆四层分析]]
- [[entities/openclaw-architecture-8-part-summary|OpenClaw 架构]]

→ [[raw/articles/on-device-harness-qwen38-27b-portable-computer|原文存档]]