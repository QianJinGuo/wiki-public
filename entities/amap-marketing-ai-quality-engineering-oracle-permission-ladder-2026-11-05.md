---
title: "高德营销 AI 质量工程：预期必须来自需求，权限必须逐级审批"
created: 2026-11-05
updated: 2026-11-05
type: entity
tags: [ai-quality-engineering, oracle-problem, agentloop, graph-engineering, langgraph, safe-autofix, permission-ladder, self-healing-tests, evidence-model, quality-gates, amap]
sources:
  - raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05
confidence: 0.88
provenance_state: extracted
review_value: 8
review_confidence: 8
---

# 高德营销 AI 质量工程：预期必须来自需求，权限必须逐级审批

## 一句话结论

把 AI 纳入软件质量保障，真正的难点不是"AI 能不能写测试"，而是**"凭什么相信它写的测试"**。高德的答案是两条硬规矩：**预期的来源必须标注并分级**（需求规则 > 提交说明 > 实现代码），**执行权限必须逐级审批**（proposal → dry-run → apply → verification → commit → push → MR，每级由策略分别控制，"验证通过本身不会自动开启后续交付权限"）。长期目标也被写得很清醒：**"不是追求'无人参与'，而是让 AI 扩大认知和执行半径，让工程机制约束不确定性，让人保留关键决策权。"**^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 一、两侧分工：研发识别风险，测试验证风险

| 侧 | 载体 | 产出 |
|---|---|---|
| **研发侧** | **Graph Engineering 决策图**（LangGraph `StateGraph`） | 风险、代码位置、**验证条件** |
| **测试侧** | **AgentLoop 工作流** | 测试资产与**运行记录** |

目标链路："研发识别风险 → 测试验证风险 → 形成交付证据链"。作者明确当前状态：**两侧能力已分别落地，统一任务协议、跨侧自动交接、结果回流与交付门禁仍在建设**。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 二、研发侧：决策图 + 权限阶梯

**决策图的编排**（多用例并行审查，而非一次性生成）：`Generator` 形成基础 `Finding` → **PM / QA / Developer / SRE / Security 并行审查**需求边界·测试条件·稳定性·安全性 → `Reducer` 合并重复结论 → `Judge` 判有效性·严重度·处理优先级 → `Executor` / `Refiner` 生成修复方案并按验证诊断**有限次迭代**。规则处理边界明确的问题，Agent 处理需要进一步判断的风险。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

**输出的要求是关键**：不仅给修改建议，还要说明**风险是什么、判断依据在哪个位置、需要什么结果才能确认已处理**。原文的例子很具体——"新增枚举未覆盖兜底分支"必须同时给出**变更位置、可能受影响的业务条件、待验证场景**，才能成为测试侧可继续使用的输入。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

**安全修复权限阶梯**（本篇最值得抄的一条）：^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

```
proposal → dry-run → apply → verification → commit → push → optional MR
```

- 每个动作**由策略分别控制**，不是"允许修复"一个开关；
- 进入修复前先查：**策略白名单 · 风险等级 · 置信度 · 证据锚点**；
- 实际修改前 `dry-run`，修改后跑仓库验证；
- **"commit、push 和 MR 分别受控，验证通过本身不会自动开启后续交付权限。"**
- 每次尝试记录**验证结果 · 失败原因 · 操作来源**；
- 超迭代或预算上限 → 停止自动尝试、按执行状态处理本地改动并转报告项，且**"已经发生的远端动作需要单独处理"**。

这条阶梯回答了一个常见误解：**验证通过 ≠ 可以交付**。它把"能改"与"能推"拆成两件事，正是企业环境里最常被吞掉的那道门。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 三、Oracle Problem：预期结果从哪里来（全篇最有方法论价值的一节）

> "能够生成和编译测试代码，并不意味着测试验证了正确的业务行为。"

