---
title: 云原生 AI 图像视频素材设计平台架构
created: 2026-09-01
updated: 2026-09-07
type: entity
tags: [ai, cloud-native, architecture, creative, video, pipeline]
sources: [raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 云原生 AI 图像视频素材设计平台架构

## 核心问题

设计师日常使用 Midjourney、Stable Diffusion、Flux 等多种 AI 工具生成图像，但工具散落在不同平台，素材需手动下载、整理、导入设计软件，且各平台计费独立，月度订阅成本高。客户需要的是一个统一工作台：调用不同 AI 模型，在同一画布上完成从创意到成品的全流程，且支持私有化部署以满足素材合规要求。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

Open Gallery 是基于 AWS 云原生技术栈构建的解决方案，需求明确为：多模型统一接入（Claude、OpenAI、Ollama、ComfyUI 即插即用）、画布式交互（无限画布支持故事板与系列海报创作）、对话式编辑（自然语言指令直接修改画布元素）、弹性 GPU 资源（缩到零或自动扩出）、灵活部署（笔记本与 EKS 集群同一套代码）。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 三层分离架构

核心设计决策是将 GPU 推理服务与业务后端拆开。GPU 实例成本是 CPU 实例的 10 倍以上，若推理与业务绑定则每个后端副本都需跑在 GPU 机器上，造成巨大浪费；且两者的伸缩节奏完全不同——后端按在线用户数扩容，推理按队列深度扩容。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

架构分为三层：**前端层**用 React 19 + Vite 构建，状态管理选 Zustand，画布渲染集成 Excalidraw 和 tldraw，前后端通过 HTTP + WebSocket 双通道通信（HTTP 处理 CRUD，WebSocket 推送推理进度）。**后端层**是 Python FastAPI，选择 FastAPI 而非 Flask/Django 是因为异步性能——图像生成是典型 I/O 密集场景，后端内部拆为 API 路由层、服务层、工具层（对接各种生成后端的适配器）。**推理层**以 ComfyUI 为核心，运行在 NVIDIA GPU 容器中，利用其节点式工作流引擎和社区生态（ControlNet、LayerStyle、VideoHelperSuite 等）组合生成管线。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 三阶段实施路径

项目分三阶段推进，每阶段解决一个核心矛盾：

- **"能不能用"**：打通用户输入到图片生成的完整链路。设计 TOML 配置体系，所有模型提供商（Anthropic、OpenAI、Replicate、ComfyUI）通过配置声明而非硬编码，ConfigService 支持热加载。数据库层采用[[orchestrator-worker-architecture]]适配器模式，同时支持 SQLite（本地开发）和 DynamoDB（生产），通过 adapter 接口运行时切换。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

- **"能不能扛住"**：将 ComfyUI 容器化并部署到 Kubernetes 集群。关键设计是模型存储与计算分离——模型文件不烘焙到 Docker 镜像，而是通过 S3 CSI Driver 挂载为 PVC，ComfyUI 启动时从挂载点读取，更新模型只需上传 S3，无需重建镜像。ComfyUI 通过 ClusterIP Service 暴露 8188 端口，后端通过环境变量对接，实现独立部署和伸缩。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

- **"能不能省钱"**：实现弹性伸缩与冷启动优化，解决 GPU 实例按小时计费的[[ai-cost-optimization-framework]]问题。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 两级联动弹性伸缩

弹性伸缩采用两级联动设计：**Pod 级**由 KEDA 驱动，**Node 级**由 Karpenter 驱动，两者通过 Kubernetes 原生调度自动协同。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

传统 HPA 基于 CPU/内存利用率伸缩在 GPU 推理场景下不准确——ComfyUI Pod 可能 GPU 利用率已满但 CPU 利用率很低。解决方案是基于业务语义的伸缩指标：每个 ComfyUI Pod 部署 metrics sidecar，每 10 秒采集队列待处理任务数（QueuePending），上报到 CloudWatch，KEDA 通过 CloudWatch 触发器监听该指标。参数配置为：探测间隔 30 秒（避免 thrashing）、扩容稳定窗口 0 秒（零延迟扩容）、缩容稳定窗口 300 秒（防止频繁缩扩）、伸缩范围 1～10 Pod。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

当 KEDA 扩出新 Pod 但集群无可用 GPU 节点时，Pod 处于 Pending 状态，Karpenter 检测到 `nvidia.com/gpu: 1` 资源请求后自动启动匹配的 EC2 实例（g5/g6e 系列）。负载下降后节点空闲超过 2 分钟自动回收（WhenEmpty consolidation policy）。Karpenter 支持 On-Demand 和 Spot 实例混合调度：批量生成任务优先用 Spot（成本低 60-70%），实时生成请求用 On-Demand 保证可用性。端到端流程约 2-3 分钟。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 多模型路由与 Agent 编排

平台集成了多种图像生成后端——ComfyUI（复杂工作流）、Replicate（快速原型）、OpenAI（通用场景）。不同任务适合不同后端：批量系列海报走 ComfyUI（复用工作流模板），单张概念图走 Replicate（启动快无预热），对话式编辑用 Flux Kontext。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

通过 Strands Agents 框架构建编排层，用户用自然语言描述意图，Agent 根据任务类型、复杂度和各后端负载情况自动路由到最合适的生成工具，并自动构建最优 prompt，大幅降低非 prompt engineering 专业设计师的使用门槛。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 冷热分层存储架构

AI 平台的存储需求特殊：模型文件大（单个几 GB）、读多写少；应用数据小、读写频繁。采用三层存储策略：^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

- **热层**（NVMe SSD）：存放正在被 GPU 使用的模型权重，通过 DaemonSet 从 S3 预同步到本地，ComfyUI 通过 hostPath 直接读取，追求极致 I/O 性能
- **温层**（S3 + CSI Driver）：存放完整模型库，配置 20 秒 metadata-ttl 缓存平衡一致性和性能，新增模型上传 S3 后所有节点自动可见
- **冷层**（DynamoDB / SQLite）：存放结构化应用数据（画布状态、聊天会话、用户配置），DynamoDB 的 serverless 特性免去运维负担^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

用户上传文件通过 Mountpoint 以读写模式挂载到后端 Pod，直接落入 S3，解决 Pod 重启数据丢失问题并为 CDN 加速预留接口。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 关键经验总结

1. GPU 推理弹性伸缩的关键不在"扩"而在"快"——KEDA + Karpenter 解决"扩多少"，SOCI 懒加载、NVMe 预热、Readiness Probe 三板斧决定冷启动体验
2. 适配器模式在"一套代码多场景部署"中被低估——SQLite/DynamoDB 适配、TOML 配置抽象看似过度设计，实则大幅降低上手门槛和运维成本
3. 不要试图重新发明推理引擎——ComfyUI 社区生态足够强大，将其作为黑盒推理微服务来编排，精力应花在其之上的产品体验和之下的基础设施上^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

## 相关概念

- [[cloud-ai-infrastructure]] — 云原生 AI 基础设施的整体范畴
- [[ai-cost-optimization-framework]] — GPU 资源成本优化的方法论
- [[orchestrator-worker-architecture]] — 编排器-工作器架构模式，Open Gallery 的多模型路由即此模式的应用
- [[mlops-engineering-methodology]] — MLOps 工程方法论，涵盖模型部署与运维

→ [[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践|原文存档]]
