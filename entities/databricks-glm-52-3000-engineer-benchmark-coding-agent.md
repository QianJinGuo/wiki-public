---
title: "Databricks CEO用3000名程序员真实任务测试GLM 5.2 — Harness选择比模型更重要"
created: 2026-07-12
updated: 2026-10-25
type: entity
tags: [coding-agent, benchmark, harness, glm, model-evaluation, ai-engineering, databricks]
sources: [raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Databricks CEO用3000名程序员真实任务测试GLM 5.2 — Harness选择比模型更重要

2026年7月，Databricks 联合创始人兼 CEO Ali Ghodsi 率领团队，基于公司3000多名工程师的真实 Pull Request，做了一次大规模编码 Agent 基准测试，覆盖多家模型和编码框架的实际表现。测试结果揭示了一个核心发现：**调用模型的框架（Harness）选择，对成本和质量的综合影响不亚于模型本身的选型**。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

## 测试方法论

测试基于 Databricks 真实业务代码库（含 Scala、Go、Rust、Java、Bazel 等多种语言和工具链），从最近的 Pull Request 中抽取出包含独立测试的任务，经过人工校准后构建基准测试集。团队利用 Unity AI Gateway 收集了所有编码交互日志以分析任务复杂度分布。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

为防止 AI 代理利用 Git 历史作弊，测试期间对 Git 历史进行了隔离。评估不依赖 LLM 裁判，而是依靠客观的测试通过情况判定结果。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

## 核心发现

### 1. GLM 5.2 逼近 Opus 4.8，开源模型迎来突破

测试中，GLM 5.2 成功跻身顶级能力梯队，在质量上与 Opus 4.8 基本持平，但单次任务成本仅1.28美元（Opus 4.8为1.94美元）。团队已着手将开源模型部署为日常开发主力。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

### 2. Token 单价 ≠ 任务成本

开发者通常通过对比 Token 单价估算 AI 费用，但测试表明这种估算经常失真。例如 Sonnet 5 的 Token 单价比 Opus 4.8 便宜约1.7倍，但实际单次任务成本为2.09美元（Opus 4.8仅为1.94美元），且完成率还低6个百分点。原因在于 Sonnet 5 需要更多轮次尝试，多消耗了1.9倍的 Token。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

### 3. Harness 对效率的决定性影响

即使调用完全相同的模型，使用不同框架（如 Claude Code/Codex vs Pi）时，任务成本可能产生两倍以上的差距，而质量保持一致。Pi 框架每次交互发送的上下文减少约三倍，通过更紧凑的工作集管理上下文，用更少轮次完成任务。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

### 4. 模型分类呈现能力梯队

测试显示模型和框架可清晰划分为三个能力梯队。开发团队决定将更多日常工作分流给 Haiku 和 GPT 5.4 Mini 级别的模型，避免盲目使用高价模型。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

## Omnigent 智能路由

基于测试结论，Databricks 开发了 Omnigent 调度系统，可在模型和框架之上按任务灵活切换不同组合，实现智能路由。该系统计划通过 Unity AI Gateway 集成，根据任务类型自动推荐最合适的模型和框架组合。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

→ [[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent|原文存档]]

> 参见 [[entities/amd跑glm-52成本只要英伟达一半|AMD跑GLM 5.2成本只要英伟达一半]] 从不同角度报告 GLM 5.2 推理成本。

---
## 深度分析

### 一、"Harness > 模型"的操作性拆解

Harness 不是 prompt 模板，而是四个可独立调参的机制层：**工具面**（暴露哪些工具、schema 多长）、**上下文策略**（每轮送什么、历史如何裁剪、工作集如何维护）、**循环控制**（何时停止、何时重试、何时升级求助）、**验证回路**（能否读到编译/测试输出并据此定向修复）。差异落在输入侧而非生成侧：同一模型换 harness，成本差两倍以上而质量保持一致，Pi 每次交互发送的上下文少约三倍、用更少轮次完成任务。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

机制上，任务成本 ≈ Σ(第 t 轮上下文长度) × 单价，而第 t 轮上下文 = 系统提示 + 工具 schema（静态）+ 累积轨迹（随 t 线性增长），总输入量随轮次近似平方增长。压缩上下文有双重收益：直接削减每轮计费，同时提高信息密度而减少绕路轮次——"更少上下文"与"更少轮次"是同一个变量的两面。

工具面是最易忽视的静态税：每个工具的名称、描述、参数结构每轮都要重发，20 个工具轻易占据数千 token；候选工具越多选错概率越高，每次选错再付一轮往返。验证回路决定"探索—修复"的收敛速度：能解析失败测试栈、只跑受影响单测的 harness，把一次修复的成本从"重读半仓库"降到"重读一个函数"。仓库规模导航（符号索引、精确 grep、LSP 跳转）同理——它决定 agent 是在几个函数间推理，还是在几十个大文件间反复采样。

### 二、基准的边界：哪些结论可外推，哪些不可

可外推的部分：内部基准解决了公开集的两个问题——污染与分布错配。SWE-Bench 类任务早已可能进入训练数据，且无法代表由 Scala/Go/Rust/Java/Bazel 混合工具链构成的私有代码库；Databricks 因此用近期 PR 自建任务，在测试期隔离 Git 历史，判定只依赖测试通过与否而非 LLM 裁判。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

不可外推的部分有三处。**其一，任务分布偏斜**：日志显示低复杂度任务约四分之一、中等约六成，高难样本只占一成多，因此"GLM 5.2 与 Opus 4.8 持平"的统计权重主要落在中低复杂度区间；长尾难题样本不足，置信区间宽到撑不起"开源已追平"的强断言。**其二，任务派生的结构性偏差**：从 PR 反推任务虽去掉了解决方案与 bug 成因，但 PR 边界本身携带信息——文件集已知、可编译、测试独立，等于把起点放在"已定位、待实现"；而最贵的"从告警到根因"的诊断型任务恰好被排除。**其三，判定是二元的**：测试通过不度量 diff 可维护性、是否顺手删改测试、是否绕过断言，人工只校准了任务构建，没校准产出。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md]

