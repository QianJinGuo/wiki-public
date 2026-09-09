---
title: "MMLU 双分数悖论：基准名修复的是名字，不是测量（评估可追溯性）"
created: 2026-09-08
updated: 2026-09-08
type: entity
tags: [llm, evaluation, benchmark, mmlu, traceability, metrology, reliability, eval-infrastructure]
sources: [raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026]
confidence: 0.7
---

# MMLU 双分数悖论：基准名修复的是名字，不是测量

> Dmitrii Zatona 以同一模型家族两个 build（build-42 accuracy 0.781、build-44 accuracy 0.79）表面同名的两份 MMLU 记录为例，论证「benchmark 名只固定了名字，不固定测量过程」，并给出 `apl-ai-eval` 验证器在 score-delta 查询下返回 `incomparable` 的完整复现与测试向量。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

## 核心命题：可比性是参考的性质，不是数字的性质

同一个 benchmark 名（`mmlu`）下，两个 build 的 accuracy 记录（0.781 vs 0.79，差 +0.009）结构上都合法，但它们的 frame 声明了不同的 runner、grader 与 dataset split。`mmlu` 标签只识别一个「数据集家族」，不是一整套完整测量流程。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

用计量学语言（VIM §2.46），计量可比性（metrological comparability）要求结果可追溯（traceable）到同一参考；而对非序数量，测量流程本身可作为该参考。`mmlu` 单独不构成这种参考——只有显式的 frame（测评环境声明）才可能满足 bridge 的 aspect/scope/procedure 约束。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

这一原则在 AI 之外已有代价：HbA1c 有 IFCC 与 NGSP 两套参考方法，数值体系不同（2007 年起 IFCC 用独立单位 mmol/mol）；AWS 的 99.99% 可用性与 Google 的 99.99% 各自动量不同（可用性 vs 磁盘访问中断口径），「99.99%」打印在不同的 measurand 上。NIST AI 800-2 草案与 NAAIMES 也把可比性绑定到协议一致性与评估/生成两分。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

## 「MMLU」不修复的五个开放变量（含已发表效应量）

| 变量 | 已发表测量 | 对数值的影响 |
|---|---|---|
| Split | 论文 test=14079 vs `cais/mmlu` test=14042、dev=285 | 同一名字两个不同分母；dev 本身是 few-shot 来源 |
| Implementation | HELM 0.637 / Eleuther harness 0.488 / 原始 0.636（llama-65b，全 5-shot） | 排行榜名次翻转过 |
| Shot 数 | `num_fewshot` 默认 0，MMLU YAML 不设 | 「5-shot」活在命令行 flag 而非任务属性 |
| Prompt 格式 | 标点/括号/空格 ~5%（Anthropic）；答案位置 gpt-3.5-turbo 67.2→60.9、llama-30b +15.2 | 最高 76 accuracy 分、排名移 8 位 |
| Grader | 精确匹配 vs llm-judge；GPT-4 判断自洽 65% / GPT-3.5 46.2% / Claude-v1 23.8%；位置偏置 | 换 grader 即换测量函数 |
| 网络可达性 | CAISI SWE-bench 事后发现联网；Scale STC 约 3% 可检索、阻断后污染子集下降约 15 分 | 环境即测量条件 |

一篇 2026 年覆盖 101,843 条结果 / 5,816 模型 / 635 基准的调查报告显示 96.5% 缺至少一项最小可复现子模式字段、温度在 93.9% 中缺失；同一模型在一个基准上被两家以 20.9% 与 61.8% 分别报告。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

## APL 协议与 `apl-ai-eval`：用内容寻址 frame 绑定数字与流程

APL 把测评 frame 做成**独立的内容寻址对象**，claim 只携带其哈希（`frame_ref.hash`）；哈希是 SHA-256 over RFC 8785 规范化字节，空白不影响摘要。`apl-ai-eval` Rust crate 把这一约定变成强制约束：`procedure` 必须含非空 `runner_id`/`grader_id`，`scope` 必须含 `benchmark_id`/`benchmark_variant`/`dataset_split`，`exclusions` 必须含三个否定标记（不证明生产就绪/部署安全/域外泛化）。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

对两个 frame 不同的合法记录，`score-delta` 查询因不满足 bridge 的 aspect/scope/procedure 约束而返回 `incomparable`（而非一个数字）；`repeatability-check` 与 runner/grader 等价 bridge 被分别钉死到对应关系类型。`apl-invalid`（记录不是良构观测）与 `incomparable`（两个良构观测不能被相减）是两条独立轴。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

## 六件 frame 不证明的事（边界声明）

frame 只把一个**声明的流程**按哈希绑定到**声明的声明**，但不保证：分数正确（MMLU-Redux 估计约 6.49% 题目含错）、两个同名标识符指同一事物（哈希字节相等不等于共享 commit/加速器/tokenizer）、数据集是同一数据集（名字解析到 14079 vs 14042 两套题）、任何阈值、任何不确定性（v0.1 无 uncertainty 字段，285 vs 14,042 的分母导致不同置信区间）、split 标签的读者语义（`test-lite` 在 crate 测试向量里命名、但无公开工件）、词汇稳定（AI-Eval v0.1 是草案，NIST 视语义化破坏性变更后两侧结果不再可比）。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

## 与既有评估知识的互补关系

本文补的是**评估基础设施的溯源/参照层**——如何把「数字=某某流程的结果」结构化地绑定并让跨团队/供应商/季度的比较在受质疑时站得住脚。它与把评估做成可操作框架的 [[entities/agent-evaluation-systematic-guide-metrics-to-closed-loop|评估指标到闭环]]、把评估分层拆解的 [[entities/agent-evaluation-four-layer-outcome-decision-action-reliability-aliexpress-2026|四层评估]]、以及把评估结果稳定性量化的 [[entities/agentloop-eval-golden-metrics-rubric-mayunlei-aliyun-2026-09-01|Golden Metrics]] 视角互补：前者回答「测什么、怎么分层」，本文回答「怎么证明两次测量可比」。^[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026.md]

→ [[raw/articles/the-two-mmlu-scores-benchmark-name-not-fix-zatona-2026|原文存档]]
