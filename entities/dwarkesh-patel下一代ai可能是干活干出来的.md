---

title: "Dwarkesh Patel：下一代AI，可能是干活干出来的"
type: entity
created: 2026-07-04
updated: 2026-09-21
tags: [wechat, ai]
rating: v7c7
sources:
  - raw/articles/dwarkesh-patel下一代ai可能是干活干出来的
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Dwarkesh Patel：下一代AI，可能是干活干出来的

**来源**: 机器之心

**发布日期**: 2026-06-28^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]


**原文链接**: http://mp.weixin.qq.com/s?__biz=MzA3MzI4MjgzMw==&mid=2651041546&idx=2&sn=fedc77937dafe2bff30de5b1d8d47e40&chksm=84e66b74b391e2622fe2705d6d20db3f35685261a164ac701af536a6a098971b92c858cfd776#rd ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

---

机器之心编辑部

硅谷著名科技播客主持人 Dwarkesh Patel 最近抛出了一个问题： AI 的下一代训练范式会是什么？ ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

Dwarkesh Patel 是硅谷近几年快速走红的科技播客主持人和写作者，年仅 25 岁，却已经凭借 Dwarkesh Podcast 进入 AI 讨论的核心圈层。他的采访对象包括 Ilya Sutskever、Andrej Karpathy、Dario Amodei、Demis Hassabis、Mark Zuckerberg 等一众 AI 与科技大牛。TIME 曾将他列入 2024 年 TIME100 AI，称他的播客已经成为许多 AI 从业者的重要收听内容。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

在最新一期的播客中，他把当下前沿 AI 实验室正在押注的路线总结为一个关键词： RLVR ，也就是 Reinforcement Learning with Verifiable Rewards，可验证奖励强化学习。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

简单说，就是让模型在大量可以自动判断对错的任务中反复试错，训练出规划、纠错、迭代和长期执行能力。今天代码、数学等领域的快速进展，很大程度上就来自这种思路。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

但 Dwarkesh 真正想追问的是： 如果下一代 AI 只靠这种「可验证任务训练」，够不够？^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]


他的答案是：可能不够。

因为一个任务光「可验证」还不够，它还必须「可刷」。^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]


这里的关键概念是 grindability，可磨性。 放在 AI 训练语境里，是「可反复刷题性」或者「可大规模 rollout 的能力」。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

代码任务就是典型的可刷任务。你可以准备一个软件仓库、一个待修复 bug、一个测试用例，然后把同一个环境复制成几千份，让几千个 agent 同时尝试。谁通过测试，谁就得分。这个过程可以并行、可复现、可重置，特别适合 RLVR。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

数学题也是类似的。答案对不对可以验证，训练环境也容易复制。^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]


但 Dwarkesh 问了一个很有意思的问题：为什么 AI 在「使用电脑」这件事上，进展反而比代码和数学慢？ ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

表面上看，电脑使用也是可验证的。比如东西有没有下单成功、活动场地有没有订好、税表有没有提交，这些结果都可以判断。但问题在于，它很难被大规模复制和回放。你不能让一千个 agent 同时去 Amazon 上反复跑同一个结账流程，因为真实网站会识别 bot、封禁账户、改变状态。你当然可以克隆 Slack、Gmail、Amazon 这样的应用来做模拟器，但这在当前阶段仍然是高成本、低扩展性的工程。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

Dwarkesh 指出： AI 在某个领域进步快，不只是因为这个领域答案可验证，而是因为这个领域能被包装成可复制、可回放、可并行试错的训练环境。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

这也解释了为什么代码、数学、游戏类任务会成为 RLVR 的天然温床，而很多真实世界任务却很难直接纳入这套训练范式。 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

接着，他把问题推向更复杂的现实世界。

- 如果我们想训练一个 AI 从零开始创

^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

→ [[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的|原文存档]] ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]

## 深度分析

### grindability 才是隐形筛选器：能力来自「能刷的任务」，而非「重要的任务」

Dwarkesh 的核心重构，是把「可验证」与「可刷」拆成两个独立条件 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]。可验证只回答「结果能不能判对错」，grindability（可磨性）回答「同一环境能不能被复制成千份、并行 rollout、随时 reset 再来一遍」。后者是训练经济学问题，不是认识论问题。于是出现一个反直觉排序：代码、数学、游戏进步最快，不是因为它们更「重要」，而是因为它们天然满足高 grindability；而订会场、报税、打官司这类结果同样可判定的任务，却因为世界不可重置、网站会识别 bot、账户会被封而卡住。由此推出一个不太舒服的结论：benchmark 上的分数更多反映「该任务被工程包装成环境的容易程度」，而不是它在真实世界中的价值与难度。

### verifier 是瓶颈，但真正的瓶颈是环境工程化能力

顺着 grindability 往下看，verifier 本身并不是最稀缺的资源 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]。判对错通常是廉价的（测试用例、编译器、单元断言），真正昂贵的是把真实场景改造成「可复制、可回放、可并行」的数字孪生：克隆 Slack、Gmail、Amazon 的行为语义，处理登录态、验证码与状态污染。这与 [[concepts/verifier-paradox|Verifier 悖论]] 有交集但落点不同——后者说的是验证带宽跟不上生成能力，前者说的是「就算验证免费，环境仍可能造不出来」。因此 RLVR 的天花板不完全由 reward 的噪声决定，而由环境工厂的产能决定；环境工程（sandbox、mock、状态快照、reset 协议）从支撑性工作变成了决定采样量的核心资产。