还有一层 Goodhart 风险：自建基准一旦进入采购与路由决策，harness 与 prompt 会朝它持续优化，基准随即失去分辨力。缓解手段是任务集按时间滚动更新，并保留只用于季度复核、不参与调优的 held-out 任务。

### 三、单价、通过率与有效成本

真正的成本分母是"解决的任务数"，不是"发起的任务数"。原始数据给出反例：Sonnet 5 的 token 单价比 Opus 4.8 便宜约 1.7 倍，但单次任务成本反而更高（2.09 美元 vs 1.94 美元），完成率还低 6 个百分点，原因是它需要更长试错、读了更多内容，多消耗约 1.9 倍 token。^[raw/articles/databricks-glm-52-3000-engineer-benchmark-coding-agent.md] 把这个结果折算到"每解决一个任务"的口径上，差距会进一步放大：若高档模型通过率为 p，低档为 p−0.06，则有效成本比 = (2.09/(p−0.06)) ÷ (1.94/p)。取 p=0.70 计算，低档反而贵约 18%；p 越低、两者差距越大。也就是说，**便宜模型的折价必须先被通过率折损抵扣，剩下的才是真便宜**。

把这个逻辑套到 harness 上，组合空间是乘性的：好 harness 把同一模型的成本打到一半，坏 harness 把最贵的模型抬到两倍以上，而质量几乎不动。最优解通常不在"最强模型 + 默认框架"，而在"够用的模型 + 上下文纪律严格的框架"。延迟同向：轮次少还缩短 wall-clock，而工程师被打断的注意力成本常高于账单。GLM 5.2 这类低成本模型在紧 harness 里的 1.28 美元量级还有下压空间，在松 harness 里则会被上下文体量吃掉全部价格优势。

### 四、对路由的启示：把模型选择变成升级阶梯

上文 **Omnigent 智能路由** 假设"按任务类型推荐模型 + 框架组合"，但路由信号需要比"任务看起来难不难"更硬：**可验证性**（有无可自动跑的测试或编译门）、**爆炸半径**（改动文件数、是否跨服务或跨语言构建）、**上下文需求**（需读多少文件，可用历史读取量分位数估计）、**同族任务历史成功率**（按目录/服务聚合，比全局通过率更有分辨力）。

更稳的实现不是训练复杂度分类器再一次性选模型——分类器误差会叠加到路由误差上，边界任务方差又极大——而是**升级阶梯（escalation ladder）**：低档模型先跑，验证回路作门；门不过则带着失败信息升级，并把上一轮 diff 与测试输出交出去。模型选择因而是由失败信号触发、可在单任务内多次发生的策略，这正是 harness 层的职责。

边界同样要写清：阶梯成立的前提是验证便宜可靠（单测、编译、类型检查）。对不可验证的探索型工作——跨仓库重构、架构选型、无测试覆盖的遗留模块——阶梯没有门，降档会静默产出看似合理实则错误的结果，此时应维持高档模型并加人工复核点。三条梯队的差别本质是"第一次就对的概率"而非"能否做对"，所以降档的代价是重试次数与延迟，只在验证便宜处划算。

## 实践启示

1. **先做 harness 对照，再谈模型选型**：固定一个中档模型跑 3 套 harness，只看平均轮次与每轮输入 token 中位数；两者差 2 倍以上时先修 harness，否则换模型的收益会被 harness 噪声淹没。参考 [[entities/claw-swe-bench-harness-evaluation-benchmark-tokenrhythm|Claw-SWE-Bench]] 与 [[entities/arena-ai-harness-tax-coding-agents|HarnessTax]] 的隔离式测法。
2. **把成本指标定义为 cost per solved task**：分母带通过率，同时记录 cost per attempt 用于诊断。任何以 token 单价为依据的模型比价都是无效估算——Sonnet 5 对 Opus 4.8 的反转就是标准案例。
3. **自建基准的构造纪律**：取近 3 个月 PR、过滤机器人提交与自动生成改动、剥离成因描述、分离测试文件、隔离 Git 历史、以测试通过为唯一判定；另留 15%–20% 任务作季度复核用的 held-out 集。同时必须补一类诊断型任务——从真实告警、issue、回滚记录反向构造"从症状到根因"的题面，否则 PR 派生集会系统性低估高档模型在定位类任务上的价值。
4. **降档条件写成可判定的规则**：可自动验证 + 爆炸半径小 + 上下文需求低于历史中位数，三者同时满足才允许降到 Haiku / GPT 5.4 Mini 档；其余走升级阶梯而非静态分配。
5. **优先压缩 harness 的三项静态开销**：工具 schema 精简与按需装配、用工作集替换全量历史重发、失败后只回灌相关测试片段。这三项在多数团队两周内可落地，且不依赖供应商的能力变化。参见 [[entities/harness-engineering-framework|Harness Engineering 概念框架]]；成本侧的横向对照可见 [[entities/cursor让马斯克的grok45咸鱼翻身追平opus-48成本比glm52还低|Cursor 让 Grok 4.5 追平 Opus 4.8]]。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

