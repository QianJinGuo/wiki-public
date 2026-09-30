---
title: "State of Open Models: Summer 2026 Observations"
created: 2026-08-15
updated: 2026-09-30
type: entity
tags: [ai, llm, open-models, huggingface, open-source-ecosystem]
sources: [raw/articles/state-of-open-models-summer-2026-observations]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# State of Open Models：2026 夏季观察

Hugging Face 在 2026 年 8 月发布的半年一度开源模型生态报告，基于 2026 年 1 至 7 月 Hub 上的下载、点赞、衍生模型与 release 数据，总结出六个关键观察：前沿规模竞赛由中国实验室主导、注意力（点赞）与采用（下载）是两种不同经济、开源权重把价值积累转移到 API 与生态位、Qwen 成为社区基础模型、小模型仍是实用层、以及 Agent 成为 Hub 的新用户。报告期内公开模型仓库从 243 万增长到 296 万，数据集达到 100 万，Spaces 达到 144 万，但分布极端：约 85.6% 的模型终身下载不足 200 次，1.5% 的仓库贡献了 99.2% 的下载量。^[raw/articles/state-of-open-models-summer-2026-observations.md]

## 前沿规模：中国实验室的月度天花板全面领先

2026 年几乎每个月，中国实验室发布的最大开源模型都比美国实验室自己的 release 更大：中国月度天花板在 754B 到 2.78 万亿参数之间，美国自己的天花板在七个月中有五个月低于 130B，例外是 NVIDIA 的 Nemotron 3 Ultra（561B）和 Thinking Machines Lab 的 Inkling（952B）。Moonshot、MiniMax、Xiaomi、Z.ai 几乎不发布 70B 以下模型，而腾讯和阿里 Qwen 覆盖从 1B 以下到万亿参数的完整谱系——大小策略成为"押注基准排名与 API 需求"还是"成为开发者标准化的家族"的意图声明。^[raw/articles/state-of-open-models-summer-2026-observations.md]

开源的中心从模型实验室转移到硬件与基础设施公司：AMD 与 NVIDIA 各发布超过 200 个新模型仓库（远超前排），LiquidAI 约 100 个排第三，开源模型成为卖芯片的证明。美国 100B 以上规模的大多数 release 是在中国模型之上构建的衍生品而非原创模型。^[raw/articles/state-of-open-models-summer-2026-observations.md]

## 注意力 ≠ 采用，许可证揭示真正的商业模式

按今年累计下载与按点赞分别取前 25 名的模型仓库，交集只有一个。2026 年新发布的模型没有一个进入下载前 25，而 25 个中有 13 个来自 2022 年：all-MiniLM-L6-v2 七个月被拉取 15.5 亿次但只有 5,156 个赞。点赞记录"发布很重要"的兴奋，下载记录"接入生产管线"的依赖，两者不可互相替代。^[raw/articles/state-of-open-models-summer-2026-observations.md]

许可证数据说明开源权重的回报不在授权费：178 个中国 20B 以上 release 中 59% 采用 Apache 2.0、22% 采用 MIT，没有任何一个带非商用限制；DeepSeek 与 Z.ai 在 700B 到 1.65T 参数规模直接使用 MIT。美国同规模带中只有 29% 是 Apache/MIT。价值回收来自 API 与云业务、硬件与平台定位、以及生态位本身，这与 [[entities/how-far-behind-are-open-models-2026|开源模型与闭源差距分析]] 中关于开源商业化的讨论一致。^[raw/articles/state-of-open-models-summer-2026-observations.md]

## Qwen 成为社区基础模型，小模型仍是实用层

按衍生模型衡量，Qwen 系模型在 Hub 上已有 151,448 个衍生仓库，是 Meta 总足迹的 2.6 倍、Llama 专属的 4.7 倍；Google 以 82,506 个跟随其后。Qwen 衍生以每天约 180–210 个新仓库的速度增长，靠的是稳定的发布节奏、尺寸覆盖和 Apache 2.0 开放许可形成的正反馈。这个位置主要由社区建立：151,448 个衍生中 Qwen 自己发布的只有极少部分，28,531 个 GGUF 转换中 Qwen 官方只发布了 54 个。^[raw/articles/state-of-open-models-summer-2026-observations.md]

