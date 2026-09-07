---
title: "EgoSuite-Open100K：10万小时全模态人类行为数据开源与具身智能数据 Scaling"
created: 2026-08-25
updated: 2026-09-07
type: entity
tags: [embodied-ai, robotics, dataset, human-video-data, scaling-law, world-model, multimodal, data]
sources: [raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026]
confidence: 0.7
related: [embodied-ai-data-market-landscape-97-players-44-billion-2026, jingshuo-tech-embodied-ai-data-infrastructure-2026, lerobot-v060-imagine-evaluate-improve, xiaomi-robotics-1-embodied-base-model-scaling-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# EgoSuite-Open100K：10万小时全模态人类行为数据开源与具身智能数据 Scaling

## 核心事件

2026 年 8 月 20 日，在 2026 世界机器人大会（WRC 2026）上，光轮智能宣布将 **10 万小时全模态人类行为数据全部开源**。数据集命名为 **EgoSuite-Open100K**，与 Hugging Face 联合发布，首批已上线 Hugging Face Hub 与 AtomGit，学术研究与商业训练均可使用。Hugging Face 官方账号罕见地单独为该项目站台，其机器人项目 LeRobot 也发长帖力挺——这类待遇在 HF 历史上仅见于 IBM/NASA 的 Surya、Databricks 的 CommonCanvas、Flux.2-dev 等标志性开源事件。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

## 数据规模与标注

EgoSuite-Open100K 是目前全球规模最大的全标注开源第一人称人类数据集。采集设计上把多样性提前设计进采集环节：**15,000+ 个独立真实采集场景**、**15,000+ 项具体任务**、**7 大类环境**（家居、餐旅、零售、运动、物流、办公、工业）、128 种场景类型、18 类任务大类。现有公开第一人称数据绝大多数集中在家庭厨房，独立场景数只有零头，该数据集把场景广度拉出了一个数量级的差距。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

每条视频带**三层标注**：
- **手部位姿**：21 个关节点，逐帧标注，教机器人手指关节怎么弯、怎么捏；
- **身体位姿**：教它怎么伸手、怎么弯腰、怎么全身协调；
- **事件级语义标注**：告诉它这一步在干什么、动的是哪个物体。

三层叠加让模型既知道「手怎么动」又知道「为什么这么动」。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

数据集分两条线：**EgoStandard** 是主体，头戴视角，跟着人的眼睛学；**EgoPro** 额外挂腕部摄像头，贴在手腕上拍手指接触物体瞬间的细节（螺丝角度、捏袋力度、插入卡槽时机）——这些最精细的操作瞬间在头戴视角里恰好被手臂挡住，而腕部视角采集成本高、公开数据里几乎见不到。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

## 物理 AI 的 Scaling Law：人类视频数据是关键资产

文章的核心论点是：训练机器人最稀缺的资源不是 GPU、不是算法，而是**高质量的人类行为视频数据**。几组证据：^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

- **Dyna Robotics DYNA-2**：预训练阶段一条机器人动作数据都没喂，只灌进超过 100 万小时的人类第一视角视频。从 1000 小时到 100 万小时，每翻 10 倍机器人都在变聪明，是一条持续上升、没有明显撞墙的幂律趋势——「物理 AI 终于有自己的 Scaling Law」。
- **跨具身迁移拐点**：DYNA-2 把跨具身迁移的拐点定位在 **1 万到 10 万小时**之间。1 万小时以下，模型在 A 厨房用 B 刀切胡萝卜就只会在 A 厨房用 B 刀切；过了 1 万小时曲线开始拐弯，到 10 万小时级别跨具身迁移信号变得明确稳定。这是机器人「开窍」的那道坎，而绝大多数团队手里的数据量远不够——**Scaling Law 谁都知道，但只有极少数玩家有资格验证它**。
- **NVIDIA EgoScale**：用 20,854 小时的第一视角人类视频训练 VLA 模型，第一次画出清晰的对数线性扩展曲线——人类数据小时数与验证损失呈对数线性关系，R² = 0.998，在 22 自由度灵巧手上平均成功率提升 54%。
- 其他佐证：Generalist AI 用 270,000 小时物理交互数据训出 GEN-0；Sunday Robotics（做出 ALOHA 的团队）证明人类示范数据采集可以走出实验室、在真实家庭环境大规模跑起来。

## 数据采集的不可能三角

10 万小时的数据采集面临「规模、多样性、标注质量」的不可能三角：量做大了覆盖面就窄，覆盖面铺开了标注深度跟不上，标得足够细规模又上不去。光轮智能的解法是把多样性提前设计进采集环节——按同一套规范作业，去什么场景、做什么任务都照着覆盖目标严格控制。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

头戴视频的抖动 + 双手关节被身体/物体/彼此遮挡，是位姿估计的噩梦，也是标注流水线上工程量最大的地方。同类数据集到了这个量级通常只放原始视频（标注成本太高），EgoSuite-Open100K 带全模态标注，质量远高于只是公开第一视角的数据集。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

## 行业痛点：数据无法拼合

当前具身智能行业最大的问题不只是数据不够多，而是各家采的数据根本没法拼在一起用——采集口径、标注规范、数据格式、时序组织都不一样。你用 A 家设备采一万小时、我用 B 家设备采两万小时，两堆数据放一起模型根本吃不下去。统一的数据标准和可拼合性，是数据成为行业基础设施的前提。^[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026.md]

## 相关

- [[entities/embodied-ai-data-market-landscape-97-players-44-billion-2026|具身智能数据市场]]
- [[entities/jingshuo-tech-embodied-ai-data-infrastructure-2026|京硕科技具身数据基础设施]]
- [[entities/lerobot-v060-imagine-evaluate-improve|LeRobot]]
- [[entities/xiaomi-robotics-1-embodied-base-model-scaling-2026|小米机器人基座模型 Scaling]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- 机器人具身智能

→ [[raw/articles/egosuite-open100k-embodied-human-video-data-open-source-2026|原文存档]]
