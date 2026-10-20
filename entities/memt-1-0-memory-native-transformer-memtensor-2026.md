---
title: "MemT 1.0：记忆原生 Transformer 与 Memory Scaling"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [memory, transformer, architecture, rsi, moe, training]
sources: [raw/articles/memt-1-0-memory-native-transformer-memtensor-2026]
confidence: 0.8
---

# MemT 1.0：记忆原生 Transformer 与 Memory Scaling

## 一句话定位

MemT 1.0 是记忆张量（MemTensor）开源的一系列「记忆原生（memory-native）」模型：不把历史经验塞回上下文，而是在 Transformer 内部增加一条**与注意力并行的记忆计算通路**，把经验编码成可训练的矩阵状态，并在生成时通过记忆分支读取。公开的两个型号是 36B（激活 3B，基座 Qwen3.6-35B-A3B）与 288B（激活 17B，基座为继续后训练的 DeepSeek-V4-Flash），团队称这是记忆原生路线**首次在近 300B 参数规模上的工程验证**。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 为什么现在讨论记忆：RSI 的第一块拼图

文章把「AI 自我改进（RSI）」拆成三块拼图：记忆、反馈、验证与更新机制。记忆负责让一次有效的改进成为下一次的起点，其余两块负责判断哪些改动值得进入下一轮——MemT 1.0 只做第一块，也是被长期低估的一块。这与 [[concepts/agent-memory-architecture|Agent 记忆架构]] 的语境不同：agent memory 关心「怎么检索、放进上下文」，记忆原生关心「怎么让模型自己学会读写的内部状态」。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 架构：主干冻结 + 超记忆 + 局部记忆

参数化上，MemT 1.0 保留基座原有前向流程，只在选定 Transformer 模块插入原生记忆分支，拆成三层：

- **主干权重**：承载已有通用能力，训练记忆时保持冻结；
- **超记忆（hyper-memory）**：读写投影、门控等可训练参数，决定如何编码、更新、读取记忆；
- **局部记忆**：动态矩阵状态，保存交互中新加入的信息。

写入时隐状态经投影生成记忆的 K/V，由擦除、写入、遗忘门控调节已有矩阵中的关联；读取时当前输入映射为记忆查询，读出后归一化并经输出门控参与生成，与注意力输出融合。更新机制基于 Gated DeltaNet-2 做逐 token 生成并做稳定化处理——消融显示，把逐 token 更新换成整轮聚合，Nano 的记忆基准平均分从约 47.0 掉到 41.0，6 分差距说明 token 级连续更新更能处理记忆衰减并纠正轮内关联。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

容量扩展走 **MoS（Mixture-of-Storage）**：把一个记忆原生模块扩成多个记忆专家，分别学习存储与路由，并采用**稠密融合**（所有专家都参与计算，路由权重决定贡献）——这与基座 MoE 的稀疏专家路由形成刻意对照。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 训练：340 亿 token 的记忆语料与教师-学生蒸馏

记忆训练语料共 340 亿 token（继续预训练 200 亿 + 后训练 140 亿）。继续预训练格式统一：把长文本切成连续片段写入记忆，再让模型拿着记忆状态回答后续查询，每处理完一段就更新状态供下一段使用。后训练包含三类数据：记忆原生的语言建模数据、记忆存储与利用数据（摘要、续写、压缩等）、以及强化长程记忆的任务数据。关键设计是**冻结的主干充当教师**（可见完整历史），MemT 1.0 作为学生只能看到自己的记忆状态，沿 off-policy 蒸馏对齐教师分布。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 实测：逼近文本记忆，优于无经验基线

| 场景 | 无经验 | 文本经验 | MemT-1.0-Nano |
|---|---|---|---|
| SWE-bench Verified 关键断点平均成功率（500 题 ×3 次） | 66.07% | 69.13% | **69.73%** |
| 三次全部成功比例 | 54.00% | 61.80% | **62.80%** |
| SWE-bench Multilingual 平均成功率 | 60.00% | 63.00% | **63.56%** |
| 11 项记忆基准平均分 | — | — | Flash 50.6 / Nano 47.0 |

作者自己的结论比较克制：程序性经验能被原生记忆保留，表现与显式文本经验接近，优势更多体现在平均成功率与重复执行的可靠性上；而文本记忆系统在长历史细节召回上仍有整体优势。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 能力形态：经验改变的是行动顺序

Django `parse_http_date` 的两位数年份修复轨迹是文章里最有信息量的细节：无经验时 agent 找到 bug 后立刻改解析逻辑，定向复现与验证发生在后续步骤；两组有经验的都是先构建复现测试再改代码，而 MemT-1.0-Nano 还在首次修改后**重新检查了一项边界条件**并修正实现。也就是说，记忆带来的不是「知识更多」，而是「工作流顺序变了、检查点提前了」。这与 [[concepts/agent-memory-systematic-framework|记忆系统框架]] 里「经验通过流程而非检索发挥作用」的判断一致。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## Memory Scaling：第三条增长曲线

团队提出的框架是：推理时扩展（test-time scaling）与训练时扩展之外，还有 **Memory Scaling**——即「同样的经历，换一种内部表示、更新规则和读取方式，能不能用得更好」。这让提升记忆效果多了一组可通过训练与架构设计调整的变量，而不是只能在外部库里调检索参数。成本侧给出了具体数字：同样 128K 长历史下，Nano 的端到端推理速度约为完整上下文方案的 **1.86 倍**，每条轨迹的持久状态占用是完整上下文 KV Cache 的 **1/66**。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 边界与判断

文章的自我限定值得保留：原生记忆**没有**证明长期上限高于外部记忆；精确删除、解释与恢复需要额外机制；对精确原文/来源/版本的查询，外部记录仍有价值。更现实的定位是分工——外部记忆负责可溯源原文，原生记忆负责执行经验的复用。对企业的含义是选型标准要加一条：不是只看单次任务表现，而是看「经过验证的经验能复用多少次」，因为重复工作流里省下的返工才是长期收益。^[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026.md]

## 关联

- [[concepts/agent-memory-architecture|Agent 记忆架构]] — 外部记忆体系的对照面
- [[concepts/agent-memory-systematic-framework|记忆系统框架]] — 经验如何改变工作流
- [[concepts/context-engineering|上下文工程]] — 把历史塞回上下文的路线及其成本
- 代码与权重：https://github.com/MemTensor/MemT-1.0 ，论文 arXiv 2610.26825

→ [[raw/articles/memt-1-0-memory-native-transformer-memtensor-2026|原文存档]]
