---
title: "StreamEval：首个实时交互视频生成基准（300 题 × 37 模型）"
created: 2026-10-23
updated: 2026-10-23
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

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]
- [[concepts/evaluation-harness-design|评测 Harness 设计]]
- [[concepts/eval-surface-rotation|评测面轮换]]
- [[entities/duomatching-few-step-video-generation-joint-marginal-2026|DuoMatching：少步视频生成]]
- [[entities/a2rd-agentic-autoregressive-diffusion-long-video|A2RD：长视频自回归扩散]]
- [[entities/seedance-25-zero-frame-video-generation-2026|Seedance 2.5 零帧视频生成]]
- [[entities/om-ai-vlx-flow-streaming-video-vlm-vlx系列开篇-2026|VLX-Flow：流式视频 VLM]]

→ [[raw/articles/streameval-interactive-streaming-video-benchmark-2026-10|原文存档]]
