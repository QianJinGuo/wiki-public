---
title: "去哪儿 AI Coding 驱动大型核心系统重构 — Harness+Loop+Task 工程化方法论"
created: 2026-08-27
updated: 2026-10-26
type: entity
tags: [qunar, ai-coding, refactor, harness, loop, task, large-scale-refactor, engineering, ai-engineering]
sources: [raw/articles/qunar-ai-coding-large-core-system-refactor-2026]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 去哪儿 AI Coding 驱动大型核心系统重构 — Harness+Loop+Task 工程化方法论

## 核心命题

大型高并发核心系统重构不适合套用小需求 AI Coding 的方式——目标从"局部功能交付"变成"系统行为控制"，AI 的局部理解偏差会被放大成系统级风险。核心方法：**先把不确定性转化成工程约束（目标可拆解、边界可限定、验证可自动化、失败可回滚），再让 AI 在约束系统里持续推进、反馈、沉淀证据**。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

## 量化结果

一次 15 万行高并发核心系统重构（约束搜索引擎 10.5 万行 QPS 10k、包装系统 4.9 万行 QPS 1.6k、预定系统 QPS 80）的实战收益：总工时约 100PD → 30PD（提效约 70%）；核心链路 P50 191ms→59ms、P90 644ms→195ms（下降约 70%）；机器资源成本节省约 45%；**无 QA 介入，顺利上线无故障**。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

## 工程化方法论：Harness + Loop + Task

### Harness 与 Loop
**Harness 负责约束边界**（AI 能在什么边界内行动），**Loop 负责反馈推进**（AI 每轮执行后如何反馈和继续）。没有 Harness，AI 在边界不清时自动补齐假设；没有 Loop，自动执行变成失控的连续改动。两者结合后 AI 在目标、边界、工具、验证、停止条件都明确的环境中执行。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 四阶段闭环
规划 → 设计 → 执行 → 验收。只有执行阶段进入高频 AI Coding，规划和设计用来对齐目标、边界、方案、验证条件。每一步有明确输入、输出、判断标准。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 人机分工
人负责方向与风险判断（目标定义/边界判断/架构取舍/风险接受/上线决策）；AI 负责高频执行与沉淀（代码扫描/方案辅助/代码修改/测试执行/Diff 分析/报告沉淀）。方向判断和方案取舍不能交给 AI。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 任务拆解：里程碑 → Phase → Task
重构目标拆成里程碑 → Phase → Task。Task 是 AI 可稳定执行的最小工程单元，分 MANUAL/CODE/CONFIG/VERIFY 四类（CODE 主要进 AI Coding，MANUAL 和关键 VERIFY 需人工判断），按依赖关系推进而非随机挑选。M1 实例 = 9 Phase、34 Task。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

Task 文件必须提前定义执行输入、验证标准和风险出口（输入/输出/修改范围/不修改内容/验收标准/风险点/回滚方案），并沉淀四类产物：Task 执行记录、验证结果、风险判断、产物链接。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 项目总览文件 = 状态源
当 Task 变多、单轮对话承载不了全部上下文，用项目总览文件作为文档导航页、状态页、上下文恢复入口、自动化执行入口。AI 按总览 + Task graph 找下一个可执行任务，执行完回写执行记录/验证结果/产物链接。把"人脑记住的进度"变成"机器可读、团队可复查、任务可接续"的工程资产。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 四道护栏（M1 边界约束）
1. **修改范围护栏**：只改相关链路和实现承接，其他代码不动。
2. **行为语义护栏**：请求/响应/异常语义不变，核心链路差异必须通过 Diff 收敛。
3. **运行安全护栏**：影子双跑、灰度切流、自动止损控制发布风险，不做一次性全量替换。
4. **副作用护栏**：影子链路不写 MQ/Redis/外部系统，新增验证不影响主流程稳定性。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### Diff 闭环
影子双跑新旧流程对比真实流量，Diff 进入"拉取日志 → 聚类分析 → 问题定位 → 修复方案 → 人工确认"闭环。AI 参与线上差异分析和修复方案生成，但辅助聚类/定位/建议，关键风险判断由人拍板。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 发布级质量控制
Task 通过 ≠ 可全量上线。Task 级验证证明单次改动局部可控；发布级验证证明新流程在真实流量下行为一致、指标稳定、异常可回退。把"开发正确"推进到"生产可控"。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

