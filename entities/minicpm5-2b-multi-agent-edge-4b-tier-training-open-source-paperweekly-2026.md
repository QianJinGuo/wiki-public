---

title: "MiniCPM5-2B 端侧多 Agent 杀进 4B 档"
created: 2026-09-09
updated: 2026-09-10
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

## 结语

两年多来 MiniCPM 一直在做同一件事：让端侧模型承担更多任务。到 MiniCPM5-2B，2B 参数已覆盖代码、长文本、工具调用和深度搜索，多 Agent 协作也开始跑起来，端侧通用 Agent 雏形随之清晰。相关工程脉络可对照 [[minicpm-v-46-13b]]、[[minicpm5-1b-forgetrain-agh-hunt]] 与 [[minicpm5-1b-forgetrain-machine-heart]]。端侧模型的竞争已不只在有限资源下完成部署，更要看有限参数预算下能承担多复杂的任务——这正是「edge-Agent + 高智能密度 = 可迁移工程」的又一次验证。 ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]

→ [[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026|原文存档]] ^[raw/articles/minicpm5-2b-multi-agent-edge-4b-tier-training-open-source-paperweekly-2026.md]