---
title: "TouchWorld: 触觉基础模型与灵巧操作 — 破晓智能/哈工大"
created: 2026-07-12
updated: 2026-09-18
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

## 深度分析

### 预测与反应：为什么单一 world model 不足以处理接触

TouchWorld 标题里的两个关键词是 Predictive 与 Reactive：前者在动作前预测子任务完成时画面该是什么样、手上压力该是什么样；后者在真正接触后持续读取触觉信号与关节状态，在原有动作上叠加「手指往左偏一点、握力再加一点」这类 delta 修正。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:74-124]

两层时钟完全不同：Reactive 层推理频率是触觉世界模型的 4 倍，每次只输出修正量而非重新生成完整动作；若杯子放偏、插头倾斜、人手突然干扰这类变化都要求上层大模型重新推理整段动作，速度根本来不及。上层负责「往哪走」，触觉负责「碰到之后别走偏」。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md:150-156]

### 触觉图预测 vs 视频预测：接触瞬间视觉被遮挡

按喷壶扳机时，视觉上「手指贴在按钮上」与「按钮真的被按下」几乎是同一幅画面，手一遮挡，图像就无法判断任务是否完成；触觉上有没有接触、压力够不够，是更直接的判据。TouchWorld 因此在预测未来画面之外，同时预测一张未来触觉图——哪根手指产生压力、压力在指尖还是掌侧、接触强度达到什么状态，相当于给机器人一个物理目标：画面对了不算完成，手指压出预期接触才算完成。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:79-104]

这与纯视觉路线形成分野：[[entities/light-interaction-world-model-inference|Light Interaction World Model]] 把物理交互压缩进视觉表征里推理，而 TouchWorld 主张接触必须成为一等输入与一等目标，[[entities/vla不够了触觉将改写具身智能新格局|VLA 不够了，触觉将改写具身智能]] 的论证也指向同一方向。

### 为什么触觉不能像一块新积木塞进 VLA

VLA 已容纳视觉、语言与动作，再加一份触觉似乎顺理成章，但触觉与视觉的信息密度、出现时机和处理速度完全不同：两分钟的任务里真正接触可能只有十几秒，图像是百万级像素而单手触觉只有几百维；同频训练时模型很容易只靠视觉「偷懒」，结果是触觉被视觉淹没。TouchWorld 因此拆成三个时间尺度：1Hz 高层拆解任务并预测视觉—触觉目标，10Hz 中层生成主体动作，30Hz 触觉反应层实时纠错。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:165-185]

消融实验支持分层并非多此一举：拿掉触觉输入后成功率从 65.0% / 57.2% 降至 43.3% / 30.0%，拿掉 30Hz 触觉修正层则降至 55.3% / 40.3%，在目标移动或抓握受扰时尤其明显。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:200-205]

### 真机证据：六项接触密集任务与数据成本

浇花、桌面清理、电源插头插入、杯子插入、擦锅、抽纸巾六项任务各自对应一种触觉难点：按压喷壶扳机、精密插接、持续调节压力、柔性物体稳定拉取，以及在多个子任务之间切换时保持抓取稳定。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md:158-164]

结果是无扰动 65.0%、人为扰动（目标移动、抓握干扰）57.2%，分别超出 Pi-0.5、FTP-1、GR00T N1.7 中最强基线 15.7 和 16.0 个百分点。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:26-37] 更值得注意的是评测成本结构：每项任务 200 条遥操作轨迹加 100 次真机评测，结论来自重复试验而非挑选个案。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md:166-172]

### 从 EgoTouch 到 HumanWBC：数据 → 世界模型 → 控制的链路

EgoTouch 用头戴第一人称相机、腕部近距离相机、手部姿态追踪与密集触觉手套，同步记录多路 RGB、双手 3D 姿态与连续压力分布，覆盖 208 项任务、1891 段交互、超过 20 小时视频与 1000 余个物体；TouchAnything 用这些视觉—触觉对齐数据训练模型从普通视频恢复接触区域与压力分布，扮演「数据放大器」。TouchWorld 的触觉世界模型先在 20.2 小时 EgoTouch 人类数据上预训练，再用 10 小时机器人演示数据微调。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:237-297]

链路末端是 HumanWBC——全身移动灵巧操作（loco-manipulation）控制模型，把感知、自主移动、全身控制、双臂协同与灵巧手操作接进同一系统；这条路线可与 [[entities/motus-2-self-evolving-world-model-dexterous-manipulation-2026|Motus-2 自演化世界模型]] 对照，差别在于 TouchWorld 把触觉抽成预测目标与纠错通道。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:344-349]

### 局限与开放性：65% 远不是终点

论文没有把 65.0% 包装成「灵巧操作已被解决」，作者明确承认距离大规模泛化还有很长距离。更真实的边界在基础设施：高自由度、全掌触觉且稳定的灵巧手尚不成熟，触觉数据难采，手套噪声大、标定漂移、批次差异明显，传感器之间数据表示不统一，统一 benchmark 也尚未建立。^[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026.md:170-190]

另一个未回答的问题是泛化：六项任务都在固定任务族与特定硬件上验证，能否迁移到新物体、新任务与新本体仍待检验，这也是 [[concepts/embodied-intelligence-frontier|具身智能前沿]] 尚未收敛的核心问题。短期可验证信号，是团队承诺逐步开源 EgoTouch、TouchAnything 与 TouchWorld 的数据、代码与模型。^[raw/articles/poxiaointelligent-tactile-robot-foundation-model-2026.md:384]

## 实践启示

1. **按时间尺度分层，而不是把能力塞进一个模型。** 1Hz 规划 / 10Hz 生成 / 30Hz 纠偏的三分层，配上拿掉触觉输入后 65.0%→43.3%、57.2%→30.0% 的消融结果，说明对接触密集任务分层是可测量的增益，而非架构洁癖。

2. **完成判据不要只写在视觉里。** 手遮挡物体的瞬间，「图像上看起来对了」与「物理上真的完成了」会分叉；为每个子任务补一张触觉目标图（哪根手指、多大接触强度），等于给策略一个不依赖视觉的物理完成信号。

3. **用「数据放大器」思路解决触觉数据稀缺。** 先用少量带传感器的对齐数据教会模型视觉与触觉的对应关系，再为海量无标注的第一人称视频补上接触监督，比单纯堆采集量更可扩展。

4. **把评测成本提前算进预算。** 每任务 200 条遥操作轨迹加 100 次真机评测才换到一组可信的成功率数字；把遥操作采集与真机重复评测当作基础设施而非一次性支出，才可能支撑持续迭代。

5. **让模型「知道做到哪一步」可能比把模型做大更有效。** 经过任务阶段监督与记忆增强的 4B 子任务规划模型准确率达 91%，反超零样本 32B 模型的 84%，说明状态进度与可执行性建模的价值未必低于参数规模。

6. **硬件与数据表示的碎片化才是真瓶颈。** 手套噪声、标定漂移、传感器表示不统一、缺统一 benchmark——这些问题比模型架构更难复制；[[entities/vision-based-tactile-sensor-overrated-critique-2026|对视觉触觉传感器的批评]] 与 [[entities/handroid-reconfigurable-robot-dexterous-hand-humanoid-unc-stanford-2026|Handroid 可重构灵巧手]] 是另外两条相关线索。

→ [[raw/articles/98-哈工大杨朔破晓智能touchworld-tactile-world-model-2026|第 2 来源原文]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

