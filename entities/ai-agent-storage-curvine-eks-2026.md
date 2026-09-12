---
title: "AI Agent 存储选型：Curvine 如何在 EKS 上支撑万级Agent运行"
created: 2026-07-07
updated: 2026-09-10
type: entity
tags: [aws, eks, storage, curvine, ai-agent, kubernetes, distributed-storage, agent-infrastructure]
sources: [raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# AI Agent 存储选型：Curvine 如何在 EKS 上支撑万级Agent运行

## 摘要

2026 年的 AI 基础设施正从"一个大模型实例服务所有请求"走向"成千上万个 Agent 实例各自独立运行"：每个 Agent 都是需要独立 POSIX 工作空间的有状态进程，如 OpenClaw 的 SOUL.md / AGENTS.md / MEMORY.md 与源码被密集随机读写，形成"万级独立文件系统"需求，K8s 侧即上万个需快速 provision 的 PVC。本文以 EKS 上 10,000 个 Agent Pod 的实测比较 EBS / EFS / S3 与 Curvine。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

## 核心要点

- **EBS 卡挂载数**：r6g 单实例 volume 上限 28（实际约 27），100 Pod/节点需近 4 倍节点。
- **EFS 卡 provision**：配额够（2025-02 起 10,000 个 Access Point），但创建受 API 速率限制。
- **S3 非文件系统**：缺 open / read / write / seek / rename / list directory 语义。
- **Curvine 拿掉限流**：对象存储之上建分布式缓存层，建 PVC 本质是 mkdir。
- **万级实测**：10,000 PVC 全 Bound、10,000 Pod 全 Running，零 Pending / Failed，核心仅 4 Pod。

## 传统方案的局限

### EBS：隔离性好，但挂载数是硬上限

每个 Agent 一个独立 EBS volume 隔离性最干净，但单实例挂载数是硬上限：多数 Nitro 实例（含 r6g）最大 attachment 数 28，与网络接口、NVMe instance store 共享，扣除主网卡后实际约 27；第 7 代后部分类型（M7i、R7i）有独立 limit。每台 r6g.4xlarge 跑约 100 个 Agent Pod，每 Pod 一个 volume 需近 4 倍节点数，利用率从 88% 掉到 20%；且 EBS 绑定单 AZ，Pod 跨 AZ 后不可达。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

### EFS：隔离机制成熟，大规模 provision 是瓶颈

EFS 配合 Access Point 可做到文件系统级租户隔离，支持 ReadWriteMany，Pod 跨节点调度无需 detach/attach，2025-02 起单文件系统最多 10,000 个 Access Point。瓶颈在 provision 速度：每个 PVC 对应一次 Access Point 创建，而 EFS API 有速率限制，CSI controller 需串行或小批量调用。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

### S3：容量无限，但不是文件系统

S3 作为归档层没问题，但 Agent 运行时需要 POSIX 语义：不支持原地修改、rename 原子性、目录列举一致性；Mountpoint for Amazon S3 也只支持顺序写新文件与读取已有文件，无法支撑反复改 context、append 日志、更新 checkpoint 的工作流。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

## Curvine：为 Agent 规模化设计的分布式缓存文件系统

Curvine 是用 Rust 从头编写的高性能分布式缓存文件系统，名字取自"Curvature Engine"（曲率引擎，《三体》里的超光速推进装置）：在 S3 之上建一层分布式文件系统缓存，向上给完整 POSIX 语义，向下以对象存储持久化，以原生 CSI 驱动以 PVC 挂载。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

### 核心架构

- **Master**：元数据管理、Worker 协调与负载均衡，Raft 保证高可用
- **Worker**：多层缓存（内存→SSD→HDD），热数据自动提升
- **客户端/K8s 集成**：FUSE 提供 POSIX 接口、兼容 S3/HDFS；原生 CSI Driver 下建 PVC 仅需 mkdir 一个目录（毫秒级）

CSI Controller 处理动态 provisioning、CSI Node DaemonSet 每节点负责 FUSE 挂载；同一挂载点的不同路径供同节点多个 Pod 独立使用，各自看到互不可见的 `curvinefs` 视图。文件元数据路径与底层 S3 对象一一对应（UFS 即对象存储），服务故障时 S3 文件仍可独立访问。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

### 与 JuiceFS 的核心差异

JuiceFS 是先行者，用 Go 实现，同为"元数据引擎 + 对象存储"模式。差异：Rust 读写路径加零拷贝，标称 100μs 级延迟、100K+ 稳定 QPS（tokio + 无 GC，小文件并发理论优于 Go）；单集群支持 50 亿小文件；元数据独立上 Curvine 与 S3 对象一一对应、故障时可独立访问，JuiceFS 拆成 Block 则无法从对象名反推原始文件；缓存上内存→SSD→HDD 多层自动分级；Curvine 以 AI 训练加速和 Agent 云原生存储为一级用例。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

## 万级 Pod 基准测试

在 EKS 上规模验证：10,000 个独立 Pod 全部 Running。环境为 99 × r6g.4xlarge（Graviton ARM64，16 vCPU / 128Gi）、Karpenter 自动扩缩、VPC-CNI、v1.31.14-eks（us-west-2），10,000 个 PVC 全部 Bound；存储集群仅 1 Master + 3 Worker = 4 个核心 Pod（CSI Controller 1、CSI Node DaemonSet 104，合计 109 个 Pod）。每节点 ~100 个 Agent Pod，CPU 88% / Memory 98%；负载为 StatefulSet（podManagementPolicy: Parallel）10,000 副本，每 Pod 131m CPU / 1190Mi（Guaranteed QoS），独立 PVC 1Gi。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

结果：PVC 零 Pending / 零 Failed，Pod 零 CrashLoopBackOff；每 PVC 实际 provisioned ~30Gi，总容量约 300TB，文件系统 curvinefs；pod-0、pod-5000、pod-9999 写入验证持久化，重启后仍在、跨节点调度视图一致；分布 102 pods × 98 nodes = 9,996 加 4 pods × 1 node = 4 = 10,000；Pod 内 `df` 见 `curvinefs 29.8G / Used 354.6M / Available 29.5G / 1%`，各 Pod 独立互不可见。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

关键结论：provision 不依赖云管控 API（本质是 FS 内 mkdir）；4 个核心 Pod 服务万级 PVC；单节点密度不受存储限制；加 Worker 即可扩展。

## 深度分析

### 瓶颈从容量迁移到控制平面

三种原生方案暴露的不是容量问题而是控制平面问题：EBS 的 28 是每实例静态配额，EFS 每次 PVC 创建都要打一次受 API 速率限制的调用。当有状态实例从"几十"跨到"上万"，上限就从"能存多少数据"变成"每分钟能建多少个卷"。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

### 共享挂载点作为密度杠杆

块存储下单节点密度被 volume attachment 上限锁死；Curvine 让 100 个 Pod 共享同一挂载点的不同路径，attachment 从"每 Pod 一个"变成"每节点一个"——块设备挂载数是虚拟化层的硬上限，CPU/内存反而能加钱放大。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

### 元数据形态决定可恢复性

最深分歧不在语言或性能数字，而在元数据与对象存储的映射：Curvine 一一对应、S3 文件可独立访问，JuiceFS 拆成 Block 则无法从对象名反推原始文件——灾难恢复时前者可从对象存储兜底，后者须先救活元数据服务。^[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行.md]

## 实践启示

1. **先量清负载形状**：是"少数大 PVC"还是"上万小 PVC"，后者会让 EBS / EFS 触顶。
2. **审查 provision 的云 API 依赖**：触发 CreateVolume / AttachVolume / CreateAccessPoint 的方案，都要按目标规模估算限流排队成本。
3. **把挂载数当容量之外的硬约束**：EBS 在 100 Pod/节点下利用率从 88% 掉到 20%。
4. **从小规模起步放量**：从 100–500 个 Pod 验证，存储集群 1 Master + 1 Worker 起步。
5. **高密度场景优先共享挂载点**：替代"每 Pod 一个块设备"的拓扑。

## 相关实体

- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 同源 Curvine 实践: [[entities/tiered-kv-cache-large-llms-sagemaker-hyperpod-curvine|Curvine 分层 KV Cache]]
- Agent 沙箱: [[concepts/agent-sandbox|Agent 沙箱]]

→ [[raw/articles/ai-agent-存储选型curvine-如何在-eks-上支撑万级agent运行|原文存档]]
