---
title: "DeepSeek做大 Mega MoE Tri Dao团队加快 SonicMoE 机器之心"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心]
provenance_state: extracted
---

## 摘要

由 FlashAttention 一作 Tri Dao（普林斯顿）与 Ion Stoica（伯克利）领导的联合团队发布了细粒度 MoE 高速训练方案 SonicMoE，能在英伟达 Blackwell GPU 上以峰值吞吐量运行，性能超过 DeepSeek 开源的 DeepGEMM：前向传播平均高出 54%，反向传播平均高出 35%，对 ScatterMoE、MoMoE 等训练框架的加速接近两倍甚至更高。其核心是算法级重设计：一方面通过重排矩阵乘法收缩顺序，避免缓存任何与专家规模成比例的中间张量，使每层激活内存与专家粒度解耦（粒度增加时保持恒定）；另一方面做 IO 感知的算子融合，Gather 融合让数据搬运在矩阵乘法核执行中同步完成，L2 缓存命中率从约 66% 提升至约 75%。^[raw/articles/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心.md]

工程层面，团队开发了统一软件抽象层 QuACK，将所有 MoE 矩阵乘法核统一为"主循环 + 可定制尾声"结构，使硬件特有优化在 Hopper 到 Blackwell 代际迁移时只需局部修改。测试基于 B300 GPU 上六个真实开源 MoE 配置（7B 到 685B，涵盖 OLMoE、Qwen3-235B、DeepSeek V3.2 等）。文章还提及 DeepSeek 此前在 DeepGEMM 中开源了走"大"方向的 Mega MoE，与 SonicMoE 的"快"形成两个不同方向的对照。SonicMoE 已在 GitHub 和 PyPI 开源，支持 H100 与 B200/B300，未来计划支持专家并行、MXFP8/FP4 精度及下一代 Rubin GPU。^[raw/articles/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心.md]

## 关键要点

- 细粒度 MoE 训练面临两堵墙：激活值显存占用与专家粒度成正比、内存访问强度比等参数量稠密模型高 12 倍（以 Qwen3 为例）。
- SonicMoE 将激活内存与专家粒度解耦且无需额外重计算代价，回答了此前业界认为不可兼得的问题。
- dH 反向核即使 HBM 数据流量增加 24%，Tensor Core 利用率仅下降约 10%，内存开销几乎被算力吸收。
- 加速主要来自 Gather 融合消除独立数据搬运核，其次来自 Blackwell 独有 CLC 调度器与 2CTA MMA 带来的更快分组矩阵乘（约 10% 提升）。
- 两年间 MoE 专家粒度提升 9 倍、每次激活专家比例降至原来的十二分之一，Mixtral 8x22B、DeepSeek V3.2、Kimi K2.5、Qwen3 均为 MoE 架构。

## 来源

- 原文: [[raw/articles/2026-05-04-DeepSeek做大-Mega-MoE-Tri-Dao团队加快-SonicMoE-机器之心.md|DeepSeek做大 Mega MoE Tri Dao团队加快 SonicMoE 机器之心]]
