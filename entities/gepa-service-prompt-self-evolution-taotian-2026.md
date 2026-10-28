---
title: "GEPA Prompt 自进化服务（淘天营销&交易技术团队 gepa-service）"
created: 2026-09-17
updated: 2026-10-26
type: entity
tags: [gepa, prompt-optimization, self-evolution, pareto, llm-judge, evaluation, taobao, reward-hacking, prompt-bloat]
sources: [raw/articles/gepa-service-prompt-self-evolution-taotian-2026]
confidence: 0.9
provenance_state: extracted
---

# GEPA Prompt 自进化服务（淘天 gepa-service）

淘天集团-营销&交易技术团队（诗咏）基于 GEPA（Genetic-Pareto, Reflective Prompt Evolution，ICLR 2026，arXiv:2507.19457，DSPy 生态）构建的 **Prompt 自进化系统 gepa-service**：给一条提示词 + 一份标注数据，自动产出更优提示词。解决 LLM Judge / 被评测 Agent Prompt 优化四大痛点——盲目性高（人工经验无数据支撑）、伪自动化（静态扫描缺深度反思）、难以复制（经验沉淀在个人）、无权衡机制（单点修补引发指标对抗性退化）。核心闭环：**推理 → 评分 → 反思 → 选择**，反思式变异（LLM 分析 Badcase → 归因至 Prompt 具体规则 → 生成候选变体）+ 帕累托前沿多目标择优（保留前沿所有候选而非单一最优，避免局部最优）。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

## 双对齐目标

- **评估器 ↔ 人工标注对齐**：LLM Judge 输出与 Ground Truth 一致，离线评测的准确性底座
- **评估器 ↔ 线上业务效果对齐**：判断与 CTR/转化率等业务指标一致，防止离线高分上线翻车

与人工经验驱动范式（启发式盲调收敛低/静态扫描难弥合语义错位/单点修补多目标退化/专家经验难跨域）的根本区别是数据驱动闭环。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

## 架构四层

| 层 | 职责 | 关键设计 |
|---|---|---|
| 配置层 | 让人"配不错" | TaskConfig 聚合 10 子配置；`build_config()` 按任务类型一次性对齐输出 schema/答案提取/评分函数三者（隐式耦合不让用户逐项填） |
| 引擎层 | 把任务跑起来 | `run_optimize`/`run_evaluate`/`validate_config` 三 API；GenericGEPAAdapter 把候选评估翻译成批量推理→解析输出→批次评分→反思轨迹四步 |
| 组件层 | 可插拔六模块 | parsers/scorers/feedback/llm/evaluate/inspector，Protocol 协议扩展，新场景实现接口注入不改引擎 |
| 服务层 | 三种用法 | CLI / FastAPI HTTP / Claude Code·Qoder Skill 对话式；任务状态全落文件系统（每任务 5 文件：config.json/job_state.json/progress.jsonl/best_prompt.txt/result.json） |

任务无关适配：2.0 版把 n 场景 n 套 Adapter 抽象为 **n 种任务类型**（二分类/多分类/评分/对比/聚类），配置驱动，"每个场景写代码"变"填配置就能用"。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

## 三模型角色分工

- **任务模型**（温度 0.0）：执行任务，追求确定性输出
- **Judge 模型**：三维归因（错误定位 / Prompt 归因 / 改进建议）——第 2 问把"答错了"翻译成"哪条规则有问题"，让反思拿到带归因的诊断而非错误堆，盲目搜索变定向改写
- **反思模型**（温度 0.7，建议比任务模型更强）：接收反思四元组（输入/原始输出/分数/反馈）生成候选

评分双层：样本级（给反思轨迹，exact_match/numerical_closeness/weighted_binary 不对称惩罚）vs 批次级（GEPA 优化目标；precision_recall 约束优化——精确率达标直接返回召回率，未达标乘 (precision/target)² 惩罚，表达"精确率≥80% 前提下最大化召回"）。**Inspector 零配置**：字段画像（类型/唯一值/二值/长文本/缺失率）→ 并行推断标注字段/任务类型/输入字段 → DatasetProfile 置信度评估，含建议模板与种子提示词。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

## 三个反直觉工程认知（全库零覆盖）

**① 冻结区（结构性防 reward hacking）**：GEPA 默认整段 prompt 可被反思模型重写——删掉一条约束往往能提分（约束限制行为空间，去掉就少犯"做多了"的错）；纯规则评分下这就是奖励黑客。防御必须**结构性禁止改动**而非事后扣分：prompt 划分**冻结区**（硬性约束/工具调用协议/输出 Schema）与**可变区**（判断优先级/决策倾向/异常处理/表达方式），GEPA 多 component 结构让选择器只返回可变区，冻结段不进反思上下文。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

