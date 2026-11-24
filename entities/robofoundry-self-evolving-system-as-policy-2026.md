---
title: "RoboFoundry：Self-Evolving System-as-Policy 具身框架"
created: 2026-10-14
updated: 2026-11-14
type: entity
tags: [embodied-ai, agent-harness, self-evolution, robotics, system-as-policy]
sources: [raw/articles/robofoundry-self-evolving-system-as-policy-2026]
confidence: 0.7
---

# RoboFoundry：Self-Evolving System-as-Policy 具身框架

NTU、北航、NUS、上交与云蝶科技联合提出，定位为首个把智能体系统本身形式化为「自进化的系统即策略」（Self-Evolving System-as-Policy）的具身框架：**基座模型冻结，进化发生在模型周围的上下文系统、技能系统与执行接口**，借助共享语义接口在不同机器人间复用 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md]。

## 叙事支点：从 Code-as-Policy 到 System-as-Policy

EmbodiedBench 上同一个 GPT-5.5：直接使用平均成功率 56.9%；不改一个参数、只让周围系统持续进化 → 72.7%。差距全部来自模型之外：哪些信息进入上下文、哪些经验被保留、决策如何落到物理动作。

与 [[entities/agent-self-evolution-flywheel-tencent-2026]]（单组件记忆/技能进化）的区别：RoboFoundry 把**整套系统**当作可进化对象，且诊断信号来自执行中暴露的问题（而非仅用于评测）。与浙大 [[entities/hugagentos-zju-self-evolution-agent-os-2026]] 同属「系统级自进化」家族，但本作聚焦具身 + 跨机器人语义接口复用。

## 深度分析

### 系统即策略：把模型周围的整套系统当作可优化对象

论文完成的是一次「策略」定义的范围迁移。Code-as-Policy 时代，模型把感知与控制原语组合成程序，决策核心仍是「模型 + 精心搭建的外围」；而既有工作大多只改进记忆、技能或交互 harness 中的某一个组件，执行中暴露的问题也只用于评测或人工调试，同样的弱点在回合之间反复出现 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:20]。RoboFoundry 的回答是把智能体系统本身形式化为策略：基座模型冻结，被优化的是模型周围的上下文系统、技能系统与执行接口 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:24]。

系统的存在形态是文件：模型像开发者维护代码库一样，用 cat、grep 查看，用 add、modify、delete 修改，并分为服务当前任务的任务级系统与跨任务、跨本体持续保留的通用系统两层 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:43]。这与 [[concepts/agent-harness-engineering-paradigm]] 的主张互为印证——harness 不是一次性装配物，而是持续演化的资产；RoboFoundry 的特殊之处在于让进化的驱动信号直接来自执行轨迹，与 [[concepts/agent-self-improvement-loops]] 的闭环框架同构，只是闭环对象从模型或单个组件扩大到了整套系统。

### 内外双循环与六能力归因：有诊断的进化，不是无约束重写

进化按内外两层循环推进：内循环执行任务并记录轨迹，外循环根据轨迹提出修改，再交回后续执行检验。修改作用在两个「进化面」上——上下文面通过 SAVE、RETRIEVE、UTILIZE 三类操作，决定当前回合的活跃上下文与跨回合的文件系统记忆里记什么、取什么、怎么用；技能面覆盖从原子技能、「移动—抓取—放置」式组合流程到失败恢复树的全部层级 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:45-49]。

关键在诊断环节：失败或低效先被归因到六种能力之一——感知、推理、规划属决策能力，保存、检索、利用属记忆管理能力；系统只在对应的进化面上提出修改，且只有修复了当前任务失败的修改才被提交 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:62]。这构成论文强调的「有诊断、有验证的进化闭环，而不是无约束的自我重写」。记忆侧「写入前先归因」的治理思路与 [[concepts/agent-memory-architecture]] 一致；提交前的强制验证则与 [[concepts/eval-optimizer-firewall]] 的防火墙模式同构。与腾讯飞轮路线（见上文关联）相比，RoboFoundry 的归因与修改对象从单个组件扩展到了整套系统。

### 回放验证与晋升门：通用能力如何被守门沉淀

