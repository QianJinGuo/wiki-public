---
title: "BEACON：里程碑引导的 Long-Horizon 语言智能体策略学习（ICML 2026）"
created: 2026-08-22
updated: 2026-11-07
type: entity
tags: [agentic-rl, reinforcement-learning, long-horizon, credit-assignment, grpo, milestone, llm-agent, training, icml]
sources: [raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026]
confidence: 0.82
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# BEACON：里程碑引导的 Long-Horizon 语言智能体策略学习（ICML 2026）

## 问题：长程任务的信用分配病根

用强化学习训练语言智能体时，任务是一步一步完成的，但奖励只在最后一刻给出。以主流 GRPO 为例，它采用轨迹级稀疏终局奖励，整条轨迹共享同一个优势值，在长程任务中暴露两个缺陷：**信用误分配（Credit Misattribution）**——前 20 步都走对、第 25 步出错，整条轨迹被判负，正确的早期动作一起挨罚；**样本利用率低（Sample Inefficiency）**——长程任务成功轨迹本就稀少，一组 rollout 全部失败时组内奖励方差为零，优势函数归零，这批样本对训练毫无贡献。^[raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026.md]

## 方法：里程碑引导的三步框架

浙江大学联合百度提出 BEACON（Milestone-Guided Policy Learning）。里程碑由环境给定的指示函数 Φ 判定（无需额外训练模型），三个环境的实现各异：ALFWorld 通过物体持有/加热状态等谓词判断关键步骤；WebShop 通过搜索结果、商品属性、下单确认页等检查点判定进展；ScienceWorld 直接读取环境自带 subgoal_completed 接口。^[raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026.md]

框架三步：

1. **轨迹分段**——在里程碑边界处把整条轨迹切成若干段
2. **段内时间衰减奖励**——已完成段内动作获得 `r = R · γ^(t_k − t)` 塑形奖励，离里程碑越近信用越高，部分成功不再被浪费
3. **双尺度优势估计**——轨迹级优势沿用 GRPO 把握全局，段级优势只在到达同一里程碑的轨迹间比较，线性组合 `Â = A_traj + λ · A_seg`

整个方法只引入两个超参数（衰减因子 γ 与段级权重 λ），实现简单。^[raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026.md]

## 实验结果

在 ALFWorld、ScienceWorld、WebShop 三个基准、1.5B 与 7B 两个模型规模上一致超过 GRPO 和 GiGPO：ALFWorld 长程任务成功率从 GRPO 的 **53.5% 提升到 92.9%**；有效样本利用率从 **23.7% 提升到 82.0%**；零优势比例从约 55% 降到约 10%。^[raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026.md]

- 领先幅度随任务长度单调扩大：相对增益从短程 +26.2% 扩大到长程 +73.6%，而 GiGPO 在长程趋于平台
- 收益来自策略优化而非里程碑模仿：用 oracle 轨迹做 SFT（行为克隆）仅 43%，BEACON 达 91.4%
- 收敛速度约为 GRPO 的 2.4 倍（第 50 迭代达 60% 成功率，GRPO 要到第 120 迭代），策略熵平滑下降
- BEACON 信用集中率 CCR=0.84 在各方法中最低但性能最好——保留渐进式梯度信用优于激进地只奖励里程碑动作

## 意义

BEACON 直接命中长程 RL 的信用分配病根：任务视野越长，里程碑式信用分配的优势越明显。它把「任务本身有组合结构，信用分配也应该有结构」这一直觉落地为可训练、仅两个超参数的简单框架，为长程语言智能体的 RL 训练提供了 GRPO 的可直接替代方案。^[raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026.md]

## 深度分析

### 为什么里程碑是「免费」的结构信号

BEACON 最聪明的设计决策，是坚持让里程碑指示函数 Φ 完全由环境给出——ALFWorld 用物体持有/加热状态谓词，WebShop 用购物检查点，ScienceWorld 直接读取 `subgoal_completed` 接口——而不训练任何额外的奖励模型。这避免了一类常见陷阱：学习的奖励模型会被策略「钻空子」（reward hacking），且每个环境都要单独调参。三个环境共用一套训练管线、批次数据里的里程碑字段自动路由，说明这种可验证的进展信号（verifiable progress signal）在三类异构环境中都天然存在，只是此前没人把它接进信用分配。代价是适用范围受限于环境能否枚举子目标：对完全没有中间信号的自由形式任务，BEACON 退化为普通 GRPO。

