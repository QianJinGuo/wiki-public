---

title: "Google DeepMind Co-Scientist 升级：AI 首次在真实实验室制造半导体"
created: 2026-09-01
updated: 2026-11-23
type: entity
tags: [deepmind, co-scientist, ai-for-science, multi-agent, semiconductor, materials-science, gemini]
sources: [raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Google DeepMind Co-Scientist 升级：AI 首次在真实实验室制造半导体

## 概述

Google DeepMind 与杜克大学联合发布了 AI 科研系统 Co-Scientist 的重大升级版本。Co-Scientist 是 Google 基于 Gemini 构建的多智能体科研系统，此前版本主要聚焦于计算机内的假设生成。此次升级的核心突破在于：**Co-Scientist 首次成为一个以真实实验执行为基础的闭环科研伙伴**。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

## 核心突破

### 从"想"到"做"的范式转变

当前 AI 科研面临的最大瓶颈：「AI 能在计算中提出什么」与「能在物理世界中验证什么」之间的鸿沟。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

### 材料科学：半导体制造

**1. 无毒新路线合成 MXene**
- Co-Scientist 针对杜克实验室 CVD 系统的具体几何结构和现有原料
- 从 272 个候选方案中提出以六氯乙烷替代剧毒四氯化钛作为前驱体
- 经过 25 轮迭代优化，成功生长出高结晶度的二维层状晶体 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

**2. 一次成功合成三种单层半导体**
- 仅根据设备硬件约束，在没有任何历史配方参考的情况下
- **第一次尝试中即成功合成高质量单层 MoS₂**
- 推广到实验室此前从未合成过的 MoSe₂ 和 WS₂，同样一次成功
- 借助 Gemini 3 Deep Think，**数分钟内**生成配方并转化为设备控制指令 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md]

### 生物学与计算机科学验证

- **生物学**：从稀疏成像数据中预测工程大肠杆菌群集形态，四项形态指标中三项达到定量吻合
- **计算机科学**：自主设计推理时扩展架构 Agent_H，在 HealthBench Hard 与 Professional 基准上超越六个前沿模型

### 可靠性机制

- 将幻觉和抄袭惩罚直接纳入优化目标
- 30 位领域专家、450 次独立评审的双盲研究证实机制有效
- 内置安全系统拒绝了 98.7% 的有害研究请求

## 深度分析

### 闭环之后：当假设生成直接驱动物理执行

此前版本的 Co-Scientist 与大多数 AI for Science 系统一样，工作在"提出猜想、人类验证"的模式下：AI 的产出止步于计算内的假设，物理世界的验证是外部环节。这次升级把实验执行纳入了智能体自身的行动回路，改变的不仅是效率，更是错误的性质。论文指出，纯计算型系统容易出现"奖励作弊"——编造看似合理却并不存在的实验结果——因为评价它的信号本身就在计算内闭环；而当结论必须经受高温反应炉的检验时，造假的空间被物理现实压缩了。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:22-28]

这就是"计算验证"与"实验台验证"之间的鸿沟（compute-vs-bench gap）：前者的评价函数可以被骗，后者不能。把真实设备接入智能体回路，本质上是为科研智能体引入了一个不可收买的奖励信号。这一转变让 Co-Scientist 从[[concepts/multi-agent-systems]]式的"头脑风暴协作体"走向[[concepts/embodied-intelligence-frontier]]意义上的具身行动者，也印证了一个更一般的判断：AI 科研的瓶颈正在从"能不能想出来"转移到"能不能在物理世界里做出来"（参见[[concepts/ai-r-and-d-bottleneck-shift]]）。

### 25 轮迭代：一次自主实验的循环解剖

MXene 的合成路径揭示了自主实验的真实成本结构。直接"自下而上"CVD 合成 Ti₃C₂Tₓ 是长期未解的难题，Co-Scientist 从 272 个候选方案中选定六氯乙烷替代剧毒的四氯化钛作为前驱体，但这只是起点——真正的工作发生在随后的 25 轮迭代优化中，每轮都以 X 射线衍射、扫描电镜等表征结果作为下一轮调整的依据。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:44-50]

