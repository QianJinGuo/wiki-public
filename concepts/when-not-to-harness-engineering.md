---
title: 何时不要 Harness Engineering
created: 2026-09-05
updated: 2026-09-05
type: concept
tags: [harness-engineering, anti-patterns, boundary-conditions, negative-results, cost]
confidence: 0.7
provenance_state: merged
---

# 何时不要 Harness Engineering

> 本页是库内自己标记的缺页：[[entities/harness-engineering|Harness Engineering 主页]]的反方标注曾写道，"反方观点（AI Coding 的核心瓶颈不在工程体系而在模型能力本身）长期无独立页面"。2026-09 反方立场透镜轮（[[drafts/wiki-emergent-viewpoints-2026-09-adversarial|涌现稿]]）补建本页。姊妹页：[[comparisons/model-capability-vs-harness-engineering|模型能力 vs Harness Engineering 对抗页]]、[[queries/negative-results-registry|负结果登记簿]]。

## 边界条件一：模型代际更新会主动侵蚀 Harness 价值

这不是反方的外部攻击，是 [[concepts/harness-engineering-framework|框架主页自己记录的]] "Harness 衰减"：Opus 4.5→4.6 一代更新使 harness 相关成本节省 38%——**上一代模型需要的脚手架，下一代模型不再需要**。推论：为当前模型短板建立的工件（冗余的 checklist、防御性 prompt 结构、多层校验）是负债，模型一升级就变成纯摩擦。主页引述的实践（Manus 一年重构多次、LangChain 一年三次、Vercel 移除 80% 工具反而更快）说的都是同一件事：**harness 的默认终态是删除，不是累积**。如果团队没有配套的删除机制，就不该开始建设。

## 边界条件二：harness 干预在强模型上可能为负

库内已有的最干净的负结果：[[entities/gene-gep-evomap-qinghua-strategy-genes-arxiv-2604-15097-2026|Gene-GEP 实验]]（Pro 模型上 Skill 使成绩 60.1→50.7，-9.4）——更强模型 + 更长 Skill = 更差的直接证据。与之互证的：[[entities/世界模型的deepseek时刻魔芯flash-world-model降本70跑出50fps实时交互|低成本专用系统]]和[[entities/computer-use-45x-more-expensive-than-structured-apis|结构化 API 比 Computer Use 便宜 45 倍]]都说明，**问题合适时，绕过 agent+harness 整条栈直接用确定性接口，便宜一个数量级**。判断顺序应该是：确定性 API → 单次调用 → 薄 agent → 重 harness；harness 是最后一档，不是第一档。

## 边界条件三：任务不值得时，harness 成本永远不回本

harness 的全部收益前提是"任务会被重复执行、且失败代价高"。一次性脚本、原型探索、Karpathy 意义上的 vibe coding 起点——这些场景上工程约束直接消灭快速探索的价值（[[entities/martin-fowler-ai-rd-harness-nondeterminism-devnote|Fowler 框架]]与 [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering|Karpathy 立场]]的分歧核心，见对抗页）。**为一次性任务建 harness，是用生产成本买玩具产出。**

## 边界条件四：安全边界不在 harness 里

[[entities/nomshub-cursor-remote-tunnel-sandbox-breakout-straiker|NomShub 沙箱逃逸链]]表明 harness 的执行层隔离可被 Shell builtin + 隧道外联组合攻破。把"我们上了 harness"当安全结论是范畴错误——隔离语义见 [[concepts/agent-sandbox|Agent 沙箱与执行容器]]，逃逸面评估独立于工程体系成熟度。

## 反向检查清单（上 harness 前四问）

1. 这个短板**当前模型**还有吗？（对表 [[concepts/claim-half-life|半衰期]]：代际 ≈59 天）
2. 有没有更便宜的确定性替代？（API/结构化接口优先）
3. 任务复用频次撑得起维护成本吗？（built to be deleted 机制存在吗）
4. 删除条件现在就写了吗？（没有删除计划的 harness 是负债）

## 检索入口

- 正方主页：[[concepts/harness-engineering-framework|框架]] · [[entities/harness-engineering|范式主页]] · [[moc/coding-agent-practice|Coding Agent 实践 MOC]]
- 对抗综合：[[comparisons/model-capability-vs-harness-engineering|两论点对抗页]] · [[queries/negative-results-registry|负结果登记簿]]
