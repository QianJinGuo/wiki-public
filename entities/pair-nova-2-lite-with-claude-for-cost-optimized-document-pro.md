---
title: "Pair Nova 2 Lite with Claude for cost-optimized document processing"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro]
provenance_state: extracted
---

> -> [[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md|原文存档]]

sha256: c8851ea611bd9d2a9d198657d99d50af19600b6dbe436f61713a013c5ec51c94 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md]

## 摘要

AWS 机器学习博客介绍了一个在 Amazon Bedrock 上构建的两模型文档数字化流水线：Amazon Nova 2 Lite 在单次 Converse 调用中完成原生多模态提取（照片检测与 bounding box、带坐标的姓名提取、页面级元数据），Claude Sonnet 4.6 随后基于页面布局做空间推理，把姓名匹配到对应人脸。在 336 页扫描年鉴上，流水线产出 3,122 条姓名-人脸关联，其中 93.3% 置信度 ≥0.95，仅 0.3% 低于 0.90。两模型拆分方案每页约 $0.033，比单模型全量方案（约 $0.10/页）便宜约三分之二，10 万页可节省约 $6,500 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md]

Nova 2 Lite 采用固定 per-image 计费（每页约 230 image tokens），成本与分辨率无关，便于规模化成本预测；只提取姓名而非全页 OCR，把输出 token 控制在约 1,000/页（全量 OCR 约 4,500）。Claude 通过 adaptive thinking 自动调节推理深度，336 页全部启用了扩展推理，推理轨迹 544–1,658 字符；单页总延迟约 22–35 秒，其中 Claude 空间推理占 20–30 秒。文章还给出批量推理（5 折）、prompt caching（最高省 90%）、reasoning budget 三类进一步降本手段 ^[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md]

## 关键要点

- 两阶段流水线：Nova 2 Lite 单次调用完成照片检测（bounding box，0–1000 坐标系）+ 姓名位置提取 + 页面元数据；Claude Sonnet 4.6 每页调用一次做姓名-人脸空间匹配，两者共享同一坐标系无需转换。
- 实测结果：336 页年鉴 → 3,122 条关联，93.3%（2,912 条）置信度 ≥0.95，202 条在 0.90–0.94，仅 8 条低于 0.90。
- 成本对比：两模型方案 ~$0.033/页（Nova ~$0.0027 + Claude ~$0.030）vs 单模型 Claude 方案 ~$0.10/页；10 万页分别约 $3,300 和 $9,800。
- Nova 2 Lite 按固定 per-image 费率计费（不随分辨率变化），reasoning 设为 LOW 即可——336 页测试中 LOW/MEDIUM/HIGH 准确率无显著差异。
- Claude reasoning token 按输出计价（$15.00/M），Bedrock batch inference 可再打 5 折（Nova 阶段降至 ~$0.0014/页）。

## 来源

- 原文: [[raw/articles/pair-nova-2-lite-with-claude-for-cost-optimized-document-pro.md|Pair Nova 2 Lite with Claude for cost-optimized document processing]]
