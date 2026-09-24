---
title: "A Tale of Two Flink Autoscalers — Netflix 流处理自动扩缩容演进"
type: entity
source_url: "https://netflixtechblog.com/a-tale-of-two-flink-autoscalers-e9f6a1b1492b"
ingested: 2026-09-23
source_published: 2026-08-21
feed_name: "Netflix TechBlog"
source: "rss"
sha256: 6af0d020f43636a0acbfc40db434bb957189687bfa806b1bd77948054230750a
---

# A Tale of Two Flink Autoscalers — Netflix 流处理自动扩缩容演进

> Netflix 自 2017 年运行 Apache Flink，2026 年已有 30,000+ Flink 作业跨多个 AWS region。本文记录其两代 autoscaler 的架构演进：从外部观察式（Mantis + Atlas 指标流）迁移到基于 Flink OSS Autoscaler 的作业内推理（TPR 算法 + Temporal workflow-per-job），含迁移决策逻辑、工程缺口修复与可泛化的三条经验教训。

## 第一代：外部观察式 autoscaler（2019）

第一代 autoscaler 形态是 Mantis 上的一个流处理作业，消费 Atlas 遥测平台提供的集群级指标（CPU、网络、Kafka lag、input-rate、consume-rate）。扩容决策综合 lag 派生的追赶时间、CPU/网络利用率阈值、历史性能观测和近期输入速率回归。因为独立于 Flink 平台运行，它不受 Flink 自身问题影响，且每个 autoscaler 节点只处理一部分 Flink 作业的指标，无需自定义分片或协调逻辑。该系统在数千条托管 pipeline 上可靠削减了 25–45% 资源消耗。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

外部观察有天花板：它通过粗粒度容器指标推断整个集群，只调节 TaskManager 总数一个旋钮，作业内所有 operator 一起伸缩。这适配简单的单 operator pipeline，但无法推理多 operator、有状态 DAG（Ads、推荐、游戏团队的定制作业）。更深的依赖问题是：autoscaler 的质量取决于底层外部系统的指标——一次网络迁移悄悄改变了流量上报方式，Atlas 部分指标失准，缺口直到生产事故才暴露。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

## 第二代：作业内推理 + OSS Autoscaler 采纳

重新评估 build-vs-buy 时，Apache Flink Autoscaler 已成熟。核心思想是估算每个 operator 的 true processing rate（TPR）：作业完全繁忙时可持续的吞吐。Flink 按 subtask 上报每秒用于实际工作的时间占比（区别于 backpressure 和 idle），用观测吞吐除以繁忙占比外推满载容量——一个每秒处理 700 条记录、70% 时间繁忙的 operator，TPR = 700/0.7 = 1000 records/sec。autoscaler 从 source 起遍历作业图，基于每个 operator 的 TPR、输入输出比和目标利用率计算每个 vertex 所需并行度，使无 operator 成为瓶颈。OSS 版可伸缩第一代搞不定的有状态多 operator 作业，且每个作业可携带自己的配置（稳定期、阈值等）。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

Netflix 的关键工程改造：
- 控制面解耦：OSS autoscaler 原生位于 Flink K8s Operator 内，Netflix Flink 平台运行自研控制面。社区将核心逻辑重构为独立库（context/state store/event handler/realizer 四个通用接口），使其可插入内部生态。
- Temporal workflow-per-job 编排：Spring Boot 服务用 Temporal durable workflow 引擎，每作业一个长运行 workflow（拉取 per-vertex 指标、运行 OSS 评估算法、通过 realizer 执行扩缩决策）。单批循环评估曾因一个慢作业拖垮全部作业的指标采集；per-job workflow 隔离了故障半径。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

三个 "社区可用 → Netflix 规模可用" 的工程缺口：
1. 高并行度指标采集：大作业从 JobManager 拉指标成为瓶颈（部分根因在 Flink runtime）。修复：JobManager 缓存瞬态指标名并一次性清理、加服务端过滤，使 autoscaler 支持到 3000 subtasks（此前 ~1000 即吃力）。部分改动进入内部 fork，FLINK-36172 等已回馈上游。^[raw/articles/a-tale-of-two-flink-autoscalers.md]
2. 保持 forward chaining：由 forward 边连接的两个 vertex 必须同并行度（记录经内存固定本地通道移交）；单独扩一个会静默把该边转成 network shuffle。内部 fork 检测 forward 连通子图并作为整体伸缩。^[raw/articles/a-tale-of-two-flink-autoscalers.md]
3. 尊重 sink 上限：部分 sink 写入容量有限，fork 增加异步 sink backpressure 检测，防止 autoscaler 把作业扩进无法吸收的 sink。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

realizer 执行前运行安全检查：区域疏散（公司级 region failover）期间拒绝缩容、验证新集群磁盘足以容纳 checkpoint state、为大集群添加 standby buffer。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

## 成效与调参

OSS autoscaler 去年在 Netflix 对定制作业 GA。客户遥测与日志团队年化 Flink 计算支出降低 58%，年省约 110 万美元。三个驱动因素：动态适配昼夜/工作日-周末流量周期（静态供给必须按峰值）、持续调整容量（无需团队在性能优化或节假日后手动调优）、统一容器规格带来更优 bin-packing 和更细的伸缩粒度。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

过度缩容也是陷阱：切得过深 CPU 饱和、lag 尖峰，而系统因指标窗口和稳定期需在每次重启后重建而无法即时反应。Netflix 现运行目标利用率 0.45（低于社区默认 0.7），以少量效率换取大规模有状态作业的稳定性——更少更平稳的 rescale 值得这点边际成本。当前有状态作业扩缩的最大成本已不在 scaler 逻辑，而在重启与状态恢复本身；Flink 2 的 disaggregated state 架构（状态存外部存储）可大幅降低恢复对状态总大小的依赖，Netflix 已开始支持 Flink 2.2 并计划试验。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

## 可泛化的三条经验

1. **指标选择比算法精妙更重要**：最有价值的调试很少关于扩缩数学，而是关于信任哪个信号。先理解指标再调算法。
2. **设合理默认值但留调优空间**：托管作业足够相似以致一个好默认值覆盖大多数；但强加单一配置会惩罚不适合的作业，需配 per-job override 并刻意隐藏需要深度专业知识的旋钮——大多数团队永远不该操心 autoscaler。
3. **采纳，再扩展（Adopt, then extend）**：2019 年因无成熟方案而自研；当强大社区项目出现时，正确动作既非永久守护自研投资、也非一夜替换，而是对新负载采纳、回馈修复、规划审慎迁移。^[raw/articles/a-tale-of-two-flink-autoscalers.md]

## 相关实体

- [[entities/from-silos-to-service-topology-why-netflix-built-a-real-time|Netflix 实时服务拓扑]]（同源 Netflix infra 系列）
- [[entities/netflix-kueue-batch-compute-migration|Netflix Kueue 批计算迁移]]（同属 Netflix 计算/扩缩容基础设施主题）

→ [[raw/articles/a-tale-of-two-flink-autoscalers|原文存档]]
