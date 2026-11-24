---
title: "EmbodiedSkills 闭环AgentLoop组织VLA长任务"
created: 2026-09-24
updated: 2026-11-16
type: entity
tags: ['embodied-ai', 'agent', 'vla']
sources: [raw/articles/embodiedskills-closed-loop-agentloop-vla-2026]
confidence: 0.7
vxc: 49
score: v=7/c=7/stars=4
provenance_state: extracted
---

# EmbodiedSkills 闭环AgentLoop组织VLA长任务

EmbodiedSkills 是浙江大学联合南京航空航天大学、Cornell、新加坡国立大学、宇泛智能与云深处科技提出的 VLA 系统层框架（arXiv:2609.01281），^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:31-39] 核心主张是：高层视觉语言模型决定下一步做什么，低层 VLA 策略完成当前子任务，运行时负责检查，验证模块负责确认动作是否真的产生了进展。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:19] 它把高层模型、低层策略与机器人环境放进一个闭环 AgentLoop，每次动作调用都经过运行时检查，动作结束后重新获取证据，后续决策使用更新后的状态。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:53-55]

## 核心要点

- **问题定位**：VLA 动作能力不断提高，但长任务执行容易出错——任务由多个连续阶段组成，需要管理子目标、执行边界和物理反馈，而不只是"观察加指令，输出动作"。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:21-23]
- **VLA model vs VLA agent**：前者学习观察到动作的映射；后者还要管理任务阶段、动作边界、执行结果和失败后的恢复。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:69]
- **提议-检查-执行**：模型输出被视为待执行的操作提议，经过技能契约与前置条件检查后才落地，被拒绝的提议连同原因和证据写入轨迹。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:73-81]
- **六阶段语义**：Observe、Plan、Preflight、Execute、Verify、Recover 构成 AgentLoop 的主要语义，运行时可依状态回到前面的阶段。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:95-107]
- **训练-部署同构**：高层 Qwen3-VL + 低层 OpenPI/π₀.₅，训练样本沿用与部署一致的上下文边界。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:111-123]
- **评测结果**：RoboTwin 2.0 五十类任务平均 86.20%（参考 π₀.₅ 为 82.74%），LIBERO 四套平均 97.40%，而 RMBench 记忆依赖任务平均仅 12.5%。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:129-153]

## 深度分析

### 为什么单个 VLA 模型撑不起长任务

在短时操作中，动作预测可以写成"观察加指令，输出动作"；一旦任务由多个动作组成，系统还要连续回答：现在正在完成哪个目标？上一步是否完成？动作是否满足执行条件？继续当前尝试还是切换子目标？^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:59-63] 以"把容器放到盘子上"为例，机器人要经历目标识别、接近、抓取、移动和放置——抓取不稳定时移动没有可靠起点，容器接近盘子但姿态不对时任务也不能判定完成。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:63] 真实环境还会叠加机械臂遮挡相机、动作块在接触阶段提前停止、物体只移动了一部分、旧观察因场景变化失效等偏差。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:65-67] 这意味着下一轮决策需要同时参考当前画面、原始计划、活动子目标和执行记录——这是一层系统问题，不是更强的动作模型能单独解决的。

### 闭环 AgentLoop：动作先当提议、通过检查再执行

EmbodiedSkills 将每次模型输出视为一项待执行的操作提议：模型提出下一步做什么，运行时检查输入是否齐全、观察是否新鲜、活动子目标是否有效、机器人和动作后端是否就绪、当前阶段是否允许调用该技能；提议通过后低层 VLA 才生成动作块。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:73-77] 动作结束后环境返回新的视觉证据和执行结果，写回任务状态再交给下一轮决策，分支逻辑是：局部目标完成则推进，未完成则保留继续尝试，证据不足则重新观察，计划失效则重新规划或恢复。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:77-81] 每个 embodied skill 以统一接口声明输入、输出、前置条件、执行过程、状态更新和失败证据，不同技能可以用不同模型或确定性程序接入同一 AgentLoop。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:85]

### 六阶段循环与有界的历史上下文