**② 提示词膨胀治理**：反思模型天然倾向不断补充说明，十几轮迭代 prompt 从几百字膨胀到几千字，每轮分数都涨、无任何信号报警。隐性代价：关键约束被淹没（注意力稀释）/推理成本线性上涨/人工无法接管回滚。抑制：**长度作为帕累托第三目标**（非加权惩罚，避免与主目标抵消）+ 硬上限门禁——目标是"更准，且不更长"。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

**③ 无 Ground Truth 场景的双通道设想**：GEPA 定位 judge align（有标注），但真正值得优化的常是被评测 agent 自己的 prompt（策略生成无正确答案，只有事后业务效果）。两点改造：**变异方向**——原生自由探索偏随机，接离线评测/线上业务效果汇总的归因分析作文本梯度 + prompt 结构缺陷诊断，把"业务上错在哪"翻译成"哪段该改"（定向变异）；**对错判据**——离线指标当 judge 构造对抗双目标 + 历史策略回溯预测（候选决策溯源历史相似场景对比业务指标，**只用于排序与否决、不回流反思**，否则"改得越少→匹配越多→分越高"把搜索拖向保守）。^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md]

## 深度分析

### 稳定性与表达力的三段折中

冻结区/可变区不是一条写作建议，而是把系统从「能提分」改成「只能沿允许的方向提分」的约束工程。冻结区承载硬性约束、工具调用协议、输出 Schema，可变区承载判断优先级、决策倾向、异常处理、表达方式^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:851-871]。其实质是把 prompt 当接口看：对外的契约（schema、协议）不可变，对内的策略才允许搜索。缺了这条边界，反思模型会优先删掉限制行为的约束——约束的本质是收窄行为空间，去掉它模型就少犯一类「做多了」的错；在纯规则评分下这是一条稳定且无人报警的提分路径，即 reward hacking。

提示词膨胀是同一折中的另一面：可变区给得越自由，反思模型越倾向不断补充说明，十几轮后 prompt 从几百字涨到几千字，而每轮分数都在涨、没有任何信号提示出了问题^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:878-891]。把长度设为帕累托第三目标而非加权惩罚，是刻意的结构选择——加权会把长度折进总分、与主目标互相抵消，而独立目标只能通过前沿比较生效，让「更准但更长」的候选被显式淘汰而不是被一个可调的系数悄悄容忍。

双通道把同一条边界推到没有标注的场景：变异方向接归因分析与结构缺陷诊断作文本梯度，对错判据用对抗双目标加历史回溯，但回溯预测只用于排序与否决、不回流反思，否则「改得越少 → 匹配越多 → 分越高」会把搜索拖向保守^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:898-907]。三个机制是同一条原则的三种用法：任何能让分数变好却不代表任务变好的通路，都要在结构上切断，而不是在评分上惩罚。

### 为什么对齐循环必须落地为服务，而不是一次离线优化

双对齐目标（评估器 ↔ 人工标注、评估器 ↔ 线上业务效果）是两个不可同时观测的约束：前者可离线测量，后者只在发布后显形^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:54-60]。因此「跑一次拿到 best_prompt 就结束」的用法缺了关键一环——没有版本、没有回滚、没有灰度，一次错误的对齐会以直接覆盖线上 prompt 的方式进入生产。服务化在这里不是工程洁癖，而是把「离线高分上线翻车」变成可观测、可回退事件的最低成本形态；文中未明写 canary 流程，但 holdout 复评、候选树可视化与五文件任务状态已经提供了它所需的前置条件。

这也解释了架构为何偏可运营性而非算法创新：任务状态全落文件系统，每任务五文件（config.json / job_state.json / progress.jsonl / best_prompt.txt / result.json），progress.jsonl 是追加流、Skill 侧 tail 一下即可实时汇报，best_prompt.txt 是纯文本、可直接替换线上 prompt^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:583-601]。失败形态也随之改变：不再是「分数不够好」，而是任务可取消、可中途回查、可在 holdout 切片上复评泛化（train/val/holdout 按 50/30/20 分层切分）^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:460-467]。

### 帕累托反思变异 vs 黄金用例调优：各自何时取胜

两条路线的差别不在算法强弱，而在「最优」的定义。黄金用例集把标准钉在少量高置信样本上，追求逐条可解释的对齐；GEPA（论文层见 [[entities/gepa-reflective-prompt-evolution-iclr2026-oral|GEPA 反思式 Prompt 进化（ICLR 2026）]]）则假定指标之间会互相拉扯，因此保留前沿上的全部候选（原文示例中四个候选只留 A、B、D）而不取综合分最高的一个^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:208-211]。与之相对，库内 [[entities/alipay-618-intent-recognition-prompt-evolution|支付宝 618 意图识别 Prompt 进化]] 走的是黄金用例自动调优，两者常被并列为「淘系 prompt 自进化」，但适用面并不重合。

