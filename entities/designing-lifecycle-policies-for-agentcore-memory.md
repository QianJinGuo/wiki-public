---
title: "Agent 记忆生命周期管理：TTL / 相关度衰减 / 合并（AgentCore Memory 架构）"
created: 2026-09-06
updated: 2026-11-26
type: entity
tags: [agent-memory, memory-lifecycle, episodic-memory, semantic-memory, procedural-memory, agentcore, memory-management]
sources: [raw/articles/designing-lifecycle-policies-for-agentcore-memory]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent 记忆生命周期管理：TTL / 相关度衰减 / 合并（AgentCore Memory 架构）

长时间运行的 agent 会从每次对话持续生成记忆，若不主动管理，会积累过期上下文，降低响应质量并带来合规风险（实测案例：客服 agent 引用四个月前已解决的账单纠纷、重复过时的部署建议）。本架构提出 memory lifecycle management：系统性地对 agent 记忆进行评分（scoring）、合并（consolidating）、剪枝（pruning）。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

## 记忆分类学（Memory Taxonomy）

三类记忆具有不同的保留需求：^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

- **情景记忆（Episodic）**：记录发生了什么——带时间戳、会话绑定、高容量；短期连续性。最优先过期。
- **语义记忆（Semantic）**：从交互中提炼的事实与偏好（如「用户偏好 us-east-1 部署」），持久、高价值、紧凑；保留时间更长，是合并的优先候选。
- **程序记忆（Procedural）**：习得的工作流与工具使用模式（如「问成本先查 Cost Explorer API」），容量低但价值最高，保留最长、剪枝门槛最高。

## 三个生命周期策略

每个策略针对无界记忆的一个失败模式：^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

**1. TTL 过期（Policy 1）**：删除超过配置 TTL 的记忆，提供硬性累积上限。建议按类型差异化：summary 30–60 天、semantic 6–12 个月、procedural 不设 TTL。TTL 先于评分/合并运行，避免浪费计算。实现上用 `ListMemoryRecords` 的 `BEFORE` 过滤 + `createdAt` 时间戳。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

**2. 相关度衰减评分（Policy 2）**：记忆老化速率不同——昨天访问的记忆比数周未动的更相关。三项加权公式组合创建新近度、最近访问新近度、访问频率。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

**3. 合并（Policy 3）**：将多条情景观察合并为一条权威事实（semantic consolidation），减少冗余、提升检索质量。

## 部署架构

运行 nightly workflow：AgentCore memory + AWS Step Functions + Amazon Bedrock，提供 CDK stack 实例，代码开源于 aws-samples。记忆作为托管资源管理，所有阈值可配置，适配不同容量 agent（低容量可仅用 TTL + GDPR 合规）。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md]

## 深度分析

### 把「何时遗忘」变成一个可调参数：pruneDays 与三权重公式
原文不暴露裸衰减常数，而是给出一个直观参数 `pruneDays`——一条无访问记忆的分数跌破相关度阈值的大致天数，内部换算为 decay_rate = -ln(threshold) / pruneDays；默认 pruneDays=45、threshold=0.3 得 decay_rate≈0.02676。评分公式三项加权：创建新近度 W_RECENCY=0.4、最近访问新近度 W_ACCESS=0.35、访问频率 W_FREQUENCY=0.25（MAX_ACCESS_BASELINE=50 时频率项饱和为 1.0），权重和为 1.0 时分数落在 [0.0, 1.0]。运维按 workload 调权：反复引用同一 runbook 的支持 bot 调高 W_FREQUENCY，追求新鲜度的实时交易助手调高 W_RECENCY。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:89-102]

### 不同 agent 原型有不同的遗忘节奏
指数衰减意味着分数在前几周骤降、随后趋平——旧但近期高频访问的记忆仍可得高分。pruneDays 按原型给推荐起点：实时支持 bot 7 天（工单小时级解决）、销售/onboarding 21 天（陈旧线索污染上下文）、通用助手 45 天、IT helpdesk/ops 90 天（事故模式季节性重复）、法律合规顾问 180 天（先例数月仍相关）。这与按记忆类型差异化 TTL（summary 30–60 天、semantic 6–12 个月、procedural 不设 TTL）是同一立场：遗忘节奏必须匹配业务周期，而非一刀切。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:104-112]^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:43-47]

