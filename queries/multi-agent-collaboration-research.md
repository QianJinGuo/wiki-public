---
title: "多智能体协作的核心模式与最佳实践是什么？"
created: "2026-05-21"
updated: "2026-05-21"
type: query
tags: [research-question, multi-agent, collaboration-patterns, orchestration, agent-architecture]
sources:
  - raw/articles/17-agent-architectures-evolution
  - raw/articles/agent-engineering-principles-architecture-practice
  - raw/articles/autoresearch-multi-agent-software
  - raw/articles/sub-agent-vs-agent-team-selection-guide
related:
  - concepts/multi-agent-systems
  - concepts/ai-team-knowledge-harness
  - entities/17-agent-architectures-evolution
  - entities/autoresearch-multi-agent-software
---

# 多智能体协作的核心模式与最佳实践是什么？

## 核心答案

多智能体协作的本质是**通过角色分解、视图多样性和交叉验证，将超出单 Agent 能力边界的任务分配给专业化的子 Agent 群体**。不同 Agent 拥有不同的知识体系、推理盲区和强项，协作可以发现单 Agent 永远发现不了的问题。^[raw/articles/17-agent-architectures-evolution.md]

但协作的前提是**协议先于协作，隔离先于并行**——没有协议和隔离的多 Agent 并行只会放大幻觉和问题。^[raw/articles/agent-engineering-principles-architecture-practice.md]

---

## 五大核心架构模式

### 1. Multi-Agent（角色分解）—— 固定流水线

将任务分解为研究员、写作、审阅等不同角色，流水线串联执行。每个角色是独立的 Agent，拥有专属的系统提示和能力边界，角色在编译时确定，流水线顺序固定。

**适用场景**：角色边界清晰、任务流程相对固定的生产环境。推荐优先级：在引入动态调度之前先用固定流水线验证角色边界是否正确。^[raw/articles/17-agent-architectures-evolution.md]

**最佳实践**：
- 先用固定流水线验证角色边界，再考虑动态调度
- 每个角色的系统提示必须边界清晰，避免职责重叠
- Validator-Creator 分离是实践证明有效的角色拆分方式

### 2. Blackboard（共享黑板）—— 动态调度

所有子 Agent 将中间产物写入共享黑板（shared workspace），Controller 在运行时动态决定激活哪个 Agent。控制流是动态的——哪个 Agent 下一步执行，由 Controller 根据当前状态判断。

**适用场景**：任务复杂度随时间增长、角色边界不清晰、需要运行时自适应调度的场景。**注意**：动态调度带来 controller 决策稳定性、信息冲突和循环风险等新失败模式，过早引入动态调度是常见反模式。^[raw/articles/17-agent-architectures-evolution.md]

### 3. Meta-Controller（入口分诊）—— 单次路由

入口级 Router 根据请求特征一次性判断应该路由到哪个专家子 Agent，之后由该 Agent 全权处理。不同于 Blackboard 的运行时全程调度，Meta-Controller 只做一次性的分诊决策。

**适用场景**：请求类型明确可分类、专家子 Agent 能力高度专业化、调度开销必须最小化的场景。^[raw/articles/17-agent-architectures-evolution.md]

### 4. Ensemble（并行冗余）—— 投票融合

同一个任务交给多个 Agent 并行独立处理，结果通过投票或评分融合得出。用冗余换可靠性——单 Agent 的随机失误可以被群体判断纠正。

**适用场景**：高可靠性要求的结论（如代码审查、安全判断），以及需要探索能力上限的任务。AutoResearch 项目的多 Agent 交叉审核本质上是 Ensemble 模式的变体：Codex 和 Claude 轮流担任实现者和审核者。^[raw/articles/17-agent-architectures-evolution.md]

### 5. PEV（Plan-Execute-Verify）—— 验证接进主回路

Plan → Execute → Verify 三步循环，失败后回重规划。与 Multi-Agent 的区别在于 PEV 将验证作为主回路的一部分，而非外部质量门禁。

**适用场景**：工具调用类任务，验证结果直接影响下一步决策。^[raw/articles/17-agent-architectures-evolution.md]

### 架构选型对照表

| 你缺什么 | 优先架构 | 原因 |
|----------|----------|------|
| 角色分工 | Multi-Agent / Blackboard / Meta-Controller | 固定→动态→分诊 |
| 高可靠结论 | Ensemble | 冗余降低偏差 |
| 工具容错 | PEV | 验证接进主回路 |

^[raw/articles/17-agent-architectures-evolution.md]

