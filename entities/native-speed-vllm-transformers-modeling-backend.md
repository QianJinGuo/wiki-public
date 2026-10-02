---
title: "Native-speed vLLM transformers modeling backend"
created: 2026-08-14
updated: 2026-10-02
type: entity
tags: [vllm, transformers, inference, optimization, huggingface, model-integration]
sources: [raw/articles/native-speed-vllm-transformers-modeling-backend]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Native-speed vLLM transformers modeling backend

Hugging Face 2026-07 发布 vLLM transformers modeling backend 提速成果：**transformers 实现的模型在 vLLM 引擎内达到（或超过）手写 vLLM 实现的速度**——模型作者无需再为每个框架各写一遍自定义实现。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

## 机制

- 旧模式：新模型要集成两次——transformers 一次 + vLLM 自定义优化一次；追求极致性能仍需手写 vLLM 实现
- 新模式：模型作者写好 transformers 实现即自动获得 vLLM 高速推理；vLLM 引擎在运行时插入注意力实现等高性能组件
- 支持范围：多数 LLM 架构；线性注意力模型暂不支持；Hub repo 中的自定义模型因未按规范编写而大概率不兼容

## 基准方法

三条件对照（唯一差异是代码路径）：`native`（vLLM 手写，基准线）/ `after`（transformers + PR）/ `before`（transformers 无 PR）。可复现 runner 以 gist 形式公开。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

## 意义

这是 [[entities/vllm|vLLM]] 生态与 [[entities/vllm-v0-to-v1-correctness-before-corrections|transformers 兼容性]] 的关键里程碑：把"写一次代码"的愿景从训练侧（transformers）延伸到推理侧（vLLM），属于 [[concepts/inference-optimization|推理优化]] 的工程范式变化——推理性能不再要求重复实现。

## 深度分析

### 从"只换注意力"到"整图重写"的能力跃迁

旧版 transformers backend 的优化思路是单点式的：假定注意力是推理瓶颈，运行时把 vLLM 的高性能注意力实现插入 transformers 模型即可。但真实部署中的性能来源远不止注意力——GPU 间并行、编译、融合算子各有贡献。新版后端用 `torch.fx` 对模型计算图做静态分析，匹配可优化模式后用 Python `ast` 直接原地改写源码，把优化范围从"一个算子"扩展到"整张图"。这本质上是把 vLLM 手写实现里的领域知识（哪些算子可融合、如何切分并行）编码成可自动应用的图变换规则。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

### 三类融合算子对应三类部署痛点

图重写产出的融合有三类，各自对应部署中的典型瓶颈：一是多对一映射到 vLLM 极致优化 kernel 的融合算子，最典型的是 MoE 模型中 Expert Parallelization（EP）所需的算子；二是 `MergedColumnParallelLinear` 与 `QKVParallelLinear`，由此可自动推导 TP（张量并行）切分计划，若 decoder block 列表易于识别还能进一步推导 PP（流水线并行）计划；三是改写后的模型仍完全兼容 `torch.compile` 与 CUDA Graphs，与专用 vLLM 实现走同一条编译加速路径。也就是说，手写实现享有的三大性能杠杆——kernel 融合、并行规划、编译捕获——现在都能从一份 transformers 代码自动获得。[[entities/moe-architecture|MoE 架构]] 模型受益尤其明显，因为 EP 算子恰是手工移植中最费力的部分。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

### 训练/推理统一：比速度更深的护城河

文中一个容易被忽视的要点：vLLM 专用实现只能用于推理，而 transformers 实现天然可用于训练——同一份模型代码可以覆盖 training、evals、RL rollouts 全流程。对做 RL 后训练的团队来说，这意味着 rollout 引擎与训练引擎共享同一个模型定义，消除了"推理实现与训练实现行为不一致"这一类隐蔽 bug 的来源。速度追平只是显性收益，"单一代码路径"才是范式级变化。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

### 兼容性边界揭示了"合规编写"的隐性契约

支持范围有一条容易被忽略的细节：Hub repo 里以独立代码形式存在的自定义模型"大概率不兼容"，原因是它们未按规范（compliantly）编写。这说明图重写并非任意 Python 代码都能吃下——`torch.fx` 静态分析要求模型代码具备足够的结构规整性（可追踪的 forward 图、标准算子调用）。对模型作者而言，这实际上把"性能"变成了一个前置约束：按 transformers 规范写模型，后续的推理优化自动到账；写得随心所欲，则退回手写移植的老路。线性注意力模型暂不支持，也印证了图模式匹配对架构范式的依赖。^[raw/articles/native-speed-vllm-transformers-modeling-backend.md]

## 实践启示

1. **新模型优先按 transformers 规范实现**：写一份合规的 transformers 实现即可同时获得训练能力与 vLLM 原生级推理速度，不要默认"要上 vLLM 就得手写实现"。
2. **迁移前先做架构自查**：线性注意力模型暂不支持、Hub repo 里的非规范自定义模型大概率不兼容——先确认架构在支持列表内，再规划迁移。
3. **RL/后训练管线可借此收敛代码路径**：训练、评测、RL rollout 与推理服务共用一份模型定义，消除双实现间的行为漂移风险。
4. **MoE 部署团队收益最大**：EP 融合算子与 TP/PP 并行计划的自动推导，直接省去 MoE 模型最繁重的手写移植工作。
5. **不要绕过编译栈**：改写后的模型仍走 `torch.compile` + CUDA Graphs，部署时应保持编译开启，否则只能拿到融合收益的一部分。
6. **基准对照可以照搬**：文中的三条件对照法（native / after / before，唯一变量是代码路径）是评估任何推理后端改动的可复用方法论，runner 脚本已公开。

→ [[raw/articles/native-speed-vllm-transformers-modeling-backend|原文存档]]
