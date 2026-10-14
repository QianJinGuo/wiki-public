---
title: "Token用量减少74.9%：Real2Gym把仿真经验变成真机可复用技能"
created: 2026-10-15
updated: 2026-10-15
type: entity
tags: [embodied, sim2real, agent, robotics, shanghai-ai-lab]
sources: [raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能]
confidence: 0.7
---

# Token用量减少74.9%：Real2Gym把仿真经验变成真机可复用技能

## 核心贡献

Real2Gym 由上海人工智能实验室联合上交大、浙大、中科大、港中文、港大提出，从人类或机器人操作视频构建**可交互仿真环境**：Agent 在其中探索、积累技能，再将经验迁移到真实机器人。论文 arXiv:2609.37089，代码与项目主页均已公开 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。

## 场景重建：物理可行性是验收标准

多模态模型生成的三维场景"重建得像"不等于"能复现操作"——机器人操作对几何误差极其敏感，杯口稍窄或容器碰撞模型封住内腔都会让画面合理的动作物理失败。Real2Gym 把**动作能否在物理引擎中完成**作为场景验收的一部分 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。

两环节管线：① 多视角输入用 Pi3X 几何线索 + SAM2 分割约束物体尺度与位置，构建 Blender 场景与 MuJoCo 环境；② 围绕**交互关键帧**（抓起/接触/释放时刻）做 Agent 自检纠错，再回 MuJoCo 执行验证支撑、接触与稳定性，轨迹导回 Blender 与原视频对照 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。

## Agent 侧：阶段级代码决策 + 技能库自进化

每轮决策目标从生成单个动作提升为**生成执行一个操作阶段的代码**（接近、闭合夹爪、抬起并验证，一次响应打包多个动作与验证条件），SAM3 提供物体定位、GraspNet 提供抓取候选。经验以相对物体的行动/检查/修正描述存入技能库，底层模型权重不变——每次新任务从独立上下文开始接收选定技能，可检验经验本身的作用 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。

## 实测结果

- DROID + EgoDex 共 24 场景：重建在内容/视角对齐、动作保真度上优于 GPT-5.6 Sol / GPT-6 Astra 基线 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。
- Agent 评估：复用首轮经验后二次执行平均成功率 **87.5%**（GPT-6 Astra Direct Mode 基线 70.8%）；策略执行 **Token 用量减少约 74.9%**；平均模型响应次数 31 → 14 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。
- Franka 真机：方块入盘/挂杯/微波炉关门 3 项任务全成（基线分别 1/3/1 次成功）；"盘子上架"失败后经人类示范 → 数字孪生 → 仿真迭代 3 次获得技能 → 真机完成，展示完整闭环 ^[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能.md]。

## 关联

- [[entities/roboflywheel-embodied-simulation-assets-2026|RoboFlywheel 百万级仿真资产]] — 同为"仿真内容供给具身训练"路线，Real2Gym 侧重从视频逐场景构建可交互环境
- [[entities/李飞飞署名具身新论文sim2real烧不起real2sim量大管饱|sim2real 烧不起，real2sim 量大管饱]] — Real2Sim 降成本的同一趋势
- [[entities/self-harness-shanghai-ai-lab-agent-improves-harness|Self-Harness（上海AI Lab）]] — 同机构的 Agent 自改进路线
- [[entities/egosuite-open100k-embodied-human-video-data-open-source-2026|EgoSuite/Open100K]] — ego 视频数据侧的互补层

→ [[raw/articles/token用量减少749real2gym把仿真经验变成真机可复用技能|原文存档]]
