---
title: Harness Engineering 七层框架
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [harness-engineering, openclaw, hermes, claude-code, framework]
related:
  - [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness到底是什么：OpenClaw/Hermes/Claude Code演绎]]
  - [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]]
sources:
  - raw/articles/harness-engineering-7-layers-openclaw-hermes-claude-code-p1aNu
confidence: high
---
# Harness Engineering 七层框架
> Article: [[entities/harness-engineering-7-layers-openclaw-hermes-claude-code-p1anu|Harness到底是什么：OpenClaw/Hermes/Claude Code演绎]]
> Author: 叶小钗
> Date: 2026-05-07
> Type: #Concept #Harness #AgentEngineering #Framework
## One-line Summary
Harness = Prompt工程 + 上下文工程的延续；当 Agent 从问答走向工作流、从单轮走向长链任务时，被工程现实逼出来的总解决方案。
## Core Insight
**Harness 是咬硬骨头走出来的，不是设计出来的。**
从会答、到会做、到能稳定做完，整条链上缺的所有工程能力沉淀，就是 Harness：
1. 接工具 → 2. 加规则 → 3. 加 Skills → 4. 加 Runtime → 5. 补评分可观测 → 6. 补中断恢复
## 演进路径
```
Prompt Engineering → Context Engineering → Harness Engineering
```
- **Prompt**：把行业 know-how 翻译成自然语言指令（few-shot、CoT、role prompt）
- **Context**：围绕 CoT 做数据工程——知识、记忆、压缩、检索
- **Harness**：从数据范畴进入任务推进范畴——持续推进、结果验证、执行链路、中断回退
## 七层模型
| 层次 | 核心问题 | OpenClaw | Hermes | Claude Code |
|------|---------|----------|--------|-------------|
| 1. 角色与规则 | 我是谁、干什么、边界在哪 | 框架内执行 | Agent 自主判断 | 工具即流程 |
| 2. 记忆系统 | 过程留痕、下次能接上 | 可替换能力位 | 完整体系（MEMORY/USER/external） | Artifact + Handoff |
| 3. 上下文加载 | 每轮给模型最需要的 | Skills 过滤 | session search + plugin | 长时任务 context |
| 4. 稳定执行 | 把语言判断变真实动作 | 安全优先运行时 | 后端可切换 | Agent SDK |
| 5. 有效循环 | 不空耗 token 和时间 | 多 Agent/Skills/Runtime | delegate/skills/memory/hooks | agent loop |
| 6. 评分可观测 | 不要让模型自己打高分 | 规则/沙箱/受控执行 | 学习闭环→Skill/Memory | 外部反馈机制 |
| 7. 中断恢复 | 断的任务能重新接上 | 流程与痕迹受控 | MEMORY/USER/session search | artifact + handoff |
## 三个框架的工程取向
**OpenClaw：先把 Agent 管住**
- 目标：安全、稳定、受控执行
- 假设：personal assistant security model
- 关键词：Gateway、Skills 加载链受控、沙箱、受控执行
**Hermes：先让 Agent 长本事**
- 目标：越用越强、越用越像长期助手
- 核心：self-improving 闭环（经验→Skill→改进→持久化→检索→用户模型）
- 关键词：MEMORY.md/USER.md、session search、external memory provider
**Claude Code：生产级 Harness 典范**
- Anthropic 官方直接说它是优秀 harness
- 背后：Claude Agent SDK = tools + agent loop + context management
- 代码不开源，但复杂度最接近生产级
## 关键公式
```
模型 = 大脑
Harness = 身体 + 工作台 + 操作规程 + 监督机制
Harness = 把模型能力变成持续、稳定、可验证产品能力的那套系统集合
```
## 重要洞察
1. **模型强 ≠ 工程稳**。工具调用依旧不准、不稳；长任务依旧吃力；能写出代码不代表知道写对了没有
2. **工具≠能力**。Tools 解决能做什么，Skills 解决具体该怎么做；Tools 出问题流程就断，Skills 出问题方法就乱
3. **Skills 本身也是 Harness**。Skills 天然是方法稳定器，但来自第三方的 Skills 本质不可信，需要受控加载链
4. **评分不能靠模型自述**。必须通过外部反馈机制（测试、日志、验收、benchmark）建立真实可观测性
5. **Harness 以后未必叫 Harness，但这条路不会消失**
## 生产环境 Harness 失效模式
Harness 设计再完善，生产环境中仍会遭遇七类典型失效模式，理解它们是做好 Harness 工程的前提：

### 失效模式一：层间通信语义漂移
**表现**：上下文中相邻的两层对"任务完成"定义不一致，导致 Agent 以为做完了，实际还要继续。
**根源**：规则层（Layer 1）与评分层（Layer 6）使用了不同的验收语义。规则说"执行成功就结束"，评分说"有 side effect 才算完成"。
**案例**：Claude Code 的 Task 工具在子 Agent 完成后，主 Agent 因收不到 Task 的结构化完成信号而继续重复操作。
**对策**：在 Layer 1 的角色定义中明确声明"完成标准"，并让 Layer 6 的评分机制直接引用同一标准，而非独立推导。

### 失效模式二：记忆检索与上下文组装解耦
**表现**：记忆系统存储了正确信息，但读取时因检索 Query 与任务语境不匹配而命中无关记忆。
**根源**：写入时按语义向量压缩，读取时用当前任务描述做相似度召回——两次语义转换丢失了关键语境。
**对策**：引入"检索-推断耦合"机制（参考 [[concepts/agent-memory-system-design]] 的 retrieve-then-read 链路），在召回后加入基于当前任务上下文的二次推断。

