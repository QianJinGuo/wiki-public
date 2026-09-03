---
title: "Evaluation & Benchmarks 主题地图 (MOC)"
created: 2026-06-15
updated: 2026-08-29
type: moc
tags: [moc, evaluation, benchmark, verifier, agent-eval, methodology, 2026]
sources: []
---

# Evaluation & Benchmarks 主题地图 (MOC)

> "传统计算机容易自动化你能写进代码的东西，这一代 LLM 更容易自动化你能验证的东西。" — Karpathy, 2026

Agent 和 LLM 的可靠性不在模型能力本身，而在你能不能**系统地、可重复地验证**它的输出。本 MOC 收口评测方法论、基准工具和验证框架三大方向。

---

## 核心方法论

- [[entities/ai-evals-methodology|AI Evals 方法论 — 评测的三种方法（自动/人工/A-B）及各自适用场景
- [[entities/evals-three-methods-of-ai-evaluation|Evals 到底在评什么 — Langfuse Lotte Verheyden 的三方法拆解
- [[entities/llm-as-a-verifier-a-general-purpose-verification-framework|LLM-as-a-Verifier — Stanford/Berkeley/NVIDIA 联合框架，20 字母 token 细粒度验证，平局率 27%→0%
- [[entities/better-harness-eval-trace-methodology|Harness Eval Trace 方法论 — Agent harness 级评测的追踪方法论
- [[entities/anthropic|Anthropic 解构 Agent Evals — 官方视角的 Agent 评测方法论
- [[entities/spotify-llm-evals-funnel-not-fork|LLM Evals: Funnel not Fork — Spotify 实践：eval 是漏斗不是分叉，42% 上线实验因次级指标回滚
- [[concepts/eval-optimizer-firewall|评测防火墙 — eval 与 optimizer 的信息/执行隔离：dev 轨迹隐藏、tests 不进镜像+拒评语义、oracle-zero 曲线、确定性检查先行]]

## Agent 评测工具与框架

- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|Agent EvalKit — AWS 开源 CLI Agent 评测工具
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation|YAML 驱动的 Agent 评测框架 — WalleZhang 声明式评测
- [[entities/claw-swe-bench-harness-evaluation-benchmark-tokenrhythm|CLAW SWE-Bench — 首个 harness 影响评测基准，8 语言 350 任务
- [[entities/claude-code-performance-benchmarking|Claude Code 性能基准测试 — 编码 Agent 性能量化
- [[entities/build-custom-code-based-evaluators-in-amazon-bedrock-agentco|Bedrock 自定义评测器 — AWS 托管 Agent 评测构建
- [[entities/agent-memory-evaluation-landscape-taobao-survey|Agent 记忆评测全景 — 淘宝团队记忆系统评测调研

## LLM 通用评测

- [[entities/ai-job-interview-model-evaluation-mollick|Mollick AI 面试评测 — 用面试题做 LLM 评测
- [[entities/adversarial-verification|对抗验证 — 红队/对抗测试方法论
- [[entities/aws-reinforcement-fine-tuning-llm-as-judge|LLM-as-Judge 微调 — AWS 强化微调 + LLM 评委
- [[entities/eva-bench-data-2-voice-agent|EVA-Bench Data 2.0 — 语音 Agent 评测基准
- [[entities/ai-traffic-cyberthreat-benchmark-2026|AI 流量网络威胁基准 2026 — 安全领域专项基准

## 验证 vs 评测：关键区分

| 维度 | Evaluation（评测） | Verification（验证） |
|------|-------------------|---------------------|
| 目的 | 比较模型/方案优劣 | 确认单个输出是否正确 |
| 基准 | 需要标注数据集 | 规则/断言/形式化描述 |
| 适用阶段 | 选型/迭代 | 生产部署 |
| 代表工具 | SWE-bench, Evals | Verifier, Test-time compute |

评测告诉你"哪个更好"，验证告诉你"这个对不对"。两者互补，不能互相替代——Spotify 的 42% 回滚数据就是证据。

---

## 与相邻 MOC 的关系

- [[moc/loop-engineering|Loop Engineering]] — 验证是 Loop 闭环的核心组件
- [[moc/anthropic-ecosystem|Anthropic 生态]] — Anthropic 的 Generator-Evaluator 架构
- [[moc/ai-skill-design|Skill 设计]] — Skill 评测（writing evaluation）是质量保障
- [[moc/amazon-aws-ai|AWS AI]] — Bedrock AgentCore 内置评测能力

---

_本 MOC 于 2026-06-15 创建，覆盖评测方法论 + 工具框架 + 专项基准。_

## 待关联概念

- [[concepts/scientific-method-ai-research|AI 研究的科学方法论]]

## 待建条目（2026-08 phd 透镜涌现的评测盲区）

以下两个评测维度在现有图谱（含 /phd 合集七系统自评）中均为空白，待立条目：

- **方向选择决策质量**：假设树的剪枝/合并决策对错率（Arbor 的 DECIDE 步、Idea Forge 打分）——产物分数无法恢复剪错分支的损失；faibench loop16 用「历史最佳轮不剪枝」实质回避了此问题
- **人-idea 匹配**：idea 可行性 × 研究者能力/每周可用小时数/算力预算的匹配度——Supervisor-Skills idea-evaluator 的独有维度，无 benchmark 跟进；IG-Bench 只评 idea 绝对质量

来源：[[drafts/wiki-emergent-viewpoints-2026-08-phd-lens|2026-08 涌现观点·观点六]]
