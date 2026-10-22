---
title: "SPACE：Perplexity 开源 Agent 沙箱平台（microVM 隔离 + 控制面 + 增量快照）"
created: 2026-10-20
updated: 2026-10-22
type: entity
tags: [agent, sandbox, agent-security, agent-runtime, infrastructure, perplexity]
sources: [raw/articles/perplexity-space-agent-sandbox-open-source-2026-10]
confidence: 0.85
provenance_state: extracted
---

# SPACE：Perplexity 开源 Agent 沙箱平台

Perplexity 于 2026-10-15 把内部 Agent 沙箱平台 SPACE 开源。它自 2026 年 7 月起替换掉商业沙箱供应商，承载全部 Computer 会话，目前每天处理数百万次沙箱创建，同生产流量下创建速度比原供应商快 3–5 倍。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

文章的核心论点值得单独摘出：**agent governance 是工程问题，不是模型问题**——可以在模型侧加 safeguard 降低不安全行为的概率，但必须在系统侧加独立控制，在 safeguard 失效时限制造 damage。SPACE 就是这层"围绕 agent 生成代码与文件的那个隔离工作区"的控制。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

## 架构要素

- **每沙箱一个小型 VM + 自己的操作系统**：在 agent 的工作环境与 host 之间建立隔离边界，而不是进程级或容器级隔离。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]
- **中央控制面**：负责 workspace 的 create / pause / resume / remove 的编排。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]
- **默认不信任沙箱**：网络访问与平台持有的凭证都在沙箱外管理，agent 可连接已授权服务，但凭证不落在它的工作环境里。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]
- **增量快照而非全量拷贝**：频繁对运行环境做 file snapshot 与 checkpoint，但只存变更，因此 create / restore / branch 的成本都很低。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]
- **会话级能力**：隔离代码执行、pause & resume、故障恢复、session branching（同一 workspace 的多个副本上并行试两条数据清洗方案，或回滚到出错前的文件）。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

上述"增量快照 + 会话分支"是它与多数沙箱产品拉开差距的地方：多数方案解决了隔离，但没解决"长会话中的状态成本"，而 agent 场景的状态是高频、可分支的。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

## 安全评估的公开数字

Perplexity 公布了对抗性测试口径：针对刻意越界的 agent 做了 108 次逃逸尝试，**没有一次从 VM 逃到 host**；但有若干次找到了绕过网络限制的方法，这些弱点已修复，且在复测中两种攻击方法都未再成功。相关弱点也披露给了其他沙箱平台的厂商。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

这个"0/108 逃逸 + 网络限制被绕过"的组合结论很有信息量：VM 边界本身在工程上守得住，实际失守点通常在**网络策略与凭证路径**这类"边界之外的配置面"，与 [[concepts/agent-security-architecture|Agent 安全架构]] 中"独立控制层"的思路一致。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

## 生态与配套工具

SPACE 已进入他人云平台：Akamai 完成测试并在其云平台上线、与 Perplexity 合作推向客户；Crusoe 将其纳入 Managed AI Services；CoreWeave 发布了在 CKS 上部署 SPACE 的指引。Perplexity 同期还开源了 BrowseSafe（检测网页中隐藏恶意指令的模型与 benchmark）、Numbat（检测/调查/阻断危险 agent 行为的工具包，7 月开源）与 Bumblebee（扫描开发机风险包、扩展与 AI 工具配置，5 月开源）。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

## 深度分析

### 边界选型：为什么 agent 代码要 microVM

分野不在"安全等级"，而在**攻击面被推到哪一层**。容器共享宿主内核，syscall 面即攻击面，一个内核提权漏洞就等于逃逸；gVisor 类方案插入用户态内核解释 syscall，收窄攻击面但付出兼容性与 I/O 折扣；microVM 把整个内核搬进 guest，攻击面退到 VMM 与设备模型这两个刻意做小的组件。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

选它的理由是负载分布：agent 执行模型生成、未经人审阅的代码，临时装来源不明的依赖，行为集合远宽于自家后端。此时"内核也在边界内"是唯一能把内核 0-day 的爆炸半径限制在可丢弃 guest 里的结构；容器方案要么假设代码已审阅，要么用 seccomp 白名单赌自己列全了 syscall，而 agent 不断换工具链，白名单几乎必然过宽。代价是冷启动与 syscall 兼容性：Firecracker 路线靠裁剪设备集与半虚拟化才把启动压到百毫秒级（[[entities/firecracker-bedrock-agentcore-multi-tenant|Firecracker 多租户实践]] 记录同一组取舍）。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

### 控制面/数据面切分才是可审计的前提

控制面只管 create / pause / resume / remove，数据面承载执行与文件。真正决定价值的是**三样东西被移出沙箱**：出口策略、平台凭证、artifact 与 volume 挂载。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md] 任一样留在内部，边界就从"VM"退化为"VM 加一份 agent 有写权限的策略文件"；凭证尤其致命——token 落进 agent 文件系统之后，隔离只是在保护一个已经交出去的秘密。

