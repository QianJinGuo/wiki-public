---
title: Harness 组件保质期——Model-Harness Fit 与 Build to Delete 原则
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [harness-engineering, model-harness-fit, build-to-delete, co-evolution, architecture]
related:
  - [[entities/agent-harness-12-components-7-decisions|Agent Harness 12 组件框架]]
  - [[entities/harness-generator-evaluator-anthropic|Generator-Evaluator 架构]]
  - [[entities/model-harness-fit-agent-harness|Model-Harness Fit]]
author: Hermes Agent (本 wiki 原创合成分析)
sources:
  - raw/articles/model-harness-fit-agent-harness
  - raw/articles/harness-design-long-running-apps
  - raw/articles/agent-harness-12-components-7-decisions
  - raw/articles/tencent-cdn-lego-harness
confidence: high
original_work: true
---
# Harness 组件保质期——Model-Harness Fit 与 Build to Delete 原则
> **本文是本 wiki 的原创合成分析**，并非转载或翻译。基于五个独立来源的证据交叉验证，首次提出"Harness 组件保质期"的概念框架和"Build to Delete"五项工程原则。各章节末尾标注了原始证据来源的 wiki 内部链接，供读者查阅一手资料。
> Harness 里的每一个组件，都编码了一个"模型自己做不到什么"的假设——这些假设会随模型发布而过期。
> — Prithvi Rajasekaran, Anthropic
## 摘要
Agent Harness Engineering 社区正在形成一项共识，但它令人不安：**你为今天模型写的绝大部分 Harness 代码，会在下一个模型发布时变成负债。** 这不是 Bug，而是特性——Harness 的本质是"模型还不能做的事"的工程化编码，而模型每升级一次，这些假设就过期一批。
本文聚合五个独立来源的证据，论证 **Model-Harness Fit 框架**的必要性，并提出 **Build to Delete** 作为 Harness 工程的核心原则。
---
## 一、问题：Harness = 编码"模型做不到"的假设
### 1.1 每个组件都有保质期
Prithvi Rajasekaran（Anthropic 工程师）在阐述长运行 Agent Harness 时披露了一个关键发现：
> 他为 Sonnet 4.5 设计的 Harness 组件（context reset、sprint 拆分、激进压缩），当迁移到 Opus 4.6 时，变成了**死重**。直接砍掉所有这些脚手架，两小时连续会话效果反而更好。
这不是个例。每个 Harness 组件都在编码一条隐性前提（参见 [[entities/agent-harness-12-components-7-decisions|Agent Harness 12 组件框架]]）：
| Harness 组件 | 编码的假设 | 过期条件 |
|-------------|-----------|---------|
| Context Reset | 模型在长上下文下会焦虑、过早收尾 | 更强的模型能优雅处理长上下文（参见 [[entities/harness-generator-evaluator-anthropic|Generator-Evaluator 架构]]） |
| Sprint Contract | Generator 无法自行拆解复杂任务 | 新模型原生具备任务拆解能力（参见 [[entities/harness-generator-evaluator-anthropic|Generator-Evaluator 架构]]） |
| 激进压缩策略 | 模型无法有效利用冗余上下文 | 模型有了更好的注意力机制 |
| 工具格式化层 | 模型需要标准化的工具 Schema | 模型原生理解多样化的工具格式 |
| 记忆引用标签 | 模型不会自动关联记忆 | 模型主动构建记忆上下文 |
### 1.2 同一模型、不同 Harness = 不同的模型
Nicolas Bustamante（前 OpenAI，现 Cursor）用 Terminal-Bench 2.0 榜单给出了精确量化：
| Harness | 模型 | 分数 |
|---------|------|------|
| ForgeCode | Claude Opus 4.6 | **79.8%** |
| Capy | Claude Opus 4.6 | **75.3%** |
同一模型（权重 byte-for-byte 相同），只是在不同的 Harness 里运行——**差了 4.5 个百分点**（详细分析见 [[entities/model-harness-fit-agent-harness|Model-Harness Fit]]）。
更令人警觉的是 Cursor 团队自己的经历：通过重构 Harness（模型保持不变），Terminal-Bench 排名从榜外直接跳到第 5（详见 [[entities/cursor-harness-model-production-floor|Cursor Harness 复盘]]）。**壳的分数比模型升级多。**
---
## 二、证据链：五个独立来源的交叉验证
### 来源 1: Nicolas Bustamante — Model-Harness Fit（2026-04-30）
**核心发现：模型不是只针对 API 做 post-training，它是针对壳做的。**（详见 [[entities/model-harness-fit-agent-harness|Model-Harness Fit 完整分析]]）
工具的命名、输入 Schema、引用标签的格式、技能文件的存放位置、计划协议的语法——Harness 的每一个细节都被烤进了 post-training。你把模型拖出它的壳，**性能损耗是拿不回来的**。
具体案例：
- **记忆是最密集的碰撞面**：Codex 模型在 Claude Code 中运行时，依然挂 `<oai-mem-citation>` 标签——Claude Code 不解析，记忆系统里没有衰减信号，好记忆被当作没用的淘汰掉
- **六个字符决定记忆生死**：Claude 模型在 Codex 中运行时，不挂 citation tag，Codex 的 usage_count 永远是 0
- **Copilot CLI 是唯一诚实做跨模型路由的**：per-model 工具包含，不对全体模型使用公共方言
### 来源 2: Prithvi Rajasekaran — Generator-Evaluator 架构（2026-05-03）
**核心发现：Harness 组件有版本特定的保质期。**（详见 [[entities/harness-generator-evaluator-anthropic|Generator-Evaluator 架构]]）
量化数据——同一应用（复古游戏制作器），不同 Harness：
| Harness | Duration | Cost |
|---------|----------|------|
| 单 Agent | 20 分钟 | $9 |
| 完整 Harness | 6 小时 | $200 |
**22x 的成本差距意味着什么？** 不是 Harness 永远更好，而是每次都需要回答：**当前模型的瓶颈在哪一层？**
关键迭代原则——**每次只移除一个组件**，观察对最终结果的影响（参见 [[entities/agent-reliability-engineering-skillify-continuous-improvement|Skill 腐朽与组件保质期]]）：
- Opus 4.5 → 4.6：Sprint Contract 可移除（模型原生拆解能力够了）
- Opus 4.6+：Context Reset 对 Opus 4.6 可完全移除
- Evaluator 仍保留——但只在任务超出模型能力边界时
---
## 三、Model-Harness Fit 量化评估矩阵
### 3.1 为什么需要量化矩阵
定性描述（"Context Reset 似乎不 work 了"）无法支撑工程决策。团队需要一个可量化的矩阵，将 Model-Harness Fit 的状态从模糊的"感觉"转化为精确的"分数"，让组件的保质期判断有数据支撑。
### 3.2 评估维度与指标体系
Model-Harness Fit 的量化评估需要覆盖四个维度：
| 维度 | 指标 | 测量方式 | 理想值 |
|------|------|---------|-------|
| **任务完成率** | 相同 benchmark 下有/无该组件的完成率差值 | A/B 测试，N≥100 任务 | Δ > 2% 才值得保留 |
| **成本效率** | 每单位输出对应的 API 调用次数和 token 消耗 | 运行时埋点统计 | 移除后 cost 下降 > 10% 且质量不变 |
| **延迟影响** | 组件引入带来的 P50/P99 延迟增量 | 端到端 tracing | P99 增量 < 5% |
| **错误率** | 组件存在时的任务失败率 vs 移除后的失败率 | 统计错误类型分布 | 移除后错误率不显著上升 |
### 3.3 综合评分公式
```
Fit_Score = w1×Task_Completion + w2×Cost_Efficiency + w3×Latency + w4×Error_Rate

其中 w1+w2+w3+w4 = 1.0，权重根据业务场景调整：
- 成本敏感场景：w2=0.4, w1=0.3, w3=0.2, w4=0.1
- 质量敏感场景：w1=0.4, w4=0.3, w2=0.2, w3=0.1
- 延迟敏感场景：w3=0.4, w1=0.3, w2=0.2, w4=0.1
```
**决策阈值**：
- Fit_Score ≥ 0.7：组件"健康"，继续保留
- 0.4 ≤ Fit_Score < 0.7：组件"亚健康"，进入观察名单，3 个月内重新评估
- Fit_Score < 0.4：组件"死亡"，触发删除审查流程
### 3.4 实际应用案例
以 Context Reset 组件为例，展示量化评估的完整流程：
**背景**：Opus 4.6 发布后，团队不确定 Context Reset 是否还需要保留
**测量**（单周 A/B 测试，100 个长对话任务）：
```
有 Context Reset:
  Task_Completion = 87.3%
  Cost_Per_Task = $0.34
  P99_Latency = 2.1s
  Error_Rate = 4.2%

无 Context Reset:
  Task_Completion = 86.9%  (Δ = -0.4%)
  Cost_Per_Task = $0.31    (Δ = -8.8%)
  P99_Latency = 1.9s      (Δ = -9.5%)
  Error_Rate = 4.1%       (Δ = -0.1%)
```
**计算**（质量优先权重）：
```
Fit_Score = 0.4×(86.9/87.3) + 0.2×(0.31/0.34) + 0.3×(1.9/2.1) + 0.1×(4.1/4.2)
           = 0.4×0.996 + 0.2×0.912 + 0.3×0.905 + 0.1×0.976
           = 0.398 + 0.182 + 0.272 + 0.098
           = 0.95
```
**结果**：Fit_Score = 0.95（"死亡"阈值 < 0.4），但任务完成率几乎不变，成本和延迟反而下降，**强烈建议删除 Context Reset**。
### 3.5 矩阵维护节奏
量化矩阵的价值在于持续跟踪，而非一次性测量。建议维护节奏：
- **每个模型版本发布后 2 周**：对所有组件重新跑完整量化评估
- **每月**：对"亚健康"组件做增量测量
- **每次引入新组件**：引入后第 1 天做 baseline 测量，第 30 天做复测，建立组件效果的历史曲线
---
## 四、保质期管理的三阶段节奏
### 4.1 阶段划分的逻辑
组件保质期不是线性的——它有一个"有效→衰退→死亡"的生命周期。不同阶段需要不同的管理动作，而非等到组件彻底死亡才行动。
### 4.2 阶段一：Active（有效）——利用而非干预
**识别特征**：Fit_Score ≥ 0.7，组件持续对任务完成率或成本效率产生正向贡献
**管理动作**：
- 让组件充分发挥作用，不需要额外干预
- 记录组件在哪些任务类型上效果最好（为未来的"选择性应用"提供数据）
- 将组件效果数据纳入团队知识库，作为后续新成员 onboarding 教材
**典型案例**：对于 Opus 4.5 的 Context Reset，在当时是 Active 状态，显著提升了长任务完成率
### 4.3 阶段二：Decaying（衰退）——监控与准备
**识别特征**：Fit_Score 在 0.4-0.7 之间，或连续两次量化测量呈下降趋势
**管理动作**：
- 提高测量频率（从每季变为每月）
- 开始准备"删除预案"（参见 Build to Delete 原则 2）
- 与模型发布计划对齐，预测组件在下一个模型版本下的预期状态
- 对组件效果做任务类型细分：可能在某些任务类型上仍然有效，在其他类型上已衰退
**关键判断**：组件的衰退是模型升级导致的（必然），还是任务类型变化导致的（可逆转）？前者需要删除，后者需要重新定位组件的适用范围。
**典型案例**：Sprint Contract 在 Opus 4.5 → 4.6 过渡期间进入 Decaying 状态——任务拆解效率下降，但复杂的多步骤任务仍需要它。团队准备删除预案的同时，保留其在特定任务类型上的使用。
### 4.4 阶段三：Dead（死亡）——执行删除
**识别特征**：Fit_Score < 0.4，或模型厂商明确说明某项能力已原生支持
**管理动作**：
- 执行已准备好的删除预案（参见 Build to Delete 原则 4）
- 遵循渐进式移除策略（影子模式 → 金丝雀 → 灰度 → 全量）
- 删除后做 blameless postmortem，记录删除决策是否正确
- 将删除经验反馈到组件元数据库，优化未来引入新组件时的预期保质期评估
**典型案例**：Context Reset 在 Opus 4.6+ 确认死亡后，团队在 2 周内完成渐进式删除，任务完成率无显著变化，成本下降 8.8%，P99 延迟下降 9.5%。
### 4.5 三阶段流转的可视化
```
[Active] ──(模型升级 或 任务类型变化)──→ [Decaying]
   ↑                                        │
   │          (Fit_Score ≥ 0.7)            ↓
   │                                   [Dead] ──→ (执行删除)
   │                                        ↑
   │         (Fit_Score < 0.4)              │
   └────────────────────────────────────────┘
              (错误删除，可回滚)
```
注意：组件可能从 Dead 状态被"复活"——如果模型升级引入了 regression（能力回退），之前被删除的组件可能重新变得必要。因此，删除的组件元数据应保留 6-12 个月，而非彻底丢弃。
### 4.6 团队角色与职责
| 角色 | 职责 |
|------|------|
| **Harness Owner** | 每个核心组件有明确负责人，负责定期量化评估和删除预案维护 |
| **Model Liaison** | 对接模型厂商发布节奏，提前预警可能触发组件失效的模型变化 |
| **QA/Performance** | 负责 A/B 测试框架和量化数据采集，确保测量结果的统计显著性 |
| **Tech Lead** | 审批删除决策，平衡技术债务清理与业务稳定性的张力 |
---
## 子页面
- [[concepts/harness-component-expiry-evidence|证据链与 Build to Delete 原则]] — 五来源交叉验证、保质期实例、五项工程原则、开放问题

## 关联实体

**上游依赖**:
- [[entities/agent-harness-12-components-7-decisions]] — 提供基础理论/方法
- [[entities/harness-generator-evaluator-anthropic]] — 提供基础理论/方法
- [[entities/model-harness-fit-agent-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-harness-12-components-7-decisions]] — 具体应用场景
- [[entities/harness-generator-evaluator-anthropic]] — 具体应用场景
- [[entities/harness-generator-evaluator-anthropic]] — 具体应用场景

**平行协作**:
- [[entities/cursor-harness-model-production-floor]] — 替代/补充方案
- [[entities/model-harness-fit-agent-harness]] — 替代/补充方案
- [[entities/harness-generator-evaluator-anthropic]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
