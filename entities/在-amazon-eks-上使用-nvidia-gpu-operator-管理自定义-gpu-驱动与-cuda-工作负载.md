---

title: "在 Amazon EKS 上使用 NVIDIA GPU Operator 管理 GPU 驱动与 CUDA"
created: 2026-06-10
updated: 2026-11-20
tags: [aws, fine-tuning, k8s, memory, mlops, nvidia, observability, tool-use, vision]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 在 Amazon EKS 上使用 NVIDIA GPU Operator 管理 GPU 驱动与 CUDA

## 摘要

这篇 AWS 中国博客记录了一次在 Amazon EKS 1.34 上完成的实际部署与验证：在**不预装 NVIDIA driver** 的 Ubuntu 22.04 EKS optimized AMI 节点上，用 NVIDIA GPU Operator v26.3.1 自动安装指定版本 driver（535.309.01），并在 EC2 G5（NVIDIA A10G）实例上跑通 CUDA 12.2 workload。文章同时验证了 self-managed 与 managed 两种节点组的等价性，并引入一个不存在的旧 driver 版本（535.15.04）观察失败模式，最后给出了"EKS 管节点组生命周期、GPU Operator 管节点软件栈、业务容器管 CUDA runtime"的三层职责划分。

## 核心要点

- Pod 想通过 `nvidia.com/gpu` 申请 GPU，前提是节点具备完整 NVIDIA 软件栈：kernel 可加载 driver、容器运行时集成 NVIDIA Container Toolkit、集群运行 device plugin、节点正确打上 GPU 标签 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:59-65]
- 预装 driver 的 accelerated AMI 在需要自定义 driver 版本时会与 GPU Operator 职责重叠，升级与排障边界不清；推荐 AMI 不装 driver/CUDA Toolkit，全部交给 GPU Operator ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:69-71]
- 同一 Ubuntu 22.04 AMI、相同 G5 实例与 Operator 配置下，self-managed 与 managed node group 均验证通过；无特殊 ASG 需求时 managed node group 应作为生产默认 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:75-92]
- driver 版本不能只看"理论兼容"：必须确认 GPU Operator 支持矩阵和 NVIDIA Container Registry 中存在对应 driver container image，535.15.04 就因镜像不存在而进入 ImagePullBackOff ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:132-134]
- 一个节点只能有一个 NVIDIA driver 版本；需要不同 driver 时应拆分 GPU 节点池并用 label/taint/nodeSelector 隔离，CUDA runtime 则可以随业务镜像自由变化 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:180-185]
- 集群必须保留至少一个普通非 GPU 节点组：GPU 节点带 `nvidia.com/gpu=true:NoSchedule` taint，GPU Operator 控制面、CoreDNS、监控组件若无 toleration 会 Pending ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:110]
- Kiro + AWS MCP 的 AI 运维模式（只读巡检优先）能把 EKS/K8s/节点层状态串联起来，适合 GPU 平台巡检、排障与知识沉淀 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:98-106]

## 深度分析

### GPU Operator 的组件架构与分层职责

文章给出的部署拓扑是：Operator 控制面（gpu-operator controller、node-feature-discovery master/gc）运行在**普通节点组**，而一组 GPU 相关 DaemonSet 运行在 GPU 节点组上——nvidia-driver-daemonset、nvidia-container-toolkit-daemonset、nvidia-device-plugin-daemonset、gpu-feature-discovery、nvidia-dcgm-exporter 和 validator pods ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:140-158]。GPU Operator 不是 EKS 托管插件，而是通过 Helm 装进集群的一组普通 Kubernetes workload ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:140]。

实际部署中 Operator 自动完成了六件事：安装指定版本 driver、安装 Container Toolkit、部署 device plugin 暴露 `nvidia.com/gpu`、运行 GPU Feature Discovery 打标签、部署 DCGM Exporter 出监控指标、运行 validator 做端到端验证 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:205-212]。由此带来的收益是新 GPU 节点加入即自动装好软件栈、driver 版本可用 Helm values/GitOps 固定、AMI 无需内置 CUDA Toolkit、GPU 节点初始化流程可标准化可审计 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:216-222]。文章总结的三方边界是：EKS 管控制面/节点组/EC2 生命周期，GPU Operator 管节点上的 NVIDIA 软件栈，业务 workload 自选 CUDA runtime ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:226]。

### Driver 安装生命周期与"节点级单版本"约束

GPU Operator 安装 driver 的机制值得细看：nvidia-driver-daemonset 调度到 GPU 节点后，运行的是一个具备 host 访问能力的**特权驱动容器**，它使用预编译 kernel module 或 DKMS 编译，将 nvidia.ko 等模块加载进 host kernel，并把 libcuda.so 等 userspace 库放到 host 的 `/run/nvidia/driver/`；业务 Pod 再通过 NVIDIA Container Toolkit 借用这套 host 库访问 GPU ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:160-178]。也就是说，Operator 的驱动容器本质上仍是在给 host 装 driver，装完后同节点所有 GPU workload 共用。

