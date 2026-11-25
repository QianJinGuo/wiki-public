---
title: "Loop Engineering 四层循环栈：从 Agent 循环到爬坡循环 — LangChain 官方框架"
created: 2026-06-18
updated: 2026-11-25
type: entity
tags: [loop-engineering, agent, architecture, harness, langchain, loopcraft]
sources: [raw/articles/loop-engineering-langchain-four-layer-loopcraft]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

LangChain 团队（2026）提出的 Agent 循环工程 4 层栈框架，将「loopcraft」（循环工艺）从概念推向可落地的分层架构。文章以 LangChain 内部文档 Agent 为贯穿示例，每层对应一个 LangChain 产品原语。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

## 理论基础：Loopcraft 与 Agent 的"辛辣教训"

Loopcraft（循环工艺）指围绕 Agent 叠加循环以成倍放大效能的技艺。swyx、Steipete、Boris Cherny、Andrej Karpathy 等人不约而同得出同一结论：**不应再手动给 Agent 写 prompt，应设计循环让循环去驱动 Agent**。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

类比 Rich Sutton 的"苦涩的教训"（Bitter Lesson），LangChain 提出 Agent 的"辛辣教训"（Spicy Lesson）：**不要亲自修修补补，把精力放在能随 Agent 数量扩展的系统上**——目标设定和编排调度才是杠杆所在。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

## 四层循环栈

### 第一层：Agent 循环（模型 + 工具）

最基本的循环：模型反复调用工具直到任务完成。LangChain 原语：`create_agent`，选择任意模型接入工具即可运行。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

文档 Agent 示例：接收文档改进请求 → 模型规划起草 → 克隆仓库、读文件、写文档、提交 PR。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]


### 第二层：验证循环（Agent + 评分器）

在外层包裹验证循环：评分器（scorer/grader）检查 Agent 输出，不达标则带反馈重试。评分器可以是确定性的，也可以是基于 Agent 的（LLM-as-Judge）。LangChain 原语：`RubricMiddleware` 或 `create_agent` 的 `after_agent` 钩子。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

**权衡**：验证增加延迟和成本，但当质量比速度更重要时（大多数生产环境）值得。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]


### 第三层：事件驱动循环（验证 + 系统）

将 Agent 接入生态系统，事件触发（Webhook、cron、Slack channel）自动运行。Agent 不再是手动调用的工具，而是持续运行的组件。LangChain 原语：LangSmith Deployment / Fleet channels。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

cron 的流行用法是 openclaw 的"心跳"机制——Agent 变成始终在线的主动助手。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]


### 第四层：爬坡循环（系统 + Engine）

最重要的层次：自动化改进过程。生产 trace 输入分析 Agent → 发现问题 → 改写框架配置（prompt/工具/评分器）。LangChain 原语：LangSmith Engine（trace 分析 Agent）。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

**关键洞察**：反馈箭头深入内部直接更新 Agent 循环本身——外层循环的每次迭代让内层循环更高效。未来可接入 RL 微调，将 trace 作为训练信号。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]


## 人类监督

自动化不等于去人化。四层中每层都有人类监督的天然节点：^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

| 层次 | 人类角色 |
|---|---|
| Agent 循环 | 敏感操作/工具调用前确认 |
| 验证循环 | 敏感工作流的评分者 |
| 事件循环 | 输出返回前审批 |
| 爬坡循环 | 框架改进部署前审查 |

## 四层汇总表

| 循环 | 作用 | 影响 | LangChain 原语 |
|---|---|---|---|
| L1: Agent 循环 | 模型反复调用工具直到完成 | 自动化工作 | `create_agent` |
| L2: 验证循环 | 输出评分，不通过则反馈重试 | 保证质量 | `RubricMiddleware` |
| L3: 事件循环 | 事件触发运行，更新系统 | 规模化运作 | LangSmith Deployment / Fleet |
| L4: 爬坡循环 | trace 分析 → 改进配置 | 持续改进 | LangSmith Engine |

## 深度分析与实践启示

### 四层栈如何改写 Agent 可靠性的经济学

传统可靠性靠"把单次运行调得更对"换取，成本随可靠度陡增。四层栈改成分层采购：L1 只管"能做"；L2 用评分器把"做对"变成可判定检查项（确定性规则或 LLM-as-Judge 均可），带反馈重试堆出高成功率交付；L3 把可靠性摊销进事件流，cron/webhook/Slack 触发即廉价重试，Agent 成为持续运行组件；L4 把"改进可靠性"本身自动化。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

可靠性计价单位由此改变：不按单次输出、而按循环收敛速度计价。验证循环为每次运行多付延迟与成本（质量重于速度时值得），爬坡循环却让审查的边际成本随 trace 积累递减，这正是"人类判断力与 token 投入共同复利"的微观机制。 [[entities/loop-engineering-feedback-control-system]] ^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