## 里程碑拆解原则
拆解顺序按"风险能否逐步收敛"而非"哪个模块好改"：**先保证行为一致，再优化结构和性能，最后灰度收口**——收益不是第一优先级，先让系统行为可信，才有资格继续性能优化和旧链路退场。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

## 核心洞察
- AI 负责高频执行，人负责关键判断。约束越清楚，AI 产出越稳定；验证越充分，团队越容易建立对自动化的信任。
- 把复杂问题拆成可验证的 Task，AI 执行就不依赖单次对话的临场发挥，而依赖结构化输入、自动化验证、持续沉淀。
- 最终沉淀的是团队能力（任务拆解方式/项目总览/验证脚本/Diff 分析流程/发布检查项/风险处理经验），下次可复用。
- **一句话**：AI Coding 在大型重构中的价值不是替代工程治理，而是把工程治理变得更细、更连续、更容易被执行。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

## 深度分析

### 规格不在文档里，而在 15 万行代码的补丁史里

Greenfield 顺利，是因为规格写在代码前面——[[entities/spec-driven-development-harness|Spec-Driven Development]] 把"要什么"抽出来变成模型可对齐的输入。遗留系统重构相反：**规格已被编译进实现，且是被历代补丁编译进去的**。某个 `if` 或许只是 2019 年兼容脏数据的补丁，或许是性能兜底——这些"为什么"不在代码里。

错位由此产生：AI 读到的是"更合理的写法"，成功标准却是"行为一模一样，包括那些不合理的地方"。模型默认补齐缺失假设，边界不清时按训练分布里的"好代码"重写。所以 Harness 在这里不是"提示词更小心"，而是三样可机械检查的东西：**改动范围**（哪些文件允许进 diff）、**行为判据**（逐字段比对而非"读起来对"）、**失败出口**（可独立回滚）。还有一条常被低估：迁移不可逆，QPS 10k 的链路不能停服试错，"改错重来"这个最常用的恢复手段直接失效。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### Loop / Task 拆解的本质：把爆炸半径变成可回滚的单位

人团队按**认知负荷与代码归属**切重构，AI 没有"熟悉"只有上下文，失败模式不是"改得慢"而是"改得又快又自信"，切分依据必须换成：**这一步失败，能否被独立观测、独立回滚**。

Phase 与 Task 分工不同：Phase 是**验收单元**（可观测的阶段性状态），Task 是**执行单元**（一次可 diff 的改动）。分开的意义是承认"Task 全绿 ≠ 可上线"——34 个局部正确的改动累加仍可能产生系统级偏差，所以要插入只准验证、不准推进的关卡。这与人团队的 strangler fig 骨架相似，差别在记忆载体：人靠老人记得旧链路为何这么写，AI 靠 Task 文档与项目总览文件——[[concepts/harness-loop-architecture|Harness/Loop 架构]]里"状态外化"在此不是效率优化，而是唯一可行性来源。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 量化结果能证明什么，不能证明什么

100PD → 30PD、P50/P90 双降 70%、省 45% 资源，数字很硬，但能证明的范围比看起来窄。

**能证明**：同一目标、同一批人、同一套约束下，"执行"的边际成本被压低了，省下的主要是代码扫描、批量修改、验证执行、报告沉淀这类重复动作。

**不能证明**四层：**缺反事实**——分母是"计划工时"而非"实际会发生的工作量"，30PD 的真实参照系可能是"这事根本不做"；**归因混淆**——P50 191ms→59ms 是"合并链路 + 统一数据源"这个架构决策的功劳；**幸存者偏差**——失败的重构不会写成公众号文章，而 30PD 的前提是护栏、影子双跑、34 个 Task 已就位，准备工时不在分子里；**"无 QA 介入"是结果不是原因**——它成立是因为验证被前移成自动化 Diff 与影子比对，读成"以后可裁 QA"是把结论用反了。站得住的指标只有"上线后指标稳定无故障"，而样本量是 1。^[raw/articles/qunar-ai-coding-large-core-system-refactor-2026.md]