六阶段中，Observe 只使用部署时可见的图像与机器人状态（模拟器隐藏状态不作策略输入）；Plan 保留完整计划并明确当前子目标；Preflight 在物理动作发出前拦截缺失输入、过期观察和非法阶段转移；Execute 用动作预算限制单次尝试长度，一个子目标可对应多个动作块；Verify 读取动作后的场景变化，决定继续、推进、重新观察或重新规划；Recover 处理无法继续的尝试，恢复后的操作重新过执行前检查。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:99-107] 历史记录采用"有界的部署一致上下文"：完整计划和活动子目标始终保留，最近的决策、技能结果、执行报告和错误证据按序加入，较早记录按固定预算压缩或删除。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:111-115] 任务执行到一半时，当前子目标、此前动作块和验证结果继续留在上下文里，下一轮模型据此继续，无需重新猜测已完成的阶段。观察、计划、动作和验证结果都带来源与新鲜度信息，关键证据变化时依赖它的旧结果被标记为需要刷新。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:91]

### 训练-部署同构与评测证据的边界

训练数据按同一套接口组织：高层样本包含任务指令、当前图像、阶段、完整计划、活动子目标、近期历史和上一轮执行结果；低层样本用子任务级专家轨迹，将观察、机器人状态、简洁子任务和对应动作绑定。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:123-125] 一条结构化轨迹可同时供给规划器、动作策略、验证器和恢复模块。评测上，RoboTwin 2.0 五十类任务每类 100 episode、按环境终局评测器计算，平均 86.20% 对参考 π₀.₅ 的 82.74% 提升 3.46 个百分点，39 个任务超过参考、1 持平、10 略低；LIBERO-Long 从 92.4% 升到 93.6%，是四套中最大提升，说明子目标与动作边界在更长任务序列中更容易体现作用。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:129-145] 但 RMBench 四个记忆依赖任务平均仅 12.5%——统一接口能承载带历史的子任务调用，长期尝试记录、状态记忆和依赖历史的高层决策仍是主要难点。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:149-153] 三组消融（共 5000 episode）进一步拆出三项作用：去掉 Verify 后系统保留完整动作预算却无法按真实进度调整，提前进入下一阶段或发出无效动作并让后续阶段继承错误状态；去掉语义子任务平均成功率降到 34.4%；每个子任务只执行一个动作块时只有 19.5%。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:157-165]

## 实践启示

1. 具身系统评估动作模型时应区分 VLA model 与 VLA agent：前者只考映射质量，后者要考任务阶段管理、执行边界与失败恢复——选型与 benchmark 设计都要按后者补齐。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:69]
2. 把模型输出当作"待验证的操作提议"而非直接指令，配合前置条件检查与拒绝记录，是任何长任务 agent（不限于机器人）都可以借用的运行时模式。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:73-81]
3. Preflight 在动作发出前拦截问题、Verify 在动作结束后读取真实进展，两者配对才能避免"提前进入下一阶段"和"无效动作空转"两类典型失败。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:157-159]
4. 训练数据按部署接口组织、上下文边界训练与推理一致，可以避免模型在训练和正式运行时看到不同类型的信息分布。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:113-123]
5. 动作预算与语义子任务缺一不可：消融显示去掉子目标语义掉到 34.4%，固定单动作块掉到 19.5%——长任务需要"明确目标 + 弹性执行空间"的组合。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:161-165]
6. 记忆依赖任务（12.5%）与常规操作任务（86.20%）的巨大落差提示：闭环执行组织解决的是执行链路问题，长期状态记忆与依赖历史的高层决策仍是独立且更难的子问题。^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md:149-153]

## 关联

- [[entities/simplememvla-native-visual-context-robot-memory-2026|SimpleMemVLA：原生视觉上下文作为机器人记忆]]
- [[entities/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026|RoboSkill：具身技能的探索-执行-进化]]
- [[entities/sensenova-robodojo-harness-ablation-embodied-agent-2026|RoboDojo：具身 agent harness 消融]]
- [[entities/xiaomi-robotics-1-embodied-base-model-scaling-2026|Xiaomi Robotics-1 具身基座 Scaling]]
- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[concepts/harness-loop-architecture|Harness 循环架构]]
- [[concepts/agent-memory-architecture|Agent 记忆架构]]
- [[concepts/agent-orchestration-patterns|Agent 编排模式]]

→ [[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026|原文存档]] ^[raw/articles/embodiedskills-closed-loop-agentloop-vla-2026.md]
