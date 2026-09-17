---
title: "Agent Harness 6 种运行模式与 SDB 方法论"
created: "2026-07-14"
updated: 2026-09-17
type: "entity"
tags: [agent, harness, architecture, runtime-patterns, sdb, reliability]
confidence: 0.8
provenance_state: "extracted"
sources: [raw/articles/agent-harness-6-runtime-patterns-pikachu]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Agent Harness 6 种运行模式与 SDB 方法论

> Stanford 独立研究者 Vasundra Srinivasan 提出的生产级 LLM Agent 运行时架构方法论（arXiv 2605.20173），涵盖随机-确定性边界（SDB）、6 种运行时模式、5 步选择流程和 12 类失败签名。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

## Stochastic-Deterministic Boundary（SDB）

SDB 是 LLM 随机输出与系统确定性写入之间的接口，论文将其形式化为四部分契约 ^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]：

- **Proposer**：LLM 输出（天然带随机性）
- **Verifier**：确定性检查代码（JSON schema、权限、规则、安全）
- **Commit**：验证通过后的持久化写入（DB、API、消息队列）
- **Reject Signal**：验证失败时返回的类型化响应（让模型自我修正）

在 5 个主流开源 Agent 框架（21 个 LLM→action 调用点）审计中发现，19 个已有明确的 Verifier 和 Commit 逻辑，但从未被显式命名 ^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]。

## 三个正交维度

| 维度 | 核心问题 | 形式化来源 |
|------|---------|-----------|
| Coordination（协调） | 工作怎么拆分和组合？ | Hewitt Actor 模型 |
| State（状态） | 系统怎么记忆？ | CAP 定理、事件时间 vs 处理时间 |
| Control（控制） | 谁决定什么运行、何时停止？ | 控制理论、Erlang 监督树 |

> LLM Agent 是分布式系统经典理论在「随机提议者」这一新成员出现后的重新组装。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

## 6 种运行时模式

1. **分层委托（Hierarchical）**：主管 Agent → 子 Agent，对话式 Agent 的典型形态
2. **分散-聚合 + 补偿（Scatter-Gather + Saga）**：任务打散并行跑，失败时按 Saga 补偿回滚
3. **事件驱动排序（Event-Driven）**：事件作为真相来源，按工作流网推进——有「重放分歧」风险
4. **监督者 + 门控（Supervisor + Gate）**：Policy/Budget/Role 三类 Gate，高风险操作标配
5. **共享状态机（Shared State Machine）**：分布式状态机做单一真相来源
6. **人在回路（Human-in-the-Loop）**：LLM 提议 → 人类审批 → Commit

## 可靠性分解公式

$$y(t) = μt + σξ(t)$$

- σ（模型方差）：由模型质量决定，随模型迭代持续压缩
- μ（架构动量）：由模式选择 + SDB 设计决定，换模型不会自动变好
- 当 σ → 0 时，μ 主导整体可靠性

> 当 LLM 强到「随机性几乎消失」那天，Agent 系统的可靠性瓶颈将 100% 落在架构选择上。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

## 典型失败签名

| 模式 | 故障 | 缓解 |
|------|------|------|
| P3 | Replay Divergence（重放分歧） | 版本化消费者 + 提示版本控制 |
| P2 | Saga 补偿失灵 | 精确触发条件判断 |
| P4 | Gate 配置错误 | 按业务调阈值 + Policy-as-Code |
| P5 | 共识冲突 | 额外序列化层 |
| P6 | 审批超时 | 异步审批 + SLA 监控 |
| P1 | 子 Agent 输出未聚合 | 强制汇聚回主管再 commit |

## 深度分析

### 为什么「换个更强模型」救不了 Agent

可靠性公式 y(t) = μt + σξ(t) 把两个来源明确拆开：σ 是单次调用的随机方差，归模型质量管；μ 是架构动量，归模式选择与 SDB 设计管。σ 的下降是确定趋势，模型每迭代一轮就被压掉一截；μ 却不会因为换模型而自动变好，因为它描述的是「提议者」之外的那些结构性选择。由此推出一个反直觉结论：模型越强，架构选择在可靠性中的权重越大——当 σ 趋近于零时 y(t) 几乎完全由 μt 决定，继续加码模型能力的边际收益趋近于零。论文对 21 个已发布失败案例的分类也指向同一处：15 个（71%）事故的根因落在随机输出与确定性写入之间的那条边界上，而不是落在模型不够聪明上。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

### SDB 四部分契约的工程含义