第二个收益是审计：越权动作必须穿过同一代理点，日志、限流、吊销都成为单点操作。多租户下隔离与可审计性来自同一结构而非两套机制，这正是 [[concepts/agent-security-architecture|Agent 安全架构]] 里"模型之外加独立控制层"的形态——控制层不是又一个检查函数，而是所有流量必须经过的物理路径。代价是控制面成为单点与瓶颈，代理必须协议感知而非 TCP 转发。

### 增量快照：决定试错经济学的原语

快照成本决定 agent 能多便宜地犯错。全量拷贝让每次分支都复制整个工作集，branch 于是只留给人预先计划；增量快照把 branch 变成近乎免费，agent 才能"先试、错了回滚、再走另一条路"——这是 [[concepts/agent-sandbox|Agent 沙箱]] 从隔离容器升级为运行时的分水岭。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

快照要含进程与内存、恢复成"能继续跑"的环境，这是多数平台能做镜像分发却做不好会话状态的原因。成本则被转到存储侧——写放大与恢复时的随机 I/O 常比 CPU 更早成为瓶颈。

### 公开安全数字覆盖了什么

口径是 108 次逃逸尝试无一次从 VM 到 host 成功，但若干次绕过网络限制，已修复且复测未再成功。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md] 分母是红队在有限时间内想到的攻击集合，不是攻击面的穷举；0/108 是好信号，不是安全证明。

覆盖的是 VMM、设备模型与已知绕过手法。不覆盖：guest 内核 0-day、微架构侧信道、借"被允许的出口"外泄（策略问题，边界管不住）、镜像与依赖供应链。最值得注意的恰是"网络限制被绕过"——失守点在**策略面而非虚拟化面**，与 [[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|Cursor 远端隧道沙箱越界]] 的形态一致。microVM 解决"代码乱跑"，凭证外置与 egress 代理解决"代码乱连"，两道闸不能互相替代。

### 生态位：自建底座 vs 托管 API

托管 API 卖隔离、容量与运维，自建买控制权、数据不出域与可修改性。SPACE 开源而非托管，把"可被云厂二次分发"当分发策略：Akamai 已上线，Crusoe 纳入 Managed AI Services，CoreWeave 给出 CKS 指引。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

自建的负担：镜像供应链、容量与碎片（长会话占实例、突发创建又要弹性）、per-session 成本（快照存储与恢复 I/O 常比 CPU 贵）。这三项不因开源而消失——开源降低从零设计的成本，不降低长期运营的成本。判据：会话短、合规要求低时托管几乎总是更便宜；若要长期保存状态、自持凭证、做分支式探索，自建 microVM 的边际收益才盖过运维开销。

## 实践启示

1. **按"内核是否在边界内"选隔离层，而不是按宣传的安全等级。** 执行模型生成代码时共享内核的假设通常不成立；若必须用容器，seccomp 白名单要按工具链版本维护并定期重放，否则会静默腐化。

2. **凭证当作永不进入沙箱的外部资源。** 沙箱外注入与刷新，沙箱内只留"能调用"的能力。验收很直接：让 agent 在沙箱里跑 `env` 并遍历 home，不应出现长期有效的 secret。

3. **egress 按"目的域名 + 方法"授权，并按会话签发短期规则。** 只按端口或 IP 段放行等于没有策略；被绕过的正是网络限制，这层需要独立红队用例。

4. **把快照做成一等公民，先写成本模型再接业务。** 上线前算清单次 checkpoint 写入量、分支存活分布与恢复时的 I/O 峰值；若 branch 比重跑更贵，分支只是文档里的功能。

5. **红队报告按攻击面分类记账，不要只报一个 0/N。** 至少四栏：VMM/设备模型逃逸、guest 内核提权、策略与 egress 绕过、供应链污染。分类之后才看得出真薄弱环节——本例中它不在虚拟化。

6. **自建前做一次含运维人头的六个月总成本对比。** 没有专职负责人时，开源底座的自由度常被运维债务吃掉；可对照 [[entities/claude-managed-agents-self-hosted-sandbox-enterprise|Claude Managed Agents 沙箱]] 与 [[entities/cubesandbox-v070-agent-sandbox-infrastructure-tencent|CubeSandbox]] 的取舍。

## 与 wiki 其他沙箱记录的对照

wiki 里已有若干"agent 运行时 / 沙箱"路线：[[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|Microsoft MXC 执行容器]] 走的是 OS 原语 + policy 隔离，[[entities/claude-managed-agents-self-hosted-sandbox-enterprise|Claude Managed Agents 自托管沙箱]] 走的是托管服务内嵌沙箱，[[entities/cubesandbox-v070-agent-sandbox-infrastructure-tencent|CubeSandbox]] 与 [[entities/opensandbox-aliyun-cloud-agent-sandbox-vibecoder|OpenSandbox]] 是国内云厂的沙箱基础设施。SPACE 的差异点是：把 **microVM 隔离 + 平台侧凭证托管 + 增量快照/会话分支** 作为开源底座一起给出，并附公开的对抗测试口径。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

同厂的 [[entities/perplexity-numbat-agent-security-2026-07-29|Numbat]] 是"检测与响应"侧，SPACE 是"隔离与执行"侧，二者合起来构成 Perplexity 的 agent 安全栈；[[concepts/agent-sandbox|Agent 沙箱]] 概念页可作为这条线的总览入口。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

→ [[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10|原文存档]]
