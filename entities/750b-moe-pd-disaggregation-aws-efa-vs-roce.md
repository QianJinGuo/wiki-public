---
title: "750B MoE PD 分离推理：AWS EFA vs 自建 RoCE 通信架构实战对比"
description: "AWS China Blog 2026-06 实战案例：750B GLM-5.1-FP8 MoE 模型在 2P2D 分离推理下，从自建 ConnectX+RoCE 集群迁移到 AWS P5en+EFA 的完整通信架构验证与端到端性能对比"
created: 2026-06-18
updated: 2026-06-19
type: entity
tags:
  - aws
  - inference
  - moe
  - efa
  - roce
  - rdma
  - pd-disaggregation
  - prefill-decode
  - sglang
  - deepseek
  - ai-infra
sources:
  - raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation
review_value: 8
review_confidence: 7
review_recommendation: worth-reading
review_stars: 3
---

# 750B MoE PD 分离推理：AWS EFA vs 自建 RoCE 通信架构实战对比

## 概述

AWS China Blog 2026-06-12 发布的工程实战案例：将 750B GLM-5.1-FP8（256 Expert, top-k=8）MoE 模型的 PD 分离推理（2 Prefill + 2 Decode 节点，每节点 8×H200 GPU）从客户自建 ConnectX+RoCE 集群迁移到 AWS P5en+EFA，**在相同模型/架构/负载下系统对比两种高性能网络的设计哲学与端到端性能差异**。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

> **Background**：本文档基于 AWS China Blog 2026-06-12 实战案例建立。综合了 AWS HPC 团队对 EFA/SRD 协议的设计哲学论述、客户实际生产数据、SGLang/DeepEP/UCCL-EP/Mooncake 等开源通信库的协同工作机制，提炼出"PD 分离推理中 EFA 与 RoCE 各擅胜场"的工程结论。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**核心结论**：EFA 在 Decode 阶段高频跨节点小消息上 TPOT 高 ~31%（来自 CPU Proxy 中转的 per-message 开销），但在极端尾延迟上改善 73%（SRD 多路径 spray 设计）。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 测试配置（2P2D 分离推理）

| 维度 | 配置 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|------|------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 模型 | GLM-5.1-FP8（78 层、256 Expert、top-k=8、750B 参数） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 输入长度 | 120,000 tokens（长文本，KV Cache 传输压力最大） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 输出长度 | 1,000 tokens | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 硬件 | 4× P5en.48xlarge（8×H200/节点 + 16×EFA/节点，双向 900 GB/s NVLink + 3,200 Gbps EFA） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 框架 | SGLang（UC Berkeley/LMSYS，原生支持 PD 分离） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| KV Cache 传输 | Mooncake Transfer Engine（底层 libfabric → EFA RDMA） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| MoE 通信 | DeepEP + UCCL-EP（EFA 传输后端） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 网络拓扑 | Cluster Placement Group + Capacity Block for ML（同 Layer ii 网络节点） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## Prefill 与 Decode 的通信模式差异

理解 PD 分离推理的关键是认识到：**Prefill 与 Decode 在通信特征上截然相反**。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

### Prefill：所有高频通信走 NVLink

- **Attention 层**：CP=8 切序列（按 token 切分），卡间 all-gather/reduce-scatter 沿序列维交换 K/V（NVLink，节点内 900 GB/s）
- **MoE 层**：EP=8 集中在节点内（每卡 32 Expert），All-to-All 完全不出节点
- **跨节点通信仅 1 次/PP chunk**：Stage 0 → Stage 1 传递中间激活值（约 200 MB BF16/chunk），频率低、延迟不敏感
- **结论**：EFA 在 Prefill 中只承担低频 PP send/recv，TTFT 仅差 9%

### Decode：唯一高频通信是跨节点 MoE All-to-All

- **Attention 层**：DP=16 + dp-attention，每卡持有完整权重，**完全无通信**
- **MoE 层**：EP=16 跨 2 节点（每卡 16 Expert），每 token 触发 75 层 × 2 次 = **150 次跨节点小消息通信**
- **每次消息数据量很小**（约 16 token hidden state），但延迟极度敏感
- **关键挑战**：单次 per-message 延迟会被 150 次累积放大

## 端到端性能对比（AWS EFA vs 客户自建 RoCE）

