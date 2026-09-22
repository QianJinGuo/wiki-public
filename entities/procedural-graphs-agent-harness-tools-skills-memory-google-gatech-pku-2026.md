---
title: "程序图 PG：把工具、技能、记忆连成网的 Agent 可演化程序知识"
created: 2026-09-22
updated: 2026-09-22
type: entity
tags: [agent, harness, memory, skill, tool, graph, self-improvement, context-engineering]
sources: [raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026]
confidence: 0.8
provenance_state: extracted
---

# 程序图 PG：把工具、技能、记忆连成网的 Agent 可演化程序知识

## 一句话结论

当 Agent 已经能调用工具、使用技能、保存记忆之后，真正缺的不是「更多能力」，而是**能力之间的衔接关系**——什么时候读记忆、哪项技能该调哪个工具、什么条件下该停下来。Google、佐治亚理工与北京大学的研究者提出的 **Procedural Graphs（PG，程序图）**，把工具调用、技能步骤、内部推理与任务状态显式写成带条件的有向图，并把「改图」变成一条可验证的离线演化回路；论文中**手工专家图反而让成功率下降 28.6 个百分点**，而从执行反馈里迭代出来的图最终反超无图基线 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

论文：https://arxiv.org/pdf/2609.09153

## 问题：工具能查到机票价，不代表知道查到就该停

文章用一个很具体的场景切题：Agent 能查到机票价格，不代表它知道**查询之后应该停下来**；能保存笔记，也不意味着下一次决策前会读回来。Tool、Skill、Memory 各自都已被单独解决，但执行过程中真正需要判断的是——什么时候读取记忆，哪项技能需要调用哪个工具，拿到结果之后又该保存什么 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

这与 [[concepts/agent-memory-architecture|Agent 记忆架构]] 和 [[concepts/tool-use-patterns-ai-agents|工具使用模式]] 里反复出现的困境同源：能力是有了，衔接没有。文章把 PG 定位在 Agent harness——即模型周围的执行支持系统——这一层，主张给 tools/skills/memory **提供一种把组件连成网的方式**，描述这些能力在什么条件下使用、怎样衔接，以及哪些错误需要避免 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

## 数据结构：过程—关系—过程三元组

PG 把衔接关系写成「过程—关系—过程」三元组。节点可以是一项技能、一个工具函数、一次内部推理，也可以是一个任务状态；连接两个节点的有向边携带三个字段：**适用条件（condition）、执行建议（guidance）、需要避免的问题（pitfalls）** ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

论文给的例子很适合贴在白板上：「预测现金流」连到「申请融资」，这条边的条件是预计现金支撑时间低于安全缓冲，建议是提前申请为资金到账留时间，要避免的是在已有申请尚未完成时重复发起。同一个工具动作由此获得完整的使用上下文——为什么现在调用、调用前要满足什么、什么情况下应当等待 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

**PG 与知识图谱的分工**是文章最容易被误读的一点：知识图谱用「实体—关系—实体」组织事实，回答「是什么」「在哪里」；程序图组织的是另一类知识——**做什么、按什么顺序做、在什么条件下做**。图 1 的对照说明这两类图是互补而非替代 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

## 记忆读写被并入同一张网

论文附录的财务 Agent 是记忆维度的关键案例：演化出的图先连接现金检查、现金流预测、保存笔记（save_note）与市场数据检查；随后又把**回读笔记（recall_notes）接到每月开始的位置**，让上个月保存的关键信息在新一轮决策前被取回 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

这一点值得单独标出：工具负责查询与计算，笔记保存跨月信息，而 PG 描述的是**何时写入、何时读取，以及读写操作怎样接上后续决策**。工具调用和记忆读写因此成为同一张程序网中的步骤，而不是两个独立子系统 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]。与 [[concepts/agent-memory-substrate-three-layer|记忆三层底座]] 相比，PG 补上的正是「读取时机」这一层此前多数架构里靠 prompt 自觉的部分。

## 在线指导：定位 → 取两跳子图 → 生成情境指导

