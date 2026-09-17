---
title: "T-Mem：腾讯 PCG 的「联想回忆」式长期记忆系统（EMNLP 2026）"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [agent-memory, long-term-memory, retrieval, memory-graph, emnlp, tencent, agent, episodic-future-thinking]
confidence: 0.75
provenance_state: extracted
sources: [raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026]
---

# T-Mem：腾讯 PCG 的「联想回忆」式长期记忆系统（EMNLP 2026）

腾讯 PCG 团队（郭伟东、王达凯、汪子轩、刘辉、徐羽）提出 **T-Mem**，把长期记忆的检索假设整个翻过来：与其在读取时寻找相似度，不如在**写入那一刻**就把「这条记忆将来会在什么情境下被用到」预先存成桥接线索。论文 *T-Mem: Memory That Anticipates, Not Archives* 已被 EMNLP 2026 主会议接收，代码与数据开源（`github.com/Sherlockwz/T-Mem`），工作已在 QQ AI 伙伴项目中上线。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

## 问题：所有主流记忆系统共享同一个相似度闸门

长期陪伴类对话场景里，用户不会年复一年用同样的话重复同一件事，记忆系统必须在措辞、话题甚至语境都变的情况下，依然把当年那段记忆找回来。但今天几乎所有长期记忆方案——无论底层用向量库、图数据库还是其他存储——在检索环节都共享同一个假设：只要查询与记忆足够「像」，记忆就能被召回。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

作者引用认知科学的「情景未来思维」（episodic future thinking）：人可以通过预演未来场景，把本应在当前对话中想起的记忆关联起来，而这并不需要对方在相似场景下说出相似的话。可教给机器的却只有「找相似内容」这一种检索方式——问题不在某个环节，而在设计根基本身。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

一个具体例子：用户几个月前随口说过「小张海鲜过敏，上周都吃进医院了」。几周后团建聚餐前问「下周团建我们去哪儿呢」——两句话之间不存在任何词汇或语义相似度，相似度检索必然失手，而联想回忆应当把那条过敏信息带出来。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

## 建模：2×2 象限与 QI–QIV 触发族

T-Mem 用两条正交轴切分记忆问题：**方向轴**（描述性回忆 vs 联想性回忆）与**粒度轴**（单条事实 vs 完整场景）。四个象限中各放一个触发族，编号 QI–QIV 与象限一一对应。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

- **QI 实体触发**（描述向／事实粒度）：给单条事实贴泛化标签，强化相似检索；
- **QIV 场景触发**（描述向／场景粒度）：把整段场景写成多维档案；
- **QII 桥接触发**（联想向／事实粒度）：在事实粒度预判「这条事实将来会在什么情境里被用到」，把「小张海鲜过敏」连到「团建餐厅选择」的触发线索；
- **QIII 前瞻触发**（联想向／场景粒度）：顺着对话往前多想几步，把未来可能的触发情境先记下来。

文章对这套思想的概括是：相似度回答「你像什么」，触发器回答「你会被怎样想起」。之所以要填满整个坐标系，是因为主流记忆系统几乎全挤在「描述」这半边——存储做得再精巧、更新做得再及时，相似度闸门不变，能力就被锁死在半边。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

## 一写一读两条链路

**写入链路**解决「记住什么、怎么记」：对话进来后先切成一个个有完整情节的独立场景，再把多个场景归入相同主题，接着从场景里抽出原子化事实条目并把场景与事实连接成「场景—事实图」，最后为每条事实和每个场景配上 Trigger。关键工程约束是这一步不在问答时刻占用响应时间——对话每推进一段，后台就把记忆库更新一段；提前铺路的开销通过线索去重、适时合并与增量更新来收敛。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

**读取链路**解决「新问题来了，怎么把对的记忆翻出来」：自上而下逐层检索——先判断问题落在哪个主题，再在该主题下挑出相关场景，最后取出具体事实交给模型作答。与传统方案的关键差别在于**联想线索在检索最开始就参与**：哪怕一个问题和某条记忆毫无表面相似，只要它在语义上「踩中」了这条记忆的线索，记忆也会被直接带进候选，而不会被主题过滤挡在门外。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

## 性能：两个基准双双刷新 SOTA，跨域落差只有 5.45 个百分点

- **LoCoMo**：整体准确率 **80.26%**，刷新 SOTA。LoCoMo 是主流的长对话事实记忆基准，但题目本质上是相似召回——查询与记忆共享词面、实体即可命中。T-Mem 在这里刷 SOTA 说明它的基础记忆能力不输任何主流系统。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]
- **LoCoMo-Plus**：达到 **74.81%**，同样刷新 SOTA。该基准在 LoCoMo 基础上刻意加入一个「认知子集」，其中每道题的线索与答案之间都抹掉了词汇与语义相似度，只靠叙事或因果线索相连——任何靠相似度检索的系统在这里都会直接失灵。据作者所称，这是目前唯一能单独检验「联想记忆」的公开基准。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

最能说明问题的是**跨域落差**：主流系统从 LoCoMo 掉到 LoCoMo-Plus 平均要掉 28–50 个百分点，T-Mem 只掉 **5.45 个百分点**。作者由此论证联想记忆并非锦上添花的加分项，而是相似度路线从结构上缺失的那半边能力。^[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026.md]

## 要点

- 记忆系统的价值不在于忠实归档对话流，而在于**写入时就预见未来会通过怎样的线索触及它**——这条被文章当作设计注脚的判断，可以当作评价任何记忆架构的第一性问题；
- 「预判性写入」的代价必须被工程约束住（线索去重／合并／增量更新），否则每条记忆都铺路会让存储与写入开销失控；
- 联想召回与相似召回不是替代关系，而是需要同时覆盖的两个象限——只补相似度会继续在 LoCoMo-Plus 类子集上崩掉。

## 相关

- [[entities/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026]]
- [[entities/agent-memory-architecture]]
- [[entities/agent-memory-evaluation-landscape-taobao-survey]]
- [[entities/funes-agent-memory-layer-huggingface-2026]]
- [[entities/tencentdb-agent-memory-hierarchical]]
- [[concepts/agent-memory-substrate-three-layer]]
- [[concepts/context-engineering]]

→ [[raw/articles/t-mem-anticipatory-memory-trigger-tencent-pcg-emnlp-2026|原文存档]]