| 指标 | 客户 RoCE（ConnectX） | AWS P5en EFA | 差异 | 根因 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|------|---------------------|---------------|------|------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Mean TTFT** | 11,904 ms | 12,977 ms | EFA +9% | Prefill 计算密集、网络非瓶颈 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Mean TPOT** | 13.80 ms | 18.05 ms | EFA +31% | 150 次 MoE 通信累积 ~28us/次 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Mean ITL** | 42.75 ms | 54.92 ms | EFA +28% | 与 TPOT 一致 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **P99 ITL** | 56.20 ms | 66.98 ms | EFA +19% | 尾部差距收窄 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Max ITL** | 434.30 ms | 116.99 ms | **EFA -73%** | SRD 多路径 spray 结构性优势 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Mean E2E** | 25,686 ms | 31,006 ms | EFA +21% | TPOT 差距 × token 数累积 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**解读**：对用户感知最关键的"最慢那个 token"改善 3.7 倍——在线服务 SLA 保障中，尾延迟稳定性往往比中位延迟更关键。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 为什么 EFA 在 Mean 上慢 31%：UCCL-EP 的 CPU Proxy 架构

### DeepEP 在 IB 上的 IBGDA（InfiniBand GPUDirect Async）

```
GPU kernel warp → 写 NIC doorbell 寄存器 → NIC 立即 DMA → 网络传输
                  （零 CPU 参与，硬件闭环）
```

ConnectX 系列网卡向 GPU 暴露 doorbell 寄存器，GPU kernel 可直接触发 RDMA——全程不经过 CPU。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

### UCCL-EP 在 EFA 上的 CPU Proxy 中转

