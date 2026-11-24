---
title: "浙大 LongDS-Bench 长程数据分析基准"
created: 2026-09-24
updated: 2026-11-16
type: entity
tags: ['benchmark', 'data-analysis', 'agent']
sources: [raw/articles/longds-bench-zhejiang-emnlp-2026]
confidence: 0.7
vxc: 56
score: v=7/c=8/stars=3
provenance_state: extracted
---

# 浙大 LongDS-Bench 长程数据分析基准

**摘要**：浙江大学、蚂蚁集团及联合实验室提出 LongDS-Bench，系统评测 Agent 在长程多轮数据分析中维护"分析状态"的能力。基准从真实 Kaggle 分析工作流构建 68 个任务、2,225 轮交互，覆盖六个领域，论文已被 EMNLP 2026 主会录用。v1.1 细化任务规范并推出 24 任务、777 轮的 Lite 子集；最新 v1.1-Lite 榜单上 GPT-6 Astra 以 78.17 分暂列第一，Claude Fable 5.1 以 76.53 分紧随其后。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:17-29]

## 核心内容

### 基准定位：考察"分析状态"的持续正确使用

LongDS 面向长程、多轮数据分析任务，评估 Agent 能否在连续分析中始终用对当前需要的数据、指标和中间结果。任务来自真实 Kaggle 分析工作流，覆盖商业、社区、教育、地学、社会公益和体育六个领域，平均每项任务约 32.7 轮交互。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:45-51]

动机来自真实场景：一次分析不会停在单个问题，用户看完结果还会缩小样本范围、调整指标权重、比较几种假设、再按早先定义复核结论；同一句"重新计算排名"在不同轮次可能要用不同的数据子集、计算公式和中间表。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:58-62] 更隐蔽的是，一段代码即使运行成功，也可能用了错误版本的数据或指标——通常没有报错，看似正常的结果已偏离用户意图，而 Agent 继续沿用这些中间结果时，错误会传播到后面的计算。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:64] LongDS 将这些依据统称为分析状态，包括数据范围、指标定义、假设和可复用的中间表等。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:68]

### 与既有数据分析基准的区别

既有数据分析基准通常给出完整目标让 Agent 开展工作；交互式基准则通过多轮对话澄清需求或逐步提供分析指导，以完成初始任务。LongDS 的考察轴不同：工具调用次数多、输入文本长、交互轮次多只是让任务变长，它重点考察后续请求如何使用前面留下的分析状态。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:74-76]

LongDS 的每一轮都有新的分析请求和独立的参考答案，回答后续问题时 Agent 还需继承、修改或恢复此前建立的状态。难点在于分析过程会留下多个有效版本，最新的版本未必适合当前问题——例如沿用新指标定义的同时恢复早先的数据范围，Agent 必须弄清每次修改何时有效、影响哪些计算。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:78-84] 论文将状态行为归纳为六种模式；后续轮次默认继承此前有效状态，因此继承不单列为标注类别，且同一轮可同时涉及更新、回退等多种行为。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:86-90]

### 任务构建与评测管线

研究团队从 64 个 Kaggle 竞赛或公开数据集收集 256 份 Notebook，经数据可用性、可执行性、分析深度等筛选后保留 77 份；先人工构建 3 个多轮长程种子任务，抽取任务构建 skill 后由 Codex 辅助构建其余初始任务。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:100-104] 每轮配有用户请求、可执行参考代码、参考答案及状态/依赖标注；初始任务随后经历专家审查、Agent 辅助验证与一致性检查，团队重新执行参考代码并删除无法修复的任务。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:106-108]

评分使用 DeepSeek-V4-Pro 逐轮打分：与参考答案在语义和数值上一致记 1 分，否则 0 分；汇总时先算每项任务的逐轮正确率，再对所有任务等权平均。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:112] 论文中的 36 轮 Netflix 市场机会分析展示了状态组合的难度：第 18 轮把长电影时长门槛从 110 分钟临时放宽到 100 分钟、仅用于一次比较，默认分数需保留；第 24 轮则要按第 23 轮的当前分数选前八市场，取回第 21 轮引入导演惩罚前的分数，再与第 12 轮基线比较——选市场、取分数、做比较各用不同轮次的结果。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:116-126]

### v1.1、Lite 子集与运行方式

随论文被 EMNLP 2026 主会录用，团队发布 v1.1：重新核查整个基准，修订可能引起歧义的任务表述、修正部分参考答案错误、明确并列结果的处理规则和输出要求，同时保留原有分析逻辑与跨轮次状态依赖。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:132-136] Lite 子集从 v1.1 完整版选出 24 个能区分 Agent 表现的任务，覆盖全部六个领域和多种状态演化行为，保留所选任务的全部内容与轮次（共 777 轮，平均每任务仍约 32 轮），以更低成本完整检验长程状态管理。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:138-140]

运行方式上，除 DSGym 外提供 Codex、Claude Code、Kimi Code 和 Qoder 四种原生 CLI 运行器，均支持本地与 Docker 评测；Docker 模式下每项任务使用独立容器，运行器通过原生会话续接，让所有轮次在同一 Agent 对话中连续完成。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:142-144]

### 榜单结果与错误模式分析