### 长时程任务与可验证奖励的结构性错配

把视野拉到长时程（long-horizon）：任务越接近真实价值，反馈越慢、变量越多、环境越 non-stationary ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]。一次创业以年计，一场官司无法从同一初始状态复制成一千个平行宇宙，选举更是多重偶然性的叠加。这类任务在 RL 术语里同时具备三个反训练性质——reset-free、non-stationary、credit assignment 极稀。于是出现错配：RLVR 能在可刷环境里练出规划、纠错、迭代与长期执行这些通用技能，而真实交付物需要的恰恰是它练不到的东西——从一次含糊的客户反馈、一次开砸的会议、一条组织内部的隐性流程里提取信号。这也是 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 语境下「泛化到不可刷任务」这份乐观主义始终是实证问题而非口号的原因；它必须等真实部署数据来裁决。

### 数据墙的另一面：稀缺的不是文本，是「真实任务轨迹」

pretraining 撞的是人类文本墙，RLVR 撞的是另一堵墙——可刷任务墙 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]。而 [[concepts/scaling-laws|Scaling Law]] 叙事里没有被定价的那类数据，恰恰是最值钱的：模型在真实组织里与真实用户协作一周所产生的轨迹——它在哪里失败、哪条建议在现实中根本行不通、哪些隐性流程没写进任何文档。这类信号目前被系统性浪费——in-context 学到的东西随会话结束蒸发。由此，学习必须「回到权重」（learning back to the weights）：不是把 KV cache 无限拉长、不是把所有历史塞进上下文，而是像员工工作半年后形成判断力那样，把经历压缩成少量高价值知识。这直接让稳定性-可塑性权衡这个老问题在新范式下回归：增量写权重的代价没有被解决，只是变得无法回避。

### 新增一条 scaling 轴：从评测场走向生产场

Dwarkesh 给出的机制草图有两条：on-policy self-distillation（OPSD），让长会话里已积累经验的上层模型充当 teacher，把「有完整上下文时的判断」蒸馏给基础模型，从而不需要外部可验证 reward 也能获得 token 级密集监督；以及 dreaming，即由模型依据真实观察自造模拟环境，在其中反复演练策略再把经验压回权重 ^[raw/articles/dwarkesh-patel下一代ai可能是干活干出来的.md]。他的时间表是：RLVR 先产出 competent agent → 部署进真实工作一周 → 用户给出 thumbs up/down 或一段工作评价 → 把这次任务学到的东西蒸馏回基础模型。这条流水线一旦跑通，训练重心就从「发布前的可验证任务」迁到「发布后的生产流量」；第四条轴 test-time training 意味着算力配置要改写为「评测场造 agent + 生产场喂 agent」的双循环。

## 实践启示

1. **把「可刷性」当作一级指标来评估任务池。** 在决定把哪个业务场景投给 RL 或者做专项训练之前，先回答三个工程问题：这个环境能否被复制成上千份？能否随时 reset 到同一起点？失败态是否会污染后续轨迹？任一为否，先别投 RL，先投环境工程。代码、数学、带测试的仓库天然满分；网页操作、桌面软件、真实交易则需先造 mock 与 sandbox。

2. **优先投资「环境工厂」而非 reward model。** verifier 通常便宜，环境复现昂贵。把预算放在 sandbox 快照、状态注入、bot 风控规避、应用克隆（Slack/Gmail/ERP 的行为语义）与 reset 协议上，收益比调 verifier 更大。并行轨迹数 × 可复现度，决定了 RL 曲线的斜率。

3. **建可验证闭环的顺序：先确定「不可作弊的终局判据」，再倒推轨迹录制。** 每个任务至少要有一个不依赖模型自评的终局信号（测试通过、接口 2xx、账单状态变更、人工 thumbs）。若只能用 LLM-as-judge，必须防止它既当生成方又当评审方——验证带宽跟不上生成能力时，错误交付只被「合法化」，不会被拦住。

4. **为「发布后学习」预留接口，今天就能做的最小版本，是把部署轨迹结构化留存。** 即使暂时不做权重更新，也应把真实任务轨迹（上下文、动作、失败点、用户反馈）以可训练格式落库：谁在什么组织里、拿模型做什么、哪一步崩了、用户最后给了什么评价。这批数据是未来 OPSD、[[entities/anthropic-dreaming-claude-managed-agents-ovz5v7jjkqdksu9xmxwt8w|Dreaming 类能力]] 与 test-time training 的唯一原料，现在不留，将来无从补录。同时把「会话内学到的经验」与「写回权重的经验」分开记账，避免把上下文适应误判为能力增长。

5. **度量口径要改：从 benchmark 分数转向「可刷迁移半径」。** 更该测的是：在可刷环境里练出的规划与纠错能力，迁移到不可刷任务时衰减多少？用分布外的真实长任务（多天、跨工具、需人类反馈）作为评测场，记录首次成功率与「需要人类接管前的自主步数」。同时显式追踪过拟合信号——分数涨而真实交付不变，说明任务被刷穿了。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

