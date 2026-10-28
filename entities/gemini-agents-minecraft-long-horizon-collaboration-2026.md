---
title: "Gemini 4 Argon 四智能体 27 小时通关《我的世界》：长程协作的失败经济学"
created: 2026-10-26
updated: 2026-10-26
type: entity
tags: [agent, multi-agent, long-horizon, evaluation, benchmark, gemini, deepmind]
sources: [raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026]
confidence: 0.75
provenance_state: extracted
---

# Gemini 4 Argon 四智能体 27 小时通关《我的世界》：长程协作的失败经济学

Google DeepMind 让四个 Gemini 4 Argon Agent 在一张随机《我的世界》地图上，从赤手撸树打到击杀末影龙：**27 小时、229 条互相消息、写下 2700 多行 JavaScript、集体阵亡 605 次**。研究人员没有分配任务、没有给攻略，只给了一句"击败末影龙"。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 接口选择：用代码玩，而不是用鼠标玩

Agent 不看屏幕也不操作键鼠，而是通过开源库 **Mineflayer** 读取角色位置、血量、背包与周围环境，再自己编写 JavaScript 控制角色赶路、挖矿、合成、战斗；一段代码跑完读结果、查报错、再写下段。这个接口选择直接决定了任务性质——**它考的是"写程序 + 读反馈"的闭环，而不是视觉-动作映射的灵巧度**。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 自组织分工：10 分钟内的角色分化

开局 10 分钟，四个 Agent 主动商量出分工：gyldenia 负责食物、床与打火石；bulicame 下矿挖钻石；janesick 炼铁与制工具；auratian 搭建下界传送门。不到两小时凑齐铁器与钻石并点亮传送门。**没有任何中心调度器**，分工从对话中涌现，且角色在后续 27 小时里基本保持稳定。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 共享记忆与事故复盘：把"坑"写成团队资产

进入下界后混乱开始：janesick 带着 12 份烈焰粉被烧死、掉落物被恶魂火球炸光，几只 Agent 立刻聚在一起复盘原因、写下解决办法，并提醒所有队友别在同一处反复踩坑。5 个半小时后公共箱子里已攒到 15 颗末影珍珠。这类**"踩坑 → 复盘 → 写入共享记忆 → 广播"**是长程多智能体系统里最稀缺的行为，因为它要求 agent 把失败当作可转移的知识而非个人事故。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

协作的极限案例是抢救：auratian 带着全部 12 颗末影之眼溺水身亡，两只 Agent 立刻向 bulicame 报出死亡坐标与物品消失时间，bulicame 赶到现场在**只剩两分半钟**内把 12 颗眼睛全部捞回。定位也用了量化手段：janesick 在两处投掷末影之眼、每 50ms 记录一次飞行轨迹并求交点，把要塞锁在距村庄约 1.3 公里处，与真实传送门相差 43 米。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 长程瓶颈出现在"未观测变量"，不在执行能力

第 13 小时激活末地传送门后，剩下近 14 小时全部卡在末影龙面前——巨龙先后杀死它们 69 次，血条压不下来。直到第 21 小时，auratian 才找出原因：**塔顶 10 颗末地水晶一直在给巨龙回血**。没有合适工具，它用最朴素的办法解决——往脚下垫土爬上柱顶，用手逐个打掉水晶；4 个多小时后水晶清空，最后一颗消失 39 分钟后末影龙倒下，完成最后一击的 bulicame 只剩 2.5 点生命值。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

这段过程的诊断价值大于结局：前 13 小时证明它们会采集、制造、探索、战斗、资源管理与协作；后 14 小时暴露的是**长程任务里最贵的失败模式——对系统隐含机制的不知情**。8 小时重复冲击同一个 Boss 而没有先做机制归因，说明"自我纠错"在缺乏外部反馈结构时退化为"更努力地重复"。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 成本结构：520 美元买来的长程样本

