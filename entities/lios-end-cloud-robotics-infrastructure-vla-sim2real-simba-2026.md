---
title: "LiOS 端云协同基础设施：具身智能柔性操作与虚实迁移（招商局狮子山人工智能实验室）"
created: 2026-08-20
updated: 2026-08-29
type: entity
tags: [embodied-ai, vla, sim2real, robotics, flexible-manipulation, end-cloud-infrastructure]
sources: [raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026]
confidence: 0.7
related: [concepts/embodied-intelligence-frontier, concepts/robotics-embodied-ai, entities/abot-agentos-robot-agent-os-amap-2026]
---

# LiOS 端云协同基础设施：具身智能柔性操作与虚实迁移

## 背景：柔性操作是具身智能的「技术试金石」

衣物折叠被公认为具身智能领域的技术试金石：衣物属无定型柔性物体，材质厚薄、褶皱、缠绕、摩擦力、弹性形变均存在强动态不确定性，考验柔性感知、双臂协同、接触力控、长程作业、状态恢复五大能力。行业主流方案普遍存在「仿真优秀、真机拉胯、场景受限」痛点，根因是仿真与真机的**虚实迁移鸿沟**（硬件刚性差异、装配误差、夹爪稳定性、底层控制精度偏差）。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md]

## LiOS 三层架构

招商局狮子山人工智能实验室自研 **LiOS 端云协同基础设施**，将具身智能从分散系统集成推进到 OS 级统一基础设施，分三层：^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md]

- **云侧**：多模态大模型分布式训练与推理优化、多模态数据湖仓管理、高并发仿真评估；可在 Qwen3-VL-30B-A3B / 235B-A22B、Wan2.2-T2V-A14B、DINOv3、V-JEPA-2 等基座上构建 VLA、WAM、WM 等具身基础模型。
- **端侧**：LiOS Runtime 接入异构机器人本体/传感器/末端执行器/边缘计算，实现传感器同步、运动控制、安全执行、接管恢复，结合 Real-time Chunking 等策略保持实时性与安全边界。
- **端云协同**：以低延迟可计算数据流把真实机器人现场接入云端，支撑推理、远程接管与数据回流；WebRTC/GStreamer 图传方案实现约 30ms（网络部分 24ms）「本地相机到云端显存」单向端到端延迟，较通用中继方案加速 2.1～6.9 倍，单路 GPU 解码吞吐每秒数千帧。

## 数据闭环与成果

实验室构建「覆盖训练、部署、轨迹采样与 Real2Sim 遥操作」的数据迭代闭环：分布式并行训练使模型训练吞吐提升 5 倍以上，仿真环境并行化重构使评测效率提升 4 倍以上。2026 年 6 月晋级 ICRA 2026 LeHome Challenge 决赛并击败包括 2025 BEHAVIOR-1K 冠军、Ilya 等对手，斩获全球第一名。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md]

三大核心能力演示：多双臂平台并列叠衣（统一接入/任务编排/控制执行框架屏蔽硬件差异）、多类衣物折叠（短袖/长袖/长裤拓扑自适应）、大形变整理（成团缠绕/褶皱/遮挡下拖拽、拉伸、展平恢复至可折叠状态）。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md]

## 相关

- 具身智能前沿 → [[concepts/embodied-intelligence-frontier]]
- 机器人具身 AI → "机器人与具身 AI"
- 端云机器人 OS 范式对比 → [[entities/abot-agentos-robot-agent-os-amap-2026]]

→ [[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026|原文存档]]
