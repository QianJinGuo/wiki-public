---
title: "SHE 与 SafeEvolve：Agent 安全 Harness–Policy 协同进化（上海AI Lab）"
created: 2026-10-15
updated: 2026-10-15
type: entity
tags: [agent, harness, safety, post-training, self-evolution, sft, rl]
sources: [raw/articles/上海ai-lab连发两项研究agent安全也能自我进化]
confidence: 0.8
provenance_state: extracted
---

# SHE 与 SafeEvolve：Agent 安全 Harness–Policy 协同进化

> **Background**：本文档基于上海人工智能实验室联合复旦、上交、港科大、浙大团队先后提出的 SHE（arXiv 2608.09885）与 SafeEvolve（arXiv 2609.02786）两项工作的微信公众号解读整理，聚焦"Agent 安全机制如何从执行失败中持续学习"这一技术路线。

## 核心问题：从静态规则到可进化安全机制

传统 Agent Harness 的安全配置（System Prompt、拦截规则、工具权限）是部署前一次性人工设计，进入开放环境后无法覆盖新网页、新文件、新工具组合暴露的风险；而一次安全失败往往是模型、Harness、工具、记忆与外部环境共同作用的结果，整体改 Prompt 难以定位修复效果、堆拒答规则损害任务能力、重训模型又难以及时进入运行时控制。SHE 与 SafeEvolve 分别从 Harness 与 Policy 两层给出持续改进路径。→ [[raw/articles/上海ai-lab连发两项研究agent安全也能自我进化|原文存档]]

## SHE：轨迹驱动的安全 Harness 演化（arXiv 2608.09885）

- **四类可编辑安全组件**：System Prompt（全局安全行为）/ Rule Bank（结构化可复用规则）/ Safety Memory（反复出现但未解决的经验）/ Tool Policy（工具权限与执行边界）——组件化解决安全功能耦合，不同失败路由到不同组件，局部修复不重写整个机制。
- **归因引导的局部演化**：每轮收集完整执行轨迹（上下文、模型响应、工具调用、环境反馈、任务结果），从危害领域、攻击面、失败模式三角度结构化诊断"风险在哪一步出现 / 哪个组件负责 / 如何修改"，只对相关组件局部更新；每个候选更新需通过有效性检查与安全—效用验证，禁止用无差别拒绝换表面安全。
- **细粒度安全边界示例**：应用推荐场景不禁止所有应用任务，而是提炼为"可介绍比较、可给官方下载指引、缺授权不做设备级安装、不把未完成操作说成已完成"。
- **实验**：对比静态 SafeHarness，平均攻击成功率 17.1%→5.5%（约 1/3），攻击条件任务可用性 31.6%→47.6%，未参与演化的 AgentHarm 上危害得分 19.8%→9.8%；学到的安全边界可跨 Agent 模型迁移，无需逐模型重新演化。→ [[raw/articles/上海ai-lab连发两项研究agent安全也能自我进化|原文存档]]

## SafeEvolve：Harness–Policy 协同进化（arXiv 2609.02786）

- **Harness 侧**：从轨迹提炼 Safety Prompt + 分层 SkillBank（通用安全原则 + 任务/工具/失败模式相关的安全操作流程）。
- **Policy 侧两阶段 SFT–RL**：第一阶段 Harness-use SFT 冷启动——在进化 Harness 指导下筛选同时通过安全性与效用验证的成功样本，让模型学会"何时调用哪个安全 Skill、识别不可信内容、区分用户目标与外部诱导、权限不明时暂停确认"；第二阶段 Harness-augmented RL——Verifier 分别检查任务完成、危险行为、工具调用有效性，训练目标是情境化行动（正常任务继续完成 / 有害请求安全拒绝 / 环境注入忽略恶意内容继续原目标 / 权限不明先确认）。
- **闭环结构**：Rollout 产生经验 → Harness 外化经验（可审计、可修改、可回滚的资产）→ SFT 帮 Policy 使用经验 → RL 推动内化 → 新 Policy 产生 On-policy 轨迹反哺 Harness 发现覆盖盲区。
- **实验**：Qwen3.5-4B 上 AgentDojo 攻击成功率 2.37%→0.79%、干净任务效用 59.79%→61.86%；AgentHarm 危害得分 56.45→12.27、安全拒答率 28.98%→83.83%；Qwen3-4B 上 AgentDojo 攻击成功率 13.38%→2.42%、任务效用 44.33%→60.82%。→ [[raw/articles/上海ai-lab连发两项研究agent安全也能自我进化|原文存档]]

## 与 wiki 既有 Harness 论述的关系

- 与 [[concepts/harness-engineering-framework]] 的组件化分解相呼应，但本文聚焦安全子集：将 Harness 拆为 System Prompt / Rule Bank / Safety Memory / Tool Policy 四类**可编辑**组件并做归因路由。
- 与 [[entities/agent-harness-architecture]] 的"外围控制层"定位互补：SHE/SafeEvolve 论证 Harness 不应只是静态防护层，而应从执行失败中持续学习。
- 与 [[concepts/ai-safety]] 的静态 guardrails 视角形成演进关系：安全从"一次配置、长期使用"转向"边运行、边学习、边修复"。
- 对 [[concepts/verifier-driven-development]] 的启示：SafeEvolve 的 RL 阶段用 Verifier 分别校验任务完成/危险行为/工具有效性，与"验证器驱动"的训练范式一致。
- 设计模式汇总见 [[queries/ai-agent-safety-guardrails-design-patterns]]。

## 关联

- [[concepts/harness-engineering]] — Harness 工程总纲
- [[entities/agent-harness-12-components-7-decisions]] — Harness 组件划分的另一框架
- 代码：SHE `github.com/RainbowQTT/SHE`；SafeEvolve `github.com/MaoPopovich/SafeEvolve`