据网友估算，这次通关约花 **520 美元**：累计处理约 **13.5 亿输入 token、生成 1600 万输出 token**，调用模型约 **2.4 万次**，其中约 **91% 的输入由缓存提供**。这组数字比胜负更有工程意义——长程任务的成本几乎全部由"重放上下文"构成，缓存命中率而非模型单价决定账单；同时它给出了一个可对标的单位：**一次 27 小时端到端任务的边际成本量级在百美元级**。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 评测意义

《我的世界》在这里不是游戏，而是一个**依赖链极长的任务图**：采集、制造、探索、战斗、资源管理、协作，前一步没做完后一步接不上。用它做评测补上了静态 benchmark 的两个缺口：跨小时级的状态延续，以及多 Agent 之间的信息不对称与救援。判断这类结果时应同时看三个轴——完成度（是否通关）、代价（token/美元/墙钟）、以及过程质量（分工是否稳定、复盘是否产生可复用记忆）。^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md]

## 深度分析

### 代码即动作：接口选择先改写了任务的性质

Mineflayer 提供的是可编程的状态读取与动作 API，而不是屏幕像素与键鼠事件^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:50-56]。这一步把"感知-控制"问题换成了"程序综合与调试"问题，代价与收益都很明确。收益有三：动作空间可组合，一段挖阶梯的代码能被队友直接复用；失败信号是返回值或异常，归因而非猜测；产物是可审计的代码，2700 多行 JavaScript 在任务结束后仍然存在。代价是把 GUI Agent 最难的部分（精确操作、时延）换成了代码 Agent 最难的部分（状态建模、幂等、部分失败）。因此这次通关不能被读作"Agent 会玩《我的世界》"，只能被读作"Agent 能维持一个二十七小时的写码-观测闭环"。参见 [[entities/code-as-agent-harness-survey]]、[[concepts/tool-use-patterns-ai-agents]]。

### 无人调度：分工从"可见的能力"里长出来

开局十分钟内四个 Agent 自行认领食物、挖矿、炼铁、建门^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:58-70]。没有中心调度器却能分化，需要三个前提同时成立：队友状态可读（同一个库暴露位置、血量、背包），广播足够便宜（全程 229 条消息），以及任务图存在串行瓶颈——食物与工具不解决，挖矿与建门就接不上，于是"解锁他人"的角色天然被优先认领。角色能稳定二十七小时，则更像承诺机制而非规划结果：一旦在某条支线上积累了工具与配方知识，切换角色的重建成本高于协作收益。这提示自组织不是免费的，它要求环境把同伴状态暴露出来，并要求任务图有可被抢占的关卡。参见 [[concepts/agent-role-specialization]]、[[concepts/multi-agent-collaboration-patterns]]。

### 事故记忆：把个人事故改写成团队资产

十二份烈焰粉被烧毁、掉落物被火球清空之后，几只 Agent 聚在一起复盘、写下解决办法并广播^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:72-80]。这是六百零五次阵亡没有拖垮队伍的真正原因：单次死亡很便宜（复活即可），重复同一种死亡才昂贵。它们的循环——踩坑、归因、写入共享记忆、广播——本质是把情节记忆压缩成语义规则，让一次代价换来全队的先验。但这套记忆没有作用域标签，下界学到的规则在末地可能失效，缺失的正是"这条经验在什么条件下成立"的元数据。抢救事件则展示了另一类资产：不是知识而是注意力路由——垂死 Agent 报出坐标与掉落物消失倒计时，把一次损失转成一个两分半钟的硬截止任务^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:88-96]。参见 [[concepts/agent-memory-architecture]]、[[concepts/episodic-vs-semantic-memory-agent]]。

### 长程瓶颈是未观测变量，不是执行带宽

