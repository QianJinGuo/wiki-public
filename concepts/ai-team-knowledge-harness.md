---
title: AI Team 知识沉淀体系
created: 2026-04-27
updated: 2026-08-01
type: concept
tags: [knowledge-management, harness-engineering, team-knowledge, knowledge-lifecycle, context-engineering, workflow-automation, karpathy-llm-wiki, knowledge-layering]
related:
  - [[entities/context-engineering-three-memory-paradigms]]
  - [[entities/harness-engineering|Harness Engineering]]
  - [[entities/thin-harness-fat-skills]]
sources: [raw/articles/tencent-knowledge-harness-practice]
review_value: 9
review_confidence: 10
review_score: 90
confidence: high
---
# AI Team 知识沉淀体系
## 概述
**AI Team** 是腾讯内部一个 AI 工程交付团队的实践体系，核心主张是：**Harness 不是目的，知识才是护城河**。工作流只是管道，知识才是流过管道的活水。该体系将知识管理作为 Harness Engineering 的核心能力而非附属品，系统性地解决了团队知识沉淀、共建共享、按需消费、生命周期治理的闭环问题。
> "将来的技术护城河不在模型，而在垂直领域知识的沉淀。"
## 核心架构：三维正交知识体系
### 三个维度
| 维度 | 回答的问题 | 构成 |
|------|------------|------|
| **存储层**（在哪） | 知识存在哪里？ | Layer 0-P / 0-T / 1 / 2 / 3 — 从个人到团队到项目 |
| **知识类型**（是什么） | 知识描述的是什么？ | model / decision / guideline / pitfall / process |
| **成熟度**（多可信） | 知识经过多少验证？ | draft → verified → proven |
### 五层存储架构
| 层级 | 路径示例 | 说明 |
|------|---------|------|
| Layer 0-P | `~/.ai-team/` | 个人偏好，纯本地不共享 |
| Layer 0-T | `team-conventions/` | 团队约定，Git 共享 |
| Layer 1 | `tech-wiki/` | 技术知识，跨项目通用 |
| Layer 2 | `biz-wiki/{domain}/` | 业务知识，按领域划分 |
| Layer 3 | `docs/knowledge/` | 项目知识，随项目走 |
**向上提升机制**：Layer 3 的项目知识，如果被判定为跨项目通用，会自动提升到 Layer 1 或 Layer 2。
### 五种知识类型（MECE 原则）
| 类型 | 定义 | 示例 |
|------|------|------|
| **model** | 实体定义、数据结构、关系图 | "广告计划包含预算/出价/投放时段三个核心字段" |
| **decision** | 技术选型、架构决策及理由 | "选择事件驱动而非 RPC 同步" |
| **guideline** | 推荐做法或禁止做法 | recommend: "公共模块变更后的兼容性检查清单" |
| **pitfall** | 已知风险、故障模式、排查步骤 | "广告预算扣减在高并发下会超扣" |
| **process** | 业务流程、状态机、操作步骤 | "广告审核：提交→机审→人审→上线" |
### 三级成熟度 + 自动衰减
```
draft（新提取，单一来源）
  ↓ 在 1 个工作流中被成功引用
verified（单项目验证）
  ↓ 在 ≥2 个不同项目中被验证
proven（成熟/可信赖）
```
**自动衰减机制**：
- proven 条目 12 个月未被引用 → 降级为 verified
- verified 条目 6 个月未被引用 → 降级为 draft
- draft 条目持续未引用 + Lint 标记 → 归档，移出活跃索引
## 团队知识库：独立 Git 仓库
### 目录结构
```
team-knowledge.git
├── knowledge-catalog.md          # 全景目录（Agent 查询入口）
├── .knowledge-config.yaml       # 团队配置
├── team-conventions/           # Layer 0-T: 团队约定
├── tech-wiki/                  # Layer 1: 技术知识
├── biz-wiki/                   # Layer 2: 业务知识
├── project-profiles/           # 项目画像
└── contributions/             # 贡献暂存区
```
### 三种团队角色
| 角色 | 权限 |
|------|------|
| maintainer | 裁决内容冲突、审批 proven 提升、管理成员 |
| contributor | 通过工作流自动贡献（创建/验证/标记矛盾） |
| reader | 只消费知识（查询/注入），不贡献 |
### 贡献模式：借鉴区块链思想
- **不可篡改的追加日志**：`log.md` 只追加不修改，每条变更记录贡献者、时间、会话哈希
- **贡献可溯源**：`evidence.contributors[]`，类似 Git blame，粒度为知识条目级
- **共识机制**：draft→verified: 1 人验证; verified→proven: ≥2 人 + ≥2 项目
### 冲突解决策略
| 冲突类型 | 处理方式 |
|----------|----------|
| 纯新增（不同条目） | 自动合并 |
| 证据追加（同条目验证） | 自动合并 |
| 内容矛盾 | 写入 `contributions/conflicts/`，通知 maintainer 裁决 |
| 成熟度冲突 | 保留较低成熟度 + 标记 contradiction |
## 知识成熟度与上下文工程的关系
AI Team 的三级成熟度模型（draft → verified → proven）与更广泛的上下文工程（Context Engineering）文献之间存在深层联系。Context Engineering 将记忆分为隐式记忆（MSA/KV cache）、参数记忆（D2L/LoRA）和外部记忆（RAG/向量库）三类，而 AI Team 的知识成熟度体系实际上是在**外部记忆层之上又叠加了一层组织验证层**。

