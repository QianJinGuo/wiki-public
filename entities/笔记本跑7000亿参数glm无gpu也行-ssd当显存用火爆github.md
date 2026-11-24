---
title: "Colibrì：SSD 当显存的 CPU 分层 MoE 推理框架（32k Star）"
created: 2026-09-28
updated: 2026-11-14
type: entity
tags: [inference, moe, cpu-inference, ssd-offload, quantization, open-source]
sources: [raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github]
confidence: 0.75
provenance_state: extracted
---

# Colibrì：SSD 当显存的 CPU 分层 MoE 推理框架（32k Star）

> GitHub 开源项目 Colibrì：纯 C 实现、零引擎依赖的分层推理框架，25GB RAM 笔记本跑 744B GLM-5.2，32GB 内存挑战 2.8T Kimi K3，GPU 非必需。

## 核心思路：把「先装进去再计算」倒过来

MoE 架构留下的操作空间：GLM-5.2 总参数 744B，但每 token 实际只激活约 40B——绝大多数专家当前根本用不上，不必常驻昂贵的高速内存。Colibrì 把模型拆成**常驻**与**临时调用**两部分：Attention/Embedding/共享专家等 Dense 部分（约 17B，int4 后 9.9GB）常驻 RAM；19456 个路由专家（int4 后约 370GB）全部放 NVMe SSD。生成 token 时 Router 先选出需要的专家，未命中高速内存的部分才从 SSD 读取。作者 JustVugg 将其类比为**针对模型权重的 JIT**：不把 744B 看作必须常驻的整体，而是变成可在 SSD/RAM/VRAM 间动态调度的数据。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

## AI Memory Multitiering：越常用的专家住得越近

冷缓存时 GLM-5.2 在 12 核 CPU + 25GB RAM 开发机上只有 0.05-0.1 token/s——能跑但有亿点慢。提速靠三层协同：

- **LRU 缓存**：最近调用的专家优先留 RAM，命中即免跑 SSD。
- **热度统计**：运行中记录专家使用次数，高频「常客」获更高缓存优先级，尽量留在快存储层。
- **预取**：相邻层专家路由相关性明显，提前一层预测专家的可预测性达 **71.6%**——当前层计算时后台预读下一层可能需要的专家，计算与 SSD I/O 重叠。
- **双 SSD 并行**：支持放第二份模型副本，把专家读取分摊到两块盘并行利用带宽。

关键设计原则：**专家放在哪里只决定速度，不改变模型本身**——Router 的选择不因存储位置改变，权重精度完全相同，不会偷偷少算专家。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

## 性能坐标与模型覆盖

- 128GB RAM 纯 CPU 桌面机：约 1.8 token/s。
- 6×RTX 5090 全专家常驻：5.8-6.8 token/s。
- 覆盖 9 个模型家族：Qwen3.8-Flash-Next / Qwen3.6 / OLMoE（小）→ DeepSeek V4 Flash 284B → GLM-5.2/5.3 744B → GLM-5.3-Flash 321B → Thinking Machines Inkling 975B → **Kimi K3 2.8T 总参/104B 激活**（权重约 1.6TB，RAM 32GB 起跑）。

工程形态：几百 KB 的程序 + 模型文件即可运行；Linux/macOS/Windows 预编译；每模型一套 C 适配文件但 IO/缓存/tokenizer 公共组件复用同一核心。Web Dashboard 实时显示各层专家分布，「Brain」页面可视化 GLM-5.2 全部 19456 个专家的存储层归属与调用热度。项目地址：github.com/JustVugg/colibri。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

## 洞察

