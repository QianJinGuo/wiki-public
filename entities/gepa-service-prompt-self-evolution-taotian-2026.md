---
title: "GEPA Prompt 自进化服务（淘天营销&交易技术团队 gepa-service）"
created: 2026-09-17
updated: 2026-09-17
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

## 与既有 GEPA 内容的关系

- 与 `[[entities/gepa-optimize-anything|GEPA Optimize Anything]]`（通用文本优化框架视角）不同框架、不同抽象层：本文是团队级 **LLM Judge 对齐工程系统**（业务落地视角 + 三反直觉认知），互补不重复
- 库内 `concepts/` GEPA 论文条目已归档（thin），本文的工程实现维度（冻结区/膨胀/双通道）为论文原文所无的实践增量
- 相关：`[[entities/agent-self-improvement-six-mechanisms|Agent 自进化六机制]]`、`[[entities/alipay-618-intent-recognition-prompt-evolution|支付宝 618 意图识别 Prompt 进化]]`（同为淘系 prompt 自进化实践，路径不同：618 用黄金用例集自动调优，本文用 GEPA 帕累托反思变异）

→ [[raw/articles/gepa-service-prompt-self-evolution-taotian-2026|原文存档]]
