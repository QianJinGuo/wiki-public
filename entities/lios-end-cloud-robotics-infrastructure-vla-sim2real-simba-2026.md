---
title: "LiOS 端云协同基础设施：具身智能柔性操作与虚实迁移（招商局狮子山人工智能实验室）"
created: 2026-08-20
updated: 2026-11-26
type: entity
tags: [embodied-ai, vla, sim2real, robotics, flexible-manipulation, end-cloud-infrastructure]
sources: [raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026]
confidence: 0.7
related: [concepts/embodied-intelligence-frontier, concepts/robotics-embodied-ai, entities/abot-agentos-robot-agent-os-amap-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
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

## 深度分析

### 虚实迁移是系统工程问题，不是单点算法问题

原文把「仿真优秀、真机拉胯」的根因拆成硬件刚性差异、机械装配误差、夹爪稳定性、底层控制精度偏差四类现实因素——这意味着仿真里再流畅的策略，落地误差也无法靠单纯加大训练量消除。LiOS 的对策不是「把仿真做得更像」，而是把迁移当成一条流水线来修：训练侧用分布式并行训练与高性能算子优化把吞吐拉高 5 倍以上；评测侧对仿真环境做并行化重构，效率提升 4 倍以上；再以 Real2Sim 遥操作把真机轨迹批量转化为仿真可用的训练资产。这与「Real2Sim 量大管饱、sim2real 烧不起」的路线判断互相印证——[[entities/李飞飞署名具身新论文sim2real烧不起real2sim量大管饱|Real2Sim 数据规模化]]的价值正在于让仿真数据便宜到可海量生成，再用少量真机数据校正残差。决定能否跨越鸿沟的是仿真与真机两条数据腿的配比和流速，而非某一项算法突破，[[entities/embodied-intelligence-sim-to-real-active-inference-behavior-tree-intrinsic-motivation-chenzhiyan-2026-06-17|sim2real 系统方法]]视角同样支持这一判断。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md:34-46]

### 云侧与端侧的分工：上限交给云端，底线留在端侧

LiOS 三层架构的分工逻辑清晰：云侧决定模型能力上限（分布式训练、数据湖仓、高并发仿真评估、支撑百亿参数扩展），端侧守住执行安全底线（传感器同步、运动控制、安全执行、接管恢复、本地状态管理），端云协同层把两者焊接起来。信息量最大的工程数据是图传链路：本地相机到云端显存约 30ms 单向延迟（网络部分仅 24ms），较通用中继方案加速 2.1～6.9 倍，单路 GPU 解码吞吐每秒数千帧，最终使云端推理频率高于端侧 5090。这实际上反转了「边缘优先」的行业直觉——不是把小模型塞进端侧，而是把像素以近本地延迟送上云端跑大模型。该取舍成立的前提是延迟工程做到位：图传围绕「画面直接进入云端模型」设计，用 GPU 编码、解码与色彩转换消除 CPU-GPU 往返搬运；而一旦网络抖动或断连，端侧接管恢复与本地状态管理就是安全网。这与[[concepts/local-vs-cloud-agent-deployment-strategy|端云部署权衡]]中「延迟敏感留端侧、规模敏感上云侧」的原则一致，只是把「延迟敏感」的范围从推理扩展到了像素传输。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md:76-94]

### 柔性操作为何是试金石：没有规范状态表示的高维接触问题

