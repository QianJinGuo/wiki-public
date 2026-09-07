---
title: LoopWM (Looped World Models)
type: entity
tags: [world-model, looped-transformer, latent-dynamics, reinforcement-learning, parameter-sharing, test-time-compute]
created: 2026-06-25
updated: 2026-09-07
confidence: 0.8
provenance_state: merged
sources:
  - raw/articles/loopwm-looped-world-models
  - raw/articles/2026-06-30-AI圈刚开始谈Loop-Engineering-两位95后博士已经盯上了人类闭环-机器之心
  - raw/articles/loopwm-loop-world-models-2026-07-01
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# LoopWM (Looped World Models)

LoopWM 是 FaceMind Research Asia 提出的循环 Transformer 世界模型，核心思想是将同一组 Transformer 块在潜空间中反复调用来推演环境状态，用迭代深度替代参数深度。约 10 亿参数在 ScienceWorld 上达到 68.4% EM，超越 Claude Opus 4.6（47.2%）。^[raw/articles/loopwm-looped-world-models.md]

- **论文**: [Looped World Models (arXiv 2606.18208)](https://arxiv.org/abs/2606.18208)
- **机构**: FaceMind Research Asia

## 核心问题

世界模型越往长程仿真走，两个问题越突出：^[raw/articles/loopwm-looped-world-models.md]

1. **误差累积**：每步预测偏一点，多滚几步轨迹完全变样
2. **固定深度浪费**：主流架构对「简单匀速运动」和「多体碰撞」分配的计算量几乎一样，不合理

堆深度可以缓解但参数量、显存、推理延迟一起涨。循环 Transformer 用共享参数迭代来解决这个矛盾。^[raw/articles/loopwm-looped-world-models.md]


## 架构：四模块 + 循环核心

^[raw/articles/loopwm-looped-world-models.md]

1. **观测编码器**：原始观测 → 潜嵌入
2. **动作嵌入器**：离散/连续动作 → 同一潜空间
3. **循环动力学核心**（Prelude–Recurrent–Coda 三段式）：
   - **Prelude**：观测-动作拼接经 Transformer + LayerNorm 压成条件信号（先做 LN 防止循环放大幅度）
   - **Recurrent**：参数共享 Transformer 多次迭代 refine 潜状态，循环次数增加但参数不增加
   - **Coda**：独立参数 Transformer 读出最终潜状态
4. **预测头**：轻量 MLP 解码观测/奖励/终止信号

形成双循环：内循环单步转移内反复 refine，外循环沿环境时间前进。^[raw/articles/2026-06-30-AI圈刚开始谈Loop-Engineering-两位95后博士已经盯上了人类闭环-机器之心.md]


## 关键设计

### 谱稳定性约束

对状态保留矩阵做谱范数约束，保证对角元严格为负 → 指数映射后落在 (0,1) → 对角收缩矩阵。约束在训练中按构造成立，不靠梯度裁剪。^[raw/articles/loopwm-looped-world-models.md]

### 自适应深度（早退门）

推理时轻量退出门在每个循环步输出概率，超过阈值即停。对标 100 层固定深度，简单轨迹只需 1 次循环，FLOPs 降 ~25×。测试时可设更大循环数 → test-time compute scaling。^[raw/articles/loopwm-looped-world-models.md]

### 可变深度训练

循环次数从 Poisson 分布采样，每条序列独立。消除大部分 loss spike，反向传播只回传到采样步数。^[raw/articles/loopwm-looped-world-models.md]

### 延迟解码 (Deferred Decoding)

给定动作序列，只在潜空间反复注入动作推进，最后一步才调用一次解码器。有效计算深度 = 多次共享参数 Transformer 调用，解码器只跑 1 遍。^[raw/articles/loopwm-looped-world-models.md]

配合两道约束防漂移：潜一致性损失（冻结编码器对齐）+ 谱收缩预算监控。defer 长度从 1 起步渐进拉长。^[raw/articles/loopwm-loop-world-models-2026-07-01.md]


## 实验结果

### ScienceWorld（全面领先）

^[raw/articles/loopwm-looped-world-models.md]

| 模型 | EM | F1 | BLEU | Entity |
|------|-----|------|------|--------|
| LoopWM (~1B) | **68.4%** | **85.3%** | **80.7%** | 83.9% |
| Claude Opus 4.6 | 47.2% | 72.8% | 64.4% | 72.3% |
| Gemini 3 Flash | 30.8% | 68.9% | 51.1% | **73.8%** |
| Qwen 3.5 Flash | 10.0% | 46.9% | 26.7% | 63.0% |

### ALFWorld

EM 略低于 Claude（51.6% vs 53.0%），但 BLEU-4 四模型最高（71.6%）。Entity 偏低是短板。^[raw/articles/loopwm-looped-world-models.md]

### 延迟解码步数效应

动作链越长优势越明显。Step 5 EM 相对 Gemini 提升 +113.8%。单任务 Boil Step 4 达 +700.9%，Melt Step 5 达 +557.7%。LoopWM 自身 Step 1–5 的 EM 在 67.2%–68.6% 窄幅波动，对 rollout 深度有鲁棒性。^[raw/articles/loopwm-looped-world-models.md]


## 意义与局限

**核心洞察**：同样 10 亿参数，把预算从「加宽网络」改成「潜空间多迭代几轮」，ScienceWorld EM 拉到 68.4%。迭代深度是继参数量、数据量之后第三个值得调的缩放维度，与 LLM test-time compute scaling 同源。^[raw/articles/loopwm-looped-world-models.md]

**局限**：
- 延迟解码只是循环收益的一个切面，循环本身增益需更细分解
- 连续视觉环境验证尚不完整
- 与 RSSM、自回归视频 token、扩散式世界模型的边界不清
- 完整 scaling law 缺硬图
- 训练依赖渐进长度、Poisson 深度采样等工程配方

## 第 2 来源 — 机器之心：AI圈刚开始谈Loop Engineering，两位95后博士已经盯上了人类闭环数据

v×c=56 的量子位文章（第 3 来源）与机器之心本篇同为大众科技媒体对 LoopWM 论文的二次解读，但切入角度不同。^[raw/articles/loopwm-loop-world-models-2026-07-01.md]

机器之心本篇（2026-06-30）的独特视角：^[raw/articles/2026-06-30-AI圈刚开始谈Loop-Engineering-两位95后博士已经盯上了人类闭环-机器之心.md]

1. **"人类闭环数据"视角**：强调 FaceMind 不仅做世界模型，还从人类操作数据中提取闭环训练信号，而非只依赖仿真环境
2. **团队背景深挖**：陆弘远（95后清华博士）、韦怡然（前微软/阿里），团队技术路线从端侧全模态 → 世界模型的演进
3. **融资与商业视角**：Pre-A 轮数千万，星连资本领投，360跟投，奇绩创坛参投
4. **具身落地验证**：已在真机机械臂、GUI Agent 环境验证

→ [[raw/articles/2026-06-30-AI圈刚开始谈Loop-Engineering-两位95后博士已经盯上了人类闭环-机器之心|原文存档]]

## 第 3 来源 — 量子位：Loop世界模型论文登顶Hugging Face

量子位本篇（2026-07-01）从 Loop Engineering 业界叙事切入，将 LoopWM 定位为"AI认知层换挡"的关键基础设施。^[raw/articles/loopwm-loop-world-models-2026-07-01.md]

独特贡献：
1. **Loop Engineering → LoopWM 的叙事桥接**：从开发者体验的 Loop Engineering（AI怎么持续干活）自然过渡到世界模型 LoopWM（AI在持续干活时如何理解世界）
2. **技术细节更丰富**：引用了论文中 100× 参数效率、25× 单步推理 FLOPs 减少、两个数量级整体计算节省等具体数据
3. **Adam's Law 外部验证**：陆弘远提出的 Adam's Law 已获 Anthropic 关注和验证
4. **X社区讨论**：引用了海外社区对 shared transformer block、adaptive compute、spectral stability 等技术词的讨论
5. **论文链接**：https://arxiv.org/abs/2606.18208

→ [[raw/articles/loopwm-loop-world-models-2026-07-01|原文存档]]

## 关联

- → [[raw/articles/loopwm-looped-world-models|第 1 来源（原始论文）原文存档]]
- → [[raw/articles/2026-06-30-AI圈刚开始谈Loop-Engineering-两位95后博士已经盯上了人类闭环-机器之心|第 2 来源（机器之心）原文存档]]
- → [[raw/articles/loopwm-loop-world-models-2026-07-01|第 3 来源（量子位）原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

