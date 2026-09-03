---
title: Build to Delete 工程原则与开放问题
created: 2026-05-13
updated: 2026-08-01
type: concept
tags: [harness-engineering, build-to-delete, engineering-principles, open-problems, model-harness-fit]
sources:
  - raw/articles/model-harness-fit-agent-harness
  - raw/articles/harness-design-long-running-apps
  - raw/articles/agent-harness-12-components-7-decisions
  - raw/articles/tencent-cdn-lego-harness
related:
  - [[concepts/harness-component-expiry-and-build-to-delete|Harness 组件保质期总览]]
  - [[concepts/harness-component-expiry-evidence|证据链与 Build to Delete 原则]]
---
# Build to Delete 工程原则与开放问题

| Harness | 模型 | 分数 |
|---------|------|------|
| ForgeCode | Claude Opus 4.6 | **79.8%** |
| Capy | Claude Opus 4.6 | **75.3%** |
同一模型（权重 byte-for-byte 相同），只是在不同的 Harness 里运行——**差了 4.5 个百分点**。
更令人警觉的是 Cursor 团队自己的经历：通过重构 Harness（模型保持不变），Terminal-Bench 排名从榜外直接跳到第 5。**壳的分数比模型升级多。**
---
## 二、证据链：五个独立来源的交叉验证
### 来源 1: Nicolas Bustamante — Model-Harness Fit（2026-04-30）
## 关联实体

**上游依赖**:
- [[entities/model-harness-fit-agent-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/harness-generator-evaluator-anthropic]] — 具体应用场景
- [[entities/agent-harness-12-components-7-decisions]] — 具体应用场景

**平行协作**:
- [[entities/harness-engineering-systematic-framework]] — 替代/补充方案
- [[entities/tencent-cdn-lego-harness]] — 替代/补充方案