参数规模小于 1B 的模型占历史下载总量的 83%，100B 以上只占 1%。万亿参数模型能触达普通开发者靠的是 [[entities/llama-cpp-deployment|llama.cpp 本地部署]]：7 月快照已包含约 284B 的 DeepSeek-V4-Flash 与约 2.8 万亿的 Kimi-K3 的 GGUF 构建，本地推理从笔记本上的 8B 变成跨几台消费级机器的万亿 MoE。声明 gguf 库的仓库数增长 464%、lerobot 194%、Apple mlx 148%，而 transformers 只有 16%——运行时层比建模核心增长快三到七倍。Qwen 系的 GGUF 月下载 3,960 万次，接近 Gemma（2,080 万）的两倍、Llama（750 万）的五倍以上。^[raw/articles/state-of-open-models-summer-2026-observations.md]

## 深度分析

### 开源与闭源的差距正在被"生态位"而非"分数"重新定义

这份报告最有信息量的结论不是"中国模型更大"，而是开源模型的胜负逻辑已经从单点基准分数转移到生态嵌入深度。中国实验室用最宽松的许可证（178 个 20B 以上 release 中 81% 是 Apache 2.0 或 MIT，零非商用限制）把前沿权重送给所有人，说明它们根本不指望授权费回收成本——回报在 API/云、硬件定位和生态位本身。这与 [[entities/how-far-behind-are-open-models-2026|开源模型与闭源差距分析]] 的判断一致：开源与闭源的差距在能力维度持续收窄，而在商业维度的竞争才刚刚开始。DeepSeek 与 Z.ai 把 700B 到 1.65T 参数的模型直接放 MIT，本质上是把"模型"降级为获客手段，把"生态位"升级为资产。

### 硬件公司接管开源中心，模型成为芯片的"证明材料"

发新模型最多的两个组织是 AMD 和 NVIDIA（各 200+ 仓库），而不是 Google 或 Meta——开源的重心已经从模型实验室转移到硬件与基础设施公司。这是一个结构性信号：当模型本身不再稀缺，"能跑在你的硬件上"成为新的差异化。反向的对称竞争也在发生，中国模型越来越多针对国产芯片优化。而美国 100B 以上的 release 大多是中国模型的衍生品（分发与优化层），真正的原创者只剩 Thinking Machines 的 Inkling、NVIDIA 的 Nemotron 系列等少数几家——这与 [[entities/the-inference-shift|推理转移]] 描述的价值向推理侧迁移是同一枚硬币的两面。

### 运行时层是本季度增长最快的信号，也是最被低估的护城河

数据上最锐利的对比：模型仓库增长 21.5%，而 gguf 库声明增长 464%、lerobot 194%、Apple mlx 148%。运行时层以建模核心 3-7 倍的速度增长。llama.cpp 的存在直接改写了发布策略的游戏规则——实验室不必再发小模型来"够到"开发者，社区量化层几天内就能让万亿 MoE 跑在消费级机器上（7 月快照已包含 284B 的 DeepSeek-V4-Flash 和 2.8T Kimi-K3 的 GGUF 构建，本地推理详见 [[entities/llama-cpp-deployment|llama.cpp 本地部署]]）。这解释了为什么 Moonshot、MiniMax 敢于只发 70B 以上模型：小模型的"可达性"已被运行时层外包。但值得注意的反差是：前十大模型家族的官方 GGUF 转换都极少（Qwen 28,531 个 GGUF 中官方只发了 54 个），官方转换、量化文档与制品签名是一个低成本高回报的空白。

### 与春季报告相比，真正变化的是"谁在使用 Hub"

