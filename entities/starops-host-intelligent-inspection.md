---
title: "STAROps Host Intelligent Inspection — 24/7 AI Doctor for ECS"
created: 2026-08-01
updated: 2026-10-08
type: entity
tags: [STAROps, host-inspection, Alibaba-Cloud, SysOM, ECS, SRE, AIOps, kernel, observability]
sources: [raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026, raw/articles/sysom-巡检-skill-一键锁定根因]
confidence: 0.6
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# STAROps Host Intelligent Inspection

STAROps 主机智能巡检是阿里云全域智能运维平台 [[entities/alibaba-agentic-cloud|STAROps]] 的一项核心能力，面向主机（ECS）基础设施层提供「自动体检 + AI 医生」式巡检——从事后救火转向事前防护。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

STAROps 的名字代表运维四层理念：全域感知（Sense）、目标导向（Target）、自主运维（Autonomy）、业务韧性（Resilience）。主机智能巡检正是「业务韧性」的关键实现——自动为每台主机做一次覆盖 **CPU、内存、磁盘、网络、GPU、内核、硬件七大领域、超过 50 个检查项** 的全面检查。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 与 RUM 智能巡检的姊妹关系

主机智能巡检与 [[entities/starops-rum-intelligent-inspection|STAROps RUM Intelligent Inspection]] 是同一平台下的两个互补能力：RUM 巡检关注**用户体验层**（页面、业务路径、设备段的体验退化灰色地带），主机智能巡检关注**基础设施层**（主机内核态与硬件层的隐患）。二者共同构成 STAROps 巡检能力矩阵。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 分层架构：STAROps 编排 + SysOM 深度诊断

STAROps 与阿里云操作系统控制台的运维组件 **SysOM** 分工协作：

- **STAROps**：面向用户的统一入口和智能运维大脑——自然语言下发巡检/排障，负责编排流程、关联全域数据、生成分级报告
- **SysOM**：操作系统层专业诊断能力提供方——提供 memgraph（内存分析）、diskanalysis（磁盘诊断）等专项诊断算子

调用关系：主机智能巡检发现异常 → STAROps 作为编排者并发调用 SysOM 底层诊断工具做深度定位 → 内核级结论汇总进巡检报告。一个是「望闻问切、统筹开方」，一个是「深入病灶、精准化验」。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 覆盖深度：内核态 + 硬件层

普通监控只看 CPU/内存/磁盘「大众指标」，主机智能巡检把探头伸进更深层：^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

- **内核事件**：softlockup（业务线程饿死）、hungtask（进程僵住超两分钟）、RCU stall（内核死锁前兆）、conntrack 表满（新连接被悄悄丢弃）
- **硬件层**：处理器退化 MCE 错误、内存条即将失效 ECC 错误、硬盘濒临故障 SMART 告警、光模块老化丢包 CRC 错误

## 诊断证据链模式

一次巡检 = 发现 + 定位 + 根因 + 方案。以「业务突然变慢」为例：发现「iowait 持续偏高」→ 定位「MySQL 进程频繁 fsync」→ 根因「sync_binlog=1 让每次事务提交都要等磁盘」→ 直接给可执行方案。这背后是 STAROps 的 AI 诊断引擎，把资深 SRE 几小时甚至几天的排查功力压进一次自动巡检。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 真实案例

1. **大促前 Slab 内存预警**：SReclaimable 异常升高占内存 30%，AI 诊断出日志采集 Agent 遍历 /proc/*/fd 产生海量 dentry 缓存，调整采集策略后释放 8GB 内核内存，避开大促 OOM 风险
2. **conntrack 表满**：使用率 92% + 「table full, dropping packet」，AI 判定 K8s 短连接场景 conntrack_max 默认值不足，调参 + keepalive 后超时消失
3. **数据库慢查询**：磁盘写延迟 5ms→200ms，AI 诊断为 O_SYNC 叠加同盘混部 IOPS 争抢，分离 WAL 与数据盘后回落到 5ms

## Agent 化五步巡检流程

用户一句自然语言「/主机巡检，对当前 workspace 的所有 ecs 做一次巡检」即可触发：Agent 读取 Skill 补齐参数（region/UID/workspace/time_range）→ 调用 SLS 异常事件查询扫描（演示：3 类 CRITICAL、102 台实例、1380 事件）→ 对关键实例并发调用 SysOM（memgraph/diskanalysis）→ 汇总分级报告（🔴 P0 需立即处置 + 🟢 P2 持续观察）。这是 **Skill + Agent + SysOM 三件套**的落地形态。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 深度分析

### 「慢性病→急症」模型与巡检的时间维度价值

原文的核心洞察是把基础设施故障重新分类：内存连续爬升直到 OOM、conntrack 表悄悄逼近满载、光模块老化直到重传率飙升——大多数故障是慢性病累积成的急症，而传统监控只在指标触顶时告警（为时已晚），人工巡检受精力限制无法高频深度排查。主机智能巡检的价值不在单点技术而在时间维度前移：七大领域 50+ 检查项把「病发前」的信号（SReclaimable 异常爬升、ECC 错误、SMART 告警）变成可消费的报告。这与其姊妹能力 [[entities/starops-rum-intelligent-inspection|RUM 智能巡检]]（体验退化灰色地带）构成同一时间哲学在两层栈上的实现。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

### STAROps/SysOM 分层是「编排智能」与「领域诊断」的解耦范例

架构上值得抽出的模式：STAROps 做统一入口、自然语言下发、编排流程、关联全域数据、生成分级报告；SysOM 提供内核态专项算子（memgraph 内存分析、diskanalysis 磁盘诊断）。调用链是巡检发现异常 → 编排者并发调用深度诊断 → 内核级结论汇总进报告。这个分工与 SRE 团队的自然分工同构——「望闻问切、统筹开方」与「深入病灶、精准化验」是两种能力，强行合并会让编排层被诊断细节淹没。Agent 化五步流程（自然语言指令 → Skill 补参 → SLS 异常扫描 → 并发 SysOM 诊断 → 🔴P0/🟢P2 分级报告）则是这个解耦在对话式运维入口的落地。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

### Skill 开源把巡检从产品能力变成可移植的专家经验

2026-07-28 开源的 SysOM 巡检 Skill（19 项巡检执行项，`npx skills add aliyun/alibabacloud-aiops-skills`）把同一套能力以两种形态存在：向人是开箱即用的巡检产品，向 Agent 是可调用的技能包（支持 Qoder、Claude Code 等宿主）。对照评测显示接入 Skill 后 Agent 在对话轮次、工具调用次数、耗时上大幅下降，且提升集中在内核专家判断类场景（socket 缓冲区泄漏、vmalloc 异常、memcg 误判）——越依赖专家经验的问题，Skill 价值越大。内部评测异常识别准确率 80%+、高风险项误判率 0。37.4 秒完成 19 项巡检并把 40 分钟的人工排障（python 进程独占 12.95GB 匿名内存）压到 5 分钟内，验证了「巡检→诊断→处置」闭环的实际收益。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]

### 证据链完整性是 AI 运维区别于告警系统的分界线

三个真实案例共有的结构是「发现 → 定位 → 根因 → 方案」四段完整证据链：iowait 偏高 → MySQL fsync 频繁 → sync_binlog=1 → 给出参数方案；SReclaimable 占 30% → 日志 Agent 遍历 /proc/*/fd 产生 dentry 缓存 → 调整采集策略释放 8GB。对比传统告警只回答「有没有异常」，AI 运维的分界线在于能否回答一线同学的三问：严重程度如何？根因在哪？下一步谁做什么？每条建议经线上验证、按紧急/中期/长期三步走、每项巡检配故障注入测试用例，是把「AI 说的」变成可信结论的工程保障。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 实践启示

