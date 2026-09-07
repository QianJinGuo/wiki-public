---
title: "TouchWorld: 触觉基础模型与灵巧操作 — 破晓智能/哈工大"
created: 2026-07-12
updated: 2026-09-07
type: entity
tags: [embodied, robot, tactile, foundation-model, manipulation, icml-2026]
confidence: 0.75
provenance_state: extracted
sources: [raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026, raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# TouchWorld: 触觉基础模型与灵巧操作 — 破晓智能/哈工大

破晓智能（PHANES AI）与哈工大（深圳）杨朔教授团队发布 TouchWorld，一种兼具预测与反应能力的触觉基础模型，面向机器人灵巧操作。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md]

TouchWorld 让触觉同时承担两种角色：行动前预测"应该碰成什么样"，接触后再根据真实反馈快速纠错。模型不仅预测未来画面，还同时预测未来触觉图——哪根手指应产生压力、接触强度应达到什么状态。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md]

在浇花、桌面清理、电源插头插入、杯子插入、擦锅、抽纸巾六项真机任务中，TouchWorld 在无额外干扰场景下取得 65.0% 的平均成功率；加入目标移动、抓握干扰等扰动后成功率为 57.2%，分别超过最强基线 15.7 和 16.0 个百分点。每项任务采集 200 条遥操作训练轨迹，并进行 100 次真机评测。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md]

创始人杨朔曾获 Google PhD Fellowship（全球 9 人之一），博士阶段工作入选 ICLR Best Paper Finalist，26 岁任哈工大（深圳）长聘教授、博导，同年获评国家级青年人才。→ [[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026|原文存档]]

## 第 2 来源 — 量子位

量子位对破晓智能（PHANES AI）的报道聚焦于其创始人杨朔的创业故事和技术路线全貌。杨朔（26 岁哈工大长聘教授、博导，Google PhD Fellowship 获得者）创办破晓智能，围绕"机器人如何真正学会操作"搭建从数据、模型到控制的完整能力链。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md]

报道详细介绍了 Touch 系列技术路线：EgoTouch（触觉数据采集）→ TouchAnything（从视频恢复触觉）→ TouchWorld（触觉世界模型）→ HumanWBC（全身移动灵巧操作控制），构成一条从数据到世界模型再到全身控制的完整链路。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md]

TouchWorld 的核心架构包含 Predictive（触觉目标预测）和 Reactive（高频触觉反馈修正，频率为 World Model 的 4 倍）两个模块，在浇花、拔插头、擦锅等六项真机任务中达 65.0% 平均成功率。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md]

→ [[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026|第 2 来源原文]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