---

## 编排模式：集中式 vs 分布式

### 集中式编排（统筹者模式）

统筹者（Orchestrator）在起点接收任务，在终点汇总结果，中间通过异步委派将子任务分发给专业 Agent。统筹者模式的关键优势是**可持久化**：人与统筹者单点交互，session 结束后的产出是分支/PR 等可持久化工件，协作上下文可以跨越 session 存在。

典型实现：OpenClaw 的 Pi Agent 担任统筹者角色，通过 MessageBus 分发任务给专业 Agent 子群体。^[raw/articles/agent-engineering-principles-architecture-practice.md]

### 分布式编排（去中心化）

没有全局统筹者，Agent 群体通过协议自发协作。极端形式是 Cellular Automata 架构：LLM 只设计局部规则，全局行为从局部交互中涌现。^[raw/articles/agent-engineering-principles-architecture-practice.md]

### 两种模式的核心对比

| 维度 | 集中式（统筹者） | 分布式（去中心化） |
|------|-----------------|-------------------|
| 任务协调 | 统筹者统一分配 | Agent 间协议自发协调 |
| 状态管理 | 统筹者维护全局状态 | 共享黑板或分布式状态 |
| 失败恢复 | 统筹者触发重路由 | 依赖 Agent 自主决策 |
| 适用规模 | 中小规模角色固定 | 大规模灵活协同 |
| 工程复杂度 | 较低（协议简单） | 较高（需处理循环和信息冲突） |

^[raw/articles/agent-engineering-principles-architecture-practice.md]

---

## 通信协议：协作的基石

多 Agent 通信协议的设计直接决定系统的协作质量和可维护性。核心原则是**协议先于协作**——先设计好 Agent 之间的通信契约，再开始协作开发。

### JSONL Inbox 协议

每个子 Agent 拥有独立的 JSONL 格式 Inbox 文件，主 Agent 将任务写入对应子 Agent 的 Inbox，子 Agent 轮询处理。JSONL 的追加写入特性天然适合异步任务队列，避免并发写冲突。^[raw/articles/agent-engineering-principles-architecture-practice.md]

### 任务图（Task Graph）

明确子任务之间的依赖关系和执行顺序，用有向无环图（DAG）管理。任务图是可持久化的——跨 session 恢复时只需重新加载图结构，不需要重新理解任务分解逻辑。^[raw/articles/agent-engineering-principles-architecture-practice.md]

### Worktree 隔离

每个子 Agent 在独立的工作目录中操作，文件系统级别隔离。隔离确保子 Agent 的操作不会相互干扰，也使得问题定位更容易——错误行为可以精确归因到某个 Agent 的工作空间。^[raw/articles/agent-engineering-principles-architecture-practice.md]

### 摘要回传机制

子 Agent 只向主 Agent 回传摘要信息，探索细节留在自己的消息历史里。过度传递信息会引发误解和上下文膨胀——**幻觉会互相放大**，当一个 Agent 的错误信息传递给另一个 Agent 时，错误可能被二次放大。^[raw/articles/agent-engineering-principles-architecture-practice.md]

---

## 任务分解策略

### 角色驱动分解

按专业角色划分：研究员 Agent（负责信息收集）、实现者 Agent（负责代码编写）、审核者 Agent（负责质量把关）。适用于流程相对稳定、可提前定义角色边界的场景。^[raw/articles/17-agent-architectures-evolution.md]

### 层级分解（Hierarchical Decomposition）

Meta-Controller 在入口做一次分诊，将任务路由到下一级专业 Agent；专业 Agent 内部可能再分解为更细粒度的子任务。这是大型软件系统模块化思想在 Agent 架构中的映射。^[raw/articles/17-agent-architectures-evolution.md]

### 交叉验证分解

同一任务由两个不同 Agent 独立完成，结果通过评分或投票融合。AutoResearch 项目的实践：Codex 和 Claude 轮流担任实现者和审核者，5 维度量化评分 ≥ 9.0 才允许合并。交叉验证的核心价值在于**多样性发现**：不同模型有不同的推理盲区，交叉审核能发现单 Agent 视角下永远看不到的问题。^[raw/articles/autoresearch-multi-agent-software]

### 反馈驱动迭代分解

将任务分解为多轮迭代，每轮审核的反馈直接注入下一轮的 Agent 提示词，Agent 针对性改进而非盲重试。这种机制将迭代效率从"掷骰子"提升到"有方向地试错"。^[raw/articles/autoresearch-multi-agent-software]

---

## 评测框架

