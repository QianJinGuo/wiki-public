---

title: "高德 Marketing AutoResearch：营销增长 AI Native 经营托管框架"
description: "高德技术 2026-06-09 发布：Marketing AutoResearch —— AI Native 经营托管框架，3 层技术架构（业务协议 + Agent Team + 真实反馈）+ 5 层解耦工程实现（业务协议/评估口径/工具适配/Agent Team/Runtime）+ 可视化中枢 + 协作入口。5 个真实案例 + 脱敏数据：年度化利润增量千万级；7小时5次自主决策收窄利润缺口-20%→-3%；高弹性区域+8% 相对提升；失败分层重新分配后6天平均 lift 1.020。"
source: [[raw/articles/gaode-marketing-autoresearch-ai-native-practice]]
review_value: 8
review_confidence: 8
review_recommendation: strong
review_stars: 4
date: 2026-06-09
created: 2026-06-09
updated: 2026-08-29
tags: [autoresearch, marketing-agent, gaode, amap, ai-native, growth-marketing, business-protocol, agent-team, real-feedback-loop, human-on-the-loop, decision-intelligence, long-term-operation, multi-decision-arbitration, pre-holiday-layout, recovery-from-failure, prod-decision-agent, case-study]
type: entity
provenance_state: synthesized
sources: [raw/articles/gaode-marketing-autoresearch-ai-native-practice]
---

> -> [[raw/articles/gaode-marketing-autoresearch-ai-native-practice|原文存档]]

# 高德 Marketing AutoResearch：营销增长 AI Native 经营托管框架

## 一句话

高德信息业务中心 2026-06-09 发布的 **Marketing AutoResearch**——面向"长期经营研究问题"的 AI Native 经营托管框架，由人定义**目标/约束/可行动空间/治理边界**，**Agent Team** 在业务协议边界内持续**假设→小步实验→真实反馈→经验沉淀**循环迭代。**3 层技术架构 + 5 层解耦工程实现**，5 个真实脱敏案例验证：年度化利润增量千万级，7 小时 5 次自主决策，零人工介入。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 核心问题：营销系统的"结构性断层"

**算法模型已经能预测用户转化、补贴弹性、策略收益，但真实经营决策仍大量依赖人工看盘/解释/调参/复盘**。模型给出"局部信号"，业务需要"连续决策"；系统能算一次策略，但很难长期回答：哪里加力、哪里收缩、哪里是噪声、哪里值得继续试。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

**核心场景**：营销发券 / 补贴分配 / 城市策略 / 人群分层 / 节假日节奏。**反馈每天更新、风险每天重评**——一次性分析/报告/调参无法支撑长期经营优化。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 三大技术架构

### 1. 业务协议 (Business Protocol) — 给 Agent 一块可安全研究的实验场

**第一层不是 prompt，而是业务协议**——把过去依赖人工经验判断的运营规则转化为**机器可执行、可审计、可版本化的生产边界**。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

**核心价值**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
- 不把 Agent 绑死
- 给 Agent **安全、可控、可验证的研究空间**
- 没有协议 → Agent "自治"变成越权
- 有了协议 → Agent 能在明确授权内持续探索

### 2. Agent Team — 多角色协作

**经营决策不是单点动作，而是一条生产链路**：^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

> 看见现象 → 解释变化 → 生成动作 → 通过安全门 → 小流量执行 → 读取反馈 → 沉淀经验 → 下一轮继续

**关键原则**：**LLM 不直接承担修改线上参数的风险角色**，而是承担**研究员 / 审稿人 / 复盘者**角色；**确定性工具负责计算 / 求解 / 安全 / 发布**。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

**架构价值**：发挥大模型在推理/解释/假设生成上的能力，同时**避免模型直接越过生产系统风险边界**。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### 3. 真实反馈 — 让策略接受业务验证

**AutoResearch 与普通 Agent 最大区别**：**不以"生成答案"为终点，而以"真实反馈"为下一轮起点**。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

