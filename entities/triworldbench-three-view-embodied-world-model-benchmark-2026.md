---
title: "TriWorldBench：首个三视角具身世界模型榜单（北大/清华/北航/上交/中科大）"
created: 2026-08-27
updated: 2026-09-07
type: entity
tags: [ai, world-model, benchmark, embodied-ai, evaluation, multi-view, robotics]
sources: [raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026]
confidence: 0.68
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# TriWorldBench：首个三视角具身世界模型榜单（北大/清华/北航/上交/中科大）

> **v×c score**: 56 | stars=4
> **来源**: [[raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026|TriWorldBench 原文存档]]

北大、清华、北航、上交、中科大联合发布**首个三视角具身世界模型榜单 TriWorldBench**，引入新评测标准：当机器人拥有多个摄像头时，头部视角与腕部视角看到的场景是否保持同一真实世界。^[raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026.md]

## 三视角一致性：评测核心

基准的核心假设是：多视角（head-wrist）生成应共同解释同一次机器人操作。头部摄像头负责任务指令、机械臂轨迹与最终结果；腕部摄像头负责接触、抓取稳定性、滑移与局部几何。评测不要求不同视角画面在像素上相同，而是判断它们能否共同解释同一次操作。^[raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026.md]

## 六维评测体系

基准包含 **500 个三视角同步 episode、50 个机器人操作任务**，围绕六个维度汇总 **19 项评测信号**，以 **TWB-Score** 作为榜单总分：^[raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026.md]

- 三视角一致性（central，head-wrist 语义判断 + 联合三视角问答）
- 任务对齐（指令是否真正执行、机械臂运动是否合理、抓取/移动/释放操作是否正确）
- 物理与 3D 一致性（物体位置关系、动作变化过程、跨视角空间对应）
- 运动质量 / 时序一致性 / 视觉质量

## STATE 标注：区分"动"与"稳"

TriWorldBench 从参考机器人轨迹构建 STATE 标注，识别当前动作阶段、活动机械臂、每个视角预期运动/静止。该动的腕部长时间冻结会被扣分，不该动的相机出现多余运动同样被识别——稳定不再等同于"全程不动"。^[raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026.md]

## 定位

TriWorldBench 不是简单排名，而是**诊断工具**——通过把三路视频建立成"同步数据→动作状态→多层结果"的完整评测链路，帮助研究团队解释模型为何得分高/低，把清晰改进方向反馈给研究者。它让世界模型评价从"看起来像"走向"真正可用"，是具身世界模型评测的标准化补充。

## 相关方向

- [[entities/world-model-evaluation-position-paper-nju-2026|世界模型评估立场论文（南大）]]
- [[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0 跨本体多视角世界模型]]
- [[entities/feifei-li-masked-visual-actions-world-model-2026|李飞飞 masked-visual-actions 世界模型]]
- Agent 评估基准
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[entities/vbot-embodied-genome-cross-embodiment-inheritance-qinhailong-2026|Vbot 具身基因组]]

→ [[raw/articles/triworldbench-three-view-embodied-world-model-benchmark-2026|原文存档]]
