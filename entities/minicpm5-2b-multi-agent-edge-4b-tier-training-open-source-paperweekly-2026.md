---

title: "MiniCPM5-2B 端侧多 Agent 杀进 4B 档"
created: 2026-09-09
updated: 2026-09-30
type: entity
tags: [minicpm, edge, multi-agent, llm, open-source, training, rl, post-training]
sources: [raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026]
confidence: 0.7
---

## 端侧 Agent 雏形初现

面壁智能（ModelBest）最新开源的 **MiniCPM5-2B**，把「高智能密度」路线推向了新的节点。在大模型越做越大的背景下，它证明了 2B 参数规模并不只停留在聊天和基础问答——响应速度、运行成本和本地完成能力之外，一个端侧模型已经能真正接手并完成 Agent 任务。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

除了模型算法和权重，MiniCPM5-2B 还同步开放了部分训练 **recipe**、**RL 框架**，以及覆盖 Code、Agent SFT、RL 和预训练数据精炼的一系列数据成果，让开发者有更多可研究、可复现的空间。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

## 能力不止一档：杀进 4B 档

在 Artificial Analysis Intelligence Index 上，MiniCPM5-2B 以 23 分拿下全球 4B 以下模型第一，表现甚至超过了 Qwen3.5-9B。整体评测平均得分 53.9，高于 Qwen3.5-4B 的 51.1 和 Granite 4.2-3B 的 42.7；其中代码推理拿到最高的 5/5，工具调用与搜索 Agent 场景也拿到 3/3，SWE-bench Verified 与 NoLiMa 长文本任务同样位居前列。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

实际跑出的任务包括：梳理企业采购材料、读取产品页面生成简报、跨 5 个官方页面相互对照找条款，以及把 8 篇 Agent 经典论文并行喂给模型——8 个 Subagent 同时开工、两个 Reviewer 复核，最终整理出完整报告。从读取文件、访问网页到多来源整理和并行执行，MiniCPM5-2B 已把 2B 模型能做的事从「回答问题」推进到「完成任务」，站上了端侧模型的 Pareto 前沿。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

## 数据家底打包开源

这些 Agent 能力并非最后一轮微调临时补出，而是从 Agentic 预训练、SFT 到大规模 RL 专门围绕 Agent 优化，并配有一套贯穿各阶段的数据治理体系： ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

- **UltraData-SFT-Agent-2609**：约 50 万条的 Agent 后训练核心数据池，覆盖工具调用、网页搜索、Code Agent、Office 文档处理、多轮记忆与数据库交互；同一任务放入不同 Agent Harness 反复采样，最终以沙箱、测试用例和文件产物等方式真正验收。
- **UltraData-Code**：从约 1.92 亿个公开 GitHub 仓库出发，逐层清洗去重、筛选算法价值、合成结构化样本；相同预算下替换一半算法代码数据后，EvalPlus 与 MultiPL-E 分别再提升 8.42 和 8.07 个百分点。
- **UltraData-RL-2609**：更关心题目能否验证、奖励标签是否可靠、对当前模型有无学习价值，把算力集中在「够得着但还没完全学会」的样本上。
- **UltraX 预训练数据精炼**：不直接重写整段文本，而是预测保留、删除、替换、插入等结构化编辑操作并由程序确定执行。五套语料平均表现全部最高，50 组「任务 × 语料」组合拿下 34 个最佳；FineWeb 上少用 4B tokens 反而更高，UltraX-Preview 总计约 100B tokens、1.14 亿条样本。截至 2026 年 9 月，UltraData 系列累计下载量超 266 万。

## RL 框架同步开放

洞见来自 [[deepseek-code-harness]]、[[agent-harness-engineering-paradigm]] 等训练工程沉淀：RL 基础设施本身同样值得开放。OpenBMB 随模型开源了自研强化学习框架 **Meshy** 与面向长思维链 RL 的 **JustRL II** 训练配方。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

- **Meshy**（角色驱动 SPMD）：不再设置中央控制器、摆脱 Ray 依赖，把 Inference、Training、Rollout、Teacher 等角色做成独立 Service，样本经 TransferQueue 在服务间流动，由数据是否就绪驱动；同一框架可支持同步 RL、有界异步、全异步流式训练与 OPD。
- **JustRL II**（Critic 的 token 级信用分配）：长思维链 RL 后期指标往往回落，原因是噪声标签与 GRPO 粗粒度的信用分配。JustRL II 用三阶段流水线筛选校准数据，并在保留 GRPO group 结构基础上引入 Critic 细化到 token 级信用分配。同样数据算力下，baseline 在 74% 左右见顶回落，JustRL II 则提升至 81%；用于多领域 RL 后训练并配多教师在线策略蒸馏，17 项评测全部获得提升。

## 深度分析

**参数规模与能力边界已经脱钩，Agentic 维度的领先幅度远超通用维度。** MiniCPM5-2B 在 Artificial Analysis Agentic Index 上与第二名拉开 11 分差距，而在 Intelligence Index 上领先幅度是「几分」——这说明同档模型的通用能力已经趋同，真正拉开身位的是 Agent 场景。它平均输出约 21k tokens 与同档持平，拿到 23 分靠的不是输出更长，而是同预算下更高的任务完成密度，这与面壁持续研究的「密度定律」一致：相近参数预算下可承载的智能仍在增加。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:58-74]