Colibrì 展示的 **AI memory multitiering** 是推理优化的第三条路线：不同于 [[concepts/inference-optimization|推理优化]] 里的 KV cache 管理或量化压缩，它把存储层级当作一等公民调度对象，本质是给 MoE 模型做权重分级调度系统——容量最大最慢的 NVMe 兜底，RAM 缓存常用专家，VRAM 接住最适合高速的部分。与 [[entities/deploying-kimi-k3-on-aws|Kimi K3 on AWS 部署]] 代表的「专业服务器部署」路线形成两极：跑前沿大模型不一定非要用机房里的专业服务器。「Tiny engine, immense model」——MoE 的稀疏激活特性 + SSD 带宽增长，使消费级硬件跑万亿参数模型成为工程可行的选项。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

## 深度分析

### MoE 稀疏激活是 CPU 推理可行的根本前提

Colibrì 能成立的因果链要从 MoE 的条件计算讲起：GLM-5.2 总参数 744B，但 Router 每个 token 只激活约 40B（约 5.4%），这意味着绝大多数专家权重在任何给定时刻都是「死重」。传统稠密模型必须把全部权重视为常驻资源，而 MoE 把「模型大小」和「每步计算量」解耦了——前者只决定存储需求，后者才决定算力需求。Colibrì 正是抓住这个解耦做文章：Dense 部分（Attention/Embedding/共享专家，约 17B）访问频率 100%，必须常驻 RAM（int4 后 9.9GB）；19456 个路由专家（int4 约 370GB）访问频率高度稀疏，可以沉到 NVMe SSD 兜底。作者 JustVugg 把这类比作针对权重的 JIT——不是提前把整个程序装进内存，而是观察真实执行路径、按需加载热点。关键在于这不是近似或降级：专家放在哪里只决定速度，不改变 Router 的选择和权重精度，模型行为与全量常驻时完全一致。[[concepts/moe-mixture-of-experts-2025|MoE 架构]] 的稀疏性把「内存墙」从硬约束变成了可以用 I/O 调度换算力的软约束。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

### SSD 当 VRAM 的性能账：冷启动 0.05 token/s 到 1.8 token/s 差了 20-30 倍

分层调度的收益可以用实测数据倒推：12 核 CPU + 25GB RAM 冷缓存时只有 0.05-0.1 token/s——每个 token 几乎都要从 SSD 现读几十上百个专家，I/O 完全主导延迟；同样机器跑热之后（LRU + 热度统计生效），常用专家驻留 RAM，速度上限由 CPU 算力和缓存命中率共同决定；128GB RAM 纯 CPU 机器可达约 1.8 token/s，说明 DRAM 容量每翻一倍、可缓存的专家比例就上一个台阶。换算成带宽逻辑：int4 后每层需读的专家总量若全部走 PCIe/NVMe（顺序读约 3-7GB/s），与 CPU int4 计算吞吐相比是数量级差距，因此性能工程的核心不是「读得快」而是「少读」——命中率每提升一个百分点，端到端延迟近乎线性下降。这也解释了为什么 6×RTX 5090 全专家常驻只能到 5.8-6.8 token/s：当所有权重都在 VRAM 时，瓶颈回到 CPU/软件栈本身的解码效率，硬件堆到顶收益也封顶了。冷热两级数据之间 20-30 倍的差距，就是这套调度系统全部价值的量化标尺。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

### 专家局部性与预取：把串行的 SSD I/O 重叠进计算窗口

这套系统真正的技术亮点是对「专家路由局部性」的挖掘。文章披露相邻层之间的专家路由存在明显相关性——当前层被点名的专家，下一层有 71.6% 的概率再次被选中（提前一层预测的可预测性）。这不是巧合而是 Transformer 的结构特性：同一段语义内容在相邻层往往由同一批专家持续处理，路由呈现「热点连续」的时序模式。Colibrì 利用这一点做流水线化：当前层计算的同时，后台预读下一层大概率需要的专家，把 SSD I/O 藏进计算窗口，串行变并行。配合 LRU 缓存（最近用过的专家留 RAM）、运行时热度统计（高频「常客」获更高缓存优先级）、双 SSD 副本并行分摊读取带宽，四招合起来构成一个完整的存储层调度栈。这本质上是把操作系统的虚拟内存页调度思想（工作集、预取、多级缓存）迁移到了模型权重域，只是「页」变成了专家、「访问模式」变成了 Router 决策。71.6% 这个数字也说明预取的上限：仍有近三成预测失败要现场读盘，这就是冷热波动和长尾延迟的来源。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

