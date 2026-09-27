---
title: "Colibrì：SSD 当显存的 CPU 分层 MoE 推理框架（32k Star）"
created: 2026-09-28
updated: 2026-09-28
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

## 关联

- [[concepts/inference-optimization|推理优化]] — 同主题其他路线（KV cache/量化/投机解码）
- [[entities/deploying-kimi-k3-on-aws|Kimi K3 on AWS]] — 同模型的云端部署路线对照
- [[entities/claude-opus-5-发布比-fable-5-强仅一半价格|Claude Opus 5 发布]] — 模型成本坐标参照

→ [[raw/articles/笔记本跑7000亿参数glm无gpu也行-ssd当显存用火爆github|原文存档]]