- 系统持续读取**实验桶 / 对照桶 / 归因窗口 / 长期指标**
- **有效策略** → 经验沉淀
- **无效策略** → 记录失败原因
- **高风险策略** → 安全门拦截或降级

**核心论断**："这让 Marketing AutoResearch 从'会生成策略'升级为'能研究策略'。"^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 五层解耦工程实现

| 层 | 职责 | 独立演进 |
|----|------|----------|
| **业务协议** | 治理边界、可行动空间、违规拦截 | 新增行业补齐协议 |
| **评估口径** | 真实反馈定义、实验桶/对照桶归因 | 新增场景补齐口径 |
| **工具适配** | 确定性计算、求解、安全、发布 | 新增策略动作注册工具 |
| **Agent Team** | 研究员 / 审稿人 / 复盘者多角色 | 通过 Runtime 编排接入 |
| **Runtime** | 编排、状态、可视化中枢、协作入口 | 增量添加新 Agent 角色 |

**解耦价值**：新增行业、新增策略动作、新增 Agent 角色都**无需重写整套系统**。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### 可视化中枢（7 大模块展示）

> "**在增长系统里，AI 最大的问题不是'不够聪明'，而是'聪明但说不清'**"^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

展示内容：当前观察、风险判断、动作生成、安全门通过、实验执行、反馈读取、经验沉淀——**让 AI 的每一次判断都能被观察、被追溯、被复盘**。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### 协作入口（5 类追问）

业务和算法同学可追问：当前为什么这个判断？风险信号是什么？实验执行情况？复盘结论？策略调整建议？让 AutoResearch **成为组织协作的一部分**，而不仅是后台能力。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 实践验证（5 个真实脱敏案例）

**整体结果**：在真实托管场景中实现**规模、效率、用户侧核心指标同时胜出**，**年度化利润增量预期达到千万级**。**收益不是来自一次性大促加码**，而是 Agent Team 在安全边界内持续做小步判断 + 小流量验证 + 资源再分配——**把多个维度的边际改进累积成整体收益**。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### Case 1: 极端波动下的自动修复

某次业务波动中，核心利润指标快速下探接近预设防线。系统在 **7 小时内自主完成 5 次决策，零人工介入**：^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

1. 识别风险 → 进入防守策略 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
2. 风险继续扩大 → 进一步收缩低效补贴 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
3. 边际指标开始改善 → **停止过度防守** ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
4. 继续收缩可能伤害有效交易 → 转向温和恢复 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

| 指标 | 数值 |
|------|------|
| 核心利润缺口 | **-20%** → **-3%** (收窄 17pp) |
| 核心交易指标 | 仍保持 **+4%** 正向增长 |
| 自主决策 | 5 次 / 7 小时 / 零人工介入 |

**核心论断**："系统能在两个目标之间做**动态仲裁**：该防守时防守，该停止过度防守时也能停下来。" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### Case 2: 外部事件不等于盲目加码

某大型线下活动 + 异常天气 → 系统识别到核心区域自然需求可能上升。**传统直觉"热点来了就加补"，Agent Team 判断**: 当自然需求足够强时，继续加补可能只是补贴**本来就会来的需求**而非创造新增需求。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

**决策**：核心区域维持保守水位，预算转向**更具弹性的周边区域**。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

| 指标 | 数值 |
|------|------|
| 高弹性区域 | **+8% 相对提升** |

**核心论断**："AutoResearch 要判断的不是哪里热，而是哪里补了才真正有**边际收益**。" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### Case 3: 节假日前的前置布局

长假前系统通过**搜索热度 / 日历信息 / 历史弹性先验**识别到出行需求进入提前预订窗口。**前置三阶段布局**：^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

1. **假期前** → 高弹性区域小步探索，锁定早鸟需求 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
2. **假期期间** → 自然需求增强后逐步收缩低弹性补贴 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
3. **低弹性区域** → 维持基准或小步观察，避免"节日一刀切" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