关键区别在于：通用 RAG 系统（如 [[entities/context-engineering-three-memory-paradigms]] 中描述的方案）只负责存储和检索，不负责验证知识的真实性和适用范围。而 AI Team 的 proven 级别对应的是**在多个不同业务场景中被验证仍然有效的知识**——这种跨项目验证在 RAG 语境中相当于做了多次针对性的召回评估（recall evaluation），只不过评估指标不是召回率，而是**决策正确率**。

这一设计对 [[entities/harness-engineering|Harness Engineering]] 的发展有重要启示：Harness 的核心价值不仅是提供工具调用能力，还包括维护一套可靠的知识库，使 Agent 在做决策时能调用经过组织验证的事实而非仅依赖模型自身权重中的统计模式。当 Harness 与知识库解耦时（如纯 prompt-based agent），模型的输出质量完全取决于 prompt 表达能力和模型本身；而当 Harness 内嵌了 proven 知识库时，Agent 的决策质量还额外取决于知识库的覆盖率和时效性——这正是 Karpathy 在 LLM Wiki 中所说的 "knowledge compounding" 的核心机制。
## 知识作为团队复利资产：量化价值分析
AI Team 体系的核心经济学洞察是**知识具有复利属性**，而多数 AI 团队在工具链上的投入被消耗在重复造轮子上，没有形成积累。以下从信息论和组织行为学两个维度分析这一现象。

从信息论角度，单个 decision 条目（技术选型决策）的价值不在于它记录了什么，而在于它**节省了多少未来决策的信息熵**。一个 proven 级别的 decision 条目，其信息量等价于：在这类决策上，未来团队成员无需再次进行等效的信息收集、方案评估和风险权衡。假设一个技术选型决策平均需要 4 小时，而团队每年面临 50 次类似决策，则一条 proven decision 条目的年化价值约为 200 小时。随着知识库中 proven 条目增加，新成员的启动时间（onboarding time）会指数级下降——这与 [[entities/thin-harness-fat-skills]] 中"技能文件随使用积累"的观点一致，只是 AI Team 将这个过程从个人层面推广到了团队层面。

从组织行为学角度，知识不共享的根源在于**贡献成本不对称**：知识贡献者承担全部时间成本，但收益由全体成员共享。AI Team 的解决方案不是道德呼吁，而是工程机制：贡献行为通过工作流自动化嵌入（每次成功的工作流执行自动记录 evidence），共识机制（2人+2项目验证）使单次贡献的时间成本可接受，同时区块链式的不可篡改日志使贡献记录永久可见可溯，解决了"我贡献了但没人看见"的问题。这个机制与 [[entities/tmall-ai-coding-practice-team-knowledge-base|Tmall AI Coding Practice Team Knowledge Base]] 中描述的团队知识库实践形成了跨公司的对照验证。
## 知识工作流嵌入：从"事后记录"到"实时沉淀"
AI Team 体系的关键工程创新在于**将知识沉淀嵌入日常工作流**，而非作为独立的"文档撰写任务"。传统知识管理的失败原因是将知识贡献视为额外工作，而 AI Team 的做法是让知识贡献成为工作流执行的自然副产品。

### 触发式知识提取机制
每次工作流成功执行时，系统自动提取以下知识：
- **decision 条目**：工作流做出的关键选择及其理由（从执行日志中反推）
- **pitfall 条目**：遇到的错误类型、根因、解决方案（从异常处理中提取）
- **process 条目**：实际执行路径与预期路径的差异（从路由日志中提取）

