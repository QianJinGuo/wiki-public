---
title: "MemCurrent: Databricks 生产级 Agent 持久记忆架构"
created: 2026-10-21
updated: 2026-10-23
type: entity
tags: [agent, memory, harness, databricks, evaluation, production]
sources: [raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026]
confidence: 0.8
provenance_state: extracted
---

# MemCurrent: Databricks 生产级 Agent 持久记忆架构

Databricks AI Research 在 2026-10 发布的 MemCurrent，把 agent memory 从"检索增强的花活"拉回到工程权衡的主线上：生产 agent 真正难的不是能不能写出 demo，而是同时守住 **quality（长尾任务答对）** 和 **efficiency（规模化下的成本与延迟）**。作者的前提判断是，多数 memory 技术在两个维度间做零和交换，而记忆之所以值得投入，正因为它是少数能同时改善两侧的手段——但"历史越多越好"是错的，把全部交互塞进上下文既贵又会让 agent 迷失。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 三类记忆接入方式与它们的代价

文章先把"怎么接记忆"拆成三种模式，这个分类本身比具体实现更有迁移价值。**plugin** 模式在会话的固定点自动读写，更新即时、行为可预测，但运行时代价与延迟高，且在不需要记忆时也照样烧 token；**tool** 模式把读写决策交给 agent，灵活但质量受 agent 自身能力约束；**offline pipeline** 模式在后台按计划或触发器更新记忆，延迟到达但运行时成本最低（异步批量）。三者的取舍表实际上是在回答一个设计问题：**你要把记忆的"决策成本"放在运行时还是放在离线批处理里**。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

MemCurrent 的选择是不选边，而是分层：**episodic memory** 用 plugin 方式 append-only 地记录每个 step（原始历史可回溯、可审计），**derived memory** 则是结构化的"沉淀物"（用户偏好、指标定义、纠正过的错误、成功工作流），由两条通道产出——在线 tool 通道处理显式请求（例如"别在 /tmp 下写脚本"），离线 dreaming 通道从每段新会话里蒸馏隐式经验并清理陈旧条目。两类记忆都落在生产级数据库里，支撑多用户与低延迟读取。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 评测：自建 benchmark 与消融

作者没有只跑公开 benchmark，而是指出 Mem0 一类的开源评测多为"模拟人格的开放式闲聊"，与真实工作负载脱节；他们的内部 benchmark 来自真实数据工作区的 12K+ 会话，先挖掘出 3K 条含记忆价值信号的候选，再为每条草拟带评分标准的后续问题。这个构造过程值得记下来：**记忆评测的瓶颈在"构造出必须依赖历史才能答对的问题"，而不在打分模型**。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

结果分两层。在三个开源 benchmark 上，MemCurrent 以约 12 倍更少的输入 token（8.4K vs 103K）达到与 full-context 相当的水平，在历史最长的两个上甚至超过 full context（把全部历史塞进 prompt 反而让 agent 混乱）；内部 benchmark 上则把 pass rate 提升到 3 倍，同时少用 31% token、快 26%，在 pass-rate/token 平面上位于 Pareto 前沿，压过 Mem0、PropMem 与 OpenClaw memory。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

消融给出了最有信息量的数字：无记忆 22.3%，仅 episodic 35.8%，仅在线 tool 通道 53.9%，仅离线 dreaming 63.5%，两者结合最高。也就是说 **derived memory 承担了主要收益，episodic 的价值在于补回蒸馏时被丢掉的细节**；而"离线整合"比"在线即时写"更强，说明记忆质量的瓶颈是 consolidation 的时机与方法，而非读写的即时性。这一结论与 [[concepts/memory-consolidation-decay|记忆整合与衰减]] 的既有判断一致，也直接反驳了"记忆越实时越好"的直觉。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 与既有记忆框架的关系

在 wiki 的记忆聚类里，MemCurrent 的位置是"生产化落地层"：[[concepts/agent-memory-architecture|Agent 记忆架构]] 给出分层蓝图，[[concepts/episodic-vs-semantic-memory-agent|episodic vs semantic]] 区分记忆类型，[[concepts/memory-derivation-observability|记忆派生与可观测]] 讨论蒸馏链路，MemCurrent 则用一组可对照的数字把这些抽象层落到"哪一层贡献了多少"的实证上。它同样呼应 [[concepts/working-set-vs-long-term-memory|工作集与长期记忆]] 的核心命题——上下文窗口不是记忆，记忆是窗口之外那部分被验证过、可复用、可清理的知识。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