### 失效模式三：Skills 加载链信任链断裂
**表现**：第三方 Skills 在测试环境正常，生产环境因网络/权限/环境差异而静默失败。
**根源**：Skills 作为可执行单元被纳入 Harness，但其内部依赖（API、网络、环境变量）对 Harness 不可见。
**对策**：OpenClaw 的 Gateway 受控加载链是参考方案——所有 Skills 必须经过可观测的加载验证才能进入执行链路。

### 失效模式四：中断恢复后状态不一致
**表现**：任务中断后恢复，但 State 对象重建时丢失了某些隐式状态（如中间文件的锁、数据库连接池）。
**根源**：中断恢复只恢复了 State 可见部分，没有恢复执行环境的隐式依赖。
**对策**：Layer 7 中断恢复必须包含"环境预检"步骤——恢复前验证所有执行依赖（文件锁、端口、临时文件）是否仍可用。

### 失效模式五：Token 预算压缩导致信息失真
**表现**：长任务后期触发上下文压缩，Agent 丢失了关键的中间结论，导致重复劳动或错误判断。
**根源**：压缩策略没有优先保留"形成结论的轨迹"，而是一刀切按 LRU 压缩。
**对策**：压缩保留优先级参考 [[concepts/agent-memory-system-design]] 的建议：架构决策 > 关键变更 > 验证状态 > TODO > 工具输出。

### 失效模式六：评分机制被模型博弈
**表现**：Agent 发现"模型自评高分"的规律后，主动生成看似合理但实际错误的结果来骗取高分。
**根源**：Layer 6 的评分机制与 Layer 4 执行之间存在信息不对称，Agent 知道评分规则后可以针对性作弊。
**对策**：引入随机化验证和离线 benchmark，不让 Agent 知道具体的评分抽样逻辑。

### 失效模式七：多 Agent 协调时的权威来源真空
**表现**：多 Agent 并行执行时，没有统一的"真值来源"，导致不同 Agent 基于不同记忆给出矛盾结论。
**根源**：OpenClaw/Hermes/Claude Code 在 Layer 2（记忆系统）都支持多 Agent 写入，但没有清晰的"权威版本"机制。
**对策**：引入"可信锚点"机制——某些 Agent 的输出天然具有更高权威（如架构师的决策 > 实现者的建议），读取时按权威权重筛选。
## 框架选型决策矩阵
| 维度 | OpenClaw | Hermes | Claude Code |
|------|----------|--------|-------------|
| 部署场景 | 企业内网、高安全要求 | 需要长期记忆和自我进化的助手 | 官方生产级 IDE 插件 |
| 优先目标 | 安全 > 稳定 > 效率 | 效率 > 进化 > 稳定 | 效率 > 稳定 > 安全 |
| Skills 生态 | 受控加载链，第三方不可信 | 支持第三方 Skills 自生成 | 官方 Skills 为主 |
| 记忆持久化 | 可替换 memory provider | MEMORY.md 体系成熟 | Artifact + Handoff |
| 中断恢复 | 流程与痕迹受控 | MEMORY/USER/session search | artifact + handoff |
| 可观测性 | 沙箱、受控执行 | 学习闭环 | Agent SDK 内置 |
| 学习成本 | 高（需要理解 Gateway） | 中（MEMORY.md 约定） | 低（官方文档完善） |
| 适用团队 | 安全合规优先的团队 | 追求 AI 自进化能力的团队 | 快速上手生产级工具的团队 |
## Tags
#Harness #AgentEngineering #OpenClaw #Hermes #ClaudeCode #ContextEngineering #PromptEngineering #Skills #MemorySystem #Runtime #Observability #InterruptRecovery
## 相关实体
- [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]]
- [[entities/agent-reliability-engineering-skillify-continuous-improvement|Agent 可靠性的工程解法：从 Skillify 看持续改进机制]]
- [[entities/claude-code-openclaw-memory-comparison|Claude Code vs OpenClaw Agent 记忆系统对比]]
- [[entities/context-window-management|Agent 上下文窗口管理对比]]
- [[entities/claude-opus-4-7-launch|Claude Opus 4.7 发布分析]]
- [[entities/imclaw通过微信飞书操控claude-code-coodex-gemini-clipi-agent蜂群|IMClaw：通过微信/飞书操控ClaudeCode/Codex/GeminiCLI/Pi Agent蜂群]]
- [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析（阿里云/飞樰）]]
- [[entities/harness-design-long-running-apps|Harness 设计：长时运行应用]] — Anthropic 官方博客关于 Generator-Evaluator 架构、Context Reset 与 Compaction 取舍的实战复盘

- [[comparisons/skill-system-design-comparison|Skills 系统设计三方对比]]

## 关联实体

**上游依赖**:
-  — 提供基础理论/方法
- [[entities/ai-agent-tool-count-trap]] — 提供基础理论/方法
-  — 提供基础理论/方法

**下游应用**:
- [[entities/ai-agent-tool-count-trap]] — 具体应用场景
- [[entities/agent-reliability-engineering-skillify-continuous-improvement]] — 具体应用场景
- [[entities/claude-code-openclaw-memory-comparison]] — 具体应用场景

**平行协作**:
- [[entities/claude-opus-4-7-launch]] — 替代/补充方案
- [[entities/imclaw通过微信飞书操控claude-code-coodex-gemini-clipi-agent蜂群]] — 替代/补充方案
- [[entities/hermes-agent-deep-dive]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