### 双尺度优势如何同时保住全局与局部

单独看三个步骤各自不新——轨迹分段、衰减塑形、分组比较在 reward shaping 文献里都有原型——组合起来才有效。关键在双尺度优势 `Â = A_traj + λ · A_seg` 的分工：轨迹级优势保留 GRPO 的组内相对比较，确保最终成败仍是主导信号；段级优势只在「到达同一里程碑」的轨迹间比较，这等价于一个条件化的对照组——同样走到加热苹果这一步的轨迹，谁在段内更高效。这样后面段的失败波动被隔离在段边界之外，不会污染前面动作的估计。实验里信用集中率 CCR=0.84 最低但性能最好，恰好印证了这一点：渐进式衰减把信用摊在整段上，比激进地只奖励里程碑动作保留了更多「怎么走过去」的梯度信息。

### 收益来自优化而非模仿的证据链

「效果好是不是因为变相模仿了里程碑轨迹」是这类方法最常受的质疑，作者的对照实验回应得很干净：直接在 oracle 轨迹上做 SFT（行为克隆）只有 43%，BEACON 达到 91.4%。差距近 50 个点说明模仿必经节点远远不够——节点之间的探索质量才是性能来源。配合训练动态看：BEACON 第 50 迭代达 60% 成功率（GRPO 需 120 迭代），策略熵平滑下降而非崩塌式收敛，一致的里程碑反馈带来的是稳定的逐步精炼。零优势比例从约 55% 降到约 10% 则解释了收敛快的机制：原本方差为零整组报废的失败 rollout，现在大部分携带非零梯度回到训练。

### 局限与开放问题

任务长度单调扩大的领先幅度（短程 +26.2% → 长程 +73.6%）暗示方法与问题高度匹配，但依赖环境给定 Φ 也划定了边界：里程碑必须可枚举、可判定，稀疏分布的细粒度子目标或环境不暴露中间状态的场景无法受益。两个超参数（γ、λ）虽少，段级权重 λ 如何随任务长度自适应仍是开放问题；里程碑粒度的选择（每步算一个还是每个子目标算一个）文章未系统消融。此外 GiGPO 在长程上趋于平台而 BEACON 持续扩大领先，说明按状态分组与按里程碑分组捕捉的是不同的结构——前者是状态等价类，后者是任务进度结构，后者的信息量在长程下更大。

## 实践启示

1. **先翻环境文档再动手**：接入任何新环境前，先找它自带的进度信号——谓词状态、检查点接口、subgoal 事件。ScienceWorld 式的 `subgoal_completed` 接口比训练奖励模型便宜得多，也更难被 hack。
2. **部分成功不该被一票否决**：如果你的 RL 训练里大量 rollout 因终局失败而零梯度（零优势比例 > 40% 是危险信号），优先考虑按任务结构切分信号，而不是加大 rollout 数量硬砸样本。
3. **两个超参数起步**：先只引入 γ 和 λ，别急着加课程学习或奖励模型。BEACON 证明结构化信用分配的主要收益来自「利用任务结构」本身，实现复杂度可以很低。
4. **用 SFT 对照排除模仿假象**：评估类似方法时，跑一组 oracle 轨迹行为克隆作基线——若方法不超过它，收益很可能来自轨迹模仿而非策略优化，泛化性存疑。
5. **任务越长越该用**：相对增益随视野单调扩大（+26.2% → +73.6%），在多轮工具调用、跨应用工作流这类 30 步以上任务上，轨迹级稀疏奖励基本不可用，里程碑式分配是默认选项。
6. **保留渐进信用，别只奖关键步**：CCR=0.84 最低却性能最好的结果提醒我们，只给里程碑动作满额奖励会丢掉段内路径质量的信息；衰减塑形（越近越高、逐步递减）是更稳的折中。

## 相关实体

- → [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026|Agentic RL 六框架实践地图]]
- → [[entities/agent训练最容易踩的坑credit-assignment-is-all-you-need|信用分配 Credit Assignment]]
- → [[entities/seed-self-evolving-opd-long-horizon-agent-rl-tsinghua-zju-2026|SEED 自进化 OPD 长程 Agent RL]]
- → [[entities/towards-long-horizon-agents-survey-mozi-space|Long-Horizon Agent 综述]]
- → [[raw/articles/beacon-milestone-guided-policy-learning-long-horizon-agent-zju-2026|原文存档]]
