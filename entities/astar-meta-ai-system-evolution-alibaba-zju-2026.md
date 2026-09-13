---
title: "Astar: 用 AI 指导 AI 系统进化（阿里 + 浙大）"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [self-evolution, rsi, meta-ai, alibaba, recommender, post-training, reward-model, rl, harness]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026]
---

# Astar: 用 AI 指导 AI 系统进化（阿里 + 浙大）

阿里与浙江大学联合推出专门用于指导 AI 系统进化的 LLM **Astar**：它不从通用语料学「万金油经验」，而是去深度学习 AI 系统自身的历史版本记录（代码提交 + loss 曲线 + 业务指标涨跌），把这些真实演化经验内化后，根据当前系统状态自适应地探索未来的演化方向。论文为 arXiv 2608.27287；落地于阿里核心推荐业务后，离线 HitRatio 提升 23.6%、在线 GMV 提升 4.86%。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

## 问题定位：想 idea 仍是卡脖子环节

普通软件工程与 AI 模型迭代的差异在验证成本：改一段普通代码跑个测试用例几秒见分晓，改一个工业级模型却要经历改代码→重训→离线评估的昂贵周期，通常数小时到数天。试错成本高昂意味着团队没有算力去穷举所有 idea，于是「把宝贵的算力优先砸给哪个改进方向」长期依赖资深算法专家的直觉。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

直接把代码丢给通用大模型并不可靠：通用模型吸收了海量公开论文与开源代码，深谙前沿原理，但面对特定业务数据和高度定制化的现有系统时，这种通用知识难以直接转化为实战经验——工程师照改、等三天三夜，结果指标纹丝不动甚至反向跳水，算力与时间成本全部打水漂。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

Astar 的切入点是一个被忽略的数据源：AI 系统自身的历史版本记录。每一次 Git 提交都完整记录了模型从上一版到下一版改了什么，以及随之而来的 loss 曲线与业务指标涨跌——这是一份现成的、带真实业务反馈的避坑指南。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

## 四座大山与四条对策

把版本历史变成训练数据需要跨越四个挑战，Astar 为每个挑战设计了一条对策。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

- **数据太少（C1）→ 两两扩增**：不再局限于相邻版本演进，而是把历史上任意两次实验配对（例如对比版本 1 与版本 5），拉出完整 loss 曲线、以谁更低判定更优版本。稀疏记录因此变成密集训练样本，且模型能同时学到短期微调技巧与长期跨越式升级。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]
- **满库是水（C2）→ 两级过滤**：第一道用代码调用图分析（Reachability）与抽象语法树（AST）剥离打印日志、死代码与格式调整；第二道用 LLM 做语义分析，无法归类为明确「优化意图」的改动当噪音过滤。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]
- **空间太大（C3）→ 三级分层提示**：训练数据打上「一级主方向→二级细分模块→三级具体动作」标签，推理时模型先定大方向（如优化策略）、再聚焦细分模块（如 Muon 优化器/梯度正交化）、最后落实具体动作（如梯度降噪），把近乎无限的搜索空间约束成有迹可循的结构化生成过程。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]
- **验证太贵（C4）→ 离线奖励模型**：用历史正负样本训练奖励模型，对生成的演化方案一秒预测其降低 loss 的概率。这个秒级代理专家既能从海量灵感中初筛最靠谱的几个上机验证，也为 Astar 的强化学习后训练提供低成本反馈，使其能主动探索前人未尝试的方向。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

## 训练与结果

基于处理后的演化语料与代理奖励模型，Astar 采用标准三阶段训练：中间训练（mid-training）→ 监督微调（SFT）→ 强化学习（RL）。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

在相同推荐系统优化场景下，通用大模型（含 GPT-5.5、Claude-4.8-Opus 等前沿模型）虽能给出方向，但真实业务中的直接执行成功率存在局限；相比之下即便是参数量最小的 0.6B 版本，单次生成有效优化方向的成功率（S@1）就达到 54.35%，明显超越最强的通用模型 GPT-5.5（30.71%），也超过人类资深算法专家（32.29%）；8B 版本进一步推到 67.86%。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

这组数据指向一个趋势：在垂直工程决策领域，喂给模型高质量、浸透真实系统反馈的「领域演化史」，与盲目增加通用参数规模同样是提升性能的关键路径。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

## 深度分析

### 把「科研直觉」变成可训练对象

