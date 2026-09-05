---
title: 通道枚举判据
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [isolation, security, evaluation, channel, boundary, systems]
confidence: 0.7
provenance_state: inferred
---

# 通道枚举判据（Channel Enumeration Criterion）

> 2026-09 跨簇迁移透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|涌现稿]]）从两个不同话题域的隔离机制中合成的通用判据：**一条隔离边界的安全/有效程度，等于其通道枚举的完整程度；任何未枚举通道都是已经存在的漏洞，无论边界声明多强。**

## 判据陈述

一个隔离声明（"评测面对 optimizer 保密"/"代码在沙箱内执行"）要成立，必须满足：

1. **通道封闭**：被隔离系统与外界的一切交互通道被显式列出；
2. **逐通道最小化**：每个通道只开放当前目的所需的最小语义（faibench 评分容器 `--network=none`：评分不需要网络，网络通道直接封死而非"限制"）；
3. **未枚举通道的失效语义**：发现未声明通道时**大声失败而非默默放行**（faibench："不挂载 tests/ 就大声失败，而不是给出可疑分数"——把"通道清单不完整"从漏洞升级为可观测故障）。

## 两簇证据（同一判据的两次实例化）

**信息隔离侧**（[[concepts/eval-optimizer-firewall|评测防火墙]]）：
- 泄漏通道实例：执行轨迹流向更新提议器（AutoDesign 显式切断）；锚点烘焙进镜像（faibench 永不烘焙）；打分历史流回评审上下文（防火墙页对本 wiki quality-loop 的自反）。
- 共同点：每条泄漏都不是"权限被攻破"，而是**存在一条没人列入清单的合法通道**。

**执行隔离侧**（[[concepts/agent-sandbox|Agent 沙箱]]）：
- 逃逸通道实例：[[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|Shell builtin 绕过 + 远程隧道外联]]——沙箱策略枚举了"文件/进程"通道，漏了"内置命令语义"+"既有隧道"两条。
- [[entities/microsoft-mxc-execution-containers-agent-sandbox-origin|MXC]] 的 AppContainer 分级本质是把通道清单做进 OS 原语。

## 使用方式

- **审计问句**（替代"安全吗/隔离吗"这种无效问题）："这个边界的通道清单在哪？最近一次新增通道是什么时候？谁有权加通道？"
- **与相邻判据的分工**：[[concepts/claim-half-life|半衰期]]管断言的时间衰减，[[queries/prediction-ledger|台账]]管预测对账，[[queries/negative-results-registry|负结果簿]]管变差实证，本判据管**边界声明的可审计性**——四者都是"把隐式假设显式化"的不同切面。
- **失败语义优先**：枚举不完整不可修复（通道天然开放），可修复的是**让缺口可见**——任何隔离方案的设计评审应先问"缺口出现时我多久能知道"。

## 参见

[[comparisons/information-isolation-vs-execution-isolation|信息隔离 vs 执行隔离]]（本判据的来源对撞页）· [[concepts/agent-security-architecture|Agent 安全架构]] · [[concepts/evaluation-harness-design|评测 Harness 设计]]
