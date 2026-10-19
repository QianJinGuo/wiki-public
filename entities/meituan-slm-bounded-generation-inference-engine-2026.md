---
title: "美团 SLM 生成式高并发推理引擎：有界生成负载下的静态 DAG 编译执行"
created: 2026-10-19
updated: 2026-10-19
type: entity
tags: [inference-optimization, slm, serving, meituan, aotinductor, fp8]
sources: [raw/articles/meituan-slm-bounded-generation-inference-engine-2026]
confidence: 0.8
provenance_state: extracted
---

# 美团 SLM 生成式高并发推理引擎：有界生成负载下的静态 DAG 编译执行

美团搜索与推荐平台为 SLM（0.5B~1B 小参数量语言模型）在线落地重写了一套推理执行路径：在平均端到端时延低于 20ms 的约束下，Prefill-only 有效 MFU 达到 67%，吞吐较 SGLang 提升 10 倍。其出发点是一个被通用推理引擎忽略的工作负载特征——搜推场景的生成任务具有确定的上界。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

## 有界生成负载：设计约束的源头

搜推场景用 SLM 做意图理解、相关性判断、生成式推荐，请求形态主要是两类：Prefill-only（十余条样本一次前向、无 decode）与固定深度 Beam Search（decode 次数有上界）。与开放式长文本生成不同，这类负载的控制流与状态容量稳定、单次生成短，现有通用大模型推理引擎并非在该延迟约束下设计，因此团队从「具有确定 decode 次数上界的请求」这一前提出发重新设计在线执行路径。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

## 静态 DAG 的分层编译执行架构

系统以静态 DAG 组织完整业务请求，节点按其数据依赖离线编译（AOTInductor），框架统一提供节点调度、Tensor 搬运、设备同步、资源预分配、请求上下文与并发管理。正确性验证的第一层就是执行链路一致性：同机、同输入、同部署产物下，Python Artifact Runtime 与 C++ DAG Runtime 的输出 Tensor 逐字节一致，用于确认节点调度、buffer 管理、数据搬运与协议转换没有改变部署产物的语义。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

针对性优化沿计算与数据路径展开：变长样本的 **Packed Prefill**（Packed GQA 相对显式 Torch Packed Attention 的收益随 prompt 变长而扩大，说明变长 attention 实现是长文本路径的主要瓶颈）；**请求内共享前缀的 K/V 布局**（shared-prefix attention 通过减少 K/V 副本、拼接 Tensor、分页索引与数据搬运，使长 prompt Beam Search 的完整 DAG 吞吐提升约 20%）；**固定 Beam Search 的单节点编译**（逐 step 对齐候选 Token、候选分数、parent index、累计分数、suffix K/V 重排与最终序列）；**密集矩阵计算的选择性 FP8**（保留敏感计算精度，使 Prefill-Only 完整 DAG 吞吐提升约 24%）；以及 CPU 文本处理——tokenizer 文本处理由毫秒级压缩到十微秒级，使 CPU 热路径不再占用主要时延。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

## 容量口径与性能结果

容量测试采用固定在途请求数的 closed-loop 方式，单卡容量取同时满足平均端到端时延低于 20ms、P99 低于 80ms 的最高完成 QPS，服务入口再用 deadline、超时退出与拒绝策略阻止过量请求占用后端资源。一个关键观察是：无约束峰值吞吐并不等于满足业务时延目标的容量——以短 prompt Beam Search 为例，在选定容量点继续增加并发，QPS 仅增加几个百分点，平均端到端时延却上升 30% 以上并越过 20ms 目标；进入饱和区间后，增加并发主要转化为排队、资源争用与单请求执行时间增长。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

论文定义了「混合精度有效 MFU」：把每个请求中不同精度的逻辑矩阵 FLOPs 分别除以对应 dense 理论峰值（BF16/FP8 取约 150/300 TFLOPS），再按相同请求口径下的完成 QPS 汇总，只统计 linear、LM head 与 attention 的逻辑矩阵 FLOPs，排除冗余计算、动态量化、逐元素与非线性、softmax、Top-K 与 Beam 状态更新、tokenizer/结果编码/DAG 调度以及数据传输同步。结论是 Prefill-Only 模型的有效 MFU 已经较高，继续只优化 GEMM 的端到端边际收益有限；短 prompt Beam Search 则需继续关注 Top-K、搜索状态与短 shape 计算。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

## 复用边界：什么时候不要用这套框架

框架级能力（DAG Runtime、执行引擎、资源生命周期、并发管理、链路一致性检查）可由不同模型直接复用；节点级语义（Packed Prefill、固定 Beam Search、shared-prefix attention）可由数据布局与状态结构相同的模型复用；而具体 Triton/CUDA kernel、FP8 覆盖范围、分块配置与执行并发必须按目标 shape、GPU 与软件环境重新选择。专项优化只面向控制流与状态容量稳定、单次生成较短的 SLM 负载——开放式长文本生成、decode 长度高度动态、sequence 持续进出、强依赖 continuous batching、跨请求公共前缀长且命中率高、KV 状态规模大需要动态分页或淘汰的场景，通用 LLM serving 系统的动态能力仍然更匹配。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

对推理工程的启示：当负载的 decode 深度可界定，把执行路径从动态调度改为离线编译的静态 DAG，可以把时延优化从「提高单算子效率」升级为「消除调度与数据搬运开销」，代价是失去动态 batching 与长文本生成的通用性。^[raw/articles/meituan-slm-bounded-generation-inference-engine-2026.md]

## 关联

- 概念：[[concepts/inference-optimization|推理优化]]
- 相关实体：[[entities/sglang-inference-deployment-practice-benchmark-tuning|SGLang 部署实践与基准调优]]、[[entities/sglang|SGLang]]、[[entities/jd-oxygen-xllm-inference-engine|京东 Oxygen xLLM 推理引擎]]、[[entities/tokenspeed-agentic-inference-engine|TokenSpeed Agentic 推理引擎]]、[[entities/quail-ai-sql-inference-engine-modal-cmu-billion-tpm|Quail AI SQL 推理引擎]]

→ [[raw/articles/meituan-slm-bounded-generation-inference-engine-2026|原文存档]]