第十三小时激活传送门，之后近十四小时卡在末影龙面前，被击杀六十九次，直到第二十一小时才由 auratian 找到塔顶十颗水晶在给巨龙回血^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:98-120]。前十三小时证明采集、制造、探索、战斗、协作都在能力范围内；后八小时的重复冲击则证明，当关键项落在观测模型之外时，"更努力"是唯一可得却完全错误的梯度。真正的修复只花了四个多小时，且手段极其朴素——垫土上柱、徒手拆水晶——说明成本全部消耗在寻找变量，而不是执行变量。工程含义是：长程系统需要独立的停滞检测器，在指标平台上把重试预算切换成归因预算，并提供一个显式的"给环境加仪表"动作（读日志、做探针、跑反事实），而不是继续加步长。文章引用的那句总结说得很准：目标函数降不下去时，先找是哪一项一直在把你往回拉^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:156-158]。参见 [[concepts/harness-long-running-task]]、[[entities/towards-long-horizon-agents-survey-mozi-space]]。

### 账单解剖：重放上下文与评测口径

五百二十美元、十三点五亿输入 token、一千六百万输出 token、约两万四千次调用、九成以上输入命中缓存^[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026.md:126-128]。把比例算开：输出只占输入约 1.19%，平均每次调用吃进约 5.6 万输入 token、吐出约 667 token，平均每 4 秒一次调用，单次调用约 0.02 美元。账单主体因此不是生成，而是重放历史；能压成本的手段是缓存命中率、上下文裁剪与"少次长调用"，而不是更便宜的输出。换算成单位也有用：二十七小时端到端任务的边际成本在百美元量级，任何以天计的多智能体系统都该先按这个量级做预算。对 benchmark 的推论是双重的：其一，必须报告成本归一化的成功率（每美元完成度、每里程碑花费），否则靠六百次死亡硬撞出来的成功与一次规划成功的系统得分相同；其二，这是单次运行、无方差、无消融的存在性证明，不是测量——它能说明"可行"，不能说明"可复现"。参见 [[concepts/context-window-economics]]、[[concepts/ai-cost-optimization-framework]]、[[concepts/evaluation-harness-design]]。

## 实践启示

1. **接口先于能力。** 优先给 Agent 可编程、可重放、可审计的行动接口，而不是像素与鼠标；它把调试成本从"猜画面"降到"读报错"，也让行动可以沉淀成队友可复用的代码。
2. **先暴露状态，再考虑调度器。** 自组织分工依赖同伴状态可读与广播便宜；把共享世界状态和消息通道做扎实，比提前写死一个中心规划器更能应对长尾。
3. **把停滞当作一等信号。** 指标连续多个周期不动时，强制转入归因模式：列出所有可能持续抵消进展的隐藏项，设计探针逐个排除，而不是提高重试次数或步长。
4. **复盘必须落成有作用域的写入。** 每次事故至少留下"触发条件、错误动作、替代动作、适用范围"四元组；没有作用域的经验会在环境切换后变成新的坑。
5. **同时记三个账：完成度、代价、过程质量。** 只报成功率会奖励蛮力；补上美元与时延，再补上角色稳定性与复盘复用率，才能区分"学会了"与"撞上了"。
6. **把英雄故事降级为假设。** 单次长程通关只提供存在性证明；要变成工程结论，需要多随机种子、消融（去掉共享记忆、去掉消息通道）与失败样本的对照。

## 关联

- [[concepts/agent-evaluation-benchmark-frameworks]] — 长程/端到端任务相对静态 benchmark 的评测定位
- [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop]] — 完成度、成本、过程质量三轴的度量框架
- [[entities/mineexplorer-开放世界多模态评测基准-longcat-2026]] — 同为《我的世界》系开放世界评测，但侧重多模态感知而非长程协作
- [[entities/code-as-agent-harness-survey]] — "写代码作为 agent 行动接口"的同类范式

→ [[raw/articles/gemini-agents-minecraft-long-horizon-collaboration-2026|原文存档]]