对做 harness 的人，最可直接借用的两条是：(1) 把记忆拆成"append-only 原始日志 + 可重建的派生层"，让记忆可审计、可回滚（这与 [[entities/agent-memory-architecture|Agent Memory Architecture]] 中的日志化主张同构）；(2) 把 consolidation 放到离线 dreaming 流水线，用批量换成本，同时把"显式指令"留给在线 tool 通道以免延迟。值得注意的是评测口径：作者用 Claude Opus 4.6 作 agent 模型、GPT 5.6 sol 作 memory 模型，并在 LangGraph 与 OpenAI Agents SDK 两套 harness 上取均值，这意味着数字衡量的是"记忆层"而非某个 harness 的偶然表现。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

## 深度分析

### 成本结构：三种接入方式算的不是同一笔账

plugin 的账单由流量决定，而非由命中率决定：无论这一轮是否需要历史，固定点上的读写都会发生，成本随会话数线性增长且不可逃避。tool 的成本由"决策次数"决定，只在 agent 判断需要时才付，代价是把判断权交给模型能力——省下的 token 是概率性的，浪费的往返是确定的。offline pipeline 把成本挪到 ingest 侧由批处理摊薄，付的是时效。这三者并不是同一条轴上的三个刻度，而是三笔结构不同的账。

原文里有一条容易被略过的脚注值得单独拎出来：在三个开源 benchmark 上各后端的延迟几乎一样（4-7 秒/题），因为绝大多数问题单跳即可作答，于是只挂 plugin 就够了，再给 agent 加检索工具属于严格被支配的配置。这句话实际给三种模式划了适用域——tool 模式的成本优势只在"多跳、长程、记忆读取相对稀疏"的负载里才兑现；在单跳召回场景里，多一次工具调用只是多一次往返、多一轮 token。这也是 [[concepts/context-window-economics|上下文窗口经济学]] 在记忆层的具体形态：省不省钱取决于"读取频率与真实需要的比值"，而不是取决于技术选型的新旧。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

MemCurrent 绕开"plugin 太贵、tool 不稳"的夹缝，做法不是选边而是**按粒度分工**：plugin 只负责在会话起点注入一份自动生成的索引（便宜、大小固定），tool 负责按标识取完整条目（贵但按需），dreaming 负责变换与整合（最贵但可离线）。于是会话起点的固定成本与"索引有多大"成正比，而不是与"记忆有多少"成正比——这是让 plugin 模式不烧钱的唯一办法，也是这套设计里最不显眼却最关键的一处工程挪移。

### 评测方法论：真正的资产是漏斗，不是分数

公开 benchmark 测的是"从合成人格的闲聊里找回一句先前说过的话"，单跳、以答案打分。内部 benchmark 同时改了三件事：领域换成真实数据工作区（表语义、指标定义、工作流），计量单位换成多步任务是否完成（记忆要喂给动作，而不只是喂给回答），覆盖换成 5 类记忆 × 11 种场景的交叉。这三处改动合起来才叫"贴近生产"，单独改任何一处都不够。

构造漏斗才是可迁移的部分：12K+ 会话里挖出 3K 条带记忆价值信号的候选，用 7 条准则的对抗式 judge 逐条筛（知识是否真的存在、是否针对该用户、是否解题所必需、是否自然、是否可验证），再人工复核，最后只留下 114 条用例、29 个用户、2,233 段历史会话。从 12K 到 114 大约是 1% 的产出率——这就是一份记忆 benchmark 的诚实成本，也解释了为什么多数团队退回去用合成集。其中"解题所必需"是承重的那一条：没有它，更强的模型靠推理或猜测也能过，记忆的贡献就无法被测量。干扰项取自同一用户的其他会话而非他人会话，让检索难度落在"人对了、会话错了"这个真实档位上，而不是白送。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

另一个应当抄走的设计是天花板：gold injection 给出"记忆完美"的上界（78.6%），于是 68.4% 可以被读成"闭合了八成以上的差距"，而不是一个孤立的绝对分。这套"漏斗 + 必需性门禁 + 天花板"的三件套，比任何单点分数都更接近 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]] 里讲的构念效度问题：先证明题目真的在考记忆，再谈分数。

### 消融数字证明的是机制，不是排名

四个数排开：无记忆 22.3%，仅 episodic 35.8%，仅 derived 66.1%，两者叠加 68.4%。derived 的边际贡献约为 episodic 的十九倍，而 episodic 在 derived 之上只再加 2.3 个点——但这 2.3 点买到的是一份 append-only、可回溯、可审计的原始轨迹，是 pass rate 无法定价的合规与回滚资产。用分数判断"episodic 可有可无"是误读，它换的是另一类收益。