SDB 的价值不在于发明新组件，而在于给已有实践命名。审计 5 个主流开源框架的 21 个 LLM→action 调用点，其中 19 个已经有明确的 Verifier 与 Commit 逻辑，只是从未被显式标识。命名之后，契约归属才变得可执行：Verifier 与 Commit 必须由确定性代码持有，不能交给模型自评——一旦让 LLM 自己判断「我这次输出是否合法」，边界就重新退化为随机的。Reject Signal 则是模型侧的自我修正接口：验证失败时返回类型化的拒绝原因，模型才有机会带着信息重试；缺了它，agent 只剩盲重试一条路，重试越多成本越高而收敛性越差。所以补齐 SDB 不是多加一层校验，而是把「失败」从异常转成被设计过的输入。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

### 6 种模式是积木而非单选

模式目录里的六项不是互斥选项，而是可以叠加的构件。生产系统里最常见的组合是 [[concepts/agent-orchestration-patterns|分层委托]] 负责拆解、监督者 + 门控守住高风险操作、人在回路兜住不可逆写入。叠加带来可靠性收益，也带来协调开销：每多一层监督，就多一次状态同步与一次超时判定的机会，控制面的复杂度随之上升。权衡点在于可逆性——可逆操作交给自动 Gate 就够，不可逆操作才值得付出人审的延迟成本。论文的提醒很直接：生产事故里绝大多数事后看都是「用错了模式」或「漏掉了配套机制」，而非模式数量不够。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

### 失败签名与模式的映射

六类签名的价值在于把故障定位回模式层。P3 重放分歧出在事件驱动模式：同一份事件日志在不同模型版本或提示版本下产生不同输出，因此需要版本化消费者加提示版本控制，再叠一层输出差异检测。P2 的 Saga 补偿失灵源于 LLM 非确定性让「补偿何时该触发」难以判断，缓解只能落在更精确的触发条件上。P4 的 Gate 配置错误本质是策略漂移，阈值要按业务调，并用 Policy-as-Code 保证规则随时可覆盖。P5 的共识冲突是随机提议者与状态机一致性要求之间的结构性摩擦，只能靠额外序列化层吸收。P6 把审批改成异步并挂 SLA 监控，避免人成为阻塞点。P1 的解法最朴素：强制子 Agent 输出汇聚回主管再 commit，堵住「只派不收」的漏口。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

### 5 步流程的输出是一份 ADR

五步流程从分类运行时、选状态主干、用协调包装、用控制边界一路到排序构建，最终交付物是六行架构决策记录。ADR 的意义在于把架构选择从口头共识变成可审阅、可回滚的书面资产：新成员能读懂当初为什么这样切分，出事故时能对照记录判断到底是模式选错还是配套机制缺失。第五步的自检清单尤其可操作——逐个检查每个 LLM→action 边界是否都具备 Verifier、Commit 与 Reject Signal，缺口一目了然。放回更大图景看，SDB 处理的是运行时面，而 skill 设计、schema 约束与 [[concepts/agent-evaluation-benchmark-frameworks|评估体系]] 处理的是静态面，两者互补：静态面决定模型看到什么，运行时面决定模型的输出如何被接纳。^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]

## 实践启示

这套方法论落到执行可以拆成一组具体动作，配套的可观测性建设参见 [[entities/agent-harness-observability-production|生产环境 Harness 可观测性]] ^[raw/articles/agent-harness-6-runtime-patterns-pikachu.md]：

1. 逐个枚举 LLM→action 边界，为每一处标注是否具备 Verifier、Commit 与 Reject Signal；缺项优先补齐，尤其不要用模型自评代替确定性校验。
2. 对高风险、不可逆的写入一律加 Gate（Policy/Budget/Role），并把策略写进代码而非文档，保证规则更新能随发布生效。
3. 长时任务把事件日志当主干：事件溯源天然提供可重放与审计能力，但必须同时给消费者和提示词打版本，否则 P3 会在模型升级那天集体爆发。
4. 用模式目录而非单练一种模式：先按 5 步流程选主干与协调方式，再叠加控制边界，最后用自检清单回扫，并把选择写进 ADR。
5. 把人在回路当异步 SLA 而不是阻塞点：定义审批时限与超时后行为，让人审延迟可度量、可预期，而不是让 agent 在等待里空转。
6. 换模型之前先看 μ：模型升级只压缩 σ，若事故根因落在边界、模式或配套机制上，升级不会自动带来可靠性提升。

→ [[raw/articles/agent-harness-6-runtime-patterns-pikachu|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