候选规则不是直接生效的。系统先把成功的修改抽象为候选规则，再检索其他任务中同类能力缺口的修复记录，回放这些任务修复前的状态，让应用候选规则的系统重新给出方案，并由系统自身充当验证者，把新方案与当时实际奏效的修复比较打分；只有平均得分不逊于原有修复，候选规则才写入通用系统 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:64]。图 3 的示例展示了晋升门的实际约束力：前三次修改把 held-in 成功率从 21.4% 推到 50.0%，两个只达 42.9% 的候选被拒，最后一个把成功率推到 78.0% 的候选通过 held-out 检查后才标记晋升 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:66-72]。

落成文件的规则也经过刻意抽象：针对清洁剂指令改写出的新规则不含清洁剂、水槽、沙发等任何具体物体，因而可跨任务共享；整个进化过程记录在 .robofoundry/ 目录的「进化账本」里，可回溯查看 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:79-92]。改进还按两种时间尺度分流：可复用的组合技能长期沉淀进共享系统，试验中临时生成的恢复逻辑只留在当次试验 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:125]。这一「回放—评分—晋升」守门与 [[concepts/verifier-driven-development]] 的验证优先原则一致，也区别于 [[entities/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026]] 的探索式技能习得——后者强调探索产出技能，前者强调晋升门槛约束沉淀。

### 语义接口与执行解耦：跨本体复用的经济学

复用来自一层共享语义接口：语义层只处理跨本体共享的任务逻辑，执行层把决策适配到具体机器人，后端可以是冻结的 VLA（如 π0.5）、编码智能体生成的函数或 CuRobo 等视觉运动接口；由于进化作用在语义一侧，一台机器人上进化的系统换一套执行适配即可在另一台机器人上复用，也能接到不同的基座模型 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:54]。

实验数据支持这套解耦的价值：五个基座模型在 EmbodiedBench 上全部提升，基座间最大差距从 19.6 个百分点缩到约 11.5 个百分点 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:102-106]；RoboMemArena 上 TSR 比所有基线至少高出 39.0% ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:117]；真机上套娃任务经在线进化后成功率达 70%，单独使用 π0.5 只有 30%，叠毛巾技能仅在蓝毛巾上进化即零样本迁移到外观、几何、材质都不同的绿粉毛巾，成功率超 80% 对不足 20% ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:131-135]。系统层增益可以部分弥合基座能力差距——强弱模型接入同一进化系统后表现趋同，这与 [[concepts/embodied-intelligence-frontier]] 所述具身竞争焦点正从模型转向系统的判断吻合；与浙大 HugAgentOS 的「Agent OS」定位（见上文关联）相比，RoboFoundry 多出的正是这层可移植的语义接口。

## 实践启示

1. **冻结基座、进化系统是比微调便宜的增益路径**：不改一个参数即从 56.9% 到 72.7% ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:16]，消融进一步显示通用进化比只保留任务级修改的 Lite 版相对高出 11.2% ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:111]——先问「系统能不能进化」，再考虑动模型。
2. **给自进化装上「归因—验证—晋升」三道门**：先归因到六种能力再动对应进化面 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:62]，只有修复当前失败的修改才提交、跨任务回放验证后才晋升 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:64]，自进化系统最贵的失败是无约束自我重写。
3. **经验规则的颗粒度决定复用半径**：写进系统的规则不含任何具体物体才能跨任务共享 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:79-92]——抽象化不是文风偏好，而是复用的前提条件。
4. **语义接口与执行后端解耦是跨本体复用的前提**：进化作用在语义一侧，换执行适配即可迁移 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:54]，设计多本体或多后端系统时应把「任务逻辑」与「本体绑定」分层。
5. **系统级自进化必须可审计**：进化账本让每一步修改可回溯 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:92]，缺了账本的自我修改系统在生产环境中不可运维、不可回滚。
6. **两种时间尺度分开管理**：可复用的组合技能长期沉淀，试验内临时恢复逻辑不外泄 ^[raw/articles/robofoundry-self-evolving-system-as-policy-2026.md:125]，防止任务特定逻辑污染通用系统。


## 关联
- → [[raw/articles/robofoundry-self-evolving-system-as-policy-2026|原文存档]]
