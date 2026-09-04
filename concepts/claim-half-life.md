---
title: 断言半衰期
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [meta, temporal, half-life, epistemics, knowledge-decay]
confidence: 0.7
provenance_state: inferred
---

# 断言半衰期（Claim Half-Life）

> 断言半衰期 = 一条断言从入库到被库内新证据修正、证伪或术语性取代的典型时滞。提出于 [[drafts/wiki-emergent-viewpoints-2026-09-time-slice|2026-09 时间切片透镜轮]]：wiki 按文章存断言，但不同类型的断言衰变速度差异极大，读库时应按类型打折，此前没有任何字段或页面承载这个信息。

## 库内首测数据（2026-09-05）

| 断言类型 | 半衰期（首个测得值） | 证据对 |
|----------|---------------------|--------|
| 术语类（vibe coding → agentic engineering） | ≈ 30 天 | [[entities/karpathy-vibe-coding-agentic-engineering-v2|karpathy 系列 v2]]（05-09）→ [[entities/karpathy-software3-vibe-coding-dead-agentic-engineering|Software3 判决页]]（06-10） |
| 模型代际"最强"类 | ≈ 59 天 | [[entities/claude-opus-4-8-system-card-zvi|Opus 4.8]]（06-02）→ [[entities/claude-opus-5-vending-bench-andon|Opus 5]]（07-31） |
| 发布预期 vs 交付类 | ≈ 109 天 | [[entities/deepseek-v4|V4 论文解读]]（05-13）→ [[entities/连夜实测deepseek-v4-pro-正式版低于预期不推荐接入codex|正式版实测]]（08-30） |
| 机制/原理类（评测防火墙、经验抽象度等） | 未测得（0 条被推翻） | [[concepts/eval-optimizer-firewall]] 等 |

## 读库协议

1. **按类型打折**：引用"最强/最新/SOTA"类断言时，先查其入库日——超过 60 天的按历史事实读，不按现状读；发布预期类超过 ~110 天必须有交付侧页面对账才可引用。
2. **机制类可长持**：未被推翻的机制类洞见（如 [[concepts/eval-optimizer-firewall|评测防火墙]]）没有观测到的衰变，适合作为 MOC 骨架；清单/榜单类是快照（见下）。
3. **快照语义**：清单、榜单、盘点类页面入库即冻结，应带时点标记，按时点而非新鲜度解读——当前 `news_stale_90d` 口径盖不住它们（[[drafts/wiki-emergent-viewpoints-2026-09-time-slice|透镜轮观点五]]）。

## 与既有机制的关系

- **≠ supersedes**：[[drafts/supersedes-relation-design|supersedes 机制]]处理的是权威转移（v2 修正/吸收 v1）；时间冲刷不产生替代页，只让老页权威流逝——给后者的容器是 [[queries/prediction-ledger|预测对账台账]] 与本页的半衰期标注。
- **≠ contradicted_by**：[[concepts/harness-engineering-framework|矛盾机制]]处理共时对立观点；半衰期处理历时失效。一页可同时两者皆有。
- **测量方式**：时间切片透镜每轮回访（见 [[queries/emergence-lens-rotation|轮换表]]），新对账样本直接更新上方表格；若扫描器后续扩展时间谓词配对，可半自动化。

## 待测假设

- "兼容/迁移指南"类断言半衰期是否与底层 API 弃用周期同步？
- 中国厂商生态页（wechat 来源为主）与美国厂商生态页的半衰期是否有系统性差异？
