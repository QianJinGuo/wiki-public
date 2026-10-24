---
title: "vLLM 分离式推理的 push 式 KV 传输：TTFT 降约 3 倍、TPOT 保持平稳"
created: 2026-10-20
updated: 2026-10-25
type: entity
tags: [vllm, kv-cache, disaggregated-inference, nixl, latency, inference-optimization]
sources: [raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026]
confidence: 0.65
---

# vLLM 分离式推理的 push 式 KV 传输

分离式（disaggregated）部署把 Prefill 与 Decode 拆到不同节点，代价是**每个请求都要把 KV cache 从 prefill 节点搬到 decode 节点**。AWS 这篇工程分析对比了两种搬运语义：**pull（decode 节点主动拉）** 与 **push（prefill 节点主动推）**，结论是 push 在高负载长上下文下显著更优。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]

## 关键数据（1P1D，128 输出 token，decode 节点关闭 prefix caching）

- **TTFT**：低负载短提示下两者几乎相同；随 QPS 与输入长度上升，pull 在 8 QPS / 16k 输入退化到 **约 1744 ms**，push 保持在 **约 540 ms**，约 **3 倍**改善。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]
- **TPOT**：轻载时两者接近（5.6–5.8 ms）；高负载长上下文时 pull 升到 **约 10.4 ms**，push 稳定在 **约 7 ms**。

## 机制解释：谁在关键路径上

差距的根因是**描述符提交发生在哪个线程**：pull 模式在 decode worker 上**内联**提交 NIXL 描述符，且位于引擎循环中同步执行，直接阻塞 token 生成；push 模式把描述符准备与传输提交移到**后台线程**，脱离关键路径，因此每 token 延迟在高并发长上下文下仍保持平稳。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]

这与 [[concepts/inference-optimization|推理优化]] 的一条通用规律同构：**延迟优化的核心不是「搬得更快」，而是「别把搬运放在生成的关键路径上」**。同为 KV 层优化，[[entities/tiered-kv-cache-large-llms-sagemaker-hyperpod-curvine|分层 KV cache]] 解决容量，push 模式解决时序。

## 实验设计的严谨点

对比时特意在 decode 节点**关闭 prefix caching**，否则重叠提示可以复用本地缓存从而绕过传输，导致对比失真——这是一个容易踩的评测陷阱：**缓存复用会掩盖传输开销**。^[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026.md]

## 适用判断

- 何时值得：PD 分离 + 长上下文 + 中高 QPS（≥4 QPS）场景，pull 模式的同步描述符提交会成为 TTFT/TPOT 的主要瓶颈。
- 何时无差别：低负载、短提示，两种模式几乎等价，改用 push 只增加实现复杂度。

## 局限

原文抓取不完整（仅保留 push/pull 对比与图表解读段落），缺少 push 模式在极端并发下的背压/内存放大分析，以及与 NIXL 之外的传输后端（如 EFA/RDMA 自定义路径）的对比。相关实体：[[entities/disaggregated-prefill-decode-llm-inference-sagemaker|SageMaker 分离式 PD 推理]]、[[entities/750b-moe-pd-disaggregation-aws-efa-vs-roce|750B MoE 的 PD 分离与 EFA/RoCE 对比]]。

## 深度分析

### 一、pull 与 push：谁发起，谁拥有交接的所有权

两种语义的差别不在「数据往哪个方向流」——KV 最终都必须从 prefill 落到 decode——而在于**控制面的发起权归谁**。pull 模式下 decode 是请求方：先取得该请求在 prefill 侧的元数据（block 列表、远端地址），再由 decode worker 自己组装并提交 NIXL 描述符，且这次提交在引擎循环内同步执行。消费端被迫兼任传输客户端，生成线程要为尚未到达的 KV 做控制面工作。push 模式把发起权交给生产端：prefill 算完即主动写入 decode 预先注册的缓冲区，decode 只需事先登记内存句柄，之后不再在生成路径上等待或发起。

这个所有权转移直接减少首 token 之前的往返次数。pull 至少需要 decode→prefill 的取数请求、prefill 侧的就绪确认、prefill→decode 的数据回传三段；push 把它压成单向数据流加一次完成通知。TTFT 里「传输与交接」这一项的往返次数下降，正是 3 倍差距的结构性来源，而非带宽变大。

### 二、为什么 TTFT 掉 3 倍而 TPOT 保持平稳

TTFT 是一次性成本，由排队、prefill 计算、KV 交接、首 token 采样四段串成；TPOT 是 KV 常驻之后每个 decode step 的固定成本，由权重带宽、attention 读 KV、采样决定。push 只改前者中的交接段，对后者的算法构成没有影响，所以一旦 KV 到位，两种模式的每 token 成本应落在同一基线——这正是标题里「flat」的含义：push 的 TPOT 曲线在高负载长上下文下仍然平（约 7 ms），而 pull 会爬升（约 10.4 ms）。