比数字更有信息量的是 episodic 失效的**机制**：agent 只在约三分之一的用例里真的去搜原始历史，而搜的时候通过率有 60%。也就是说它的短板是"不知道要去找"（调用问题），不是"找到了没用"（内容问题）。derived memory 能跳到 66.1%，很大程度上靠的是会话起点注入索引这件事本身——它把"库里有什么可查"变成 agent 的前置知识。由此可以推一个可检验的假设：若给原始轨迹也配一份索引，也许不蒸馏就能拿回大部分差距。换言之，"蒸馏的收益"里混着一部分**路由收益**，而我们平时把两者一并记在了"抽取质量"头上。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

抽取策略同理：离线 dreaming 63.5% 对在线 tool 53.9%，两者写进同一个库，所以这近 10 个点的差距隔离出的不是"写了什么"，而是"何时写、以多大视野写"。在线通道只在用户显式开口时触发，覆盖面受限于用户想得起来说什么；dreaming 看得到每一段会话，还具备单次在线写永远不具备的能力——跨会话发现矛盾并合并。这才是它领先的真实原因，而不只是"批处理更便宜"。还有一处必须提醒：两组消融是嵌套的，不构成可加关系。抽取策略那一组（53.9 / 63.5 / 66.1）只测 derived memory，叠上 episodic 后的 68.4 属于另一层，不能把 10 点和 2.3 点相加。

### 这套设计会在哪里断掉

**领域变化快于整合周期时。** 文件是可变的、冲突由流水线"解决"、读取方不校验新鲜度，这套设计没有显式的有效期契约。在"当前负责人""当前配额"这类小时级漂移的对象上，滞后的派生记忆比没有记忆更糟——它会以高置信度的语气给出过期事实，而 agent 没有怀疑它的习惯。

**正确性可以被廉价验证时。** 原文两个案例的价值分别是"不要重复已知的坑"和"不要重新探索路径"，而在代码、SQL、单测这类有廉价验证器的场景里，重推一遍往往比维护一座库更划算，经济学是反过来的。这也是 [[concepts/when-not-to-memory-architecture|何时不要建记忆系统]] 那条边界的另一面。

**单用户、低频、短程时。** 数据库、索引、dreaming 流水线、评测漏斗全是固定成本；在几百段会话的规模上，三层分工只是纯开销，收益还抵不上运维注意力。

**多租户与合规场景。** 共享的可变派生库加上自动跨会话合并，是交叉污染与"删除语义"的双重雷区；append-only 轨迹又把"被遗忘权"变成跨轨迹硬删，而原文未讨论这一点。同样缺席的是写入侧信任：能自动保存"纠正"的通道，也能自动保存一个错误的"纠正"，而它一旦进了索引就会被后续会话当成先验。

## 实践启示

1. **按粒度分工，而不是三选一。** plugin 只注入索引/指针，工具只按标识取 payload，dreaming 只做变换与整合。检验标准很硬：会话起点的固定成本应当随索引大小增长，而不是随记忆体量增长。这条分工也是 [[concepts/harness-tool-design-evolution|harness 工具设计演化]] 里"工具该薄还是该厚"的一个正面样本。
2. **把"解题所必需"做成评测的硬门禁。** 每条候选问题先过对抗式 judge（存在性、用户特异性、必需性、自然度、可验证性），干扰项取自同一用户的其他会话。1% 的漏斗产出率是正常的；为凑样本量而放宽必需性，等于自废测评。
3. **报"距天花板的距离"，不报绝对分。** 先跑 gold injection 拿上界，再报 gap closed（本例 68.4% 对 78.6%，闭合 >80%）。同时公布 token 与延迟，否则"更强"很可能只是"更贵"。
4. **固定底层 LLM、跨两套 harness 取均值。** 至少 LangGraph 加一套别的 harness，各跑 3 次；否则你测到的是某套 harness 的偶然表现，不是记忆层本身。
5. **把公开集上的领先基线拉进自己的真实负载复测。** 本例中在开源集上位于 Pareto 前沿的 Mem0，内部集上掉到 43.9% 且更慢更贵——这个落差是评测构念效度的体检项，应当作为上线前必跑项，而非竞品八卦。
6. **上线前先写新鲜度契约与删除语义。** 给派生条目补上来源会话、写入者、有效期；约定冲突裁决规则，以及用户要求遗忘时的硬删路径。这两件事在 114 条用例里测不出来，但通常是企业落地的第一道门。^[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026.md]

→ [[raw/articles/memcurrent-persistent-memory-production-agents-databricks-2026|原文存档]]
