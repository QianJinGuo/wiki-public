---
title: 涌现验收门
created: 2026-09-10
updated: 2026-09-10
type: query
tags: [meta, emergence, quality-gate, scoring, dashboard]
confidence: high
---

# 涌现验收门（Emergence Acceptance Gate）

> 涌现产出（透镜轮涌现稿及其补页）在 2026-09-10 之前只有两条软约束：lint 0 error + 新颖性陈述（[[queries/emergence-lens-rotation|透镜轮换]]合约第 4 条）。基线审计证明这不够：8 篇存量涌现稿中 7 篇 claim 级引用为零，度量数字裸奔。本页把两个外部方法形式化为硬门——**spark-to-paper 的双模式证据门控**（proposal 模式禁前瞻数字 / data_aware 模式数字必溯源）与 **faibench 的 oracle-zero 评分设计**（触发任一零分硬门即整稿记 0，不给部分分）。机制出处见 [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 phd 七系统透镜]] 观点一、二。

## 双写作模式（沿用 spark-to-paper 语义）

| 模式 | 适用 | 数字纪律 |
|------|------|----------|
| **proposal 模式** | 前瞻性 drafts（涌现稿中的预测、路线、建议段） | 禁止无溯源的前瞻数字——"X 将翻倍"这类量化断言必须进 [[queries/prediction-ledger|台账]] 或删除数字 |
| **data_aware 模式** | 综合性正文（观点、分析、判分段） | 每个度量数字必须能溯源到 raw 来源（claim 级引用 `^[...]`）或库内测量页 |

模式不是二选一：一篇涌现稿可以同时含两类段落，每段按其性质受对应纪律约束。这与 lint 的既有约定一致——引用只上散文段，列表/表格/引用块豁免。

## 零分硬门（oracle-zero，无部分分）

| 门 | 名称 | 判据 | 机械检查 |
|----|------|------|----------|
| **ZG1** | 数字溯源门 | 散文段含度量型数字（百分比/小数/倍数/量词计数/金额）且段落无 `^[` 引用 | ✅ 可机械判定 |
| **ZG2** | 前瞻断言台账门 | 含时间量化前瞻断言（将/预计/到 YYYY 年/半年内等）却未关联 [[queries/prediction-ledger|台账]] | ✅ 可机械判定 |
| **ZG3** | 新颖性陈述门 | 缺"本 wiki 此前不存在 X"类陈述（合约第 4 条） | ✅ 可机械判定（正则近似） |

**拒评语义**（faibench "宁可拒评不出可疑分"的直译）：任一硬门触发 → 整稿 `REJECT`，不进入评分排序、不接受"整体不错就放行"。修复硬门后重审。无法机械判定的质量维度（论证是否原创、综合是否超出双亲之和）**不由本门裁决**，留给透镜轮人审——本门只拦"机械可查却不及格"的部分。

**豁免**：度量轮按合约 v2 免新颖性定额，在稿 frontmatter 写 `gate_exempt: [ZG3]`。豁免是逐门声明，不是整体免检。

## 机械检查

```bash
node scripts/emergence-gate.mjs          # 报告模式: 全量审计, exit 0
node scripts/emergence-gate.mjs --strict # 门禁模式: 有 REJECT 则 exit 1（透镜轮验收、pre-commit 可挂）
node scripts/emergence-gate.mjs <file>   # 单稿审计
```

只读，不写库。

## 基线（2026-09-10 首次全量审计）

| 稿 | ZG1 | ZG2 | ZG3 | 判定 |
|----|-----|-----|-----|------|
| [[drafts/wiki-emergent-viewpoints-2026-07|2026-07 结构信号]] | 4 | — | 1 | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 phd 七系统]] | 3 | — | — | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-09-time-slice|2026-09 时间切片]] | 8 | — | — | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-09-concept-gap|2026-09 概念缺位]] | 5 | — | — | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-09-adversarial|2026-09 反方立场]] | 6 | — | 1 | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-09-cross-cluster|2026-09 跨簇迁移]] | 1 | — | — | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-09-external-probe|2026-09 外部探针]] | 3 | 1 | 1 | REJECT |
| [[drafts/wiki-emergent-viewpoints-2026-09-metrology|2026-09 度量轮]]（ZG3 豁免） | 1 | — | — | REJECT |
| **合计** | **31** | **1** | **3** | **8/8 REJECT** |

两点解读：**(1)** 这是 2026-09-05 合约定型前写就的历史稿，按当时的软约束多数"合规"——债务属于标准升级，不属于违约；**(2)** 唯一 claim 级引用较多的 2026-07 稿也只有 2 处 `^[`，说明涌现稿从未真正执行过综合页的引用纪律。修复策略：**不溯改历史稿**（与 116 页合成债同一处理纪律），债务登记于此表；自下一轮起 ZG 门为硬约束。

## 与其他机制的关系

- **上游**：[[queries/emergence-lens-rotation|透镜轮换]]合约 v3 把本门纳入每轮验收；[[queries/lens-proposals|提案队列]]是产出前的分流层（提案卡人审先行，减少整稿返工）。
- **下游**：通过 ZG 门的涌现稿，其时间量化断言仍须走 [[queries/prediction-ledger|台账]]回访——ZG2 只查"有没有关联"，真值对账在台账侧。
- **边界**：本门不替代 [[concepts/claim-half-life|半衰期]]测量与 wiki-lint 的引用结构检查；三者分别管"数字可溯、断言可证、结构可查"。