春季报告还在讨论"哪些模型被发布"，夏季报告第一次有了 agent 流量数据：Claude Code 从 4 月的 67.8% 跌到 5 月的 6.4% 再回到 7 月的 44.4%，Codex 稳步爬升到 20.8%，5 月有 59.8% 的 agent 流量来自数据集尚未命名的新 harness——没有在位者的市场，一次 release 就能移动一半流量。Hub 也开始为机器读者重构基础设施（机器可读 Markdown、agent trace 一等公民、MCP 服务器）。7 月的自主 agent 入侵事件（详见 [[entities/openai-huggingface-agent-intrusion-incident-2026|Hugging Face agent 入侵事件]]）则给出了另一半答案：闭源模型的安全护栏拒绝了攻击代码分析，最终由自家基础设施上的量化开源 GLM-5.2 完成——开源模型在安全敏感场景中意外成为唯一可用的分析工具，这是报告之外最值得咀嚼的细节。

## 实践启示

1. **选基座模型看衍生生态，不看发布热度。** Qwen 以 151,448 个衍生仓库（Meta 的 2.6 倍）成为社区默认基座，且 180-210 个/天的新增主要来自社区而非官方。选型时应优先考虑衍生生态规模、GGUF 可得性和许可证（Apache 2.0 优先），而不是点赞榜——点赞与下载的交集只有 1 个模型。
2. **用下载量而非点赞判断生产依赖。** 点赞衡量"发布很重要"，下载衡量"接入了排程运行的生产管线"；all-MiniLM-L6-v2 七个月 15.5 亿次下载只有 5,156 个赞。做技术调研或竞品分析时，把点赞当情绪指标、下载当依赖指标，混用两者是最常见的判断错误。
3. **自托管路线不必等小模型。** llama.cpp + GGUF 已让 2.8T 级 MoE 可以跨几台消费级机器部署，如果对数据主权或成本敏感，先检查目标模型的 GGUF 社区生态再决定是否走 API，2026 年本地/自托管与 API 的成本差距比一年前显著缩小。
4. **开源模型的商业模式押注在 API 和生态位，评估其可持续性时要看正反馈循环而非权重本身。** 家族覆盖（1B 到万亿）、稳定发布节奏、宽松许可证三者互相强化才会形成 Qwen 式飞轮；只有单点旗舰的开源 release（哪怕刷新 benchmark）难以沉淀为基础设施。
5. **规模策略是意图声明，可以反推实验室的战略。** 只发 70B+ 的实验室在押注基准排名与 API 需求（frontier-only），全谱系发布的在竞标"开发者标准化家族"的位置。预测一家实验室下一季度的动作时，先看它的尺寸策略属于哪个阵营。
6. **为 agent 读者优化你的接口。** Hub 上的第一名用户已经是 agent：机器可读文档、agents.md、结构化 API 成为流量入口。构建面向开发者的产品时，假设主要"读者"是自动化客户端，文档和接口的机器可读性直接决定采用率。

## Agent 成为新的用户

7 月发布的 agent-usage 数据集第一次记录了编码 Agent 调用 Hub 的流量：Claude Code 7 月占 44.4%，但 4 月是 67.8%、5 月只有 6.4%，而 Codex 从 10.4% 稳步升到 20.8%——没有在位者的市场里，一次 release 或一个默认值变更就能在一个月内移动一半流量。7 月近四分之一的 Agent 标记流量来自数据集尚未命名的 harness，4 到 7 月出现了十几个新客户端标识。Hub 同时开始为机器读者建设：3 月论文提供机器可读 Markdown，4 月 Agent trace 成为一等数据集类型并上线 agents.md 端点，7 月 MCP 服务器上线 hf_fs 工具，MCP 协议本身进入 Linux 基金会旗下 Agentic AI Foundation。7 月还发生了首个有记录的自主 Agent 持续入侵事件，HF 用自家基础设施上的量化开源模型 GLM-5.2 完成了攻击代码分析。^[raw/articles/state-of-open-models-summer-2026-observations.md]

整体来看，地理再平衡在加速：开源模型从模型实验室移向硬件与基础设施公司，中国前沿模型之间的竞争吸引大量社区关注，而 Agent 首次成为 Hub 上排名第一的用户。报告强调，AI 竞赛既是短跑也是马拉松——开源 LLM 生态 的胜负取决于模型家族与开发者之间能否形成正反馈循环，并最终嵌入基础设施。^[raw/articles/state-of-open-models-summer-2026-observations.md]

→ [[raw/articles/state-of-open-models-summer-2026-observations|原文存档]]