→ [[entities/model-harness-fit-agent-harness|Model-Harness Fit 实体页]]
**核心发现：模型不是只针对 API 做 post-training，它是针对壳做的。**
工具的命名、输入 Schema、引用标签的格式、技能文件的存放位置、计划协议的语法——Harness 的每一个细节都被烤进了 post-training。你把模型拖出它的壳，**性能损耗是拿不回来的**。
具体案例：
- **记忆是最密集的碰撞面**：Codex 模型在 Claude Code 中运行时，依然挂 `<oai-mem-citation>` 标签——Claude Code 不解析，记忆系统里没有衰减信号，好记忆被当作没用的淘汰掉
- **六个字符决定记忆生死**：Claude 模型在 Codex 中运行时，不挂 citation tag，Codex 的 usage_count 永远是 0
- **Copilot CLI 是唯一诚实做跨模型路由的**：per-model 工具包含，不对全体模型使用公共方言
### 来源 2: Prithvi Rajasekaran — Generator-Evaluator 架构（2026-05-03）
→ [[entities/harness-generator-evaluator-anthropic|Generator-Evaluator 架构实体页]]
**核心发现：Harness 组件有版本特定的保质期。**
量化数据——同一应用（复古游戏制作器），不同 Harness：
| Harness | Duration | Cost |
|---------|----------|------|
| 单 Agent | 20 分钟 | $9 |
| 完整 Harness | 6 小时 | $200 |
**22x 的成本差距意味着什么？** 不是 Harness 永远更好，而是每次都需要回答：**当前模型的瓶颈在哪一层？**
关键迭代原则——**每次只移除一个组件**，观察对最终结果的影响：
- Opus 4.5 → 4.6：Sprint Contract 可移除（模型原生拆解能力够了）
- Opus 4.6+：Context Reset 对 Opus 4.6 可完全移除
- Evaluator 仍保留——但只在任务超出模型能力边界时
### 来源 3: 12 组件框架（2026-04-20）
→ [[entities/agent-harness-12-components-7-decisions|12 组件框架实体页]]
**核心发现：Harness 的脚手架隐喻——脚手架不盖楼，但工人够不到更高的楼层。**
量化数据：
- **TerminalBench**：只改 Harness，从榜外→第 5
- **砍掉 80% 的工具，性能反而提升**——工具太多了模型不知道选哪个
- **Harness 才是产品**：相同模型 + 不同 Harness = 完全不同体验
12 个组件中，哪些有最短保质期？来自框架本身的判断：**越"厚"的组件（sprint contract、context reset、激进压缩），保质期越短。**
### 来源 4: 系统性框架 — 1.6% vs 98.4%（2026-04-30）
→ [[entities/harness-engineering-systematic-framework|Harness 工程系统性框架实体页]]
**核心发现：Claude Code 51.2 万行 TypeScript 源码中，只有 1.6% 是 AI 决策逻辑，98.4% 是确定性工程基础设施。**
但关键问题是：**这 98.4% 的基础设施会随模型升级而贬值。**
LangChain 的验证提供了对照：调整了 Harness（系统提示、工具、中间件、推理模式），模型未换，Terminal Bench 从 52.8→66.5（+13.7 分）。
### 来源 5: 腾讯 CDN LEGO Harness（2026-04-28）
→ [[entities/tencent-cdn-lego-harness|腾讯 CDN LEGO Harness 实体页]]
→ [[concepts/harness-component-expiry-evidence|返回证据链]] | [[concepts/harness-component-expiry-and-build-to-delete|返回总览]]
---
## 三、Build to Delete 五项工程原则
基于五个独立来源的交叉验证，推导出以下五项工程原则：
### 原则 1：最低模型版本标注
每一个 Harness 组件必须显式标注其所依赖的最低模型版本。组件的"保质期"从该版本开始，到下一个引入breaking change的模型版本为止。
> 适用场景：新组件引入时强制标注；模型升级前先扫描所有标注了目标版本之前的组件
### 原则 2：每发布一个模型版本，跑一次 Harness 审计
每次模型升级后，团队应系统性审查现有 Harness 组件：哪些假设已经过期？哪些组件变成了死重？原则是**每次只移除一个组件**，观察对最终结果的影响。
> 迭代检查清单：Context Reset → Sprint Contract → 激进压缩 → 工具格式化层 → 记忆引用标签
### 原则 3：组件保质期三级分类
| 等级 | 典型组件 | 保质期特征 |
|------|---------|-----------|
| **短（Model-Version 级）** | Context Reset、Sprint Contract、激进压缩 | 随模型小版本升级即失效 |
| **中（Architecture 级）** | 工具路由层、记忆引用标签 | 跨1-2个大版本有效 |
| **长（Product 级）** | 核心循环机制、护栏 | 基本不随模型变化 |
> 越"厚"的组件保质期越短
### 原则 4：移除比保留更需要证据
默认状态从"保留"反转为"移除"——如果一个组件在当前模型版本下没有产生可测量的正向效果，它应当被移除，而不是保留"以防万一"。
> 核心转变： Retention 必须被 justify，removal 是新默认
### 原则 5：Build to Delete 是组织原则，不只是技术原则
当一个组件因模型升级而不再需要，但团队已经依赖它来维持审查能力时，移除它带来的风险不仅仅在技术层面——它影响团队的能力建设和审查文化。
> 腾讯 CDN LEGO 的36%误报率数据表明：AI"自信"会降低人审查的意愿
---
## 四、三项开放问题
### 开放问题 1：检测成本——如何以低成本发现"死了但没被删除"的组件？
目前的验证方式是手动审计，成本极高。是否存在一种可规模化的检测方法？
- **问题核心**：组件的"死亡"往往是静默的——它不再起作用，但不会报错
- **初步思路**：对比启用/禁用该组件时的任务完成率 + 成本，发现率 > 某个阈值即触发删除审查
- **待解决**：阈值设定、检测频率、与 CI/CD 的集成方式
### 开放问题 2：Build to Delete 与团队能力萎缩之间的张力
腾讯 CDN LEGO Harness 案例揭示了一个深层矛盾：当 AI 做大部分工作时，人的审查意愿和能力都会下降。如果一个组件的移除让团队失去了最后一道人工审查屏障，那么这个组件可能既是 Harness 的过期组件，也是团队能力萎缩的症状。
- **问题核心**：移除"过期"组件可能加速团队能力的系统性退化
- **待解决**：如何在技术删除和组织保留之间找到动态平衡？
### 开放问题 3：Harness 与模型共同进化时的双向折旧
目前的分析都假设 Harness 随模型升级而贬值。但是否存在反向情况——某些 Harness 组件在模型升级后反而变得更有价值？
- **待解决**：寻找反例（counterexamples）——升级后价值增加的 Harness 组件将帮助我们理解共同进化的动态机制
---
## 五、实践指南：如何在真实团队中落地 Build to Delete
### 5.1 启动审计：从"组件清单"开始
在开始删除之前，团队需要一个完整的组件清单。这不是简单的文件列表，而是每个组件的以下元数据：
```
Component: context_reset_v2
Type: Short-expiry (Model-Version level)
Introduced: Opus 4.4
Hypothesis_Encoded: "Model loses coherence after 50k tokens"
Current_Effect: [measured: +2.1% task completion, +18% cost]
Expiry_Trigger: Opus 4.6+ with native long-context optimization
Annotated_By: @engineer_name
Last_Reviewed: 2026-04-15
```
**第一步行动**：在下一季度规划中预留一个"组件审计冲刺"（Component Audit Sprint），为期 1-2 周，专门做历史组件的版本标注和效果测量。
### 5.2 建立"删除预案"流程
每一次引入新 Harness 组件时，同时提交一个对应的"删除预案"（Delete预案）：
1. **触发条件**：组件在哪个模型版本下预期失效？
2. **删除信号**：哪些可量化的指标变化表明组件已死亡？
3. **删除步骤**：移除该组件需要几步？涉及哪些配置文件？
4. **回滚计划**：如果删除后效果变差，如何在 24 小时内恢复？
这个预案应该在组件引入 PR 中就作为必填项，而非事后补充。
### 5.3 渐进式移除策略
原则 2 强调"每次只移除一个组件"，但实际操作中需要更精细的节奏控制：
| 阶段 | 操作 | 观察窗口 | 通过条件 |
|------|------|---------|---------|
| 影子模式 | 组件保留但日志标记"疑似过期" | 1-2 周 | 无异常反弹 |
| 金丝雀释放 | 5% 流量切换到无组件版本 | 3-5 天 | P99 延迟 < 阈值 |
| 灰度推广 | 50% 流量 | 1 周 | 错误率持平 |
| 全量上线 | 100% 流量 | 持续监控 | 2 周无异常则正式删除 |
### 5.4 文化阻力：如何说服团队接受"删除"
Build to Delete 最大的阻力不是技术，而是心理：**工程师不愿意删除自己写的代码**，因为这意味着承认"当初的判断是错的"。
实用策略：
- **不要惩罚**：将组件过期视为正常的技术债务清理，而非失误
- **量化成就**：把"本季度删除了 X 个过期组件"纳入工程成就，而非只衡量"引入了什么"
- **公开复盘**：每次删除后做 blameless postmortem，总结删除是否正确，为后续决策提供数据
### 5.5 建立"组件年龄"可视化仪表盘
让过期组件的风险可见，是推动删除的关键。可在内部 Dashboard 中增加以下视图：
- **组件年龄热力图**：X 轴=模型版本，Y 轴=Harness 组件，颜色=效果评分（绿色=高效，红色=衰退）
- **过期风险警报**：当某组件的编码假设与新模型发布说明重叠时，自动触发 Slack 通知
- **删除ROI累计图**：展示每次删除后的成本节省和性能变化
---
## 六、与 CI/CD 集成的自动化保质期检测机制
### 6.1 检测框架设计
自动化检测的核心挑战是：组件的"死亡"是静默的——它不再起作用，但系统不会报错。传统的 CI/CD 断言（单元测试、集成测试）无法发现这种"隐性失效"。
设计思路：**用影子对比（Shadow Comparison）来检测隐性死亡**。
```
CI Pipeline:
  1. Baseline: 记录当前组件集的任务完成率基线 B_base
  2. Shadow Run: 在 staging 环境临时禁用组件 C，运行相同测试集
  3. Comparison: 比较 B_base 与 B_shadow 的差值 Δ
  4. Decision:
     - Δ < 0.5%: 组件 C"死亡"，触发删除审查
     - Δ > 2%: 组件 C 仍有显著正向效果，保留
     - 0.5% < Δ < 2%: 不确定，继续观察
```
### 6.2 与模型发布节奏同步
模型厂商的发布节奏是可预期的（Anthropic 大约每季度一个大版本），CI/CD 流程应与这个节奏同步：
| 时间节点 | CI/CD 动作 |
|---------|-----------|
| 模型发布前 2 周 | 扫描所有组件的编码假设，与新版本发布说明做交叉比对，标记"可能过期"组件 |
| 模型发布日 | 启动影子对比测试套件，收集 48 小时 baseline 数据 |
| 模型发布后 1 周 | 执行"可能过期"组件的影子对比，生成删除建议报告 |
| 模型发布后 1 个月 | 对已执行删除的组件做效果复盘，更新组件元数据库 |
### 6.3 工具链推荐
以下是实践中被验证有效的工具链组合：
- **测试框架**：基于 Playwright 或 Puppeteer 的 E2E 测试套件，覆盖核心用户流程
- **A/B 流量分配**：使用 Feature Flag 服务（LaunchDarkly、Unleash）做细粒度流量控制
- **数据收集**：Prometheus + Grafana 采集任务完成率、延迟、错误率；与 CI 结果关联存储
- **告警**：当影子对比差值进入不确定区间时，通过 PagerDuty 或 Slack 通知 Harness 负责人
- **元数据管理**：用 JSON Schema 定义组件元数据，与 Harness 源码放在同一仓库，版本化管理
### 6.4 误报率控制
影子对比不是完美的，存在两种误报风险：
1. **统计噪声**：测试集本身的不稳定导致误判
2. **环境偏差**：staging 与生产环境的差异导致测试结果不可靠