### 访问信号本身需要一条基础设施管道
AgentCore 的 MemoryRecordSummary 不含 lastAccessedAt 字段，真实的访问数据要自建：CDK 配置带高级事件选择器的 CloudTrail trail 捕获 GetMemoryRecord 数据事件，Memory Scorer 每轮评分先聚合过去 25 小时日志，再把新鲜计数与 S3 持久化的历史访问台账合并——频率项因此获得全生命周期信号而非每日快照。这把记忆评分从纯算法问题变成数据工程问题：没有访问台账，W_FREQUENCY 项无从落地；也解释了架构为何必须是 nightly workflow + 五个 Lambda 的编排，而非一个定时删除脚本。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:149-151]^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:177-191]

### 合并是有损操作：置信度、护栏与失败路径
Consolidation 把低分记忆批量（默认 10 条）发给 Bedrock 合成一条语义条目，提示词要求保留关键事实、去除冗余，并输出 confidence（0.0–1.0）供标记低质量合并以人工复核。原文直言「合并天然有损」：LLM 概括五条记忆可能丢失细节，高风险域应把原文归档冷存储而非直接删除；Bedrock 失败时系统保留原记忆不变，失败删除记录日志供人工处理。Guardrails 与 grounding check 被定位为「生产必需品，不是可选添加」——这是 [[concepts/memory-consolidation-decay|记忆合并与衰减]] 理念在工程侧的完整展开。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:153-173]^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:188-191]

### 用回归测试把「遗忘的质量」闭环
剪枝与合并只有在 agent 回答不退化的前提下才有意义。测试用例是「问题 + 判据 + 最低质量分」三元组，走 before/after 四步：基线查询并用 AgentCore Evaluations 打分（LLM-as-judge，输出 0.0–1.0 归一化分）→ 跑生命周期 → 复查同题 → post 分数 ≥ min_quality_score 才通过，同时报告质量 delta。套件全自动化、可进 CI/CD——这把遗忘从运维动作升格为可验证的工程实践，与「记忆作为托管资源」的定位互为支撑。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:350-403] 这一「策略须配质量验收」的闭环与 [[concepts/agent-memory-system-design|Agent 记忆系统设计]] 的系统工程视角互补；遗忘节奏匹配业务上下文半衰期的取向，也与 [[concepts/working-set-vs-long-term-memory|工作集与长期记忆]] 的分层保留及 [[entities/admem-memory-policy-selective-memory-sjtu-tencent-2026|AdMem 选择性记忆策略]] 的按信号保留思路一致。

## 实践启示

1. **遗忘节奏按业务周期定价，而非统一 90 天**：先用原型表选起点（支持 bot 7 天、法律顾问 180 天），再按实际工单/成单周期微调。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:104-112]
2. **TTL 先行省算力**：过期排在评分/合并之前，已被时间判死的记忆不值得再花评分与 LLM 调用；低容量 agent 只需 TTL + GDPR 即可起步。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:43-47]
3. **成本主驱动是合并阶段的 Bedrock 调用**：1,000 条记忆 20% 低分 ≈ 每晚 20 次调用（约 $0.01–0.02），10 万条可达 $50–100/月；从较高的 relevanceThreshold 起步控制合并量。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:344-346]
4. **合规要落在记忆层**：GDPR 被遗忘权由独立 handler 按 user namespace 列举并逐条删除，部分失败返回 failed IDs 供重试；每次记忆变异产生结构化 JSON 日志，CloudTrail 提供不可变审计链。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:405-455]
5. **高风险内容别硬删**：合并置信度低时归档冷存储而非删除，失败删除进入人工复核队列——宁多一步人工，不静默丢事实。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:171]
6. **遗忘要有质量验收**：把回归套件（问题-判据-最低分 + before/after delta）纳入上线流程，剪枝参数的每次变更都过一遍套件。^[raw/articles/designing-lifecycle-policies-for-agentcore-memory.md:350-375]


## 关联

- 概念：[[concepts/agent-memory-lifecycle-philosophies|Agent 记忆生命周期哲学]]、[[concepts/episodic-vs-semantic-memory-agent|情景 vs 语义记忆]]、[[concepts/memory-consolidation-decay|记忆合并与衰减]]
- 实体：[[entities/agent-memory-architecture-essence|Agent 记忆架构本质]]、[[entities/structured-memory-filtering-metadata-agentcore-memory|AgentCore Memory 结构化过滤]]、[[entities/when-ai-agents-learn-to-forget-amazon-bedrock-agentcore-memory-philosophy|Agent 何时学会忘记]]

→ [[raw/articles/designing-lifecycle-policies-for-agentcore-memory|原文存档]]