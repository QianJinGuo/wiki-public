---
title: Fable 5 使用硬核指南：搞定未知盲区
created: 2026-07-04
updated: 2026-09-18
type: entity
tags: [fable, claude, agent, coding, methodology, unknowns, map-territory]
sources: [raw/articles/fable-5-field-guide-unknowns-ai-coding]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> Anthropic Claude Code 团队成员 Thariq 撰写的 Fable 5 使用指南，核心观点是「地图不等于疆域」，提出四类未知数框架和一套在实施前、中、后不断发现未知数的迭代工作流。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

## 地图 vs 疆域框架

**地图（Map）**：你交给 Claude 的东西——prompt、技能设定和上下文，代表你对要做的事情的描述。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

**疆域（Territory）**：工作真正发生的地方——代码库、现实世界及其各种限制。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

地图和疆域之间的落差，就是 **未知数（Unknowns）**。当 Claude 在工作中碰到未知数，它就得根据对你意图的猜测做出决定。要做的工作越多，Claude 可能碰到的未知数也就越多。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

Thariq 认为，Fable 是第一个让他明显感觉到工作质量的瓶颈在于自己能不能把未知数讲清楚的模型。提前把计划做足并不总是够用——有些未知数只有深入到实现阶段才会暴露出来。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

## 四类未知数

面对一个问题，Thariq 习惯把自己的认知拆成四类：^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

| 类别 | 含义 | 应对方式 |
|------|------|----------|
| **已知的已知（Known Knowns）** | 写进 prompt 的内容，明确告诉 Agent 想要什么 | 直接描述 |
| **已知的未知（Known Unknowns）** | 自己没想清楚，但清楚地知道自己没想清楚的部分 | 提问式访谈、头脑风暴 |
| **未知的已知（Unknown Knowns）** | 见到才会认出来、但从来不会主动写下来的标准和要求 | 原型验证、参考物 |
| **未知的未知（Unknown Unknowns）** | 完全没有考虑过的东西，包括不知道自己不知道的知识 | 盲区扫描（Blind Spot Scan） |

最擅长 agentic coding 的人未知数往往最少——他们对自己想要什么了如指掌，和代码库、模型行为都高度同步。但他们同样也在假设未知数的存在。减少并规划自己的未知数，正是 agentic coding 的核心技能。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

## 与 Fable 5 协作的迭代工作流

Thariq 把和 Fable 一起工作的过程，描述成一个在实施之前、之中、之后不断发现自己未知数的迭代过程：^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

### 实施之前

1. **盲区扫描（Blind Spot Scan）** — 直接让 Claude 帮你找出未知的未知，告诉它你是谁、了解多少。适合在陌生领域开始新任务时使用。
2. **头脑风暴与原型（Brainstorming & Prototyping）** — 在未知的已知（看到才知道好不好）特别多的领域，让 Claude 一起做原型，尽早暴露决策点。
3. **提问式访谈（Q&A Interview）** — 让 Claude 反过来采访你，针对模糊或不确定的地方提问，优先问那些会改变整体架构的问题。
4. **参考物（References）** — 给 Claude 一个参考物（最好是源代码，不限语言），让 Fable 直接从实物中理解意图。
5. **写实施计划（Implementation Plan）** — 让 Claude 先做计划，把最可能变化的部分（数据模型、类型接口、用户体验流程）放在最前面供审阅。

### 实施过程中

6. **实施笔记（Implementation Notes）** — 让 Claude 维护一份 implementation-notes.md，记录偏离原计划的决定（Deviations），采取保守方案继续推进。

### 实施完成之后

7. **说明文档与汇报材料（Docs & Briefs）** — 将原型、规格实施笔记打包，方便评审者快速理解，展示未知数已被考虑。
8. **测验（Quiz）** — 让 Claude 出一份关于改动的测验，只有完全答对才合并代码，确保自己真正理解发生了什么。

→ [[raw/articles/fable-5-field-guide-unknowns-ai-coding|原文存档]]

---
## 深度分析

### 瓶颈从「模型会不会」搬到了「你说没说过」

Fable 级模型带来的最关键变化，是失败模式的性质变了：过去一次任务做砸，通常可以归因于模型能力不够；现在同样的失败，更常见的归因是你没把某个关键约束讲出来。能力上限抬升之后，剩下的大部分误差都来自地图与疆域之间的落差，而不是模型的知识或推理缺口。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

反直觉的副作用是：模型越强，未知数的绝对数量反而可能更多。因为能承接的工作半径变大，而未知数随决策点数量增长——每一处需要拍板的地方，都是一次让模型去猜的机会。所以「换个更强的模型」不会消解这个问题，只会把它暴露得更充分。事前把计划做足也不构成解药：有一类未知数只有在实现阶段被代码撞出来才会现形，甚至现形之后会反过来告诉你该换一套完全不同的解法。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