缓解策略：
- **多次运行**：每个组件的影子对比至少运行 3 次，以中位数而非单次结果做决策
- **分层测试**：单元测试层（快速，但不可靠）+ 集成测试层（中速，较可靠）+ E2E 测试层（慢速，最可靠），三层结果综合判断
- **人工复核**：所有"建议删除"结论都需要 Harness 负责人确认后才能执行，防止自动化误判导致线上故障
### 6.5 实施路线图
| 阶段 | 时间 | 目标 |
|------|------|------|
| Phase 0（手动阶段） | 第 1-2 月 | 纯手工组件审计，建立元数据库，不需要自动化 |
| Phase 1（半自动） | 第 3-4 月 | 引入影子对比脚本，由工程师触发运行，结果需人工确认 |
| Phase 2（自动化） | 第 5-6 月 | CI/CD 流程集成影子对比，结果自动生成 PR，工程师审查后合并 |
| Phase 3（智能化） | 第 7-12 月 | 基于历史数据训练简单模型，预测组件过期时间，从"检测死亡"升级为"预防性删除" |
---
## 七、结论
Build to Delete 不是"不建"，而是"建了就要计划好怎么删"。在模型以季度为周期升级的世界里，大部分工程成果的生命周期比人的职业周期短得多。
五项工程原则的核心洞察：**Harness 组件编码了"模型当前做不到什么"的假设——随着模型能力的提升，这些假设会一个接一个地失效，过期组件如果不清除，就会从助力变成阻力。**
实践指南和 CI/CD 集成机制将五项原则从抽象理念落地为可操作的工程实践：组件元数据、删除预案、渐进式移除节奏和自动化影子对比，构成了完整的"检测-决策-执行"闭环。
> 参见 [[concepts/harness-component-expiry-evidence|证据链与 Build to Delete 原则]] | [[concepts/harness-component-expiry-and-build-to-delete|Harness 组件保质期总览]]
---

## 所属 MOC

- [[moc/loop-engineering|Loop Engineering]]