### 人工监督门应建在循环层级的哪些位置

四道门性质不同，选址原则：**由不可逆性与错误可检测性共同决定**。L1 门（敏感工具调用前确认）拦不可逆副作用（金融交易、数据库写入）；L2 门（人任敏感工作流评分者）处理评分器编码不了的判断——自动评分能查链接、CI、diff，"表述是否适合受众"则靠人的品味；L3 门（输出返回用户前审批）防坏输出被事件流复制 N 次；L4 门（框架改进部署前审查）最关键——一次错误改写会污染其后所有内层循环。梯度规则：层级越高、杠杆越大，门控越应保守——L1 可近乎无人值守，L4 的人类否决权应长期保留。 [[entities/loop-engineering-anthropic-playbook-orange-book-v260615-2026]] ^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

### "辛辣教训"对 harness 设计的含义与适用边界

苦涩教训说算力与学习终胜手工知识；辛辣教训把它搬到工程侧：精力应投向随 Agent 数量扩展的系统（目标设定、编排调度）。落到 harness 设计有三条推论：^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

1. **接口稳定、实现可改写**：L4 自动改写 prompt/工具/评分器的前提是配置结构化、可 diff、可回滚；harness 的价值是为机器改写提供安全落点，而非写死行为。
2. **评估先于优化器存在**：没有 L2 评分器，L4 trace 分析失去信号源，爬坡退化为盲目搜索——评分器是循环栈的损失函数。
3. **人的时间迁移方向**：从"修 prompt"转向"定义什么算好"（rubric 设计与 L4 审查）——Steipete、Boris"让循环去 prompt Agent"与 Karpathy 的杠杆问题是同一件事的工程版与个人版。

适用边界：教训对已跑通 L1-L2 的团队成立；刚起步时 prompt 工程仍是第一层可靠性基础，过早跳 L3-L4 欲速则不达。循环栈是随模型能力迁移的动态配置，切换信号是"失败发生在哪一层"。文章自己也说：每阶段早期向下深入循环补可靠性有价值，模型变强后向上扩展换杠杆更有价值。^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

### 与既有框架的对齐：L4 为何是护城河

四层与清华六件套（Skill/Spec/Tool/Act/Eval/Stop）可映射：L1≈Tool+Act，L2≈Eval+Stop，L3≈Spec（事件触发规则），L4≈Skill（从 trace 提炼技能）；锚点不同：前者绑产品原语（`create_agent`、`RubricMiddleware`、Engine），后者绑抽象层。前三层是各家共识，L4 独到之处在于把 trace 分析 Agent 当一等公民：反馈箭头深入内部、直接更新 Agent 循环本身，而非事后复盘。外层每次迭代都让内层更高效，优势随时间复利，难以被单次工程质量追平——与挑战驱动的持续改进方向一致。 [[entities/loop-engineering-addy-osmani-challengehub]] [[entities/loop-engineering-tsinghua-2026]] ^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

**实践要点**

1. **先 L1-L2 后 L3-L4**：没有稳定评分器的 Agent 不该接入事件流，否则规模化只是放大不可靠。
2. **评分器从确定性规则起步**：文档 Agent 示例全用无人工检查：测试通过、链接可达、CI 绿、diff 仅含请求改动；LLM-as-Judge 留给规则覆盖不了处。
3. **每条 trace 回答"哪层失败"**：给运行打层次标签，L4 才能把指向同一问题的多条 trace 聚合成一个 issue 而非逐条告警，请求改对应 prompt 或工具。
4. **把 prompt/工具/评分器当配置管理**：版本化、可 diff、可回滚，L4 自动改写才可审计。
5. **人工门只留不可逆与品味节点**：其余交给 L2 自动评分；每多一道门，循环吞吐降一档。
6. **自建栈可等价映射**：agent loop（L1）、pre-commit hook + lint（L2）、cron + webhook（L3）、session trace 分析（L4），不绑 LangChain 也能搭出同一架构。 [[entities/hermes-agent-skills-source-code-analysis-shuge]] ^[raw/articles/loop-engineering-langchain-four-layer-loopcraft.md]

## 相关实体

- [[entities/loop-engineering-addy-osmani-challengehub|Loop Engineering — 12 来源合并]]
- [[entities/loop-engineering-feedback-control-system|Loop Engineering: 把反馈循环放进工程现场]]
- [[entities/loop-engineering-tsinghua-2026|循环工程 — 清华 2026 框架]]
- [[entities/agent-loop-engineering-handbook-8-questions-chen-jin-tencent-self-2026|Agent Loop 8 个未解问题]]
- [[entities/hermes-agent-skills-source-code-analysis-shuge|Hermes Agent Skills 源码分析]]
- [[moc/loop-engineering|Loop Engineering 主题地图]]
- → [[raw/articles/loop-engineering-langchain-four-layer-loopcraft|原文存档]]