**失效机制写得很清楚**：假如 Agent **从实现代码推导出预期值，再用这个值检查同一段实现**，那么代码中的错误会被**一并写进断言**——测试通过，错误照旧。这就是测试的 **Oracle Problem**：如何确定正确的预期结果。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

**解法是给预期标来源分级**：^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

| 预期来源 | 可信度 | 说明 |
|---|---|---|
| **需求文档中的业务规则** | 独立于实现的依据 | 可作为预期 |
| **提交说明** | 可辅助理解意图 | 但仍需核对 |
| **仅从实现代码推导** | **只能说明代码行为** | **不能单独证明业务正确** |

阶段一专门提取 `requirement_expectations[]`，把规则及其适用条件传给后续阶段。并且**"缺少独立规则时，应明确验证能力受限，而不是把代码行为直接提升为业务标准"**。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

**正反向检查**：对关键风险点比较"实际响应 vs 需求规则 vs 用例断言"是否一致，并设计能**区分目标行为与错误行为**的场景（业务规则确实要求不同输出时，合法与非法输入应有不同断言）。但作者立刻补了限定：**"有限用例通过不等于证明所有实现路径都正确；需求本身的准确性仍需人工判断。"**^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 四、测试侧：四阶段 AgentLoop + 阶段间人工审阅

| 阶段 | 工作与产物 | 关键检查 |
|---|---|---|
| ① 变更分析 | 变更范围、风险点、需求预期 → `change-impact-analysis.md` | — |
| ② 测试用例设计 | 业务场景、输入条件、断言 → `test-cases.md` | 可构造性·重复场景·边界真实性·可观测性·环境可行性 |
| ③ 测试方案设计 | 映射到测试框架、数据准备、执行方式 → `test-plan.md` | — |
| ④ 自动化实现 | 测试代码 + 实现摘要，结合执行结果定位失败 | 可复查的验证记录 |

后续还有执行验证 → 失败诊断 → 覆盖率分析 → 报告汇总（⑤—⑧）。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

**阶段间的人工审阅是硬约束**：阶段四只能**在本阶段内**受限修订；**"如果失败根源在上游业务理解或测试设计，应暂停并请求人工确认，不绕过已经通过的审阅环节"**。理由：这样能分别检查"**测什么 / 怎么测 / 代码怎样实现**"，"避免把不同层次的问题混在一次生成中处理"。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

`Kbase` 补代码差异里缺失的**环境知识**（缓存规则、Mock 方式、TestBase 差异），以结构化条目进入用例/方案/实现三阶段并随知识库与仓库更新同步——但**"它为环境适配提供上下文，不替代需求规则对测试预期的约束"**。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 五、自愈测试的三条红线

失败先分类：**环境瞬时问题 / 数据残留 / 用例自身问题 / 预期错误 / 服务实现缺陷**。归因依靠**请求、响应、环境状态与独立需求依据**；"**证据不足时保留'待判定'，不以让测试通过为目标反向修改判断标准**"。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

自愈必须守三条：**①预期修订必须有独立依据，不能照抄实际响应；②不得通过删除断言或降低精度掩盖失败；③所有修订都保留记录。** 修订达到 **3 次**的用例被标为**"弱验证"**，作为人工复核的管理提示——但作者强调"修订次数本身不决定断言是否正确，验证结论仍取决于独立预期、执行结果和覆盖范围"。涉及上游规则或设计变更时，回到人工确认。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

> 这三条把"自愈"从"让测试变绿"重新定义成"**在独立依据下修正预期**"——前者是目标错位，后者才是工程。

## 六、一次完整的协同案例

**变更**：营销业务新增行业类型枚举 + 静态券面额。**研发侧**：`Generator` 出 `Finding`"新增类型未覆盖兜底路径"；多角色评审注意到这段逻辑关联券面额、遗漏可能影响**资金相关行为**，故 `Judge` 要求补验证。**测试侧**：七条用例，其中 `TC-03` 专门检查**合法与非法行业输入的差异**（约定：合法→配置面额，非法→兜底值）。**首次执行**：非法行业参数返回了**配置面额**，与兜底预期不符 → 排查聚焦到非法输入的处理路径 → 开发定位到**缺少兜底分支**并补充 → 复测通过 → `Finding` 标记"**已验证修复**"。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

