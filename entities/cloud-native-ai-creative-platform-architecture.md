---
title: 云原生 AI 图像视频素材设计平台架构
created: 2026-09-01
updated: 2026-10-20
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

## 深度分析

### 三层分离省下的不是机器，是两套节奏

拆开 GPU 推理与业务后端，本质是成本结构的重新分配。GPU 单价是 CPU 的十倍以上，这个倍数决定了浪费被放大的位置：只要两者同进程，每个后端副本都必然带一张卡，而副本数由在线设计师人数驱动、不是待处理任务数——两个数字常差一个量级，差额即纯浪费；拆开后两者的伸缩信号也才各自成立。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

代价是：函数调用变成跨服务 HTTP，异常语义退化。ComfyUI 某个节点失败，在调用方看来只是一个超时或非 200，卡在哪一步必须靠另采的进度事件还原——这才是双通道的真正理由，WebSocket 不是体验优化，而是后端对推理层唯一的可观测性来源。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

### 队列指标选对了，延迟仍锁在节点供给上

HPA 失效的原因很具体：ComfyUI 满载时 GPU 打满而 CPU 仍处低位，按 CPU 判断会得出"不必扩容"的相反结论。QueuePending 之所以正确，是因为它直接等于用户可感知的等待量，且与硬件型号解耦——换更快的卡，参数不必重调。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

失败模式不在指标，而在两级联动的串行依赖。Pod 扩容只把状态推进到 Pending，节点供给、镜像拉取、模型预热全串在这条路上，2–3 分钟里大头是 [[entities/reducing-container-cold-start-times-using-soci-index-on-dlam|镜像与权重就位]]，不是容器启动。窗口内队列继续上涨，KEDA 依据的却是扩容前的旧深度，容易过冲：一次尖峰常换来一批按整节点时长计费的空转实例。0 秒扩 / 300 秒缩的非对称配置是为"不排队"付的保险费，minReplicas=1 则用一个常驻 GPU 给最坏冷启动封顶——都是花钱买确定性。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

### 多模型路由是能力匹配，不是性价比排序

路由层的常见误读是"把简单的活派给便宜的模型"，原文的依据其实是任务形状：批量海报复用工作流模板，价值在模板而非单张成本，故走 ComfyUI；单张概念图无需预热，Replicate 启动开销近零；对话式编辑要理解指令与图像的指代关系，只有 Flux Kontext 这类模型能做。第三项已不是贵贱取舍，而是可做与不可做——价格表表达不了这层判断。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

用 Agent 承担判断，收益是把"意图 → 后端"从硬编码规则变成可扩展推理，并补上设计师不擅长的 prompt 构建；代价是路由成了一次 LLM 调用，多一跳延迟、一笔计费，且决策不可复现——同一句描述可能落到不同后端，回归测试与成本归因都变难。原文只写了正确路径，没交代失败路径，这恰是上生产后最该补的：置信度不足时应退回静态规则，而非报错。[[entities/state-of-routing-in-model-serving|模型服务路由]]的实践也表明，可审计的静态策略与灵活的路由智能通常分层共存。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

### 冷热分层按访问形态切，不按数据重要性切

三层的依据是"对象大小 × 读频 × 一致性要求"的组合，而非数据重要程度。模型权重几 GB、读多写少、加载延迟直接落在出图时间里，于是 DaemonSet 预同步到 NVMe、hostPath 直读；S3 存全量模型库，靠 20 秒 metadata-ttl 在新模型可见性与元数据请求量之间取平衡；应用数据小而碎、读写频繁，交给 DynamoDB 或本地 SQLite。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

关键在于预热的因果：DaemonSet 没有消除加载时间，只是把它从用户等待路径搬到节点供给路径，与 Karpenter 拉起实例的时间重叠——本来就要花的时间，叠起来才算省。故分层有个隐含前提：节点生命周期必须长于预热成本。若 Spot 中断频繁，NVMe 预热反复作废，热层反成净损耗。上传侧用 Mountpoint 读写挂载换来重启不丢数据，代价是写路径受对象存储延迟约束。

### 天真实现会在哪里先断

抽掉这些决策，最自然的实现是镜像内含模型、单进程同跑 Web 与推理、按 CPU 伸缩、素材存本地盘。它会在四处依次断裂：模型更新要重建 10GB 级镜像，发布被镜像仓库吞吐绑定；GPU 7×24 常开，空闲成本吃掉预算；CPU 指标看不见 GPU 排队，高峰不扩、低谷不缩；本地盘存素材，Pod 重建即丢。三阶段划分正是按这四个断点排序——先通、再扛、后省。顺序说明弹性是唯一可推迟的问题，存储与计算分离必须提前做——它同时是发布效率、伸缩能力与数据持久性的前置条件，也是[[entities/from-kubernetes-dev-setup-to-production-what-actually-change|从本地开发走到生产]]最常变的接缝。^[raw/articles/构建云原生-ai-图像视频素材设计平台从零到生产的架构实践.md]

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
