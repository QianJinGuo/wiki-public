---

title: "A from-scratch derivation of REINFORCE for language models"
created: 2026-09-29
updated: 2026-11-26
type: entity
tags: ["reinforcement-learning", "policy-gradient", "reinforce", "rlvr", "grpo", "training", "tutorial-deep"]
sources: [raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero]
confidence: 0.7
---

# REINFORCE 从零推导（Tyler Romero）

## 定位

一篇数学推导型教学深文：**PPO 到 GRPO 等当代 LLM RL 算法都是 policy gradient 这一个思想的精化**，文章用单一 prompt（"What is 17 × 24?"）贯穿全程，从 next-token 概率分布一步步推导出使正确答案概率上升的梯度。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md]

## 推导主线

1. **目标函数**：LLM 生成即一段 episode——每个 token 是一个 action，奖励只在结尾由 verifier 给出（RLVR：数学题有标准答案、代码任务有单测），总奖励即最终奖励。目标是最大化期望奖励 J(θ)。
2. **不可直接求导的两个障碍**：(a) 补全空间爆炸——15 万词表下 100-token 补全的枚举不可行；(b) 离散采样不可微——token ID 对参数无平滑梯度，且奖励来自 verifier/单测/人，无 autograd 路径。
3. **REINFORCE 技巧**：核心恒等式 `∇θ pθ = pθ ∇θ log pθ`（log-derivative trick），把期望的梯度改写为"采样补全上 ∇log pθ × R 的平均"——将不可微问题转化为可估计的采样平均。
4. **与 PPO/GRPO 的关系**：三者共享期望奖励目标，但 PPO/GRPO 还**改变更新本身**（clipping/reweighting 保持训练稳定）——这是"精化"而非"另起炉灶"。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md]

## wiki 中的定位

概念链路：[[concepts/rlvr-reinforcement-learning-verified-reasoning]]（RLVR 奖励面）→ 本文（policy gradient 数学面）→ [[concepts/grpo-policy-optimization-2026]] 与 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]（算法演化面）。对需要从第一性原理理解 RLHF/RLVR 训练的读者，这是比论文更可及的入口。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md]

## 深度分析

### log-derivative 技巧：把不可微问题变成可采样估计

推导的核心是一行恒等式 `∇θ pθ = pθ ∇θ log pθ`：期望奖励的梯度本要对全部补全求和，但 15 万词表下 100-token 补全根本无法枚举，且离散采样与来自 verifier 的奖励都没有 autograd 路径。恒等式把问题改写成"从策略采样 N 个补全、以各自奖励 R(yi) 加权 ∇log pθ(yi) 再平均"的蒙特卡洛估计，即 score function 估计器；它无偏——多批平均等于真实梯度——但单批可能指向很偏。文章最有用的心智模型是：policy gradient 就是"对自己的样本做 SFT、按奖励加权"——正确补全被推高，错误补全从不被直接压低，只是丢失概率份额；这也是 rejection-sampling fine-tuning（RFT）能单独成立的原因。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:35-48] ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:61-74]

### 仅终端 verifier 奖励：为什么天然是 RLVR 形状

与一般 RL"每步动作都有奖励、按折扣累计"不同，LLM 生成一个补全就是一段 episode：每个 token 是一个 action，唯一奖励在结尾由 verifier 给出（数学题有标准答案、代码任务有单测），总奖励就等于最终奖励。这带来严重的 credit assignment 问题：奖励只回答"整个补全对不对"，每个位置拿到同一个标量——A 的正确开头得不到任何分数，B 的填充 token 却分走全额奖励；token 级区分只能来自 score 本身（模型已经很确定的 token，其 score 趋于零）。引入 value function 或逐步打分器给单个 token 分配信用，正是沿着这条裂缝展开的后续研究方向；这也是 [[entities/agent训练最容易踩的坑credit-assignment-is-all-you-need]] 的主题。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:19-34] ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:87-91]

### 数值与稳定性：REINFORCE 的坑，PPO 后来补的课

估计器无偏但方差大：所有补全奖励相同的极端情形下真实梯度为零，每个样本却仍在随机推高自己的 log 概率。减去 baseline 不改变期望梯度，却大幅压方差，标准选择是按 prompt 估计期望奖励——不同 prompt 难度差异巨大，按 prompt 中心化去掉了最大的噪声源。更隐蔽的坑是 on-policy 假设：log-derivative 改写与零均值 score 两个恒等式都要求"采样分布与被求导模型是同一个分布"，而实际训练中补全由独立推理引擎生成——数值精度不同、异步架构下权重滞后——分布稍有偏差，期望 score 不再为零，减 baseline 就从无损降噪变成引入偏差。PPO/DAPO 等正是在这个估计器上加 clipping、masking、reweighting 来稳住训练。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:99-114] ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:125-131] ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:34]

### GRPO 继承了什么、改了什么

继承：GRPO 仍是同一个 score 估计器，baseline 思想保留——每 prompt 采样一组补全、用组内平均奖励做 baseline，正确补全得正 advantage 被强化，错误补全得负 advantage 被抑制，全对或全错的 prompt 贡献为零。改变有三处：(1) 用组均值取代 PPO 需要与策略同规模训练的 value network，省掉一个模型；(2) 额外除以组内奖励标准差——文章引 Dr.GRPO 的批评指出这种归一化会给简单/困难 prompt 引入偏差，故正文只用均值中心化；(3) 组均值包含补全自身奖励，期望梯度只是被等比缩小，相当于略小的步长；leave-one-out baseline 是更干净的替代。整个 GRPO 式 REINFORCE 在 PyTorch 里只有几行，advantage 是不携带梯度的常数。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:108-124]

## 实践启示

1. **用"奖励加权的 SFT"心智模型理解 RLVR 训练**：REINFORCE 与 RFT/拒绝采样微调在数学上同源——都是对自己采样的正例做监督微调；理解了这一点，SFT 到 RL 的过渡就不再是范式跳跃。参见 [[concepts/reinforcement-fine-tuning-rft]]。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:71-74]
2. **训练信号异常时先查 baseline**：若大量 prompt 组内全对或全错，其 advantage 全零、对梯度毫无贡献；先按 prompt 估计期望奖励再中心化，是成本最低的降噪手段。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:105-111] ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:118]
3. **不要默认组内标准差归一化无害**：GRPO 的 std 除法被指会给易/难 prompt 引入偏差；复现时明确自己用的是 mean-centering、leave-one-out 还是 full GRPO，三者梯度并不等价。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:112-114]
4. **把 on-policy 当成工程问题管理**：训练框架与推理引擎的数值精度差异、异步权重滞后都会破坏零均值恒等式；排查 RL 训练发散时，先验证"生成分布 = 训练分布"这一前提。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:125-131]
5. **注意 loss 归一化分母的隐含加权**：按 batch 总补全 token 数归一，分母随采样长度波动，会隐式按长度重加权补全；要精确对应推导式，应按补全条数平均。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:122-124]
6. **从最小实现建立直觉**：组中心化 advantage 的 REINFORCE 只有几行 PyTorch，先跑通它再看 PPO/DAPO 的 clipping 与 masking，比直接读论文高效。 ^[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero.md:119-122]

## 关联

- → [[raw/articles/reinforce-from-scratch-policy-gradient-llm-tyler-romero|原文存档]]
- [[concepts/rlvr-reinforcement-learning-verified-reasoning]] — RLVR 奖励函数与 verifier 概念面
- [[concepts/grpo-policy-optimization-2026]] — GRPO 对 REINFORCE 的精化
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] — PPO/DPO/GRPO 算法演化谱系