截至 2026-09-07 的 v1.1-Lite 榜单：GPT-6 Astra 暂居榜首，比 Claude Fable 5.1 高 1.64 个百分点，比同样使用 Codex 的 GPT-5.6-sol 高 7.46 个百分点；前两名接近，其他模型得分相差较多。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:148-152] 论文原版实验（DSGym、五模型）中 Gemini-3.1-Pro 以 48.45 最高，GPT-5.4 和 Claude-4.6-Sonnet 分别为 43.50 和 41.56；新版榜单的模型、任务版本、任务集合和运行框架均已变化，两组分数需放在各自设置下解读。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:156-158]

原版实验的错误分析揭示了长程失败的三个结构性事实。其一，按相对进度对齐任务后，从最初 10% 到最后 10% 进度区间整体正确率下降约 46.8 个百分点；直接依赖的历史轮次增加、追溯距离变长都对应更低正确率，回退到旧状态的请求也比初始构建更难。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:166-172] 其二，对抽样任务 3,207 个错误轮次的分析将上下文记忆错误、状态管理错误和级联错误归入长程错误，三类合计占各模型被分析错误的 52%–69%，级联错误占比最大——出错的是更早生成、又被本轮沿用的中间状态，只检查本轮代码未必找得到根因；状态管理错误包括选错版本、更新错对象或恢复不合适的历史状态。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:176-182] 其三，环境重置诊断实验（GPT-5.4）显示在弱任务上小幅改善、在强任务上正确率明显下降：状态出错时重置可减少传播，状态正确时清空又删掉后续需要的结果；且交互步骤更多的模型并不总得分更高，沿用错误状态时增加预算可能只是反复算错。 ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md:186-192]

## 深度分析

**评测轴的转移是这篇工作最重要的贡献。** 此前长程基准普遍以"更长"为轴——更多工具调用、更长输入、更多轮次；LongDS 把评测对象换成"状态演化行为"：沿用、修改默认定义、临时假设、历史回退与组合。这一轴直接对应真实数据分析中最常见的失败面，也解释了为何单轮能力强的模型仍会在长程任务上退化。

**46.8 个百分点的进度退化与级联错误的主导地位，共同指向"无报错失败"这一评测盲区。** 传统 pass/fail 评测只能捕捉报错；LongDS 的每轮独立参考答案 + 状态/依赖标注，让"代码跑通但用错版本"这类静默偏离成为可量化的信号。级联错误占比最大说明错误主要不在当下推理，而在历史中间结果的继承链上——这与上下文工程中"工作集 vs 长期记忆"的取舍直接相关。

**环境重置实验揭示了一个非单调杠杆：** 清空环境对弱任务是修复（切断错误传播），对强任务是破坏（丢失正确积累）。这提示 Agent 记忆/状态管理不存在"越多保留越好"或"越干净越好"的普适策略，需要按状态的置信度区分处置——论文给出的改进方向（记录来源轮次与有效版本、临时分支与默认状态分离、只重算受影响部分）正是这一思路的工程化。

**榜单头部集中但断层明显。** GPT-6 Astra 与 Claude Fable 5.1 仅差 1.64 个百分点，而其他模型差距拉大到 7 个百分点以上，说明长程状态管理已成为头部模型之外的显著能力分层维度。同时原版（DSGym、五模型、最高 48.45）与 v1.1-Lite 榜单不可直接比较，提醒消费基准分数时必须核对任务版本与运行框架。

**与 wiki 内其他长程基准相比**，LongDS 的差异化在于"每轮独立参考答案 + 状态依赖标注"的标注密度——PolyWorkBench、RoadmapBench、LongCat-VitaBench-2.0 等同样考察长程能力，但多数沿任务完成度打分，LongDS 是少数把"中间状态用对没有"作为一等评测对象的工作。

## 实践启示

1. **为分析对象记录来源轮次和有效版本**：让每个中间结果携带"由哪一轮、哪一版规则生成"的元数据，是抑制级联错误最直接的工程手段。
2. **把临时分支与默认状态分开保存**：像 Netflix 示例中"110→100 分钟只用于本次比较"一样，临时假设不应污染默认计算路径。
3. **检查关键中间表而非只检查本轮代码**：级联错误的根因在被沿用的历史状态，单轮代码审查定位不了它。
4. **状态冲突后只重算受影响的部分**：发现版本冲突时做增量重算，而非全量清空重置——重置在高质量状态上是净损失。
5. **评测设计可直接复用**：每轮独立参考答案、状态/依赖标注、逐轮正确率 + 任务等权平均的汇总方式，适用于自建数据分析 eval 的多层评测体系。
6. **解读榜单必须绑定设置**：模型、任务版本、任务集合、运行框架任一变化都使分数不可比，跨榜单排名需回到原始实验设置。

## 关联

- 相关概念：[[concepts/agent-evaluation-benchmark-frameworks]]、[[concepts/context-management-agent-systems]]、[[concepts/working-set-vs-long-term-memory]]
- 相关实体：[[entities/polyworkbench-cross-lingual-long-horizon-agent-benchmark-paperweekly-2026]]、[[entities/meituan-longcat-vitabench-20-long-term-dynamic-agent-benchmark]]、[[entities/roadmapbench-long-horizon-agentic-software-development-benchmark]]
- 资源：论文 *LongDS-Bench: On the Failure of Long-Horizon Agentic Data Analysis*；主页 <https://zjunlp.github.io/DataMind/>；代码 <https://github.com/zjunlp/DataMind/tree/main/longds>；数据 <https://huggingface.co/datasets/zjunlp/LongDS>

→ [[raw/articles/longds-bench-zhejiang-emnlp-2026|原文存档]] ^[raw/articles/longds-bench-zhejiang-emnlp-2026.md]