多 Agent 系统的评测比单 Agent 更加复杂，因为需要同时评估协作质量和个体贡献。

### 探索能力 vs 回归能力：Pass@k 与 Pass^k

- **Pass@k**：k 次尝试中至少一次正确 → 评估 Agent 的**探索能力上限**
- **Pass^k**：k 次尝试全部正确 → 评估 Agent 的**上线回归**能力

两个指标满足不同阶段：Agent 研发阶段需要高 Pass@k 验证能力边界，生产环境需要高 Pass^k 保证质量稳定。^[raw/articles/17-agent-architectures-evolution.md]

### 评分器确定性排序

| 优先级 | 评分器类型 | 特点 |
|--------|-----------|------|
| 1 | 代码评分器 | 确定性最高，自动化程度最高 |
| 2 | 模型评分器 | 依赖 LLM 判断，有随机性 |
| 3 | 人工评分器 | 成本高，一致性难以保证 |

^[raw/articles/17-agent-architectures-evolution.md]

### AutoResearch 5 维度加权评分

| 维度 | 权重 | 说明 |
|------|------|------|
| 功能正确性 | 35% | 实现是否符合 Issue 要求 |
| 测试充分性 | 25% | 测试覆盖是否完整 |
| 代码质量 | 20% | 规范、可读性、结构 |
| 安全性 | 10% | 是否有安全漏洞 |
| 性能 | 10% | 是否有性能问题 |

总分 10 分，≥ 9.0 达标。^[raw/articles/autoresearch-multi-agent-software]

---

## 知识沉淀：协作的持久化机制

[[concepts/ai-team-knowledge-harness|AI Team 知识沉淀体系]] 提供了团队级别知识管理的三维框架：

| 维度 | 回答的问题 | 构成 |
|------|------------|------|
| **存储层**（在哪） | 知识存在哪里？ | Layer 0-P / 0-T / 1 / 2 / 3 |
| **知识类型**（是什么） | 知识描述的是什么？ | model / decision / guideline / pitfall / process |
| **成熟度**（多可信） | 知识经过多少验证？ | draft → verified → proven |

关键原则：**Skill、Agent、工具链会随模型迭代更新，但领域知识是永恒的**。工作流可替换，知识可累积——这是多 Agent 协作形成长期价值的关键。^[raw/articles/tencent-knowledge-harness-practice.md]

Thin Harness Fat Skills 与 AI Team 知识沉淀体系的互补性：Fat Skills 强调个人层面的知识积累（Markdown 技能文件随个人使用自动进化），而 AI Team 强调团队层面的知识验证（跨个人、跨项目的知识共识机制）。

---

## 常见反模式

基于  和  的总结：

- **多 Agent 无边界**：角色职责不清晰，Agent 之间没有协议约束，协作混乱
- **过早引入多 Agent**：单 Agent 尚未稳定就上多 Agent，把单 Agent 问题放大 N 倍
- **指挥者模式（同步协作）**：人与单个 Agent 紧密互动，session 结束所有协作记忆丢失
- **信息过度传递**：子 Agent 回传全部细节而非摘要，导致上下文膨胀和误解传播
- **缺少验证机制**：没有质量门禁，多 Agent 并行产出的结果直接合并
- **约束靠期望不靠机制**：依赖 Agent"理解"应该做什么，而非用协议和工具强制约束

---

## 实践框架：决策流程

```
1. 任务复杂度评估
   └─ 是否超出单 Agent 能力边界？
       ├─ 否 → 单 Agent 即可，不要强行多 Agent
       └─ 是 → 继续下一步

2. 角色边界清晰度
   ├─ 边界清晰 → Multi-Agent（固定流水线）
   └─ 边界模糊 → Blackboard（动态调度）

3. 可靠性要求
   ├─ 高可靠性 → Ensemble（并行冗余）
   └─ 普通 → 继续角色分解

4. 协作上下文是否需要跨 session 持久化？
   ├─ 是 → 统筹者模式（集中式编排）
   └─ 否 → 去中心化协议

5. 设计完成后，先跑评测再优化 Agent
```

---

## 相关概念

- [[concepts/multi-agent-systems]] — 多智能体系统的完整架构模式和评测框架
- [[concepts/ai-team-knowledge-harness]] — AI Team 知识沉淀体系，团队协作的持久化机制
-  — 17 种 Agent 架构的完整演化分析
-  — 多 Agent 软件开发的交叉审核实践
-  — Agent 工程实践，含 Harness、上下文工程、多 Agent 隔离协议