**核心论断**："AutoResearch 不只是事后复盘，也能把外部信号转成**提前布局**。它开始具备**经营节奏感**。" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### Case 4: 从失败维度中重新发现机会

某分层维度早期实验表现不佳（人工直觉会判断"不值得继续投"），但系统在**后续模型更新**中发现真实弹性可能被低估 → **调整投资分数并重新分配资源**。^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

| 阶段 | 指标 |
|------|------|
| 早期 lift | **0.958**（落后基准 -4.2%）|
| 重新分配后 | **连续 5 天 lift > 1.0**，峰值 ~1.053 |
| 6 天平均 | **~1.020**（从负向维度转为正向）|

**核心论断**："AutoResearch 的强项不是一次判断永远正确，而是发现自己可能判断错了以后，能用**新数据修正**。" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### Case 5: 多区域并行学习

(节选可见) —— 系统支持多区域并行实验，每区域独立判断 + 全局经验沉淀。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 深度分析

### 与现有 AutoResearch 实体的差异化定位

**现有 AutoResearch entities** 全部聚焦**科研 / 软件开发**域： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

| 现有 entity | 应用域 | 核心方法 |
|-------------|--------|---------|
| [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AI 自主科研 L0-L4]] (6KB) | 科学发现 (AlphaFold / The AI Scientist) | L0-L4 五级自主性框架 |
| [[entities/autoresearch-multi-agent-software|AutoResearch 多 Agent 软件开发]] (14KB) | 软件开发 (Codex / Claude Code) | 多 Agent 异步 + 评审者回路 |
| Karpathy AutoResearch (Query Engine Bug) | 个人研究循环 | Nightly research loop |

**高德 Marketing AutoResearch** 是**第四个独立 AutoResearch 范式**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

| 维度 | 高德 Marketing AutoResearch |
|------|---------------------------|
| **应用域** | 营销增长（业务托管）|
| **核心抽象** | 业务协议 + Agent Team + 真实反馈 |
| **决策粒度** | 城市级 / 分层 / 节假日（百万元级预算）|
| **关键创新** | **业务协议 (governance boundary) + 真实反馈 (real feedback loop)** |
| **业务结果** | 7 小时 5 次自主决策收窄 -20% → -3% 利润缺口 |
| **哲学** | 人定义边界，Agent 在边界内自治 |

**与 [[entities/autoresearch-multi-agent-software|多 Agent 软件开发]] 的方法论共鸣**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
- 同样强调**多 Agent 角色分工**（研究员 / 审稿人 / 复盘者 ≈ Planner / Reviewer / Reflector）
- 同样重视**真实反馈**（业务结果 ≈ 单元测试 / Linter / 编译）
- 同样需要**安全门/边界**（业务协议 ≈ CI/CD Pipeline）
- **区别**：营销域的"实验桶/对照桶"是更复杂 A/B 测试基础设施（不同城市/分层/时段），软件开发域的"测试"更结构化

### 与高德 AI Native 系列的内部一致性

**高德"超级应用的 AI 原生研发模式"系列**已有： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

| 期 | 主题 | 核心 entity |
|----|------|------------|
| 第 1-2 期 | (背景调研)| 待补充 |
| **第 3 期** | **7×24 Self-Healing Pipeline + Agent 自进化** (研发生产线) | [[entities/gaode-ai-native-7x24-pipeline-self-healing]] (30KB) |
| **第 4 期** (本期) | **Marketing AutoResearch** (营销增长托管) | 本文 |

**两条线的并列**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
- **研发生产线**：人定义规则和边界 → AI 在规则内自主决策 → 7×24 永动 (CI/CD 自闭环)
- **营销增长线**：人定义目标和边界 → Agent Team 在边界内持续实验 → 经营托管 (实验/反馈闭环)

**核心思想一致**："**人定义规则和边界，AI 在规则内自主决策和执行**"——研发线强调"操作执行→决策审查"，营销线强调"工具调优→决策智能"。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### 与 [[entities/gaode-ai-companion-agent-architecture|高德 AI Companion Agent 架构]] 的层级关系

