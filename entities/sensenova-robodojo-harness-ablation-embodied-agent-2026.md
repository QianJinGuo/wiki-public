---
title: "SenseNova RoboRSI：给通用模型换 Harness，机器人成功率 15.83% → 54.17%"
created: 2026-10-20
updated: 2026-10-23
type: entity
tags: [harness-engineering, embodied-agent, rsi, vla, benchmark, sensenova]
sources: [raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026]
confidence: 0.8
---

# SenseNova RoboRSI：给通用模型换 Harness，机器人成功率 15.83% → 54.17%

商汤科技与南洋理工大学等机构的联合研究，用同一套 GPT-6-Astra 权重，只改 Harness（上下文知识包、动作接口、快慢模型协作方式、反馈与记忆管理），把 RoboDojo 的成功率从 15.83% 拉到 54.17%，平均分 22.96 → 60.08，**模型参数全程未动**。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]

这是「Harness 是能力放大器」在具身场景的一次量化复现：与 [[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]] 中编码 Agent 领域的结论同构——同一模型，脚手架决定上限。

## 评测设计：先补齐「不对等」的两项条件

现有具身基准多围绕专用机器人策略设计，直接拿通用模型零样本对比并不公平。团队调整了两项条件：

1. **允许利用训练集成功示范**：通过上下文学习（ICL）让模型在测试前分析训练集的图像、机器人状态、动作轨迹，整理出任务步骤、运动参考、视觉样例、完成检查与失败恢复方法，形成**冻结知识包**；测试时按当前观察检索使用，参数与知识包都不在回合内更新。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]
2. **放宽动作步数预算**：通用模型每步要观察、推理、调用工具、检查结果，1 步预算下「找到问题却来不及调整」。除 BEHAVIOR 外，其余 11 项基准步数上限扩展到最多 10 倍，仍在同一测试回合内保留状态与交互历史。

12 项基准覆盖机械臂、灵巧手、人形机器人、移动操作平台与视觉语言导航。结果（扩展预算下）：LIBERO-Long++ 100%、LIBERO-Pro 99.3%、SIMPLE 人形 98.8%、RoboDojo（42 任务 × 5 布局 = 210 回合）81.9%（标准预算 27.6%）、Bench2Dex 61.5%，而长程导航 LHPR-VLN 仅 28.9%——**追加步数解决不了长程进度追踪**。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]

## 快慢模型协作：审核节点的位置比审核频率更重要

RoboDojo 上以 GPT-6-Astra 为 System 2（推理与检查），任务微调的 π0.5 为 System 1（提动作候选），消融结果构成一条清晰曲线：

| 配置 | 成功率 | Token 量 |
|---|---|---|
| 仅 VLA（π0.5） | 6.91% | — |
| GPT-6-Astra 只下子目标、VLA 开环执行 | 5.83% | — |
| 每个候选动作都由 Astra 审核 | 23.61% | ~766 万 |
| 每 4 个候选审核一次 | 25.83% | ~280 万 |
| 上述 + ICL 知识包 | 54.17% | ~1933 万 |

^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]

关键结论：**大模型不需要审核每一个细小动作**。把检查放在合适节点，成功率与 Token 开销可以同时改善；而一旦叠加知识包，成功率翻倍但 Token 膨胀近 7 倍——具身 Harness 的成本结构是「知识换 Token」。

## 组件级消融：反馈子 Agent 有效，执行子 Agent 无效

- 增加**反馈子 Agent**：成功率 +8.89 个百分点。
- 压缩较早交互历史：成绩提升的同时 Token 降约 66.6%。
- 单独增加**执行子 Agent**：无提升。
- 动作接口：直接输出关节角度难以把空间判断转成精确控制，用末端位姿/相对方向指令更好；**动态选择动作块长度**优于固定 4 步。
- 失败模式样例：头部相机显示杯把已套上挂钩、腕部相机显示仍在桌上，模型未能及时确认接触状态——多视角冲突下的状态判定是典型短板。

^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]

## SenseNova-RoboRSI：把执行记录变成改进闭环

SenseNova-RoboRSI 把提示词、Skill、模型、工具/MCP 收进同一运行环境，执行环境与评测模块通过统一接口连接（独立 Docker 容器 + HTTP API）。每一步的观察、动作、反馈、得分都被记录，支持批量评测、中断恢复与轨迹导出；RSI 环节分析规划/动作预测/反馈/记忆组织的问题，提出候选修改并在相同条件下复测，**通过验证的改动进入下一版本，基础模型参数保持冻结**。积累的交互经验进一步用于训练 27B 具身模型 SenseNova-RoboRSI-27B-Preview。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]

这是 [[concepts/agent-self-improvement-loops|Agent 自我改进闭环]] 的具身版：改进对象是 Harness 而非权重，验证靠同条件复测。真机侧接入了 8 种配置（单臂、双臂、灵巧手、轮式移动平台、宇树 G1 人形）。

## 代价与边界

高完成率伴随可观时间成本：LIBERO-Long++ 标准预算下已超 97%，平均每回合仍消耗约 100 万 Token、53 分钟；RoboDojo 扩展预算下平均超过 3 小时。ICL 的收益也依赖「任务与示范接近」——RoboDojo 的 Open 任务缺少专属示范，知识包帮助有限，**经验向陌生任务迁移**仍是开放问题。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md]

