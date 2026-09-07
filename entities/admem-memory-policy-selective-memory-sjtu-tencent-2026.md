---
title: "别让Agent什么都记 上交×腾讯提出 AdaMem"
created: 2026-07-31
updated: 2026-09-07
type: entity
tags: ['llm-agent', 'memory', 'memory-policy', 'selective-memory', 'sjtu', 'tencent', 'rag']
sources: [raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md|原文存档]]

# 别让Agent什么都记 上交×腾讯提出 AdaMem

上海交大和腾讯合作的论文 AdaMem，解决一个被忽略的问题：LLM Agent 的记忆系统，是不是存得越多越好？答案是：不是。存太多反而会让 Agent 变笨。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]

## 摘要

AdaMem（Adaptive Memory）挑战了 LLM Agent 记忆系统的默认假设——"记住一切"。主流记忆方案（如 Mem0）奉行"来者不拒"策略，把所有事实性信息不分轻重地写入记忆库；短期可行，但长期对话（数周乃至数月）后，无关闲聊、随口日程、重复问候堆积成噪声，检索时挤占有限的上下文窗口，导致 QA 准确率持续下滑。论文把这一现象命名为 **Memory Bloat（记忆膨胀）**。AdaMem 的核心主张是：不要记住一切，要学会"记住什么"——用一套可自动学习、可回滚的 **Memory Policy** 决定每个角色/场景该记什么，并把"选择性记忆"从检索端延伸到了写入端。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]

## 核心要点

- **Memory Bloat 是真实存在的退化**：记忆库"来者不拒"写入，噪声在长期会话中持续堆积，每次检索都挤占上下文窗口，QA 准确率随对话时长逐步下降。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]
- **答案不是更多记忆，而是更聪明的记忆**：AdaMem 认为记忆系统的瓶颈不在容量而在策略——决定"记什么、不记什么"比"能存多少"更重要。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]
- **Memory Policy 是一套可编程的记忆规则**：显式定义"对每个角色/场景应该记什么"，把模糊的"记下来"变成可审计、可迭代的决策逻辑。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]
- **策略从反馈中自动演化**：每周跑一轮 QA 测试 → 定位答错项 → 反思是"漏记"还是"记多" → 给策略打补丁 → 补丁若让历史表现变差则自动回滚。全程无需重训练模型。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]
- **即插即用的工业价值**：直接嵌入现有 RAG 记忆栈即可，不要求架构重构。
- **实测收益"少即是多"**：在自建 AdaMem-Bench 上测 10 周，QA 准确率最高提升 +9.0%（85.2% vs 76.2%），同时记忆量减少 9%——更少的记忆带来更好的效果。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]

## 深度分析

### 为什么"存得越多"反而让 Agent 变笨

长期记忆的退化与原始 RAG 的噪声问题同构：无关信息不仅消耗存储，更重要的是在检索阶段"污染"相关性排序，把真正有价值的片段挤出上下文窗口。AdaMem 揭示的是写入端被长期忽视的事实——如果写入时不做策略性筛选，检索端再好的排序器也只能在噪声里挑相对不差的候选。这解释了为何单纯扩大模型上下文或升级检索器无法根治记忆膨胀：问题源头在记忆入口处的"来者不拒"。这与 [[entities/agent-memory-main-contradiction-context-scheduling|上下文调度的记忆主矛盾]] 一脉相承——容量稀缺的真正博弈点是信息取舍，而非存储空间本身。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]

### Memory Policy：从"存下来"到"有规则地存"

AdaMem 把记忆写入从隐式行为改写为显式策略。Memory Policy 本质是一组（角色/场景 → 该记什么）的映射规则，让"记什么"成为可被测试、可被审计、可被打补丁的工程对象。这套机制的闭环是：QA 测试暴露 err → 反思区分"漏记（under-memory）"与"记多（over-memory）"两类根因 → 定向补丁 → 回归验证；一旦补丁使历史指标退步就自动回滚，形成带安全网的自适应循环。这种"策略即代码"的思路，把记忆治理从一次性设计变成持续迭代的运行时行为，也与 [[entities/agent-memory-architecture-essence|Agent 记忆架构的本质]] 强调的"写入即决策"相呼应。 ^[raw/articles/admem-memory-policy-selective-memory-sjtu-tencent-2026.md]

### SOTA-aware 调度与选择性记忆架构

AdaMem 的价值还在于把"选择性"做成架构层面的一等公民，而非事后裁剪：在写入环节就用策略裁决保留优先级，配合检索侧的 RAG 裁剪共同压紧上下文占用。它服务的是"知彼"式认知——Agent 只保留对当前任务真正有用的记忆，把与目标无关的历史细节主动遗忘。这与 [[entities/memreranker-reasoning-aware-agent-memory-reranking-2026|推理感知的记忆重排序]] 一起，构成从"存得多"到"存得准"的两端收敛：一个在写入端设闸，一个在读取端重排，最终目标都是让 Agent 的注意力只落在高价值记忆上。

### SJTU × Tencent 研究定位与"When/How to Forget"

作为学术界与工业界联合的工作，AdaMem 的工程取向非常鲜明：不重训练、可嵌入现有栈、直接压缩推理成本。其"When/How to Forget"由 Memory Policy 显式承载——把"什么时候忘、怎么忘"从玄学变成可配置规则。论文同时坦承局限：验证基于合成数据集；在隐式反馈下策略推断准确率仅约 60%；评估规模偏小。这提示我们，选择性记忆的价值方向已被证明，但要跨入真实生产环境，仍需更强的策略推断信号与更大规模的实证支撑。它给出的是一份"记忆该做减法"的路线图，而非终局答案。

## 实践启示

1. **给记忆入口装上策略闸门**：不要默认所有事实都写入，为每个角色/场景显式定义"该记什么"的规则，从源头抑制 Memory Bloat。
2. **把"记错"当可调试信号**：QA 测试要能区分"漏记"与"记多"，两者根因不同，修复手段也必须分开设计，否则补丁会互相抵消。
3. **建立自动回滚的安全网**：任何记忆策略补丁都应伴随历史回归验证，一旦指标退步立即回滚，避免"修编译坏"式的不稳定迭代。
4. **写入端与读取端协同裁减**：选择性记忆（写入设闸）与 RAG 裁剪/推理感知重排（读取定序）是同一目标的左右手，应共同压紧上下文占用。
5. **从"存得多好"到"存得准好"的指标迁移**：评测不仅要看准确率，更要看记忆量、推理成本等实际代价，用"少而准"作为系统健康度的评判标准。
6. **警惕"少即是多"在不同数据下的边界**：AdaMem 在合成数据与较小规模上验证较充分，落地前需在真实长会话与真实用户反馈上复测策略推断的可靠性。

## 相关实体

- [[entities/agent-memory-main-contradiction-context-scheduling|上下文调度的记忆主矛盾]]
- [[entities/agent-memory-architecture-essence|Agent 记忆架构的本质]]
- [[entities/memreranker-reasoning-aware-agent-memory-reranking-2026|推理感知的记忆重排序]]
- [[entities/mragent-memory-reconstructed-not-retrieved-nus-icml2026|MR-Agent：记忆应重建而非检索]]
- [[entities/what-makes-good-agent-memory-system-yuanrunzi-2026|什么是好的 Agent 记忆系统]]