EFA NIC（AWS 自研 Nitro 卡）**尚未向 GPU 暴露 doorbell**，IBGDA 无法直接使用。UC Berkeley/UC Davis 团队开发的 [UCCL-EP](https://github.com/uccl-project/uccl)（OSDI 2026, arXiv:2512.19849）通过 CPU Proxy 架构跨平台适配： ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

```
GPU kernel warp → 写指令到 FIFO（GPU 显存）
                          ↓
CPU Proxy 线程 busy-poll FIFO（通过 PCIe BAR 映射读取 GPU 显存）
                          ↓
解析指令 → 调用 libibverbs (ibv_post_send)
                          ↓
EFA NIC 通过 GPUDirect RDMA 直接从 GPU 显存 DMA → 网络传输
```

**数据路径不经过 CPU 内存**（NIC 直接 DMA GPU 显存），CPU 只负责转发控制指令。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

### 延迟量化

| 步骤 | IBGDA (ConnectX) | UCCL-EP (EFA) | 差异 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|------|------------------|---------------|------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| GPU 发起 | 写 NIC doorbell (~ns) | 写 FIFO (~ns) | ≈0 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 指令到达 NIC | 立即（同一 PCIe 总线） | CPU poll FIFO → 解析 → 提交 | +3-10 us | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 网络传输 | 交换机确定性转发 | SRD 多路径 spray | ≈相同 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 接收端处理 | NIC 硬件检查 → 写 CQE | CPU poll CQ → 重排序 → 通知 GPU | +3-10 us | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **每次通信总计** | ~5-15 us | ~20-40 us | **~15-25 us** | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**反推验证**：TPOT 差距 4.25ms / 150 次 = **每次 ~28us**——与路径分析完全吻合。这是 CPU Proxy 中转的固定开销被高频小消息累积放大，**不是带宽瓶颈**。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 为什么 EFA 的 Max ITL 低 73%：SRD 多路径 spray

EFA 的 SRD（Scalable Reliable Datagram）协议在协议层与 RoCE v2 有根本性差异： ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

| 特性 | RoCE v2 (ConnectX) | EFA/SRD | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|------|---------------------|----------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 消息顺序 | 有序（per-QP，需应用管理） | 无序（应用层实现有序） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 路由 | ECMP（依赖交换机支持，路径受限） | ECMP spray（一次选 64 条路径） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 拥塞控制 | ECN + DCQCN（依赖 PFC + ECN 交换机） | 动态速率估计 + 拥塞感知（无 PFC 依赖） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 是否需专用交换机 | 是（PFC + ECN 复杂配置） | 否（运行于 UDP/IP，普通以太网） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 部署复杂度 | 中高（MTU、流控、PFC 错误配置风险） | 低（无损、零配置依赖） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 核心权衡 | 单次极限低延迟，扩展性受限 | per-message 略慢，但大规模稳定性高 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**SRD 的核心创新**：放弃 per-QP 有序交付，将每个数据包的包同时 spray 到多条路径（一次从数百上千条可用路径中选 64 条）。单条路径拥塞不阻塞其余包——AWS 实测 P99 尾延迟下降约 10 倍。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**对生产环境的意义**：RoCE 单路径遇到拥塞时无法绕行，产生 434ms 极端毛刺；EFA SRD 包分散到多条路径，天然负载均衡——这正是用户感知"最慢那个 token"改善 3.7 倍的根本原因。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## AWS 网络拓扑保障机制

要获得 PD 分离推理的优化拓扑，必须主动使用以下 AWS 原语： ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

| 原语 | 拓扑保证 | 适用场景 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|------|----------|----------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Cluster Placement Group (CPG)** | 同组实例共享 high-bisection bandwidth segment（同 Layer ii 网络节点） | 长期稳定需求 + 需要灵活扩缩容 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **Capacity Block for ML** | 自动放入 UltraCluster（low-latency, petabit-scale, non-blocking networking） | 短期 ML 任务，按天/周预约（最远 8 周后） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| **On-Demand Capacity Reservation (ODCR)** | 用户主动指定 CPG + 高二分带宽网段 | 立即生效，持续占用直到取消 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**验证**：本次 4× P5en.48xlarge（eu-south-2a）通过 Capacity Block 自动分配到同一 Layer ii 网络节点，跨节点通信仅 3 跳（实例→Layer iii→Layer ii→Layer iii→实例）。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**诊断 API**：`DescribeInstanceTopology` 返回每台实例的 NetworkNodes 列表——从底层（数组最后一个）开始向上比较，共享的最底层网络节点越深，物理距离越近。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## EKS 软件栈分层实践（内核态在 AMI，用户态在容器）

要在 EKS 上跑 MoE PD 分离推理，节点软件栈必须解决"让节点用上 GPU + EFA + 安全分配给容器"两件事： ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

| 层 | 核心职责 | 关键组件 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|----|----------|----------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 容器层 | 封装应用软件栈，可独立于节点升级 | SGLang + NCCL/aws-ofi-nccl + UCCL-EP + nixl/Mooncake TE | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| 设备资源层 | 向 K8s 调度器注册非标准设备 | NVIDIA Device Plugin（gpu: 8）+ EFA Device Plugin（efa: 16） | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| AMI 层 | 让操作系统识别硬件 | NVIDIA Driver 580.x + nvidia-container-toolkit + EFA Driver (efa.ko) + libfabric + NVMe LVM | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**关键原则**：内核驱动（efa.ko、NVIDIA Driver）随宿主机（AMI 提供），用户态库（libfabric、aws-ofi-nccl、UCCL、CUDA runtime）随容器镜像走——**保证通信库与应用版本精确对齐、镜像可移植**。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 通信软件栈协同（NCCL/Mooncake/UCCL-EP）

EFA 在用户态暴露两套接口，这些通信库分别走不同路径： ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

| 库 | 开发者 | 走 libfabric? | 走 libibverbs? | 定位 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
|----|--------|---------------|----------------|------| ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| NCCL + aws-ofi-nccl | NVIDIA + AWS | 是 | — | GPU 集合通信标准 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| DeepEP | DeepSeek | — | — | MoE Expert Parallelism 专用 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| UCCL-EP | UC Berkeley / UC Davis | — | 是（直接 Verbs） | DeepEP 跨平台适配层 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| Mooncake TE | 月之暗面（Kimi） | 是 | — | PD 分离的 KV Cache 跨节点传输 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| NIXL | NVIDIA（Dynamo） | 是 | — | 单边 RDMA KV Cache 抽象 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
| libfabric | OFIWG | — | — | 跨厂商网络传输抽象 | ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

**为什么 UCCL-EP 直接走 Verbs？** 对延迟极度敏感、需要精细控制发送队列的场景（如 MoE 150 次小消息/token），绕开 libfabric 抽象层直接调 Verbs，可省去 ~3-5 us 开销。 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 三个独家贡献（不应合并到现有 entity）

1. **CPU Proxy 架构量化对比** — 首次以"~28us/次通信"为粒度量化 UCCL-EP vs IBGDA 的延迟差异，结合 TPOT 反推验证路径分析的准确性 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
2. **Max ITL 改善 73% 的根因** — SRD 多路径 spray vs RoCE 单路径 ECMP 的结构差异，附 AWS 原始 IEEE Micro 2020 论文引用 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]
3. **2P2D 分离推理完整通信栈分层** — Attention/MoE/PP/KV Cache 五种通信模式的物理路径与库映射，从节点内 NVLink 到节点间 EFA 完整拆解 ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 相关主题

- [[entities/aws-fsx-lustre-gpudirect-sharded-llm-loading|AWS FSx Lustre GPUDirect 加载]] — AWS 训练/推理数据加载栈
- [[entities/aws-grpo-rlvr-sagemaker-math-reasoning|AWS GRPO RLVR SageMaker]] — AWS 后训练栈
- [[entities/building-blocks-for-foundation-model-training-and-inference-on-aws|Foundation Model Building Blocks on AWS]] — AWS 训练与推理基础组件
- [[entities/foundation-model-building-blocks|Foundation Model Building Blocks]] — 通用基础组件
- [[entities/glm5-scaling-pain|GLM-5 Scaling Pain]] — GLM 系列规模化的工程挑战

> [[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation|原文存档]] ^[raw/articles/750b-moe-model-roce-cluster-migration-aws-efa-prefill-decode-disaggregation.md]

## 相关实体

- [[moc/mlops-training-inference|MOC]]