1. 自建巡检体系先按「慢性病指标」列清单：conntrack 使用率、Slab/SReclaimable 占比、ECC/SMART/CRC 硬件信号，比盯瞬时 CPU 峰值更有预防价值。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]
2. 运维 Agent 架构采用编排层/诊断层解耦：让编排层消费结构化诊断结论，不要把内核分析逻辑写进编排 prompt。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]
3. 把已验证的排查路径封装成 Skill 供 Agent 调用，收益最大的场景是内核专家判断类问题；通用 LLM 自己「想想看」在这类问题上准确率明显更低。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]
4. 巡检命中高风险项后自动串联深度诊断（memgraph/diskanalysis），报告必须给到「哪个进程、多少量、占比例、可排除什么」的根因级结论才算闭环。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]
5. K8s 短连接密集场景把 conntrack_max 默认值当风险项主动巡检，配合 keepalive，可避免「新连接被悄悄丢弃」这类无告警故障。^[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026.md]

## 第 2 来源 — SysOM 巡检 Skill 开源（2026-07-28）

阿里云操作系统控制台在开源 SysOM 诊断 Skill 之后，将 **SysOM 巡检 Skill** 一并开源，两者构成「巡检发现问题 → 诊断定位根因 → 给出处置建议」的完整闭环，后续一同纳入 SysOM 技能包。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]

互补角度 4 条（相对第 1 来源）：

1. **19 项巡检执行项的技能化实现**：巡检 Skill 执行 19 项巡检（18 项 Normal + 1 项 Error 命中），把第 1 来源「50+ 检查项」的概念框架落到可安装的 skill 包（`npx skills add aliyun/alibabacloud-aiops-skills --skill alibabacloud-alinux-sysom-inspection`），支持 Qoder、Claude Code 等 Agent 宿主加载。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]
2. **真实案例数据**：一次内存告警案例——19 项巡检 37.4 秒完成，命中内存使用率 91.90%、空闲内存仅 161MB，memgraph 深度诊断定位到 python 进程独占 12.95GB 匿名内存（占总内存约 83%），处置时间「从四十分钟压到不到五分钟」。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]
3. **巡检→诊断→处置闭环 + 技能包组合**：巡检 Skill 负责事前发现隐患，诊断 Skill 负责事后定位根因，二者组合为 SysOM 技能包，实现「让 Agent 既能事后止损，更能事前预防」的主动运维模式。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]
4. **memgraph 深度诊断自动串联机制**：巡检命中高风险项后自动串起 SysOM 内置的 memgraph（内存深度分析）诊断，报告不止停在「内存高了」，而是直接给出「哪个进程占了多少、占总内存比例、可排除哪些」的根因级结论。^[raw/articles/sysom-巡检-skill-一键锁定根因.md]

→ [[raw/articles/starops-host-intelligent-inspection-ecs-ai-doctor-2026|原文存档 1]]

→ [[raw/articles/sysom-巡检-skill-一键锁定根因|原文存档 2]]

---
## 关联
- 姊妹能力: [[entities/starops-rum-intelligent-inspection|STAROps RUM Intelligent Inspection]]
- 姊妹能力: [[entities/starops-umodel-digital-twin-openapi-embedding-jingchen-2026-08-04|STAROps UModel 数字孪生 + OpenAPI 嵌入（客户集成模式）]]
- 平台: [[entities/alibaba-agentic-cloud|阿里云 Agentic Cloud]]