**Agent 能力是全链路训练的系统性产物，不是最后一轮微调的补丁。** 从 Agentic 预训练、SFT 到大规模 RL，每个阶段都专门围绕 Agent 优化，并由一套贯穿始终的数据治理体系串起来：SFT 阶段同一任务放进不同 Agent Harness 反复采样、以沙箱/测试用例/文件产物真实验收；RL 阶段按可验证性与学习价值筛题。这否定了「拿通用模型后训练一轮工具调用就能做 Agent」的捷径认知，也呼应了 [[concepts/agent-harness-engineering-paradigm]] 中 Harness 参与训练数据构造的思路。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:98-102]

**数据治理从「过滤」进化到「精修」，且收益可量化、操作可追溯。** UltraData-Code 在相同预算下替换一半算法代码数据即带来 EvalPlus/MultiPL-E 约 8 个百分点的提升；UltraX 不重写整段文本，而是让小模型预测保留/删除/替换/插入等结构化编辑操作、由程序确定执行——在 Ultra-FineWeb 这类本就干净的语料上只减 3.8% token、保留 99.2% 非空样本仍有提升，FineWeb 上少训 4B tokens 反而更高。这印证了 [[concepts/data-quality-framework]] 的方向：数据工程的杠杆点正在从「删什么」转向「怎么改」。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:126-160]

**长思维链 RL 的后期回落被拆解成两个可修的工程问题，而非训练不稳定性的玄学。** 归因是噪声标签与 GRPO 粗粒度信用分配，对策分别是三阶段数据筛选校准流水线，和保留 GRPO group 结构下引入 Critic 做 token 级信用分配——baseline 在 74% 见顶回落，JustRL II 持续升至 81%，随后配合多教师在线策略蒸馏使 17 项评测全部提升。这套「先归因、再分别在数据侧和算法侧修」的方法论，与 [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]] 中 GRPO 信用分配的演进脉络相互印证。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:188-204]

**开源的竞争单位已经从权重下沉到基础设施层。** 在多数开源仍停留在模型权重时，MiniCPM5-2B 把训练 recipe、四类数据成果、RL 框架 Meshy（去中央控制器、去 Ray 依赖的角色驱动 SPMD）一并开放——实质是把社区复现的门槛从「跑通推理」降到「复现训练」，这在 [[entities/deepseek-code-harness]] 等工程沉淀之外，为小模型 RL 后训练提供了可参考的完整基建样本，也是 UltraData 系列累计 266 万下载量背后生态策略的延续。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:32-36]

## 实践启示

- **端侧/小模型选型别再按参数量筛**：直接看 Agentic Index、SWE-bench Verified、工具调用与长文本等可验证基准；2B 级模型已能驱动「多 Subagent 并行 + Reviewer 复核」的 [[concepts/agent-harness-engineering-paradigm|Harness]] 结构，端侧 Agent 原型开发可从 2B 起步而非 7B+。
- **构建 Agent SFT 数据时加两条硬约束**：同一任务放进不同 Harness/执行环境反复采样（覆盖工具、系统约束、交互接口变化），并以沙箱、测试用例、文件产物做程序化验收——验收标准是「事情做完」而非「回答像对的」。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:112-116]
- **RL 数据集按三重过滤分配算力**：可验证性 × 奖励标签可靠性 × 对当前模型的学习价值；算力集中在「够得着但还没完全学会」的区间，完全做不出的难题用动态采样保留而非一刀切丢弃。参考 [[concepts/reinforcement-fine-tuning-rft]]。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:128-132]
- **预训练语料优先做结构化精修而非大规模删减**：让强模型生成精炼文本、与原文对齐转成可执行编辑操作（保留/删除/替换/插入）再由程序执行，全程可追溯可审计；即使语料本就干净也有收益，且能省下训练 token 预算。
- **搭长思维链 RL 管线时的一组可抄组合**：框架层参考 Meshy 的角色驱动 SPMD（Inference/Training/Rollout/Teacher 独立 Service + TransferQueue 数据就绪驱动，同一框架支持同步/有界异步/全异步/OPD）；算法层用 JustRL II 的 Critic token 级信用分配治后期回落；多领域能力整合用多教师在线策略蒸馏（参见 [[concepts/model-distillation-compression]]）。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md:178-204]

## 结语

两年多来 MiniCPM 一直在做同一件事：让端侧模型承担更多任务。到 MiniCPM5-2B，2B 参数已覆盖代码、长文本、工具调用和深度搜索，多 Agent 协作也开始跑起来，端侧通用 Agent 雏形随之清晰。相关工程脉络可对照 [[minicpm-v-46-13b]]、[[minicpm5-1b-forgetrain-agh-hunt]] 与 [[minicpm5-1b-forgetrain-machine-heart]]。端侧模型的竞争已不只在有限资源下完成部署，更要看有限参数预算下能承担多复杂的任务——这正是「edge-Agent + 高智能密度 = 可迁移工程」的又一次验证。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

→ [[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026|原文存档]] ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]