案例里需要串联的东西被列得很具体：**新增行业类型的变更 · 兜底路径 Finding · TC-03 的输入与预期 · 首次执行的返回差异 · 分支修复 · 复测结果与 Finding 状态**。为此规划的**证据模型**只有少量核心字段：`change_id` + `source_revision`（变更及版本）、`claim` / `locator`（风险或验证结论及其依据）、`verification_status`（待验证 / 人工确认 / 执行验证）。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 七、数据口径：克制得罕见

| 指标 | 看板显示 | 作者自己的解释范围 |
|---|---|---|
| 业务仓库 A 问题处置 | **94 条中 50 条"可信修复" = 53.2%** | 范围仅该次快照的 94 条；另有"上报修复 60 项，**含 10 项行号漂移误判**"，`Autofix` 为 0，"**两种修复记录口径不同，不合并为自动修复成效**" |
| 测试工作项状态 | **46 个中 34 个"全阶段完成" ≈ 74%** | 范围仅该次快照；"**未完成是流程状态，不等于发生缺陷或测试失败**"；图中"全链路交付"仅指该测试工作流，**不代表跨侧自动交付已贯通** |

并明确：**两套看板并非同一统计队列，不据此计算端到端交付效果或长期效率变化**。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 八、自陈边界

- 适用：**业务规则明确、环境可控、结果可观测**的场景；规则不完整、并发时序复杂或结果具概率性时仍需人工判断与其他验证手段；
- 未完成：**统一任务协议、跨侧自动交接、结果回流、交付门禁**；
- `Safe Autofix` **自动修复记录为 0**，现阶段"仍以风险识别、人工确认和安全边界验证为主，**尚未将自动修改数量作为建设目标**"。

## 深度分析

### Oracle Problem 是"AI 写测试"这件事的真正天花板

代码生成的质量可以由测试兜底，但**测试的质量由什么兜底**？如果预期值来自实现代码，就等于"用同一份可能出错的认知去检查它自己"——测试通过只证明**实现自洽**，不证明**业务正确**。这篇的价值是把这件事**从哲学问题变成工程约束**：给预期标来源、缺独立规则时明确"验证能力受限"。这条约束可以直接写成守门规则：**任何断言若其 `expected` 无法追溯到需求条目，就不许标记为"通过"**。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

### 权限阶梯的洞见：把"验证通过"从"可以交付"里拆出来

大多数自动化流水线的隐含逻辑是"测试绿了就合并"；这里把 `verification` 与 `commit`/`push`/`MR` **分别受控**，并明说"验证通过本身不会自动开启后续交付权限"。这消除了自动化最危险的滑坡——**一次修好不等于一次可以上线**。再配上"超预算停止 + 远端动作单独处理"，才算把"AI 能改代码"关进了笼子。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

### 阶段间人工审阅，是为了保住"分层归因"

如果允许 Agent 从"测试失败"直接跳回"改业务理解和改设计"，就再也分不清问题是**测错了、设计错了，还是实现错了**。这里规定阶段四只能在**本阶段内**修订、上游问题必须回到人工确认，本质是**用流程保住归因能力**。与高德 Data Agent 那篇的"体检门禁"、速卖通的"故障语义三分"是同一族纪律：**先分清是哪一层的问题，再动手。**^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

### "弱验证"标记是个聪明的中间态

修订 3 次就标弱验证——它既不阻止自愈（否则自愈没意义），也不假装自愈可靠（否则等于自欺）。而且作者明确"修订次数本身不决定断言是否正确"，把这条标记定位为**管理提示**而非质量判定。这类"**承认不确定但给出可操作信号**"的设计，比任何单一指标都更接近真实工程。^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