- **AI Companion Agent**：C 端用户交互（智能助手）
- **AI Native 生产线**：B 端研发提效（CI/CD 自愈）
- **Marketing AutoResearch**：B 端业务提效（营销托管）

**高德 AI 战略的三层落地**：C 端用户助手 / 内部研发工具 / 业务决策托管——形成完整的"AI Native 业务操作系统"。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

### 与 [[concepts/harness-engineering-framework|Harness Engineering]] 的深度共鸣

**业务协议 = Harness 在业务域的具象化**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
- Harness 提供**环境治理**（CI/CD、Review、Test）
- 业务协议提供**经营治理**（合规、预算、风险边界）

**真实反馈 = Harness 反馈循环的业务版**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
- Harness 反馈来自 CI 流水线
- 业务反馈来自业务结果（GMV / 利润 / 转化率）

**Agent Team = Harness 内的多 Agent Runtime**： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
- Harness Runtime 调度多个 Agent
- 业务协议约束 Agent 行为边界

### "决策智能" vs "工具调优"的范式跃迁

**从工具调优到决策智能**是本文核心论断——这一范式跃迁体现在： ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

1. **目标**变化：从"模型预测准确率" → "经营结果（GMV/利润）" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
2. **决策**变化：从"一次性输出" → "连续决策仲裁" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
3. **反馈**变化：从"离线评估" → "线上实时反馈" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
4. **责任**变化：从"工具辅助" → "业务托管" ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

这是 [[entities/gaode-sdd-harness-team-ai-coding-paradigm-ibjfu|高德 SDD Harness 团队 AI 编码范式]] 的**业务侧对应**——SDD 关注"代码的正确性"，AutoResearch 关注"决策的正确性"。 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 实践启示

1. **业务协议优先**：在 Agent 上线前先把"什么能做、什么不能做、如何判断好坏、出现风险如何停止"定义清楚（业务协议第一，不是 prompt 第一） ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
2. **风险角色交给确定性工具**：LLM 只承担研究员/审稿人/复盘者，发布/计算/安全交给工具 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
3. **可视化中枢是组织采纳前提**：AI 必须"可被观察、被追溯、被复盘"才能被业务信任 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
4. **实验桶/对照桶是关键基础设施**：没有 A/B 实验基础设施就无法做"真实反馈"循环 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
5. **失败维度值得重新投资**：早期失败的分层维度，如果后续模型更新发现真实弹性被低估，应重新分配资源 ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]

## 与已有实体的差异化定位

- vs [[entities/autoresearch-ai-scientific-discovery-l0-l4-challengehub|AI 科研 L0-L4]] — 科学发现域，本文是营销业务域
- vs [[entities/autoresearch-multi-agent-software|多 Agent 软件开发]] — 软件开发域，本文是营销业务域
- vs [[entities/gaode-ai-native-7x24-pipeline-self-healing|高德 7x24 Self-Healing Pipeline]] — 同公司研发线，本文是营销业务线
- vs [[entities/gaode-ai-companion-agent-architecture|高德 AI Companion Agent]] — C 端用户交互，本文是 B 端业务托管
- vs [[entities/tmall-marketing-ai-workflow-best-practices|天猫营销 AI 工作流]] — 营销自动化，本文是营销**研究**（研究 + 实验 + 反馈循环）
- vs 火山引擎营销策略 Agent — 营销决策支持，本文是**业务托管**（Agent 直接调整预算）

## 上线状态

- **发布**: 2026-06-09 17:45（高德技术公众号）
- **作者**: 高德信息业务中心（业务 + 算法联合作战）
- **业务结果**: 年度化利润增量预期千万级

## 原文链接

## 相关实体
- [[entities/autoresearch-marketing-growth-amap-ai-native]]

→ [[raw/articles/gaode-marketing-autoresearch-ai-native-practice|原文存档（高德技术 2026-06-09）]] ^[raw/articles/gaode-marketing-autoresearch-ai-native-practice.md]