Astar 真正被训练的不是某个模型的参数，而是**改哪里的判断力**：监督信号来自系统自身演化的正负样本（版本 A 比版本 B 好），层级提示提供方向空间的坐标系，奖励模型提供廉价验证。三者合起来把稀缺的专家直觉转译成「mid-training + SFT + RL」的标准流水线，且 0.6B 就超过了人类专家——说明该任务的瓶颈在过去不是模型容量，而是**领域反馈的组织方式**。这与 [[entities/aide2-recursive-self-improvement-weco-2026|AIDE² 递归自改进]]、[[entities/agent-self-evolution-flywheel-tencent-2026|腾讯 Agent 自进化飞轮]] 同属「让系统自己产出改进信号」的路线，但粒度更靠近模型训练侧而非 skill/提示侧，可与 [[entities/harness-evolution-papers|Harness 进化论文群]] 中的「进化什么、用什么反馈」两轴对齐阅读。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

### 奖励模型是这套闭环的单点依赖

四座大山中最具结构性的是 C4：没有秒级奖励模型，RL 后训练不可行，而 RL 恰恰是 Astar 从「会表达方向」到「会判断方向」的分水岭。奖励模型由历史正负样本训练得到，这意味着它的能力上界由历史实验的覆盖度决定——**它只能廉价评价那些在历史上以某种形式出现过的改进类型**。因此 Astar 的探索能力是「历史外推」而非「历史外」：主动探索前人未尝试的方向这一主张，仍要通过上机验证才闭环。对工业团队的直接含义是：奖励模型与演化语料是资产，且随每次真实实验单调增厚，构成数据飞轮而非一次性工程。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

### 与通用 RSI 研究的位置

如果把 [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026|AI4AI 综述]] 与 [[entities/recursive-automated-ai-research-first-steps-2026|递归自动化研究首步]] 描述的路径看作「AI 做研究」，Astar 是其中切口最窄的一类：不做假设生成、不写论文，只回答「当前这个系统的下一步改动应该是什么」，并且用线上业务指标（GMV +4.86%）而非学术 benchmark 结账。切口窄带来可验证性，也带来迁移成本——四座大山的对策都高度依赖「有长期 Git 实验记录 + 指标可回读」的重业务场景，推荐/搜索这类迭代密集域天然适配，缺少历史实验流水线的团队则没有足够语料启动。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

### 边界与开放问题

披露的定量结果集中在单一推荐系统优化场景（S@1/RM@k 与集团内离线/在线指标），跨任务类型（CV/NLP/多模态训练管线、Agent 系统）的通用性未给出证据；「三级分层提示」的层级体系由团队人工定义，层级本身的适配成本未量化；奖励模型秒级打分与真实上机结果的一致率（即代理指标可信度）也未披露。此外 0.6B 超过人类专家的对比中，人类专家的样本量与评估协议细节有限，宜按「同任务同协议下相对优势」读，而非「人类经验被替代」。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

## 实践启示

- **先问「有没有历史实验流水线」，再问要不要训练**：Astar 的原料是 Git 提交 + loss 曲线 + 指标回读，三者缺失时该路线的启动成本与收益不成比例。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]
- **稀疏实验记录可以用配对法放大**：任意两次历史实验配对 + loss 比较，即可把「相邻版本」的稀少监督扩展成密集样本，这是最低成本的数据增广手段。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]
- **过滤比采集更决定质量**：AST/可达性分析去噪 + LLM 语义归类「是否构成优化意图」两道过滤，把占多数的日志/路径/清理提交排除在外，避免把噪声当成演化规律。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]
- **用代理奖励换取探索预算**：秒级奖励模型把「哪个方向值得上机」与「上机验证」解耦，是最直接降低单次实验边际成本的做法，也为 RL 提供可持续反馈。^[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026.md]

## 相关实体

- [[entities/ai4ai-survey-composition-gap-recursive-self-improvement-2026]] — AI for AI / 递归自改进的综述框架
- [[entities/aide2-recursive-self-improvement-weco-2026]] — 递归自改进的另一条实现路线
- [[entities/agent-self-evolution-flywheel-tencent-2026]] — 自进化飞轮的 Agent 侧形态
- [[entities/agent-evolution-four-stages-six-dimensions-aliyun]] — 阿里侧 Agent 演化阶段框架
- [[entities/recursive-automated-ai-research-first-steps-2026]] — 自动化研究的早期实践
- [[entities/harness-evolution-papers]] — 进化对象与反馈来源两轴的论文群
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] — RL 后训练算法谱系

→ [[raw/articles/astar-meta-ai-system-evolution-alibaba-zju-2026|原文存档]]
