---
title: 透镜提案队列
created: 2026-09-10
updated: 2026-09-10
type: query
tags: [meta, emergence, proposal-queue, dashboard]
confidence: high
---

# 透镜提案队列（Lens Proposal Queue）

> 透镜轮的产出从"一期特稿直接落页"改为"**提案卡 → 人审 → 落地**"的例行闭环——EvoScientist 的观察聚类→skill 提案→人工 `/autoskills` 审批模式，见 [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 phd 七系统透镜]] 观点五的工程解法。动机：涌现稿观点五的诊断（"wiki 缺的不是洞察的生产，是洞察的消费管线"）+ 8 篇涌现稿 [[queries/emergence-acceptance-gate|验收门]] 基线 8/8 REJECT 的教训——先小卡先审，比整稿落完再返工便宜。

## 流程

1. **落卡**：透镜轮/机制脚本（`ledger-backpropagate.py` 判定回传、`concept-gap-detector.py` 缺位检测）产出的补页候选，先在本页登记为卡片，不直接建页
2. **人审**：裁决 `approve` / `reject`（附一句理由）；卡是 LLM 起草、人拍板，与质量闭环三裁同纪律
3. **落地**：approve 后按**产出阶梯**选最廉价形态——wikilink 边 < comparison stub < 独立补页 < 涌现稿章节；落地后回填卡状态 `landed` 并链产物
4. **过期**：`proposed` 超过 60 天未裁决自动降级为 `stale`（防待办清单自身腐烂——8 月涌现稿已观察到一次）

提案卡不设独立文件，直接在下表追加行；复杂提案（需论证 >10 行）才开 drafts 存稿并在此处链入。

## 提案卡

| # | 提案 | 来源 | 预期产出（阶梯档位） | 状态 | 提出 | 裁决 |
|---|------|------|---------------------|------|------|------|
| 1 | 注入防御评测面轮换机制补页：payload 库的 held-out 锚点与轮换纪律（security 簇最大单向缺口） | [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|跨簇迁移轮]]观点一空白 | 独立补页（`concepts/eval-surface-rotation`） | `proposed` | 2026-09-10 | — |
| 2 | [[concepts/agent-sandbox|沙箱]]四问框架增第五问："该策略保护哪类信息"（执行隔离 vs 信息隔离判别） | [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|跨簇迁移轮]]半成功待办 | 已有页增量修改（最小档） | `proposed` | 2026-09-10 | — |
| 3 | 断言动力学首测：12 月判定窗三件套（缺口复测 + 引力首测 + 台账 #2-#5 到期判定），含 `ledger-backpropagate.py` 回传收割 | [[queries/emergence-lens-rotation|度量轮]]决议 + 2026-09-10 仪器化 | 涌现稿章节 + 半衰期后验首算 | `scheduled`（2026-12，勿提前跑） | 2026-09-10 | — |
| 4 | supersedes 活边冷启动：机制进 SCHEMA/lint 但全库活边仅 1 条；时间切片透镜下一轮把已识别的术语替换链（karpathy v2/v3/v4 等）批量补 `superseded_by` 标注 | 2026-09-10 判定回传首跑发现 | wikilink 边批量补注（最小档） | `proposed` | 2026-09-10 | — |

## 与其他机制的关系

- **上游**：[[queries/emergence-lens-rotation|透镜轮换]]合约 v3 第 3 条——产出先落卡再落地；矛盾升级线与概念缺位检测器的候选照常走各自队列，仅补页动作在此汇合。
- **下游**：approve 的卡落地为页面后，涌现稿/补页过 [[queries/emergence-acceptance-gate|验收门]]（ZG 门 + lint）方可结单。
- **对账**：本页每月在 [[queries/vault-evolution-dashboard|进化仪表板]] 记一次队列深度与 landed 率——消费管线的健康指标。
