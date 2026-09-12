---
title: 何时不要建记忆系统
created: 2026-09-10
updated: 2026-09-10
type: concept
tags: [agent-memory, negative-guidance, architecture, write-threshold, adversarial]
confidence: 0.7
provenance_state: inferred
---

# 何时不要建记忆系统（When Not To Build Memory）

> 2026-09 反方立场透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-adversarial-memory|反方记忆稿]]）的产出页，与 [[concepts/when-not-to-harness-engineering|何时不要 Harness Engineering]] 同构：记忆簇的默认叙事是"建"，本页写**默认别建**的条件与写入门槛。全部论据来自簇内自认证据，无外部输入。

## 三条"先别建"的触发条件

1. **任务是单会话可完成的**：记忆的价值全部在跨会话复利；单会话任务里记忆 Machinery 是纯成本（[[queries/rag-vs-agent-memory-vs-context-window|rag-vs-memory 决策页]]自认：上下文窗口保真度最高、延迟最低）。
2. **写入没有清算路径**：生命周期哲学页自认删除"从来不是一键"——删原文不删摘要、删摘要不删已提取偏好。若一条信息的衍生谱系无法清算（或无人愿意维护清算），它进入记忆就是**不可撤销的负债**（[[raw/articles/skilljack-persistent-skill-backdoor-tencent-mindchain-2026-09|SkillJack]]：删源记录后污染轨迹仍大面积存续）。
3. **没有消融基线**：全库尚无"同任务、有/无记忆"对照（[[queries/negative-results-registry|负结果登记簿]]观察区在案）。上记忆系统前先跑无记忆基线——赢不了基线的记忆是装饰。

## 写入门槛：何等信息不配进入记忆

与"要不要建系统"并列的逐条门槛——**写入前**对每条候选记忆过一遍：

- **无失效条件的不写**：说不出这条记忆何时/因何作废，就不写入（过期谓词与 [[queries/prediction-ledger|台账]]断言同理）。
- **无衍生预算的不写**：这条信息允许影响哪些类别的未来行为？说不清即不写（衍生预算是 [[concepts/channel-enumeration-criterion|通道枚举判据]]第三簇扩展的写入侧应用）。
- **未经认证通道可达的不写**：存储若存在未枚举的读通道（bleeding-LLAMA 型），敏感类信息一律不进记忆库。
- **可即时重推导的不写**：能从当下上下文或 RAG 便宜重推导的信息，记忆留存是负收益（存储+漂移风险 vs 一次检索）。
- **模型换代可能免费解决的不写**：记忆工件会随模型升级贬值（同 harness 工件衰减的同构推论，[[queries/prediction-ledger|台账]] #11 在账）——绑定"模型当前不会"的记忆优先级最低。

## 立场光谱中的位置

本页是记忆簇五档立场光谱的最反端（[[moc/agent-memory-architecture-decision-points|决策点 MOC]]已收纳光谱）：不是"记忆无用"，是**举证责任在记忆侧**——先过门槛，再谈架构。

## 参见

- [[concepts/agent-memory-lifecycle-philosophies|记忆生命周期哲学]] — 治理侧的正题
- [[concepts/memory-derivation-observability|记忆影响谱系可观测性]] — 写入后的配套（卡 #19）
- [[entities/agent-memory-engineering-tax-aws-china-2026|记忆系统工程税]] — 建了之后的持续成本
