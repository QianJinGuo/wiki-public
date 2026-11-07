---
title: "StreamEval：首个实时交互视频生成基准（300 题 × 37 模型）"
created: 2026-10-23
updated: 2026-11-08
type: entity
tags: [benchmark, video-generation, multimodal, evaluation, streaming, interactive-video, arxiv]
sources: [raw/articles/streameval-interactive-streaming-video-benchmark-2026-10]
confidence: 0.62
provenance_state: extracted
---

# StreamEval：首个实时交互视频生成基准（300 题 × 37 模型）

StreamEval 是 2026 年 10 月 20 日挂上 arXiv（2610.28402）的**首个面向实时交互式流式视频生成的公开基准**，作者来自清华、英伟达、CMU、UC Berkeley、普林斯顿、新加坡国立大学与 Reactor。它把「实时视频好不好」这件原本只能靠感觉判断的事，变成一套含 300 道题、4 个维度 10 项指标、可复查可提交新模型的公开考卷；论文一次性评测 37 个模型、47 组组合。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

## 为什么实时视频难评

普通视频生成是「出来一段，好不好看一眼就知道」，但实时交互视频是一条**一直往前走的流**：第 10 秒要求换衣服、第 40 秒换回来、第 90 秒又反悔，每插一次话模型都必须同时答对三件事——现在要改什么、哪些东西不能动、之前改过的还算不算数。这三点无法用「截一帧好看的图」来验证，因此需要专门的评测协议而不是把离线视频生成的指标搬过来。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

## 出题设计：任务三要素 + 时间线

每道题由三个元素组成：**起始素材**（人物图、源视频、语音或参考图）、**输入时间线**（第几秒发出哪条新要求）、**判卷依据 oracle**（每条要求送达后，哪里该变、哪里必须保留）。题目按时长分五档——15 秒、30 秒、60 秒、120 秒、300 秒：短档看反应速度与执行到位程度，长档看多轮要求叠加后是否还记得前面答应过什么。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

300 道题分五类各 60 道：数字人对话、数字人表演、世界生成、文字指令编辑、参考图编辑，基本覆盖实时视频当前三条主线（数字人、虚拟世界、直播画面实时修改）。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

## 判卷：oracle 时间线 + VLM 观察 + 固定规则

评测把能力拆成 4 个维度、10 项指标（每项 0–100）：任务执行、状态一致性、视觉保真度、声画协调；不同任务只使用与自己相关的指标。打分分两步：先由视觉语言模型提取结构化观察（某时刻出现什么缺陷、外观是否匹配、指定动作是否发生），再由固定规则结合题目规定的时限与验证窗口把观察转成精确分数；没有输出或不支持某项必要操作直接记零分。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

设计上最容易出错的是「取消修改」这类语义：披风案例在 0 秒挂红色丝绒参考图、10 秒切金色缎面、20 秒取消修改，此时画面应恢复到**20 秒时点的源画面**，人物与动作绝不能「滚回」最初起点。另一个反直觉之处是编辑成功度按「完整编辑」严判——衣服换对了但把人物手里的杯子改没了，在 Edit Success 指标下仍算失败。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

五类任务**各排各的、不合并总榜**：数字人聊天与生成一个虚拟世界不是一回事，硬加总分会掩盖能力差异。这一点与 [[concepts/agent-evaluation-benchmark-frameworks|benchmark 框架设计]]中「指标必须绑定任务语义」的原则一致，也解释了为何 leaderboard 呈现为多张分类榜而非单一排名。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

## 37 个模型的结果：稳定性高、可控性差

数字人方向由生数科技 Vidu S2-Avatar 拿下对话与表演两项综合第一，其 S2-Editing 在两类视频编辑中领先；世界生成第一是港大、快手可灵等团队的 MemFlow。拆开分项后行业困境很明显：Vidu S2-Avatar 表演稳定性 97.25、动作跟随仅 36.90——而 36.90 已是该任务七个模型里的最高分；参考图编辑最严苛的完整编辑指标全场最高只有 15.00。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

落榜模型各有「错位优势」：Tavus CVI Full 音画同步领先，LiveTalk 1.3B 表演画质最优，JoyAI-Video-Edit 更擅长保留不该改的内容，Krea Realtime 虽世界生成综合分落后但指令跟随达全场最高 25.93。这类分项数据对选型的价值高于总分——实时试衣最怕哪种失误、聊天角色最依赖哪项能力，都能回到对应指标查。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

## 局限与维护

作者侧的局限是明确的：自动评分尚未与人工判断做对照，5 分钟以上的长会话、复杂场景，以及真实部署中的帧率、时延与成本都还没有进入评测范围。维护机制上 StreamEval 每月评测新增与更新后的模型并更新榜单，团队与企业可通过「Evaluate your model」提交自测结果，代码、案例时间线、数据集与评分规则均已开源（GitHub: StreamEval/Stream-Eval-Open，榜单 streameval.ai）。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

