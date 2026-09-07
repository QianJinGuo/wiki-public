---
title: "The next big breakthrough will be AIs learning on the job"
created: 2026-08-01
updated: 2026-09-07
type: entity
tags: ['raw', 'article']
sources: [raw/articles/the-next-big-breakthrough-will-be-ais-learning-on-the-job]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/the-next-big-breakthrough-will-be-ais-learning-on-the-job.md|原文存档]]

sha256: 8e20a931ea9cae9c14fd33ebda63509ca1feb3b54cc8c2160bfa3585ebead099 ^[raw/articles/the-next-big-breakthrough-will-be-ais-learning-on-the-job.md]

## 摘要

Dwarkesh Patel 在这篇长文中质疑实验室的主流押注——通过在数千个 RL 环境中训练 AI 完成数百万可验证任务（RLVR）来造出 AGI。核心论据是：一个领域不仅要"可验证"，还必须"可反复研磨"（grindable）——即能对确定性、可重放的模拟器跑大量并行 rollout。编程满足这个条件，而 computer use 不满足（无法让一千个 Agent 同时在 Amazon 上跑结账流程），更做不到的领域包括创业、打官司、赢选举等——这些 rollout 必须与真实世界交互，验证回路可能长达数月数年。模型在训练中样本效率比人类低百万倍，而大多数领域的真实数据稀疏、不可重放，因此"除非能为一个领域建造可重放的训练目标，否则模型很难取得进展"。^[raw/articles/the-next-big-breakthrough-will-be-ais-learning-on-the-job.md]

作者进而论证样本效率与持续学习（continual learning）是深度关联的问题：靠上下文学习虽样本高效，但 KV cache 每 token 增长 320KB（Llama 3 70B），内存上不可扩展，也不是人类的学习方式；而梯度更新又极度样本低效，现有在线学习（如 Cursor Tab 模型每天对 4 亿+ 请求学同一目标）无法针对特定部署学习专属知识。他提出的解法是 on-policy self-distillation（OPSD）：把长会话中积累了上下文的"老师"模型的预测蒸馏回基座模型权重——无需外层可验证奖励，且提供比 RL 更密集的监督信号，同时保留 RL 稀疏更新的优点（只改必要的部分，不覆盖已有知识）。更投机性的方向是"dreaming"：模型自建世界模拟来排练技能，若可行将成为预训练、RL、推理时算力之后的第四条 scaling 轴（可称 test-time training）。他预测到 2027 年底：RLVR 先造出足够胜任真实部署的 Agent，一周墙钟时间的协作后用户一个点赞/点踩，基座模型用 OPSD 等技术把会话所学蒸馏回权重，AI 的主要提升将来自部署后的在职学习——"每次你与 AI 交互，它都会更聪明"。Dario 在播客中"训练上下文长度与服务上下文长度不一致会导致退化"的表述被作者引为短程 RL 训练未必泛化到长程表现的暗示。^[raw/articles/the-next-big-breakthrough-will-be-ais-learning-on-the-job.md]

## 关键要点

- 约 30-50% 的实验室算力用于推理，这些算力目前对改进模型没有产出，而最有价值的学习信息恰恰只在部署时才被揭示。
- 一小时视频约消耗 100 万 token 文本的上下文，这是 computer use 进展慢的原因之一。
- EfficientZero 用 2 小时 Atari 游戏经验打败人类新手，靠的是每步在脑内跑几十局模拟对局——"dreaming"思路的原型。
- 用 Llama 3 70B 计算：上下文学习每 token 存储 320KB 信息，而预训练每 token 只存 0.075 bit，相差约 3500 万倍。
- 作者不看好"给自己写 Markdown 笔记"式的持续学习，比喻是：学生学萨克斯的方式是弹一遍、记笔记、把琴和笔记交给下一个初学者。

## 来源

- 原文: [[raw/articles/the-next-big-breakthrough-will-be-ais-learning-on-the-job.md|The next big breakthrough will be AIs learning on the job]]