### 数字口径的克制，本身就是质量的一部分

53.2%、74% 这两个数，如果按营销口径可以说成"AI 修复过半问题""四分之三工作项端到端交付"。作者反过来做了四件事：**限定快照范围、拆开上报与可信两种口径（并点出 10 项行号漂移误判）、声明 Autofix=0、说明未完成≠失败**。**在一个普遍虚报的领域里，愿意把"我们只做到这一步"写清楚，比任何指标都有说服力。**^[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05.md]

## 实践启示

1. **先问预期的来源**：断言里的 `expected` 必须能追溯到需求条目；只能追溯到实现代码的，标为"行为快照"而非"通过"。

2. **缺少独立规则时，明说"验证能力受限"**，别把代码行为提升为业务标准。

3. **修复拆成七级动作**（proposal/dry-run/apply/verification/commit/push/MR），各自策略控制，**验证通过不自动开通交付**。

4. **修复前查四样**：策略白名单、风险等级、置信度、**证据锚点**。

5. **超预算即停**：转报告项处理本地改动，**已发生的远端动作单独处理**。

6. **多角色并行评审 + Reducer + Judge**：并行出覆盖面，Reducer 去重，Judge 定优先级——三者缺一，评审要么漏要么吵。

7. **风险结论必须带三件**：变更位置、受影响业务条件、待验证场景；否则测试侧无法接手。

8. **阶段间设人工审阅**，且允许的修订**只限本阶段内**；上游问题回到人工——保住"测什么/怎么测/怎么写"的分层归因。

9. **自愈三条红线**：预期修订须有独立依据、不许删断言降精度、所有修订留痕。

10. **"弱验证"作为管理提示**（如修订≥3 次），但不要把它当成质量判定。

11. **证据模型只要少量字段**：`change_id`/`source_revision`、`claim`/`locator`、`verification_status`——够用且可查。

12. **报告数字时限定范围**：快照范围、口径差异、未完成≠失败、别把两个队列加起来算端到端。

## 与相关实体的关系

- [[entities/amap-data-agent-two-way-translation-cfhl-framework-2026-11-04|高德 Data Agent（两段翻译 + CFHL）]] 是同一公司另一团队的**分析链路**质量实践；本篇是**研发/测试链路**质量实践。两者的门禁思想一致（可得性核对 vs 预期来源核验）：**能查的事实不交给模型判断**。
- [[entities/taobao-d2c-harness-three-platform-visual-verification-2026-11-02|淘宝 D2C 视觉验证闭环]] 用**像素 diff + 四层视觉门禁**判"结果对不对"；本篇用**预期来源分级 + 权限阶梯**判"证据可不可信、动作该不该放行"——一个测结果，一个管依据。
- [[entities/vivo-pm-agent-harness-design-task-card-contract-2026-11-04|vivo PM 视角 Harness 契约]] 的"结果四类 + `evidenceRefs` 可定位 + 发布前确定性校验器"与本篇证据模型（`claim`/`locator`/`verification_status`）几乎同构，可互相对照落地。
- [[entities/aliexpress-promo-ops-autonomous-multi-agent-l3-l4-2026-11-04|速卖通选搭投]] 的"按置信度×影响面×可逆性放权"与本篇"七级权限阶梯"是同一问题的两种粒度：一个按风险分级，一个按动作分级。
- [[entities/ai-coding-agent-quality-defense-five-control-mechanisms|AI Coding 质量防御五机制]]、[[entities/baidu-ai-coding-quality-gates|百度 AI Coding 质量门禁]] 提供更宏观的门禁框架；本篇补上最容易被忽略的一环——**测试本身的预期可信度**。

→ [[raw/articles/amap-marketing-ai-quality-engineering-oracle-permission-ladder-2026-11-05|原文存档]]
