---
title: "NanoGPT Speedrun MUDD 优化：彩云科技连续两次刷新训练速度世界纪录"
created: 2026-09-02
updated: 2026-09-11
type: entity
tags: [training-optimization, gpu-kernel, nanoGPT, speedrun, architecture, skip-connections, MUDD, muon, pretraining]
sources: [raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录]
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# NanoGPT Speedrun MUDD 优化：彩云科技连续两次刷新训练速度世界纪录

## 摘要

NanoGPT Speedrun 是公开训练竞速项目：所有人用固定 8 张 H100，把 GPT-2 Small 规模模型训到 FineWeb 验证集 loss ≤ 3.28，再比拼完整训练耗时；架构、优化器与自写 Kernel 都能改，但数据不能换、最终效果不能降。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

彩云科技基础模型算法团队连续两次刷新 Track 1 纪录：第 81 次提交 MUDD Skip Connections 把 84.36 秒压到 81.78 秒，一个多月后第 85 次提交 MUDD Gates + DC MHA 又从 79.2 秒推进到 76.3 秒。两刀分别落在层间与层内：前者让每个 token 动态选择回取哪些历史层，后者让注意力头按输入动态组合。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

## 核心要点

- **基准即硬约束**：固定 8×H100、固定 FineWeb 数据、固定 loss ≤ 3.28，唯一变量是端到端墙钟时间，"省步数"与"跑得快"必须同时成立。
- **榜单是技术出道场**：Muon 最早在此验证并大幅刷榜，后被 Kimi 扩展为 MuonClip 用于万亿参数 Kimi K2；DeepSeek 的 Engram 条件记忆也被榜单吸收为 Bigram Hash Embedding。
- **两次提交**：#81 MUDD Skip Connections（84.36→81.78 秒，约 -3.1%）优化层间信息流动；#85 MUDD Gates + DC MHA（79.2→76.3 秒，约 -3.7%）优化层内注意力头协作。
- **MUDD 机制**：依当前隐藏状态生成动态权重，不受 Softmax 归一化约束，可直接给出正负系数以放大、抑制或抵消历史层信息；完整版为 Q、K、V、Residual 四类信号分别配路。
- **同方向对照**：Kimi AttnRes 用 Softmax 选层（Block 分组降开销）、字节 Seed 的 Partitioned Hyper-Connections 扩多流信息（#73 缩短约 0.66 秒，但代码版本不同、非严格对比）；维护者 Larry Dial 称该提交"easily the most advanced in quite a while"。

## 深度分析

### 基准即裁判：墙钟计分如何决定技术取舍

墙钟计分决定了取舍方向：任何新模块都要自己挣回运行成本，省了几十步但自身很慢的模块反而拖长时间，收益必须以净值为准。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md] 也正因约束完整可复现，榜单自 2025 年 10 月开记、经 80 多次提交后于 2026 年 4 月近乎躺平，剩余收益要靠几百行 Kernel 抠零点几秒——它同时是拉力赛、竞技场与实验室。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

### 残差拓扑与信号传播：改连接为何能省真实秒数

传统残差是一条事先修好的固定通道，无论 token 在处理数学题、代码还是反讽都走同一路线，信息不断累加；深层堆叠时各层信息逐渐混在一起、单层贡献被稀释，模型无力判断该保留哪一层、哪一路该加强或压低。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md] 动态连接改变的是信号回取路径的深度与选择性——不同 token 需求本就不同（有的要回头取词法信息，有的要最近几层的推理结果），装上"动态电梯"后梯度可沿更短、更相关的跨层路径回传，前向信号也不再被无差别稀释，同一门槛于是能在更少步数内达到，直接换算成墙钟秒数。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

### 两刀的技术内核：MUDD 动态路由与 Lightweight DC