但 pull 的 TPOT 为什么会爬升，值得单独说清：内联描述符提交发生在**每步都要跑一遍**的引擎循环里，且不只新请求进入时触发——正在 decode 的请求同样被这段同步控制面工作周期性打断。它抬高的不是传输时间，而是循环周期本身。因此严格地说：TTFT 的收益来自「少一次控制面往返」，TPOT 的收益来自「循环不再被插入同步工作」。前者缩小绝对值，后者消除抖动，量纲不同，不能互相折算。

### 三、原文自己划定的测量边界

**1P1D 拓扑**：没有 P:D 比例效应，decode 侧只面对单一推流来源。扩到多 prefill 后，后台线程与缓冲区要承受聚合带宽与并发推流，push 的实现复杂度与背压风险才真正暴露。**128 输出 token**：TTFT 是常数项、TPOT 是线性项，输出越短，TTFT 在端到端时延里的权重越大，3 倍比值越好看；长输出下同一绝对收益被摊薄，用户感知从「首字更快」转向「吐字更稳」，评价指标必须换。**decode 关闭 prefix caching**：这是隔离变量的正确做法（否则命中会绕过传输、使对比失真），但同时剔除了生产里最常见的优化——真实系统中命中率高则大量请求根本不产生跨节点传输，push 收益按未命中率打折；命中率低、前缀多样的负载才是它的完整收益区。

此外测试只扫到 8 QPS / 16k，拐点位置未被定位。收益随负载单调扩大意味着低负载区间几乎没有差别，而「从哪一档 QPS 开始值得切换」这个工程问题，原文没有回答。

### 四、它在 KV 传输栈里的位置

这一层有三个正交维度：**传输后端**（NIXL 之下的 RDMA/EFA 路径）决定单次搬运的带宽与延迟；**传输粒度**（整块 vs layer-wise/chunked）决定 decode 侧何时能开始第一层 attention；**调度语义**（pull/push）决定控制面落在哪个线程。push 属于最上层、也最便宜可改的一层——不动网络、不动模型，只换谁提交描述符；它与分块传输互补而非互斥：分块让首 token 不必等全部层就位，push 让提交动作不占生成线程，两者削的是同一个临界区。

排队论上，decode 的引擎循环是一个被串行化的服务台，任何内联控制面工作都等价于抬高单次服务时间、推高利用率 ρ；M/G/1 型系统里等待时间随 ρ/(1−ρ) 放大，于是同一段固定开销在低负载不可见（原文低负载两者几乎相同），高负载时被放大成数量级差异。不是传输变慢了，是排队把同步开销放大了。

## 实践启示

1. **把 P/D 交接单独基准化**：TTFT 与吞吐分开测，扫 input length × QPS 的二维网格，定位自己拓扑下的切换拐点，不要直接引用别人的 8 QPS / 16k 结论。
2. **prefix caching 留在生产决策矩阵里**：评测期为隔离变量可以关闭，但上线决策必须在开启状态下重测，按实际未命中率折算 push 的收益。
3. **按每请求 KV 字节数配带宽**：KV bytes ≈ 2 × 层数 × KV 头数 × head_dim × 序列长度 × 精度字节数，乘以目标 QPS 得到需要的持续带宽，再为后台线程的突发留余量，而不是按平均值配。
4. **TTFT 与 TPOT 作为两个独立 SLO**：push 主要买 TTFT，顺带买 TPOT 的稳定性；用单一端到端 P50 验收会同时掩盖这两件事。
5. **在自己的 P:D 比例上复验**：1P1D 是对 push 最友好的形态；扩到多 prefill 后必须补测聚合推流下的缓冲区水位、背压行为与 decode 侧内存放大。
6. **先改调度语义，再改传输后端**：push 是纯软件改动、无需换网卡或换实例族，先吃掉这层低垂果实，再评估 layer-wise 传输与更大带宽的边际收益——并把「引擎循环内的同步控制面工作」当作一类可观测的反模式持续巡检。

## 关联

- [[concepts/inference-optimization|推理优化]]
- [[entities/ai-infra-llm-efficient-inference-vllm|vLLM 高效推理栈]]
- [[entities/disaggregated-prefill-decode-llm-inference-sagemaker|分离式 Prefill/Decode 推理]]
- [[entities/tiered-kv-cache-large-llms-sagemaker-hyperpod-curvine|分层 KV Cache]]

→ [[raw/articles/vllm-push-based-kv-transfer-disaggregated-inference-2026|原文存档]]
