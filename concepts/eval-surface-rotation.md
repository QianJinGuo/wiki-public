---
title: 评测面轮换
created: 2026-09-10
updated: 2026-09-10
type: concept
tags: [evaluation, security, isolation, rotation, held-out, defense]
confidence: 0.7
provenance_state: inferred
---

# 评测面轮换（Eval Surface Rotation）

> 2026-09 跨簇迁移透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|涌现稿]]观点一）识别的 security 簇最大单向缺口：评测防火墙机制（[[concepts/eval-optimizer-firewall|评测防火墙]]）此前只保护 evaluator，防御侧的攻击面（payload 库、检测规则、wasm 签名）暴露给自适应攻击者时没有同构的保密与轮换纪律。本页把防火墙机制翻译到防御侧。

## 判据：防御面是优化器面前的评测面

攻击者就是 optimizer（[[entities/openart-agent-red-team-evolving-environment-2026|OpenArt 红队演化环境]]证明：静态环境上的攻击成功率会被超图搜索推到饱和）。推论：**任何对自适应攻击者暴露的固定防御面都会被学穿**——payload 黑名单、检测签名、过滤规则都是"评测面"，静态即衰减。这与 [[concepts/claim-half-life|半衰期]]框架里"机制类断言"的例外：防御机制不是被模型升级推翻，而是被攻击者采样推翻。

## 四件套（防火墙机制向防御侧的翻译）

| 防火墙机制（evaluator 侧） | 防御侧对应 |
|---------------------------|-----------|
| 锚点永不烘焙进镜像 | 检测锚点（真实攻击样本）不入分发物：签名库/规则集镜像内不含完整真集 |
| held-out 集 | payload 库分层：公开规则与 held-out 私域样本，后者只用于内部红队度量 |
| 轮换 | 锚点定期轮换 + 旧锚点降级为公开规则；轮换周期不预告 |
| 未枚举即大声失败 | 防御面清单化（[[concepts/channel-enumeration-criterion|通道枚举判据]]）：检测依赖的每条通道显式列出，缺口出现要可观测 |

## 现状与空白

security 簇内（[[concepts/prompt-injection-defense|注入防御]]等页）通篇没有"防御面保密/轮换"内容——攻击侧已走到环境演化（OpenArt），防御侧评测面管理仍是空白。审计问句：**防御面的 held-out 集在哪？上次轮换是什么时候？签名库分发物里有没有真集？**

## 参见

- [[concepts/eval-optimizer-firewall|评测防火墙]] — evaluator 侧的原机制
- [[concepts/channel-enumeration-criterion|通道枚举判据]] — 边界可审计性的通用判据
- [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|跨簇迁移首轮涌现稿]] — 本页缺口的原识别记录