值得对照的是同一系统在 TMD 任务上的表现：MoS₂、MoSe₂、WS₂ 三种单层半导体全部一次成功，全程约一小时。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:54-60] 两个任务的差异恰恰说明了迭代轮数不是智能体的固有属性，而是问题欠定程度的函数——TMD 的失败模式（设备敏感性导致的配方不可迁移）可以通过精确描述硬件约束来消除，而 MXene 面对的是真正未知的化学空间，只能靠"提出—执行—表征—修正"的循环逐步收敛。这为[[concepts/autonomous-agent-systems]]的实验循环设计提供了一个量化参照：已知映射关系靠约束传递一次命中，未知映射关系靠闭环迭代收敛。

### 98.7% 的拒绝率：把安全当作治理层而非补丁

可靠性设计是此次升级中最容易被低估的部分。团队没有把"AI 会编造结果"当作需要事后清理的副作用，而是把幻觉和抄袭惩罚直接写进优化目标，并把论文中的每一项定量结论与原始实验运行日志逐条比对——可追溯、可验证、可审计被当作系统属性而非评审流程。30 位专家、450 次独立评审的双盲研究证实了这些机制的有效性。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:78-80]

安全系统拒绝 98.7% 有害研究请求这一数字，标志着治理思路的转变：安全不再是部署前的静态对齐检查，而是嵌入科研回路中的实时守门机制——每个研究方向在进入实验执行之前先经过一次可测量的筛选。当智能体获得了操控真实实验设备的能力，这种"行动前的门控"就从合规要求变成了物理安全的必要条件（参见[[concepts/responsible-ai-governance]]）。

### 对 AI for Science 的含义：材料发现走出纯仿真

材料科学领域此前多为 in-silico 研究——AI 预测结构、筛选候选，制备仍由人类工艺工程师完成。此次工作展示的是另一条路线：AI 产出的不是候选列表，而是可直接在特定设备上运行的生长配方，从设计到表征验证全链路闭环。这意味着材料发现的核心资产从"预测模型"扩展为"设备约束下的工艺生成能力"，AI 首次同时占据"设计者"与"制造者"两个位置。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:84-90]

生物学（大肠杆菌群集形态预测）与计算机科学（自主设计 Agent_H 架构并通过盲评降低临床风险）两个领域的同步验证，说明闭环范式并非材料领域特有。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:72-74]  当然，边界同样清晰：CVD 系统只是半自动化，新型碳化物的原子级相结构仍待截面原子分辨电镜确认，"一次成功"距离"稳定量产"尚远。人类科学家的角色由此重新定位——设定目标、审核结果，把配方探索的海量迭代交给系统。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:88-90]

## 实践启示

1. **把硬件约束写全，是最便宜的提示工程。** Co-Scientist 一次长出三种单层半导体的前提，是拿到炉体结构、可用化学品、衬底类型等完整设备约束。实验室自动化智能体的首要设计任务不是更聪明的推理，而是把物理系统的状态空间完整地暴露给模型。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:56-60]
2. **区分"一次命中型"与"迭代收敛型"任务，分别设计循环预算。** 有明确约束映射的任务（TMD 生长）应追求一次成功以节省机时；化学空间未知的任务（MXene 合成）应预设多轮迭代，并把表征仪器（XRD、SEM）的输出格式化为智能体可读的反馈信号。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:46-50]
3. **配方到机器指令的转化要自动化，不留人工翻译层。** 数分钟生成配方、直接转译为 CVD 设备控制指令，整个流程约一小时——人工转录参数恰恰是传统工艺开发中最慢、最易出错的环节。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:60]
4. **安全门控必须可测量，并公开测量结果。** "拒绝 98.7% 有害请求"之所以有意义，是因为它建立在可复现的评测之上。为科研智能体设计安全层时，应构造有害请求测试集并报告拒绝率，而非仅声明"有安全机制"。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:78-80]
5. **结论与日志逐条对账，是幻觉治理的落地形态。** 将论文中每项定量结论与原始实验运行日志比对、把幻觉与抄袭惩罚纳入优化目标，比事后人工审查更可扩展；这套机制可移植到任何产出可核查声明的智能体系统。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:78]
6. **人是目标设定者与审核者，不是配方调试员。** 重新划分人机分工时，应把海量迭代交给系统，把目标定义、安全边界设定与最终结果审核留给人类——这是闭环科研系统里人类介入点设计的基准形态。 ^[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08.md:90]

→ [[raw/articles/google-deepmind-co-scientist-upgrade-physical-lab-semiconductor-2026-08|原文存档]]