这导出一个常被忽略的硬约束：**一个节点只能有一个 driver 版本**——不可能让 Pod A 用 535、Pod B 用 580。需要多 driver 并存时，唯一正解是拆分 GPU 节点池并用调度策略隔离；而 CUDA runtime 可以随容器镜像变化，driver 则是节点级共享能力 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:180-185]。driver container image（如 `nvcr.io/nvidia/driver:535.309.01-ubuntu22.04`）必须同时匹配 driver 版本与节点 OS，它与业务 CUDA 镜像、业务应用镜像三者职责互不相同 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:189-201]。文章的负向验证很有价值：535.15.04 虽理论兼容 CUDA 12.2，但其 ubuntu22.04 driver image 在 registry 中不存在，DaemonSet 直接 ImagePullBackOff——失败模式未损坏节点，但说明生产配置必须落在支持矩阵内 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:203]。

### CUDA 版本管理与节点组选型

文章把 CUDA 的管理位置钉死在**业务容器镜像**：节点 driver 只需满足 CUDA runtime 的最低版本要求，节点 AMI 不应安装 CUDA Toolkit ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:124-132]。客户坚持固定 driver/CUDA 版本的动因包括：业务镜像已基于特定 CUDA 构建、ML 框架/商业软件有认证组合要求、生产需要可重复部署与审计回滚、不同团队要用不同 CUDA 镜像 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:114-120]。

节点组选型上，self-managed 与 managed 的差异只在**生命周期责任**（托管升级、EKS 参与健康管理），两者都支持自定义 AMI 且 GPU Operator 兼容性等价 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:75-86]。self-managed 仍适用于已有成熟 ASG 体系、复杂 Spot/On-Demand 混合策略或完全自定义启动流程的场景 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:92]。一个实践细节：实验中 Ubuntu 22.04 EKS AMI 的 kubelet 为 1.32 而控制面为 1.34，仍在 Kubernetes version skew policy 范围内 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:230]。另一个调度陷阱是：只有 GPU 节点（带 NoSchedule taint）的集群会让无 toleration 的 Operator 控制面和系统组件 Pending，因此生产上必须保留普通节点组承载这些组件 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:246]。文章未涉及 MIG 与 cluster-autoscaler/GPULifecycle，但给出的最终生产组合（managed node group + Ubuntu 22.04 AMI + G5 + Operator v26.3.1 + driver 535.309.01 + CUDA 12.2 镜像 + 普通节点组）可作基线模板；若必须用 Ubuntu 24.04，则应换 R570/R580 driver 分支，CUDA 12.2 仍可运行（driver 向后兼容旧 runtime）^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md:264-276]。

## 实践启示

1. **把 driver 选型当作"三重匹配"问题**：版本号理论兼容 + GPU Operator 支持矩阵覆盖 + NVIDIA Container Registry 真实存在对应 OS 的 driver image tag，三者缺一就会重演 535.15.04 的 ImagePullBackOff。
2. **AMI 只做 OS，软件栈交给 Operator**：不要在 accelerated AMI 里预装 driver/CUDA Toolkit，避免"AMI 管 driver、Operator 也管 driver"的职责重叠；driver 版本通过 Helm values 或 GitOps 声明。
3. **CUDA 放镜像里，driver 放节点上**：业务镜像固定 CUDA runtime（如 12.2），节点 driver 只需满足其最低要求；CUDA Toolkit 不要进 AMI。
4. **永远保留一个普通节点组**：承载 GPU Operator 控制面、CoreDNS、监控与日志组件，防止 NoSchedule taint 导致系统 Pod Pending；用 taint 顺带防止普通 workload 误调度到昂贵 GPU 节点。
5. **多 driver 版本需求 → 多节点池**：单节点只能一个 driver，按业务团队/版本拆 GPU node pool 并用 label + taint + nodeSelector 隔离。
6. **用 AI 运维降低多层排障成本**：GPU 栈跨 EKS/EC2/AMI/kernel/driver/runtime/DaemonSet 十余层，Kiro + EKS MCP 的自然语言只读巡检适合日常，写入模式需授权收敛。

## 相关实体

- 同题异语种孪生页（归并候选）：[[entities/eks-gpu-operator-custom-driver-cuda-workload]]
- Kiro/AI 运维 EKS 实践：[[entities/ai-eks-upgrade-kiro-cli-agent-paradigm-2026]]、[[entities/kiro-cli-fluentbit-logging-solution-eks-s3-parquet-comparison]]
- EKS MCP Server（文中 AI 运维的基础设施）：[[entities/amazon-eks-mcp-server]]
- EKS 版本升级管理：[[entities/规划-amazon-eks-从-132-升级到-135关键变更识别与逐版本实施路径]]

→ [[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载|原文存档]]