衣物折叠难，根源是柔性物体不存在规范化（canonical）状态表示：材质厚薄、褶皱形态、缠绕、摩擦力、弹性形变构成连续且强耦合的状态空间，同一件衣服第二次出现时状态都不同。它同时压迫五大能力（柔性感知、双臂协同、接触力控、长程作业、状态恢复），且难度随品类、初始无序程度、拓扑复杂度指数上升——刚性物体可以靠抓取位姿分类覆盖，柔性物体不行。演示的能力排序也值得注意：大形变整理（拖拽、拉伸、大幅展平）被单列为第三项核心能力，说明系统把「从无序恢复到可折叠状态」当成显式子任务处理，而不是假设输入已规整；多类衣物折叠则按短袖/长袖/长裤拓扑动态调整视觉关注区域、操作时序与折叠路径。这正是理想实验环境与家庭场景的分界线：后者的输入分布长尾才是真机失败的主要来源，长程恢复能力（参考[[entities/icra26-symskill-robot-long-horizon-manipulation|长程操作]]研究）比单步折叠精度更稀缺。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md:96-110]

### 数据闭环是具身智能的基础设施级飞轮

原文总结的两大产业化卡点：机器人技术栈与大模型技术栈交叉带来的系统复杂度（缺统一资源纳管与开发接口，真机系统退化为高度定制化集成、难复用难扩展），以及端侧轻量低功耗约束与模型规模持续需求之间的结构性矛盾。LiOS 的回答是把闭环本身做成 OS：数据采集、训练、真机部署、闭环优化统一纳入基础设施，并显式设计「在实际运行中积累失败数据、反向推动模型与执行链路持续优化」的回路。这把具身智能的竞争力从单一模型指标转移到闭环转速——训练吞吐、评测效率、图传延迟、失败数据回流率都是飞轮的轴承，缺一个飞轮就空转。行业层面，[[entities/jingshuo-tech-embodied-ai-data-infrastructure-2026|具身数据基础设施]]已成为独立赛道，LiOS 代表大厂全栈自研一体化路线；仿真资产侧的[[entities/roboflywheel-embodied-simulation-assets-2026|仿真资产飞轮]]则补上了「闭环里仿真数据从哪来」的一环，两者共同指向同一结论：数据生产管线本身就是产品。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md:50-94]

## 实践启示

1. **做端云协同先修延迟链路，再谈模型能力。** 30ms「相机到云端显存」不是网络快，而是 GPU 编码/解码/色彩转换消除 CPU-GPU 搬运、链路围绕「画面直接进模型」设计的结果；通用中继方案 2.1～6.9 倍的差距说明这层常被低估。立项先测这条链路。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md:88-92]
2. **把失败数据当一等公民。** 闭环里显式包含「积累失败数据 → 反推模型与执行链路」；部署系统时就要设计失败案例的自动捕获、归类与回流通道，否则数据闭环只剩采集没有反哺。
3. **Real2Sim 与真机数据并行投资。** 遥操作轨迹批量转仿真资产，加上训练吞吐 5 倍、评测效率 4 倍的并行化改造，是闭环转速的前提；只优化模型不优化数据生产管线，迭代速度很快见顶。
4. **硬件抽象层决定多本体复用成本。** 统一接入、任务编排与控制执行框架屏蔽自由度、控制接口、运动范围差异，多双臂平台「无需重复搭建链路」即完成同类任务。采购或自研机器人平台时，把抽象层完备度作为一等评估项。^[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026.md:100-102]
5. **端侧底线独立于云端智能。** 实时性、安全边界、接管恢复、本地状态管理必须在端侧自成闭环，不能假设网络与云端永远可用；云端推理频率再高，也替代不了端侧安全网。
6. **按初始状态分布评估工程成熟度，而非演示成功率。** WRC 现场的多双臂并列、多类衣物、大形变整理展示的是能力边界；家庭场景的褶皱堆叠、随机朝向、成团挤压仍是长尾。评估任何柔性操作系统，先问它覆盖的「初始无序状态分布」有多宽。

## 相关

- 具身智能前沿 → [[concepts/embodied-intelligence-frontier]]
- 机器人具身 AI → "机器人与具身 AI"
- 端云机器人 OS 范式对比 → [[entities/abot-agentos-robot-agent-os-amap-2026]]

→ [[raw/articles/lios-end-cloud-robotics-infrastructure-vla-sim2real-simba-2026|原文存档]]