MUDD（Multiway Dynamic Dense Connections，多路动态稠密连接）依当前 token 与内部状态生成权重，自行决定从哪些历史层取回信息、以多大权重参与计算；与注意力打分不同，它不受 Softmax 约束，可直接生成正负连接系数以放大、抑制或抵消信息，完整版还把这套路由扩到 Q、K、V、Residual 四路，论文报告仅增约 0.23% 参数与 0.4% 计算量即等效 1.8–2.4 倍算力。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md] 擂台不接受论文原样搬运：彩云只拆出最有价值的部分做低开销 MUDD Skip Connections，依隐藏状态生成动态权重改善跨层传递、把额外计算压到最低，从而把 84.36 秒砍到 81.78 秒。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

第二刀源自 2024 年 ICML Oral 的 DCMHA（可动态组合的多头注意力）。传统多头注意力让各 Head 独立算注意力图、到输出才拼接，缺少按输入动态协作的机制，易产生 Head 冗余；DCMHA 用 Compose 模块对注意力分数与权重做跨 Head 变换，按输入决定哪些 Head 协同、哪些信息加强或重组。竞赛版 Lightweight DC 复用原有 Q/K 投影，只额外算一个 112 token 局部注意力窗口，Softmax 后把多个 Head 的注意力图合成为一张共享图，再以各自 Head Scale 作用于对应 Value，靠复用投影、缩短窗口与专用 Kernel 压住开销，使训练步数下降约 7%。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

### 坐标系与外推边界：AttnRes、Hyper-Connections 与"小透明"反超

Kimi 2026 年 3 月的 AttnRes 同样认为传统残差会让各层信息混在一起、越深单层贡献越被稀释，做法是逐 token 对历史层打分并经 Softmax 选择性回取（Block AttnRes 再按块分组以降显存与通信）；MUDD 的关键差异是不受 Softmax 约束、能给出正负系数，并把动态路由从残差流扩到 Q/K/V 四路，而字节 Seed 的 Hyper-Connections 把单一隐藏状态扩成多条可交互信息流以动态重组层间连接。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

小团队能反超，是因为墙钟基准考核工程判断力而非集群规模：预算怎么花、开销如何用 Kernel 抹平、论文模块哪些该拆该裁，都不需要几千张卡才能试。但外推必须谨慎——测试对象仍是 GPT-2 Small，3% 的提速不代表放大到百亿、千亿参数后同样成立；Muon 从榜单走向 Kimi K2 证明这条路走得通，走通的方式却是先在社区代码库反复验证、再由大团队按规模重新校准。彩云两项纪录已被后来者继续刷新，而竞速的意义本就不是让名字留在第一名，是让有效想法进入公共代码库。^[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录.md]

## 实践启示

1. **把提速折算成端到端净值再取舍**：论文里以步数或理论算力衡量的收益必须除以模块自身的运行开销，只有净值为正才在真实训练中成立；彩云两刀的胜利来自裁剪而非照搬。
2. **优先审视残差拓扑**：参数与数据预算固定时，改跨层回取路径与选择性可在几乎不增参数的前提下改善梯度回传与信号保真，比单纯加宽加深更划算。
3. **允许连接系数取负**：Softmax 只能做加权平均式取用，带正负号的动态系数还能表达抑制与抵消，为层间干预提供更丰富手段。
4. **用固定硬件+固定数据的公开基准做体检**：把架构创新的有效性从论文表格变成可复现的墙钟数字，是低成本验证的入口。
5. **迁移前重做规模与并行校准**：张量/流水并行的切分与通信会改变 Kernel 与带宽开销结构，小模型上的相对收益可能在大规模训练中被通信与显存压力吃掉。

## 相关实体

- [[entities/kimi-attention-residuals-prenorm-dilution-block-attnres|Kimi AttnRes：残差稀释与 Block AttnRes]]
- [[entities/xhc-expanded-hyper-connections|Expanded Hyper-Connections]]
- [[entities/kimi-k2-5-architecture-innovation-moonshot-2026|Kimi K2.5 架构创新]]
- [[concepts/transformer-architecture|Transformer 架构]]

→ [[raw/articles/一个国产ai小透明连续两次刷新nanogpt-speedrun世界纪录|原文存档]]