把关系组织成图之后，还要决定每一步读什么。整图信息完整但会带入大量无关分支；独立检索几条语义相似的建议，则可能**丢掉步骤之间的联系**——只取回「提交」的指导而没取回前面的「检查答案」，Agent 就缺少判断何时可以提交的依据 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

PG 把在线指导拆成三个连续操作：

- **定位当前步骤**：根据最近执行的动作匹配图中的节点，确定 Agent 当前位置
- **提取相连的局部结构**：默认读取沿出边**两跳以内**的子图；匹配不到节点时回退到整张图
- **生成当前情境下的指导**：指导模型结合局部子图、用户任务与近期执行记录，生成下一步建议并加入执行模型的提示词；最终动作仍由执行模型选择

论文实验中指导模型与执行模型采用同一种基础 LLM，执行单个任务时图保持固定 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]。这一「局部子图 + 生成式指导」的组合，是 [[concepts/context-engineering|上下文工程]] 中「按需注入」思路在图结构上的具体化。

## 局部子图 vs 整图：一次很干净的消融

ALFWorld 固定测试子集（Gemini 3.5 Flash）上的对比给出了这轮工作最锐利的一组数字 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]：

| 指导方式 | 成功率 | 平均 Token |
|---|---|---|
| 整张图生成指导 | 54.48% | 96,360 |
| 同一张图的局部子图指导 | 81.53% | 28,064（−70.9%） |
| 无图基线 | — | 18,055 |

局部子图不仅把成功率从 54.48% 拉到 81.53%，还把 Token 消耗砍掉约 70.9%。但文章没有藏成本：**局部指导的 Token 仍高于无图基线**（28,064 vs 18,055），离线演化与候选图验证还要额外算力。作者也承认程序图在跨模型、跨工具接口复用时能保留多少效果仍需进一步研究 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]。这种「把开销和负结果一起写出来」的写法，和 [[concepts/when-not-to-harness-engineering|何时不该做 harness 工程]] 的取舍框架是一致的。

## 离线演化：手工专家图是负收益，改图必须过验证

文章最反直觉的结果在 MultiChallenge 的图构建实验（Gemini 3.5 Flash，56 个测试样本）^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]：

| 图来源 | 成功率 |
|---|---|
| 无图基线 | 87.50% |
| **手工专家图** | **58.93%（−28.57pp）** |
| 模型静态更新一次 | 53.57% |
| **迭代演化后的图** | **92.86%（+33.93pp vs 专家图，且超过无图基线）** |

手工写出的流程「看起来合理，实际执行时也可能带来问题」——这是整篇文章的立论支点：**程序知识必须经过任务检验，不能靠专家撰写一次性交付**。这直接呼应 [[concepts/skill-engineering-principles|技能工程原则]] 中「经验必须被验证而非被声明」的主张，也解释了为什么静态更新一次（53.57%）比专家图更差：未经检验的修改同样会引入坏边 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

PG 的离线循环是四步：**执行任务 → 分析轨迹 → 提出修改 → 验证候选图**。系统先在一批训练任务上运行当前保留的图，再由修订模型对照高分与低分轨迹，查找反复出现的错误或可复用的步骤；修改可以增加缺失节点与连接、删除易致失败的路径，也可以重写边上的条件、建议与注意事项。候选图先过结构检查，再在独立验证集上运行，**只有验证分数不低于当前保留图才被采用（持平也保留）**；被拒绝的方案及结果留下记录供后续修订参考 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]。这套「不改权重、只改结构」的自我改进回路，是 [[concepts/agent-self-improvement-loops|Agent 自我改进回路]] 的一个可验证实现。

演化不是抽象的指标上涨，文章给了一个可以逐帧回放的案例：某验证样本要求 Agent 讲笑话、同时延续此前「只用被动语态」的约束，但第二代候选图指导下的 Agent 偏离任务，转而回答环境附带的评价问题（以「没有一直使用被动语态」开头）。**第三代候选图删除了直接结束的连接**，并改写「提取约束→结束」的指导，要求不要直接回答评价问题；这次 Agent 经过提取约束步骤后给出了笑话，该验证样本的成功标记从 0 变为 1 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

