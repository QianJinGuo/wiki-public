---
title: "SPACE：Perplexity 开源 Agent 沙箱平台（microVM 隔离 + 控制面 + 增量快照）"
created: 2026-10-20
updated: 2026-10-20
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

## 与 wiki 其他沙箱记录的对照

wiki 里已有若干"agent 运行时 / 沙箱"路线：[[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|Microsoft MXC 执行容器]] 走的是 OS 原语 + policy 隔离，[[entities/claude-managed-agents-self-hosted-sandbox-enterprise|Claude Managed Agents 自托管沙箱]] 走的是托管服务内嵌沙箱，[[entities/cubesandbox-v070-agent-sandbox-infrastructure-tencent|CubeSandbox]] 与 [[entities/opensandbox-aliyun-cloud-agent-sandbox-vibecoder|OpenSandbox]] 是国内云厂的沙箱基础设施。SPACE 的差异点是：把 **microVM 隔离 + 平台侧凭证托管 + 增量快照/会话分支** 作为开源底座一起给出，并附公开的对抗测试口径。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

同厂的 [[entities/perplexity-numbat-agent-security-2026-07-29|Numbat]] 是"检测与响应"侧，SPACE 是"隔离与执行"侧，二者合起来构成 Perplexity 的 agent 安全栈；[[concepts/agent-sandbox|Agent 沙箱]] 概念页可作为这条线的总览入口。^[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10.md]

→ [[raw/articles/perplexity-space-agent-sandbox-open-source-2026-10|原文存档]]