### 组织前提与失效边界

载体不是 prompt，是组织。三条前提缺一不可：

1. **可执行的验证是 harness 的物理本体。** 没有自动验证，Task 验收标准只能写成"看起来对"，Loop 退化成"AI 说自己改好了"。原文把"测试覆盖不足"列为重构成本——AI 没解决它，只用流量差分**临时替代**了它，前提是有可对照的线上流量。
2. **边界要清晰到能写成护栏。** "修改范围护栏"必须可机械检查（目录、接口、调用方），边界糊的系统只能退回"请只改必要处"这种无效约束。
3. **评审带宽与 oncall 归属先于选型。** AI 把 diff 产出提了三倍而 review 带宽不变，瓶颈从"写"平移到"审"，解法是 Task 小到一人一天可审完；并事先约定 AI 产出 diff 出事谁值班——没有责任主体的自动化，第一次 P1 就会被关掉。这也是 [[concepts/enterprise-ai-adoption|企业 AI 采纳]]最常被跳过的一环。

骨架与语言框架无关：接口迁移、框架升级、服务化切分都同构，[[entities/anthropic-claude-code-large-scale-code-migration-2026|Anthropic 大规模代码迁移]]与 [[entities/scarfbench-ai-agents-enterprise-java-framework-migration-ibm|企业 Java 框架迁移基准]]可见同类做法。但三类系统要绕开：**无验证信号的**（离线批处理、数仓跑批，无差分则 Loop 空转）；**状态藏在运行时与数据时序里的**（缓存、MQ、DB 中间态让"代码等价"≠"系统等价"）；**强一致与合规约束的**（计费清结算不允许两套真相并存，只能靠仿真，仿真保真度成为新瓶颈）。最根本的误用是目标含"功能该不该保留"的判断——那已是重新设计，AI 读不出"应该变成什么"。

## 实践启示

1. **先评估验证能力，再评估模型能力。** 立项前量化两件事：核心链路能否影子双跑/逐字段差分、一次 diff 的人工可审规模。任一项不达标，AI 的速度优势会被 review 队列和线上事故吃掉。
2. **把目标写成"行为不变"的可执行判据，而非"架构更清晰"。** 首个里程碑只做行为对齐，性能与结构优化单独排期计量，否则归因永远是糊涂账。
3. **预算里单列一行 harness 建设成本。** 四道护栏、Task 模板、项目总览、Diff 聚类流水线都是前置投入，摊进执行工时 30PD 就失真。
4. **用 Phase 卡点做"只准验证、不准推进"的闸门，每个 Task 附回滚方案。** 改动切到一人一天可审完，否则 AI 只是把瓶颈从编码搬到 review。
5. **先定 owner 与 oncall，再上自动化。** 明确 AI 产出 diff 的责任人、值班归属、复盘口径并写进发布流程；否则流程活不过第一次 P1。
6. **选型时先排除不该用这套方法的系统。** 无验证信号的离线系统、状态深耦合在 DB/MQ 时序里的系统、不允许新旧并存的计费清结算，应先补验证基础设施（仿真环境、对账、可回滚发布单元），再谈让 AI 执行——顺序反了投入全部沉没。

## 相关实体
- → [[entities/qunar-ai-coding-platform-practice-l0-l5-harness|去哪儿 AI Coding 研发平台实践（L0-L5 Harness）]] — 同团队平台侧实践，本文聚焦单次大型重构工程闭环
- → [[concepts/harness-engineering-framework|Harness Engineering 框架]] — 边界约束与工程化执行
- → [[entities/tmall-ai-assistant-scheduler-refactor-ai-coding-engineering-2026|天猫 AI Coding 重构工程实践]] — 同类 AI 驱动重构案例

→ [[raw/articles/qunar-ai-coding-large-core-system-refactor-2026|原文存档]]