### 地图 vs 疆域，其实就是上下文工程的另一种命名

把「地图」翻译成工程语言，它就是交给模型的那包上下文：prompt、技能设定、会话里传进来的 artifact。它的本质是有损压缩——无论塞多少材料，都只是你对意图的投影，而非代码库本身。

这给 [[concepts/context-engineering|上下文工程]] 补上了一个常被忽略的判据。通常讨论上下文工程，问的是「窗口里该放什么」；地图-疆域框架问的是另一个问题——窗口里呈现的部分占真实疆域的多少，以及没被呈现的部分会不会被悄悄补全。后者才是决定长周期任务成败的变量，也是同一套模型在不同人手里差距巨大的那部分原因。这也意味着「多喂上下文」的边际收益会迅速衰减：它能提高地图的保真度，却关不上那道缝，反而会以一份看起来完备的地图制造虚假的确定感。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

### 四类未知数里，Agent 真正能替你消掉的只有一类半

按「Agent 能在多大程度上代你发现」重新排一遍这四类，结论相当不对称。已知的已知不需要发现、只需要写清楚，Agent 在这一格帮不上忙。已知的未知是效率最高的一格：它能用访谈的形式，把你问不出来、但确实存在的决策点逐个枚举出来，相当于替你把问题空间做了一次穷举。

未知的已知无法靠对话提取——它们不在你的语言里，只在你看到东西的那一刻被激活。这时 Agent 的角色不是提问者而是生成器：快速产出多个方向的原型与参考物，把你脑子里那条隐性的验收标准逼出来。至于未知的未知，盲区扫描确实能列出候选清单，但它成立的前提是你先交代自己是谁、懂多少；不交代，模型只会按行业通行做法作答，而那恰恰不是你的疆域。所以严格说，Agent 在这里只完成了一半工作——它负责把盲区摆上台面，认出盲区仍然发生在你脑子里。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

### 沉默补全：真正的失败模式不是答错，而是看不出有洞

这套框架里最值得警惕的现象，不是模型出错，而是模型在你没意识到的地方替你做了决定，且输出看起来完好无损。指令过细时它会严格执行，哪怕换个方向明显更合适；指令过糊时它会退回行业最佳实践去猜，而最佳实践未必适配你的具体任务。两头的共同点是落差不会自我宣告，只会以「这跟我想的不一样」的形式出现在评审或生产环境里，发现得越晚返工代价越高。指南给出的应对，本质上是把隐性猜测变成可审计的痕迹：实施阶段让 Agent 维护 implementation-notes，凡遇到必须偏离计划的情况就选保守方案、把偏移记进 Deviations 一节；收尾时让它围绕改动出题考你，答对才允许合并。前者让「猜了什么」可见，后者验证「你到底懂没懂」。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

### 从 Fable 5 迁移到任意 Coding Agent

这套框架的可迁移性来自它的抽象层级：地图与疆域刻画的是人机之间的信息接口，而不是某家厂商的模型特性。任何具备快速检索与快速迭代能力的 agent，都会把这道落差放大成质量问题，因此换模型、换 harness 都不改变结论。^[raw/articles/fable-5-field-guide-unknowns-ai-coding.md]

真正随模型变化的是容错区间：模型越强，欠规范状态下仍能跑出像样结果的空间越大；但能承接的工作半径也在扩大，暴露出的未知数更多。这也解释了为什么顶尖的 agentic coding 实践者未知数最少——他们同时与代码库和模型行为保持同步，而这种同步是长期累积的结果，不是天赋，可以靠这套工作流批量生产。参见 [[entities/thariq-fable-5-usage-mindset-map-territory-unknown-unknowns|同源对照]] 与 [[concepts/ai-coding-paradigm-evolution|AI 编程范式演进]]。

## 实践启示

1. **开工前先做盲区扫描，并附上自我坐标。** 先告诉 Agent 你是谁、对这个代码库了解多少，再让它列出未知的未知；缺少自我坐标的扫描只会返回通用最佳实践。
2. **按未知数类型选工具。** 已知的未知交给提问式访谈，未知的已知交给原型与参考物，未知的未知交给盲区扫描。用错工具的典型症状是聊了很多轮，决策点一个都没减少。
3. **讲不清需求时给源代码当参考物。** 指向一个已实现你想要语义的文件或依赖目录，哪怕它是另一种语言，也比三段文字更省 token、更少歧义。
4. **把最可能变化的部分顶到计划最前面。** 要求 Agent 把数据模型、类型接口、面向用户的流程排在前面，机械式重构放最后；你只精审前半段。
5. **让每次偏移留下书面痕迹。** 实施中要求维护 implementation-notes，凡必须偏离原计划处一律选保守方案并记入 Deviations 再继续推进。
6. **合并前用测验验证你自己的理解，而不是只看 diff。** 让 Agent 基于改动出题并要求自己全部答对，答不上来就说明还有未闭合的未知数。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

