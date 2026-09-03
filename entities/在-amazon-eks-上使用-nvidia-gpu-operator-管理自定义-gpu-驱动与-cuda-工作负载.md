---

title: "在 Amazon EKS 上使用 NVIDIA GPU Operator 管理 GPU 驱动与 CUDA"
created: 2026-06-10
updated: 2026-06-30
tags: [aws, fine-tuning, k8s, memory, mlops, nvidia, observability, tool-use, vision]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载
---

# 在 Amazon EKS 上使用 NVIDIA GPU Operator 管理自定义 GPU 驱动与 CUDA 工作负载

→ [[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载|原文存档]] ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]

## 深度分析

在 Amazon EKS 上使用 NVIDIA GPU Operator 管理自定义 GPU 驱动与 CUDA 工作负载 涉及aws领域的核心技术议题。 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
### 核心观点
1. # 在 Amazon EKS 上使用 NVIDIA GPU Operator 管理自定义 GPU 驱动与 CUDA 工作负载 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
摘要：在 EKS 上结合 GPU Operator 与 Kiro+EKS MCP，管理自定义 GPU 驱动和 CUDA 工作负载。 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
2. 对平台团队来说，难点往往不只是“把 GPU 节点加进集群”，而是如何在可控的运维模型下同时满足几个要求：使用特定 GPU 实例类型、固定 NVIDIA driver 版本、让业务容器使用指定 CUDA runtime、支持节点自动扩缩容，并在日常排障中快速理解集群状态。 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
3. 本文基于一次在 Amazon EKS 上完成的实际部署与验证，介绍如何使用 EKS GPU 节点组、EKS managed node group、NVIDIA GPU Operator，以及 Kiro + AWS MCP 的 AI 运维方式，落地以下组合： ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
Amazon EKS 1. ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
4. 04 EKS optimized AMI ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
Amazon EC2 G5 / NVIDIA A10G ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
NVIDIA GPU Operator v26. ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
5. 1 ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]
NVIDIA driver 535. ^[raw/articles/在-amazon-eks-上使用-nvidia-gpu-operator-管理自定义-gpu-驱动与-cuda-工作负载.md]

### 关联实体

- [[entities/scale-robot-reinforcement-learning-with-nvidia-isaac-lab-on-]]
- [[entities/nvidia-isaac-lab-sagemaker-robot-rl-humanoid]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]
- [[entities/end-to-end-encrypted-ml-inference-with-amazon-sagemaker-ai-a]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]

