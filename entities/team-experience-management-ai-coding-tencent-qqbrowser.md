---
title: "QQ浏览器团队经验管理系统：从AI Coding对话中提纯团队经验"
created: 2026-07-30
updated: 2026-10-03
type: entity
tags: [agent, team-experience, experience-management, three-layer-governance, qqbrowser, tencent, codebuddy, agent-memory, knowledge-base, mcp-retrieval]
sources:
  - raw/articles/ai-coding-team-experience-management-tencent-qqbrowser
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---
# QQ浏览器团队经验管理系统：从AI Coding对话中提纯团队经验

QQ 浏览器平台技术团队（腾讯 CSIG）基于 CodeBuddy Plugin 搭建了一套从 AI Coding 对话中自动提取、治理、召回团队经验的完整系统。核心创新在于三层治理管道（Review/Dedup/Merge）和"三镜头"认知障碍框架，将初版 90% 的抽取垃圾率降至治理后 95% 的有效率，最终入库率约 80%。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

## 问题定义：经验断层

团队使用 AI Coding 时面临四类问题：^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
1. **经验即抛** — session 纠偏产出的经验，session 结束即清零
2. **重复踩坑** — 隐性工程约束（如"stop hook 不能同步上报"）不在文档中，换人换 session 大概率照踩
3. **知识碎片化** — 老同事的 pattern 无动力书面化
4. **AI 永远是"新同事"** — 每次进项目从零开始，缺乏团队语境

## 核心设计：从 Agent 视角定义经验

判定标准：**被召回后，Agent 能否产生正向的行为变更**。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

### 三镜头框架

文章提出了**三镜头（Three Lenses）**框架，将"Agent 不容易直接发现"的信息分为三类：^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

| 镜头类型 | 认知障碍 | 例子 |
|---------|---------|------|
| **黑话镜头** | 语义不可发现 | "D 站"字面只是字母 D，实际指盗版站点 |
| **索引镜头** | 位置不可发现 | "直达页面功能在 xhome 模块，关键类是 FastCutXXX" |
| **逻辑镜头** | 行为不可发现 | stop hook 同步上报——AI 推理不出并发阻塞风险 |

## 系统架构

### 管道流程

对话上报 → 主题分组 → 经验抽取 → Review → Dedup → Merge → 入库 → 召回统计^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

### 三层治理

**1. 抽取（Extraction）**：归纳九类典型垃圾特征（事实性错误、通用常识、对话摘要、一次性 case 等）。以 Recall/Precision/Garbage Rate 三指标驱动 Prompt 迭代。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

**2. Review**：入库前精准拦截明确垃圾。策略：**默认保留、定向过滤**。特色创新是**源码探索（Code Explorer）**：对候选经验中提到的代码方法名在源码库中做事实验证，拦截不存在的方法调用。例如某经验声称 FastScrollBar 有 attachToQBListView() 方法，源码检索发现该类只有 attachToRecyclerView() 和 attach()，正确方法属于 FastScrollBarCompat.attachToQBRecyclerView()。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

**3. Dedup**：宁严勿宽，禁止桥接合并。当 What 和 When 均相同时 Recall 达 91.67%；整体 F1=71.79%。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

**4. Merge**：唯一目标先行，保护历史边界。六轮实验 157 样本，整体 F1=94.27%。Create 最稳定（F1=96.46%），Update 存在"积极合并"倾向（Recall 96.30% 但 Precision 78.79%）。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

## 生产数据

- **规模**：6 个仓库，50+ 研发人员 ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
- **数据量**：1,236 次独立对话 → 1,022 条候选经验 → 最终入库 **789 条**高置信经验 ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
- **管线已稳定运行 30+ 次** ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
- **召回表现**（125 次真实检索）：请求级召回率 68.8%，no_hit 仅 4.8%，平均注入 2.4 条经验，平均检索耗时 1,299 ms ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

### 基础设施

底层复用公司已有设施：IWIKI 经验空间落库 → Knot 知识库自动索引（向量+关键词） → Agent 通过 Knot MCP 协议实时检索。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

## 四条方法论

1. **先定义资产边界，再做自动化沉淀** ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
2. **分层治理，别用一个 prompt 解决所有问题** — Review/Dedup/Merge 各层目标不同、prompt 不同 ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
3. **默认策略按业务风险分别设计** — Review 默认保留、Dedup 宁严勿宽、Merge 保护历史边界 ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
4. **Prompt 优化从错例中抽象规则** — 固定流程：收集错例→识别误判类型→抽象规则→多轮实验→固定评测集 ^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

## 与业界方案对比

本文系统性地对比了 ClaudeCode AutoDream（单会话自动记忆整理）、Hermes Agent（Prompt 驱动 memory）、Mem0（长期记忆基础设施），指出共同局限：聚焦"个人记忆"而非"团队质量"。本系统追求"留下更少但更可信"——带适用场景、约束边界、来源证据的工程经验。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]

## 风险与未来方向

三层治理后垃圾率降至约 5%，但仍有三个未闭合的环：^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md]
- **生产阶段**：源码探索只覆盖部分场景，事实性错误仍难纯文本识别
- **召回阶段**：命中≠采纳，缺乏观测 Agent 行为变更的手段
- **生命周期**：无自动淘汰机制，项目演进时历史经验可能集体失效

## 深度分析

