---
title: "PARPO：把个性化写进 Agentic RL 的训练目标（中科大 + 阿里）"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [agentic-rl, personalization, reward-model, memory-graph, grpo, user-profile, e-commerce, post-training, alibaba, ustc]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026]
---

# PARPO：把个性化写进 Agentic RL 的训练目标

中国科学技术大学与阿里巴巴的团队提出 **PARPO（Personalized Anchor Reward-Decoupled Policy Optimization）**，把「个性化」从推理期的 prompt 拼接上移到训练期的优化目标：同一 query 下不同用户偏好不同的规划策略、工具调用与表达方式，只优化通用任务完成度会让模型学成「平均用户最喜欢的样子」，而把历史行为直接塞进 prompt 又会把从众、流行度与上下文噪声误当成真实偏好。论文为 arXiv 2605.23382《From Correctness to Preference》，在 ETAPP、ETAPP-Hard 与商家决策场景 SJAgent 上以 Qwen3-4B/8B 训练，8B 模型 Judge 分 0.8333、SJAgent Reward 0.8270，并在 15 位人类专家与 4 组 LLM judge 的盲测中排名第一。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 三个被明确命名的问题

论文把「正确答案为什么还不够」拆成三条约束：C1 个性化奖励含义不清——通用奖励无法表达同一条轨迹对不同用户的不同价值；C2 偏好信号被行为混合——真实兴趣、流行度影响与群体从众交织在一起；C3 记忆仍然太扁平——难以显式表达用户、技能、工具、场景与历史轨迹之间的关系。这三条同时决定了下游三件套的设计边界。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 三件套：记忆、奖励、优化各司其职

PARPO 不是替换 GRPO 的一个公式，而是三个相互衔接的部件。**PSGM**（偏好对齐技能演化图记忆）在 rollout 之前把历史经验组织为 User / Skill / Tool / Scenario / Trajectory 五类节点的异构图：检索时先按 query 找到语义种子技能，再沿「技能—用户—同源技能」扩展候选，综合用户相关性、互补加成与冲突惩罚排序，因此召回的未必是文本最相似的技能，而是处在同一有效行为邻域里的技能。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

**两阶段偏好解耦奖励模型**在 rollout 之后产生个性化奖励：第一阶段用多视角用户画像缓解冷启动；第二阶段在用户—物品交互图上用 LightGCN 建立「兴趣」与「从众」两个正交分支，兴趣分支提高低流行度项目的权重、从众分支相反，从而从混合行为里提取更干净的偏好信号。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

**PARPO 优化器**把通用任务质量与个性化偏好拆成两条 advantage 轨道再做策略更新，个性化分支用指数移动平均维护**用户级历史锚点**，让当前奖励与该用户自己的历史中心比较，而不是与所有用户混合后的均值比较，缓解异质偏好下的 reward-center 与 reward-scale 失配。论文给出的分工直觉是：通用质量负责让 agent 把事情做对，个性化质量负责让它用这个用户愿意接受的方式做好。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 闭环与消融

高质量轨迹随后被总结为新技能写回 PSGM，形成「检索—交互—评价—优化—积累」的闭环。消融实验显示：去掉技能记忆后 Judge 从 0.7708 掉到 0.7006，是所有消融中最大的降幅；去掉用户锚点或兴趣/从众任一分支也都明显掉点，说明记忆、解耦与用户锚点三者缺一不可。最大优势维度出现在 User Relevance，即提升主要来自更强的用户对齐，而不只是更顺滑的文字表达。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 边界

论文自陈两处局限：人工盲测规模仍然有限；兴趣—从众解耦是操作层面的正交化，并不构成严格的因果分离。它要论证的方向是「同一个任务不再只有一种正确完成方式」——真正好的策略还必须是这个用户愿意持续使用的完成方式。^[raw/articles/parpo-personalized-agentic-rl-ustc-alibaba-2026.md]

## 相关

- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026]]
- [[entities/user-governed-personalization-agentic-recommendation-paradigm-2026]]
- [[entities/remember-when-it-matters-proactive-memory-agent-long-horizon-wu-meta-2026]]
- [[entities/agent-memory-architecture]]
- [[entities/agent-evaluation-turing-meituan-2026]]
- [[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528]]
- [[concepts/grpo-policy-optimization-2026]]
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]
