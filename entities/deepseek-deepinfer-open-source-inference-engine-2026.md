---
title: "DeepSeek 开源 DeepInfer 推理框架：把单点算子组装成整套部署工程"
created: 2026-10-20
updated: 2026-10-20
type: entity
tags: [inference, deepseek, speculative-decoding, moe, structural-output, hardware-portability]
sources: [raw/articles/deepseek-deepinfer-open-source-inference-engine-2026]
confidence: 0.75
---

# DeepSeek 开源 DeepInfer 推理框架

2026-10-20，DeepSeek 开源内部推理框架 **DeepInfer**，并同步开源生产级 Token 编解码组件 **deepseek-recipe**。此前 DeepSeek 陆续放出 FlashMLA、DeepGEMM、DeepEP 等单点算子库，DeepInfer 的定位是**把这些零散库组装成整套工程框架**——推理框架决定服务器先处理谁、计算任务怎么分配、多卡怎么配合，是出字速度与模型价格的直接决定因素。^[raw/articles/deepseek-deepinfer-open-source-inference-engine-2026.md]

## 四个核心能力

1. **核心算子与并行调度合一**：FlashMLA、DeepGEMM、DeepEP、TileKernels、DeepSelect 及自定义 TileLang 算子打包进统一运行时，内置 Attention DP、上下文并行（CP）、MoE 专家并行（EP）及负载均衡。^[raw/articles/deepseek-deepinfer-open-source-inference-engine-2026.md]
2. **生成加速**：支持 DSpark 与基于置信度调度的推测解码（默认一次最多猜 5 个候选再批量验证），并用 **SWA Bounded Replay**（滑动窗口有界重放）截断长上文冗余计算，压低 Prefill 工作量。参见 [[concepts/speculative-decoding|推测解码]] 与 [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark]]。
3. **确定性与结构化输出**：支持 Batch-invariant inference（结果与批次组合无关）与确定性采样；可直接用正则、JSON Schema 或 EBNF 约束输出格式，把格式清洗从业务层移到引擎层。
4. **双算力平台**：适配英伟达 SM100 架构 GPU，同时原生支持华为昇腾 950 NPU。

## 实测数据（DeepSeek-V4.1-Flash）

单 Scale-up 域内 4 张 SM100 跑 Prefill、64 张 SM100 跑 Decode，128K 长上下文场景下：Prefill 吞吐最高约 **29K tokens/s/GPU**；Decode 在每用户 80 tokens/s 交互速度下单卡输出约 **2200 tokens/s/GPU**。^[raw/articles/deepseek-deepinfer-open-source-inference-engine-2026.md]

## deepseek-recipe：解决「同权重不同效果」

私有化部署最常见的坑是「同一个权重，自建效果比官方 API 笨一点」，病根多在请求前后的格式转换。deepseek-recipe 把 OpenAI / Anthropic 等不同格式的请求转换成统一会话表示，按官方逻辑拼提示词、编码 Token，再把输出解析回对应 API 响应格式——目标是让社区私有化部署拿到与官方 API 一致的体验。^[raw/articles/deepseek-deepinfer-open-source-inference-engine-2026.md]

这个组件的重要性常被低估：**推理一致性不只取决于权重和引擎，还取决于 prompt/模板/tokenizer 的逐字节复现**。它与本仓 [[concepts/inference-optimization|推理优化]] 中「KV cache 与批调度」同属性能问题，但管的是「语义保真」这一层。

## 位置：从权重开源到全栈开源

开源路线图清晰可见：模型权重 → 底层算子（FlashMLA/DeepGEMM/DeepEP）→ 推理框架 + API 编解码逻辑。DeepSeek-V4.1-Flash 的大规模服务已跑在 DeepInfer 上，官方表示未来新模型的推理代码与优化会持续更新在该仓库。对自建方而言，这意味着可复用的不再只是算子，而是**编排层**：并行策略、推测解码调度、结构化约束、跨平台后端。^[raw/articles/deepseek-deepinfer-open-source-inference-engine-2026.md]

## 待验证

- 吞吐数字来自官方单场景（128K、4P/64D），未给出与 vLLM/SGLang 的同条件横向对比，也无 P99 延迟与尾延迟数据。
- SWA Bounded Replay 对长上下文任务质量的损失未披露（有界重放截断冗余计算，理论上可能影响「远处依赖」类任务）。
- 昇腾 950 支持的实际成熟度（算子覆盖率）未知。

## 关联

- [[concepts/inference-optimization|推理优化]]
- [[concepts/speculative-decoding|推测解码]]
- [[entities/deepseek-dspark-speculative-decoding-2026|DeepSeek DSpark 推测解码]]
- [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 高效推理栈]]
- [[entities/atrex-kernel-agent-qwen38-max-production-kernels-2026-09-28|Atrex：生产级算子生成]]

→ [[raw/articles/deepseek-deepinfer-open-source-inference-engine-2026|原文存档]]