这个案例最值得深挖的一点，是它对「经验」这个概念做了工程化的重新定义。团队没有沿用「有价值就存下来」的模糊直觉，而是以 Agent 的行为作为最终裁判：被召回后能否产生正向的行为变更。这实际上把经验系统的验收标准从「信息论」转向了「行为论」——一条经验的价值不取决于它写得多好，而取决于它在下游 Agent 的决策链里是否真的改变了什么。作者由此推导出的合格标准（真实对话来源、有证据、项目特有、非通用常识、可复用、且「不容易直接发现」）形成了一个层层递进的过滤器，其中「不容易直接发现」这一条最具洞察力：如果 Agent 自己能推导出来，召回它没有任何边际价值。这从根本上解释了为什么「记住更多」的通用记忆方案（Mem0、AutoDream 等）在编程场景失效——它们假设记忆大体有价值，而编程对话的高噪声特性恰恰把这个假设打得粉碎。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:44-66]

初版 90% 垃圾率这个数字背后的结构性教训比数字本身更重要。文章把废经验拆成四层叠加的问题：噪声极高（调试过程混入）、上下文脱离（抽出的经验失真）、错误引导（错的经验比没经验更糟）、价值难定义。这四层揭示了自动提取的一个根本悖论：过程录像不等于经验，而「扩大候选集」的自动化手段会同步放大噪声——不加过滤的自动提取本质上是一个垃圾放大器。这说明 LLM 的总结能力在开放域虽然强大，但把「判断什么值得留」的边界判断完全外包给模型，等于把最难的部分交给了最不适合做它的环节。三层治理管道（Review/Dedup/Merge）正是在这个教训上长出来的：先用抽取端归纳九类垃圾特征控制候选质量，再用各层独立的默认策略兜住剩余风险。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:38-54]

Review 层的「源码探索」（Code Explorer）是整个系统里最有原创性的设计。它没有停留在用 LLM 校验 LLM 的同义反复陷阱里，而是让候选经验中提到的代码方法名回到源码库做事实验证——例如某经验声称 FastScrollBar 有 attachToQBListView() 方法，源码检索发现正确方法实际是 FastScrollBarCompat.attachToQBRecyclerView()，这类幻觉直接被拦截。这一步把「事实性错误」的审查从概率问题变成了可验证问题，是治理链路中唯一不依赖模型自证的一环。相应地，Dedup 和 Merge 的策略设计也体现了「按业务风险分别设计默认值」的思路：Dedup 宁严勿宽（What 和 When 均相同时 Recall 达 91.67%），Merge 唯一目标先行、保护历史边界（整体 F1=94.27%，但 Update 存在 Precision 仅 78.79% 的「积极合并」倾向）。用召回的代价换取经验库的可信度，用保守的合并策略保护历史经验的边界，这本质上是把数据库事务的一致性思维迁移到了经验治理。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:86-91]

文章结语把整个系统的本质点破了：经验系统不是技术系统，是团队认知能力的工程化管道。三镜头框架（黑话/索引/逻辑）实际上是对「隐性知识」做了一次类型学切分——语义不可发现、位置不可发现、行为不可发现——每一切片对应 Agent 推理路径上的一个具体盲区。这比笼统的「隐性知识难以传递」前进了一大步，因为它让隐性知识从「不可管理」变成了「可分类、可采集、可验证」的对象。那些写进 git blame 但从不进文档的隐性约束，第一次有了一条不依赖当事人主动书面化的自动通道。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:14-36]

## 实践启示

- **用 Agent 行为变更定义经验价值，而不是靠人主观判断。** 建任何经验/记忆系统之前，先回答「被召回后 Agent 能否产生正向行为变更」这一个问题，并以此为唯一标准筛选一切候选条目。参照 [[concepts/agent-memory-architecture]] 时尤其要警惕「记住更多」的默认倾向——追求「留下更少但更可信」才是团队场景的正确目标。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:68-74]

- **不要用一个大 prompt 解决整个治理问题，分层治理、各层独立设默认策略。** Review 默认保留、定向过滤；Dedup 宁严勿宽、禁止桥接合并；Merge 唯一目标先行、保护历史边界。每层目标不同，prompt 和评测集也应完全独立。这与 [[concepts/data-quality-framework]] 中的分层质量门思路一致。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:86-91]

- **能回到源码验证的，绝不让模型自证。** 对候选经验中提到的代码实体（方法名、类名、模块路径）做源码级事实校验，是拦截幻觉最可靠的手段；纯文本层面的 LLM 审查无法发现「方法不存在」这类事实性错误。这一原则可推广到任何知识入库管道：凡有可执行的真值来源（代码、schema、日志），优先接入真值校验。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:86-91]

- **用固定评测集加错例驱动的方式迭代 prompt。** 固定流程：收集错例、识别误判类型、抽象规则、多轮实验、固定评测集。Recall/Precision/Garbage Rate 三指标同时看，避免单指标优化带来的隐性回退。这也是 [[concepts/evaluation-harness-design]] 强调的「先有评测面再优化」原则在经验抽取场景的落地。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:88-104]

- **警惕经验库的三个生命周期陷阱：源头噪声、召回不可观测、无淘汰机制。** 治理后垃圾率仍有约 5%；召回命中不等于采纳，团队缺乏观测 Agent 行为变更的手段；且没有自动淘汰机制，项目演进时历史经验可能集体失效。新建类似系统的团队应从第一天就把「召回后行为归因」和「经验半衰期」纳入设计，参考 [[concepts/memory-consolidation-decay]]。^[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser.md:77-82]

---
**相关条目**
- [[entities/tencent-ai-coding-practices|腾讯 AI 编程实践]]
- [[entities/tencent-tab-harness-production-practice|腾讯 TAB Harness 生产实践]]
- [[entities/tencent-ai-team-knowledge-harness|腾讯团队知识 Harness]]
- → [[raw/articles/ai-coding-team-experience-management-tencent-qqbrowser|原文存档]]
