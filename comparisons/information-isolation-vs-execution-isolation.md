---
title: 信息隔离 vs 执行隔离：两簇隔离机制的通用判据
created: 2026-09-05
updated: 2026-09-05
type: comparison
tags: [isolation, evaluation, sandbox, security, channel, comparison]
confidence: 0.7
provenance_state: merged
---

# 信息隔离 vs 执行隔离：两簇隔离机制的通用判据

> 跨簇迁移透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|2026-09 涌现稿]]）产物：把 [[concepts/eval-optimizer-firewall|评测防火墙]]（evaluator 簇的信息隔离）与 [[concepts/agent-sandbox|Agent 沙箱]]（security 簇的执行隔离）放在同一张桌上，检验两簇的机制是否互迁。

## 两个隔离，同一个失败模式

| | 信息隔离（评测防火墙） | 执行隔离（沙箱） |
|---|---|---|
| 被隔离物 | 评测面内容/轨迹/锚点 | 被执行代码的权限与可达面 |
| 对抗者 | optimizer（训练循环/自改进/打分调参） | 攻击者 + 失控的合法代码 |
| 失效形态 | Goodhart：optimizer 对可见的 eval 过拟合 | 逃逸：代码抵达未枚举的通道 |
| 库内正面机制 | 轨迹隐藏（AutoDesign）、锚点不烘焙（faibench）、held-out 双门（Self-Harness） | AppContainer 分级（MXC）、`--network=none`（faibench 评分）、写入闸门（自进化治理） |
| 库内失效案例 | 打分锚点历史泄漏 → 评分自我强化（防火墙页"本 wiki 的自反"节） | [[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|NomShub：Shell builtin + 隧道外联组合逃逸]] |

**通用判据（本轮合成）：隔离边界只在其通道枚举完整时成立。** 信息隔离漏的是"未设想的信息通道"（打分记录流回提议器），执行隔离漏的是"未设想的执行通道"（shell builtin 绕过、隧道外联）。两者不是隐喻相似，是同一个结构性质的两次实例化：**边界 = 通道集合的封闭性声明，而通道集合天然开放。**

## 互迁检验（三问）

1. **沙箱设计能否借用"锚点不烘焙"？** 能且已发生：faibench 的 `--network=none` 评分挂载就是执行隔离手段服务信息隔离目标——沙箱策略表里应有一列"这个策略保护的是哪类信息"，[[concepts/agent-sandbox|四问判定框架]]可加第五问。
2. **防火墙能否借用"逃逸链"方法论？** 能：把打分/评审管线当攻击面做红队——"评审 prompt 能否被被评内容注入"正是 [[entities/breaking-claude-code-opus-5-auto-mode-prompt-injection|注入类攻击]]在评测场景的镜像，库内尚无该场景的复现记录（空白，见涌现稿）。
3. **两簇对"隔离是组件还是属性"的答案一致吗？** 一致：[[entities/anthropic-multi-agent-conflict-frontier-red-team-2026-08|Anthropic 多智能体红队实验]]的结论"安全是整体属性而非个体属性"，与防火墙页把隔离定为"一等架构属性"同构。

## 迁移待验证清单

- [ ] 防御评测面（注入 payload 库）对攻击者/自动化防御调优器的保密与轮换机制——security 簇空白， [[concepts/prompt-injection-defense|注入防御]]页未覆盖
- [ ] 评审管线的注入红队复现（被评内容作为不可信输入的评测场景）
- [ ] 通道枚举的形式化检查（sandbox 策略 × 信息流的联合审计）

## 检索入口

[[concepts/eval-optimizer-firewall|评测防火墙]] · [[concepts/agent-sandbox|Agent 沙箱]] · [[concepts/agent-security-attack-defense|Agent 攻防]] · [[concepts/verifier-driven-development|Verifier 驱动开发]]