帕累托路线在三种条件下赢：指标本身冲突（精确率对召回率、误判对漏判）、标注带噪（单点最优容易恰好选中噪声）、同一 prompt 要服务权衡偏好不同的多个业务方。黄金用例路线在另外三种条件下赢：评分口径单一、样本稀少（几十条撑不起前沿宽度）、以及必须向非技术方解释「为什么动了这一条」。成本结构同样不同：前沿每宽一格就多一个候选要全量推理加批次评分，一次完整优化约 15-30 分钟^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:644]；黄金用例集则是每条用例一次判定，成本线性可预估。选哪条，本质是先回答「这个任务的指标之间是否真的在打架」。

### 成本、噪声与漂移：规模化后先坏在哪

最先坏的是 judge 噪声。整个闭环唯一的文本梯度来自 Judge 模型的三问归因（错误定位 / Prompt 归因 / 改进建议），Judge 一旦系统性偏移，反思模型会把偏移写成规则、再被同一套评分确认为「提分」——错误被固化而不是被纠正^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:539-553]。这正是双对齐目标里第一条必须先行、且不能一次性验收的原因：它是整条链路上唯一无法被优化掉的上游误差。

其次是 reward hacking，且在纯规则评分下不可见：删约束、缩输出、把判断推向多数类，都会稳定提分。冻结区是第一道结构性防御，长度第三目标是第二道，但两者只挡住已知的作弊形态，评分函数越简单（exact_match）剩余空间越大——这解释了业务侧为何备了 weighted_binary 的 FP/FN 不对称惩罚与 precision_recall 约束优化（精确率未达 80% 时乘 (precision/target)² 惩罚）^[raw/articles/gepa-service-prompt-self-evolution-taotian-2026.md:520-525]。把业务诉求编码进评分形状，比事后调阈值更能约束搜索方向。

最后是漂移，也是最难在离线闭环里闭合的一环。评估器与线上业务效果的对齐只能发布后观测；无 GT 场景连「事后业务效果」都要靠历史回溯近似，而回溯一旦回流反思就会自我强化。规模化的真正瓶颈因此不在优化吞吐（批量并发、2s→4s→8s 指数退避重试已把工程侧抹平），而在「评分还准不准」这件需要持续独立校验的事——它决定了这套系统是持续进化，还是把一次错误的判断放大成一条固化的规则。

## 实践启示

1. **先划冻结区，再谈优化**：把输出 Schema、工具调用协议、硬性约束设为不可变异区，只让反思模型改判断优先级与异常处理；在纯规则评分下，「删掉一条约束」永远是最省力的提分路径。
2. **长度进前沿，不进总分**：把 prompt 长度设为独立帕累托目标并加硬上限，别用加权惩罚——加权会被主目标抵消，独立的第三目标才能让「更准但更长」被淘汰。
3. **批次级指标定目标，样本级指标给反馈**：批次分数回答「这个候选整体好不好」，样本分数回答「这一条错在哪」；混用会让反思拿到无法归因的信号。
4. **无标注场景先定判据，再定方向**：先用离线指标构造对抗双目标解决「什么算对」，再谈定向变异；回溯类信号只用于排序与否决，回流即自我强化。
5. **把优化做成任务，而不是脚本**：状态外化到文件系统、holdout 复评泛化、最优 prompt 以纯文本产出——版本、回滚、灰度才有落点。
6. **先校准 judge，再优化 prompt**：judge 噪声是唯一无法被优化环自身修正的上游误差，双对齐（人工标注 / 线上效果）要作为持续校验而非一次性验收。

## 与既有 GEPA 内容的关系

- 与 `[[entities/gepa-optimize-anything|GEPA Optimize Anything]]`（通用文本优化框架视角）不同框架、不同抽象层：本文是团队级 **LLM Judge 对齐工程系统**（业务落地视角 + 三反直觉认知），互补不重复
- 库内 `concepts/` GEPA 论文条目已归档（thin），本文的工程实现维度（冻结区/膨胀/双通道）为论文原文所无的实践增量
- 相关：`[[entities/agent-self-improvement-six-mechanisms|Agent 自进化六机制]]`、`[[entities/alipay-618-intent-recognition-prompt-evolution|支付宝 618 意图识别 Prompt 进化]]`（同为淘系 prompt 自进化实践，路径不同：618 用黄金用例集自动调优，本文用 GEPA 帕累托反思变异）

→ [[raw/articles/gepa-service-prompt-self-evolution-taotian-2026|原文存档]]