## 深度分析

### 一、三个成功率数字来自三套装置

文中至少三组成功率：Harness 消融内部的 15.83% → 54.17%（同基准、同模型、只改脚手架）；预算实验的 27.6% → 81.9%（210 回合，步数扩到 10 倍）；榜单参考分 22.48%。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:23-27] 混读会得出错误因果。

仅放宽步数就值五十多个百分点，比整套 Harness 消融的增益还大；15.83% 那个基线本就压缩了行动带宽与信息条件，度量下限而非上限。上界对比更有信息量：RoboDojo 实测 81.9% 对乐观上界 90%，缺口只剩 8 点，「步数不够」已被榨干；LHPR-VLN 10 倍预算只到 28.9%。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:77-81] 这把「资源约束型失败」与「表征约束型失败」分开，也是重心从预算转向 Harness 的理由。

### 二、审核跨度的经济学

反常拐点：Astra 只下子目标、VLA 开环执行，成功率 5.83%，**比单用 VLA 的 6.91% 还低**。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:111-115] VLA 在具体轨迹上微调，擅长续出下一个动作块；逼它理解语义子目标，等于让它在没被训练过的条件分布上推断。

「每 4 个候选审一次」因此有第二重解释。766 万 → 280 万省下的主要是**轮次**：每轮 System 2 都要重发累积观察与知识包，成本随轮次近似二次增长。成功率同时升到 25.83%，说明逐动作审核还打断了本可收敛的序列——单步判据太少，模型容易把「看不清接触状态」误判成「这步错了」，于是拒绝、重采样。跨度 4 把判决点对齐到接触状态真正变化的动作块边界：时机应由状态节奏决定，而非固定计数。

### 三、子 Agent 的不对称

反馈子 Agent 带来 +8.89 点，因为它把「任务是否真的完成」外置成独立观察者。杯把案例里头部相机说已套上挂钩、腕部相机说还在桌上，主循环选了符合预期的那条再沿错误前提继续调——单线程推理无法同时持有互斥观测并主动消解。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:29-29] 执行子 Agent 无提升则相反：动作生成已有比 LLM 更可靠的执行者（VLA、脚本、末端控制），再插一层只增延迟与噪声。

压缩较早历史在成绩提升同时砍掉约 66.6% 的 Token ^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:127-129]，佐证长上下文对具身任务是负资产。判据因此不是「加不加」，而是**子 Agent 只能加在主循环看不见的地方**：观测冲突需仲裁，或粒度不够需换视角（双臂插花时移动空闲机械臂，让腕部相机看到花瓶口，把观察角度变成可规划的动作）。已有专用控制器且信息充分之处，加层只是收税。

### 四、RSI 回路搬运的是成本

RoboRSI 的改进对象是 Harness 而非权重：执行记录 → 定位规划/动作预测/反馈/记忆的问题 → 同条件复测 → 通过者进入下一版本，参数冻结。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:103-109]

闭环在 27B 那步合上：RoboCasa365 上 26.0% → 46.5%，输出 Token 59.5K → 15.9K（降 73.3%），成本约 Astra 的 1/50。^[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026.md:165-167] 机制是把「每步显式推理一遍」的检查蒸馏成行为倾向：成本只是从推理时搬到训练时。但 27B 在 RoboDojo 只有 20.83%，远低于 Astra 的 54.17%：蒸馏下来的是行为倾向而非泛化能力，离开经验覆盖过的形状就退回规模上限。这与 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 中「脱离配置栈谈分数没有意义」一致。

## 实践启示

1. **先把步数预算当一等超参调。** 判定「模型不行」前画预算-成功率曲线并标上乐观上界：贴近上界则瓶颈在决策质量，远离则在表征与进度追踪。

2. **审核点对齐状态变化，不要对齐每个动作。** 扫 1 / 4 / 8 三档跨度取拐点；逐动作审核既贵又会打断本可收敛的序列。

3. **加子 Agent 前先问它补哪个观测缺口。** 只有「观测冲突需仲裁」与「粒度不够需换视角」值得加层；在已有专用控制器的动作生成上再插执行子 Agent，实测无提升。

4. **把上下文压缩设为默认开启。** 压缩较早历史在提升成绩同时省约 66.6% 的 Token；取舍与 [[concepts/100-line-vs-managed-harness-tradeoff|100 行 vs 托管 harness 权衡]] 同构——省下的必须是重发成本，不是判断依据。

5. **动作接口给末端位姿或相对方向，粒度要匹配执行器分布。** 直接输出关节角度效果更差；给 VLA 一个语义子目标再让它开环跑，可能比它自己跑更差。

6. **执行与评测共用一套接口，逐回合落盘观察/动作/反馈/得分，再用验证过的轨迹蒸馏小模型并以同一 Harness 回归。** 没有轨迹，组件级消融就做不出来。

## 关联

- [[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]]
- [[concepts/agent-self-improvement-loops|Agent 自我改进闭环]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[concepts/agent-role-specialization|Agent 角色专业化]]
- [[entities/sensnova-u1-deep-dive-jiqizhixin-d8602ded5c51|SenseNova U1 深度解析]]

→ [[raw/articles/sensenova-robodojo-harness-ablation-embodied-agent-2026|原文存档]]