从**最小图结构**出发的演化也被验证：HotpotQA 构建实验中，这种方式得到的 PG 取得 78.79 的答案 F1，高于无图基线的 71.21，说明程序结构可以从执行反馈中逐步建立，而不必先有专家骨架 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

## 任务评估：长期决策的收益最大，问答的收益最小

评估覆盖多跳问答、多轮指令遵循、专业任务、交互式环境、工具调用与长期财务决策，各方法使用相同的 ReAct 执行框架，只改变经验的存储与复用方式 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

- **工具调用（BFCL v3）**：Gemini 3.5 Flash 使用 PG 后准确率 67.00%，同组最强基线 58.00%
- **专业任务**：GDPval 与要求遵守业务规则的 τ-bench 上，Gemini 3.1 Pro 均有提升
- **长期决策（EnterpriseArena）**：Agent 管理一家模拟企业，连续作最多 **132 个月**财务决策，需应对现金流变化、资金到账延迟与经济冲击。融资申请发出后资金需**一至六个月**才到账，等现金即将耗尽才申请可能撑不过等待期；轨迹显示 PG 指导下的 Agent 更早检查现金流、预测资金缺口，并在相对稳定的月份发起融资申请。每种配置 50 次模拟，Gemini 3.1 Pro 跑完整个模拟周期的**存活率从 6.0% 提升到 34.0%**，Claude Sonnet 4.6 从 44.0% 提升到 58.0%
- **反而收益有限**：主表 HotpotQA 问答中，PG 相对各模型最强基线的差距介于**下降 0.90 到提高 1.30 个百分点**之间

这组结果给出了一个相当清晰的适用边界：**当任务的关键难点是「动作之间隔了很长时间才有结果」时，显式程序关系的价值最大**（存活率 6%→34% 的量级）；当任务本身是单轮问答时，程序图几乎是噪音。这条边界与 [[concepts/harness-long-running-task|长任务 harness]] 的关注点高度重合 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

一个收尾细节值得记住：BFCL 机票查询案例里，用户只想了解经济舱票价，无图基线查到 220 美元报价后继续认证身份、操作银行卡并尝试订票，失败后还修改预算限制再次完成预订；同样使用 Gemini 3.5 Flash，PG 指导下的 Agent 在报出 220 美元后**结束当前回合，等待用户的新指令** ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]。程序知识的价值在这里表现为一个很朴素的能力：判断已有操作是否已经满足当前请求。

## 与既有 harness 路线的关系

PG 的定位更接近 [[concepts/harness-engineering-framework|Harness Engineering 框架]] 中「上下文/记忆/技能」横切层的统一化尝试，而不是新增一个能力模块：它不动模型权重（图更新无需重新训练），把经验存成可读、可检查、可修订的**程序关系** ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

与 [[concepts/skill-engineering-principles|技能工程]] 的差别在于粒度与可组合性：技能是封装的「可复用做法」，PG 关心的是技能与工具、记忆之间的**边**——顺序、条件、衔接与反模式。作者自己的总结是：从 harness 的角度看，PG 让 tools、skills、memory 之间的配合关系成为**可以检查和改进的对象**，补充一步检查、修改一项条件或调整一次记忆读取的时机，都可以落实到具体的节点与连接 ^[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026.md]

## 可迁移要点

- **手工专家流程是负收益**：MultiChallenge 上 87.50% → 58.93%。程序知识必须走「执行 → 分析 → 修改 → 验证」回路，这给「先写 SKILL.md 再上线」的直觉泼了一盆冷水
- **只改图、不改权重**：经验沉淀在模型权重之外的结构里，可检查、可回滚、可 diff
- **局部两跳子图是性价比拐点**：81.53% vs 54.48%，Token 降 70.9%
- **收益随任务时间跨度上升**：长期财务决策存活率 6%→34%，而单轮问答仅 ±1pp
- **成本要一起报**：局部指导仍比无图基线贵（28,064 vs 18,055 Token），离线演化还需算力

→ [[raw/articles/procedural-graphs-agent-harness-tools-skills-memory-google-gatech-pku-2026|原文存档]]