### 与既有推理优化路线的关系：第三条轴，不是替代品

把 Colibrì 放回 [[concepts/inference-optimization|推理优化]] 的整体地图里看，它的位置很清晰：KV cache 管理优化的是「序列维度的重复计算」，量化压缩优化的是「每个参数的比特成本」，而 AI memory multitiering 优化的是「参数在存储层级中的驻留位置」——三者作用在不同维度上，正交可叠加。它也不与 GPU 路线对立：VRAM 在这套体系里只是最高一层缓存，有 GPU 就让 VRAM 接住最热的部分，没 GPU 就 RAM 顶上，调度逻辑不变。真正被改变的是可行性边界——Kimi K3 2.8T 总参、权重约 1.6TB，过去意味着只能进数据中心，现在 RAM 32GB 起跑即可离线推理。代价同样要认清：吞吐量级（个位数 token/s）决定了它适合离线批处理、本地实验、隐私敏感场景，不适合在线服务的高并发。它证明的不是「CPU 比 GPU 快」，而是一个更普适的工程命题：当模型的计算密度足够稀疏时，存储层级本身可以成为一种可编程的推理资源。与 [[entities/deploying-kimi-k3-on-aws|Kimi K3 on AWS 部署]] 的专业服务器路线对照，消费级硬件路线补上了「个人可拥有」这一端。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

## 实践启示

1. **先算激活比再定部署方案**：评估一个 MoE 模型能否在本地跑，别看总参数，看激活参数与 Dense 部分大小——Dense 部分（GLM-5.2 约 17B/int4 9.9GB）必须常驻 RAM，这是硬性地板；路由专家总量只决定 SSD 占用，不决定内存门槛。Kimi K3 1.6TB 权重 32GB RAM 起跑就是这条规律的极端案例。
2. **SSD 带宽和容量同等重要，双盘值得配**：分层推理的性能瓶颈在专家读取 I/O，NVMe 顺序读速度直接决定冷路径延迟；机器有两块 SSD 时启用第二份模型副本并行分摊读取，是成本最低的提速手段。
3. **让缓存「跑热」再用，接受冷启动慢**：0.05-0.1 token/s 的冷缓存速度是正常现象而非故障——LRU 和热度统计需要运行时间积累。把预热纳入使用预期：跑长会话/批任务，而不是频繁重启短会话。
4. **把预取前提当可测指标，别当默认假设**：71.6% 的相邻层路由可预测性是这套方案的隐含赌注。换模型家族时这个数字会变（路由熵不同的模型局部性不同），用 Web Dashboard 的 Brain 页面观察专家驻留分布与调用热度，验证当前模型是否吃这套调度。
5. **明确适用场景：离线与隐私，不是在线服务**：个位数 token/s 的吞吐定位是本地实验、离线批处理、隐私敏感的单人使用，不是高并发生产 API。生产部署仍应看专业推理引擎路线。
6. **精度与路由零妥协是可用性底线**：分层调度不应改变模型行为——检查所用框架是否保证「专家位置只影响速度不影响计算」（Router 选择不变、权重精度相同），任何为了省内存而偷偷丢专家或换路由的实现都会产生质量回归。^[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github.md]

## 关联

- [[concepts/inference-optimization|推理优化]] — 同主题其他路线（KV cache/量化/投机解码）
- [[entities/deploying-kimi-k3-on-aws|Kimi K3 on AWS]] — 同模型的云端部署路线对照
- [[entities/claude-opus-5-发布比-fable-5-强仅一半价格|Claude Opus 5 发布]] — 模型成本坐标参照

→ [[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github|原文存档]]