对 wiki 的意义在于方法论可迁移：把「何时修改、保留什么、如何恢复、怎样计分」写成显式 oracle 的做法，与 Agent 评测里 [[concepts/evaluation-harness-design|评测 harness 设计]]、[[concepts/eval-surface-rotation|评测面轮换]]面对的同类问题（如何防止指标被刷、如何让评测长期可复现）同构，可直接借鉴到流式多模态系统的评测设计。^[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10.md]

## 深度分析

### 行业的真短板是「可控性」，不是「画质」

37 个模型的分数分布揭示了一个结构性事实：模型普遍已把「稳」做到很高水平，却在「听话」上集体失守。Vidu S2-Avatar 表演稳定性高达 97.25，而动作跟随最高仅 36.90；参考图编辑最严苛的完整编辑指标全场封顶 15.00。这说明现有实时视频模型的架构优化重心仍停留在感知质量与画面流畅度上，而对「指令→状态变更→持续保持」这条控制链路缺乏系统性的建模能力——稳定性和可控性目前是两条几乎不相交的能力轴。

### 流式交互本质上是状态追踪问题

时间线五档设计（15 秒到 300 秒）背后是一个关键判断：实时交互视频的核心难点不是单帧质量，而是多轮指令叠加后的记忆与一致性。短档测的是响应速度，长档测的是「还记得前面答应过什么」——这与对话系统里的上下文一致性问题是同构的。一个在 15 秒档表现良好的模型未必能扛住 300 秒档的多轮编辑，这意味着评价实时系统能力必须把「持续时间」作为独立变量，而不是默认能力随时长恒定。

### 「取消修改」暴露的是时间锚定能力

披风案例的陷阱设计（取消修改时应恢复到 20 秒时点的源画面，而非滚回最初起点）指向一类容易被忽视的能力：模型对「时间上的当前」必须有精确锚定。许多模型在取消操作时把整个场景重置回视频开头，等于把「撤销」误解为「重放」——这类失败模式说明模型内部缺乏对「源画面随时间演化」的表示，只记住了起点状态。这是流式系统与离线生成在语义层面的本质差异。

### VLM 观察 + 固定规则：对 LLM 评委漂移的工程化防御

两步打分（VLM 只负责提取结构化观察，固定规则负责把观察转成分数）刻意把「判断」和「计分」解耦。VLM 不直接给分，避免了 LLM-as-judge 常见的分数漂移与宽严不一；「没输出/不支持必要操作直接记零分」则堵住了用「优雅降级」换取部分分数的漏洞。这套分离式评分架构对任何依赖 VLM 评委会的多模态评测都有直接借鉴价值。

## 实践启示

1. **选型看分项榜，不看综合名次**：五类任务各排各的，落后模型往往有「错位优势」（如 Krea Realtime 世界生成综合分落后但指令跟随全场最高 25.93）。按你的业务场景（实时试衣怕外观错改、聊天数字人怕动作跟不上）直接查对应指标和输出记录。
2. **把 oracle 写进自己的评测**：为任何交互式/流式系统设计评测时，显式规定「何时修改、保留什么、如何恢复、怎样计分」，配合带时间戳的输入时间线。含糊的「效果好不好」式 prompt 会让评测结果不可复现。
3. **解耦判断与计分**：用 VLM 提取结构化观察（某时刻有无缺陷、外观是否匹配、动作是否发生），用确定性规则结合时限与验证窗口计分，并对缺失输出/不支持操作硬性记零，防止模型靠降级表现刷分。
4. **评测里必须包含撤销类语义**：任何支持交互编辑的系统都要测「取消修改」，且验收标准是恢复到操作时点的源状态，而不是回滚到初始状态；同时按「完整编辑」严判——改对了目标却破坏了无关元素仍算失败。
5. **把时长当独立测试变量**：短片段达标不代表长会话达标，多轮指令叠加下的记忆衰减需要用 120/300 秒档专门压测，否则上线后第一场长直播就会暴露。
6. **把基准当活文档用**：StreamEval 每月更新榜单、接受社区提交自测结果，题目与评分规则全开源。自己接入跑分之外，把踩到的新失败模式反哺为新增考题，是低成本获得行业可见度的方式。

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[concepts/eval-surface-rotation|评测面轮换]]
- [[entities/duomatching-few-step-video-generation-joint-marginal-2026|DuoMatching：少步视频生成]]
- [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD：长视频自回归扩散]]
- [[entities/seedance-25-zero-frame-video-generation-2026|Seedance 2.5 零帧视频生成]]
- [[entities/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|VLX-Flow：流式视频 VLM]]

→ [[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10|原文存档]]