这种触发式提取的价值在于：**知识贡献零额外成本**。贡献者不需要停下来写文档，系统会在工作流成功结束时自动记录关键信息。

### 知识注入的实时性
当新成员加入或跨项目协作时，知识库可以**实时注入当前任务相关的 proven 条目到 Agent 的上下文窗口**。这解决了传统知识库"需要时找不到，找到时已过时"的问题——proven 条目本身就是经过跨项目验证的知识，其时效性由自动衰减机制保证。

### 与 [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]] 的互补性
AI Team 的知识沉淀体系与 Garry Tan 倡导的 Thin Harness Fat Skills 理念形成互补：Fat Skills 强调**个人层面的知识积累**（Markdown 技能文件随个人使用自动进化），而 AI Team 强调**团队层面的知识验证**（跨个人、跨项目的知识共识机制）。两者的结合点在于：Fat Skills 是个人知识的载体，而 AI Team 的知识库是将个人知识转化为团队知识的沉淀池。具体而言：

| 维度 | Thin Harness Fat Skills | AI Team 知识沉淀体系 |
|------|-------------------------|---------------------|
| 沉淀主体 | 个人，随个人使用进化 | 团队，通过共识机制验证 |
| 知识粒度 | 技能文件（完整工作流） | 条目级（decision/model/pitfall） |
| 验证机制 | 个人反复使用验证 | 跨项目、跨成员共识 |
| 衰减机制 | 无（个人决定保留/删除） | 自动衰减（基于引用频率） |
| 共享方式 | 手动分享技能文件 | 自动进入团队知识库 |

两者结合的最佳实践是：**个人用 Fat Skills 积累第一手经验，AI Team 知识库负责将这些经验提炼为可验证的团队知识**。
## 子页面
## 核心原则
> **Skill、Agent、工具链会随模型迭代更新，但领域知识是永恒的。**
1. **工作流可替换，知识可累积**：工作流变化快，领域知识不管怎么变都有价值
2. **没有知识沉淀的工作流是一次性的**：投入工程成本搭建工具链，却没有让工具链越来越聪明
3. **知识是团队的复利资产**：成百上千条 proven 知识条目时，新成员、新项目都能站在前人肩上
4. **Big Model vs Big Harness 的务实立场**：知识工程投入是确定性回报；模型能力提升不能替代领域知识
## 相关概念
- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — 理论基础，知识管理是 Harness 的核心支柱之一
- [[entities/agent-memory-architecture|Agent Memory 架构本质]] — 记忆系统设计，知识沉淀的技术基础
- [[entities/personavlm-personalized-memory|PersonaVLM 长期个性化记忆]] — 五类记忆分层与知识管理的相似设计
- [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]] — YC 视角：知识作为技能文件积累
- [[concepts/harness-engineering-paradigm-shift|Harness Engineering 三次范式跃迁与四根支柱]]
- [[entities/context-engineering-three-memory-paradigms|Context Engineering 三种记忆范式对比]] — 记忆类型分层，对应知识成熟度的信息论解释
- [[entities/tmall-ai-coding-practice-team-knowledge-base|天猫 AI 编码实践团队知识库]] — 电商场景团队知识管理工程实践
- [[entities/harness-engineering|Harness Engineering 总览]] — 知识管理在 Harness 工程中的定位
## 参考文献
- Karpathy LLM Wiki — 知识复合增长：Ingest + Query + Lint

## 新增关联实体
- [[entities/headroom-context-compression-cache-stabilization]]
- [[entities/is-grep-all-you-need-pwc-retrieval-harness-coupling]]

## 关联实体

**上游依赖**:
- [[entities/context-engineering-three-memory-paradigms]] — 提供基础理论/方法
- [[entities/harness-engineering]] — 提供基础理论/方法
- [[entities/thin-harness-fat-skills]] — 提供基础理论/方法

**下游应用**:
- [[entities/thin-harness-fat-skills]] — 具体应用场景
- [[entities/tmall-ai-coding-practice-team-knowledge-base]] — 具体应用场景
- [[entities/thin-harness-fat-skills]] — 具体应用场景

**平行协作**:
- [[entities/thin-harness-fat-skills]] — 替代/补充方案
- [[entities/context-engineering-three-memory-paradigms]] — 替代/补充方案
- [[entities/tmall-ai-coding-practice-team-knowledge-base]] — 替代/补充方案

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
