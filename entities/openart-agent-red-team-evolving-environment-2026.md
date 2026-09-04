---
title: "OpenART Arena：长程 Agent 红队评测的环境演化"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [agent, security, eval, red-team, harness-engineering, safety, benchmark]
sources: [raw/articles/openart-agent-red-team-evolving-environment-2026]
confidence: 0.7
provenance_state: extracted
---

# OpenART Arena：长程 Agent 红队评测的环境演化

> 来源：PaperWeekly（让你更懂AI的）。论文《OpenART Arena: Scaling Agent Red Teaming via Open-Ended Environment Evolution》，复旦 + 上海AI实验室 + XSafeAI，代码开源（github.com/AI45Lab/OpenART），arXiv 2608.00677。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 核心贡献：从静态测试到环境演化

OpenART 把 Agent 红队评测从「固定环境中的一次测试」升级为「持续演化环境中的系统级评测」。它解决三个问题：①怎样构造足够长、能真实执行的测试场景；②怎样让同一场景适配不同 Agent Harness；③怎样在任务目标不变的前提下持续演化环境并寻找新的风险状态。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

Agent 进入真实工作流后，安全问题沿多步执行逐渐显现——文件、工具结果、权限、记忆、计划状态会在后续步骤被反复读取/修改，早期变化沿长程执行传播，很多步后才表现为安全失效。现有基准多基于固定或可重置环境，难以覆盖状态累积、跨步骤传播、不同 Harness 对安全结果的影响。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 三支柱方案

- **长程可执行场景**：从 50 万+ Tools/MCPs/Skills 能力库出发，在 50 个应用领域构造 1 万+ 有状态场景；每场景需执行检查和 Evaluator 验证后入测集，工具调用次数中位数达 97。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- **target-agnostic 描述 + 轻量 Adapter**：先用目标无关方式描述场景，再经 Adapter 把任务目标/安全约束/Evaluator 映射到各 Agent 原生 Harness；覆盖 15 个已部署 Agent × 5 个基础模型 = 75 种配置，可直接比较基础模型、Agent 实现与接口差异的影响。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- **固定任务、环境持续演化**：每轮从当前环境+历史攻击状态出发，策略生成候选环境变化，经目标 Agent 的 Adapter 检查只保留该 Harness 支持且允许修改的 state surfaces。支持 Workspace / Instructions / Skills / Tools / MCPs / Short-Term Memory / Plan State / Long-Term Memory 八类 state surfaces。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## Evolutionary Markov Hypergraph Attack（EMHA）

EMHA 作为参考搜索方法，采用 black-box 反馈，target 与 attacker 模型参数固定，适应过程发生在外部 attacker state（保存历史反馈、路径价值、不断演化的 graph pool）。用 Hypergraph 表示需要协同发生的环境变化：vertex = attack subgoal，hyperedge 连接其前置条件与后续 subgoals；逐步采样形成路径，由冻结的 attacker model 转成实际环境更新，一次演化可同时协调多个 state surfaces 并把上一轮反馈带入下一轮。higher Evaluator 分数增益会分配到该轮路径 hyperedges，Archive 保留各搜索区的表现较好 graph 供继续演化。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 关键发现

- 75 种配置上 pooled Strict ASR 达 85.0%；DeepSeek-V4-Pro 五轮演化中累计 Strict ASR 从第一轮 42.9% 升至第五轮 94.7%——环境持续变化能暴露初始测试未发现的问题。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- 场景越复杂，完整环境演化 vs 仅演化指令的差距越大：依赖深度最大差 17.6pp，工具调用最大差 17.2pp。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- 风险延迟明显：1 万条轨迹中，变化状态首次被使用的中位执行进度约 23%，首次不安全行为出现在 64%，两者中位相隔 37 个目标动作（≈41% 执行流程延迟）。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]
- 控制基础模型与正常任务完成度后，加入目标 Agent 身份可额外解释 7.6% 攻击成功率变化——**Agent 安全不仅取决于基础模型，也与 Harness 设计、状态组织、接口方式相关**。^[raw/articles/openart-agent-red-team-evolving-environment-2026.md]

## 相关实体

- [[entities/agent-harness-engineering-survey-2026|Agent Harness Engineering 综述]] — Harness 对系统行为的影响是 OpenART「Agent 身份解释 7.6%」的工程背景
- [[entities/agent-evaluation-survey-ibm-yale-2026|Agent 评测综述（IBM/Yale）]]
- [[entities/agent-security-three-step-sequence-harness-governance-identity-crewai|Agent 安全三步序列（CrewAI）]]
- [[entities/agent-evaluation-four-layer-outcome-decision-action-reliability-aliexpress-2026|Agent 评测四层可靠框架]]
- [[entities/agent-evaluation-turing-meituan-2026|美团 Turing Agent 评测]]

→ [[raw/articles/openart-agent-red-team-evolving-environment-2026|原文存档]]