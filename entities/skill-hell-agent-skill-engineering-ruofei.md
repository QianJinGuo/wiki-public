---
title: Skill Hell：Agent Skill 工程方法论
created: 2026-07-07
updated: 2026-10-15
type: entity
tags: [agent, skill, engineering, harness, loop, methodology]
sources: [raw/articles/raw-skill-hell-agent-skill-writing-ruofei]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> 本文基于 Matt Pocock 的 "writing-great-skills" 与 AI Engineer World's Fair 2026 Keynote，由公众号"架构师"（若飞）深度解读，系统梳理了 Agent Skill 从设计、触发、执行、维护到安全审计的工程方法论。^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

## 核心概念：Skill Hell

Matt Pocock 提出的 **Skill Hell** 概念：当 Skill 越来越容易下载、复制和安装后，团队开始缺一套评估、路由、验证和修剪的方法。与 **Tutorial Hell**（教程多但项目没长出来）、**Framework Hell**（框架多但系统边界没稳定）并列，是 Agent 工程时代的治理困境。^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

**Skill Hell 的四种失败模式**：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]
- **Duplication** — 同一个意思散在多处
- **Sediment** — 旧规则沉积下来，没人确认是否还有效
- **Sprawl** — 即便每一行都还活着，文件本身也太长
- **No-op** — 看起来像规则，实际不改变模型行为

## 信息分层（Progressive Disclosure）

Skill 应遵循三层信息架构，与 [[claude-code-why-instructions-ignored-jia-gou-x-2026|CLAUDE.md 入口上下文]]、[[loop-engineering-addy-osmani-challengehub|Loop 工程]]、[[agent-harness-architecture|Harness 工程]]形成互补：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

| 层级 | 功能 | 典型材料 |
|------|------|----------|
| 入口上下文 | Agent 进来少猜错什么 | AGENTS.md, CLAUDE.md, 项目边界 |
| 过程接口 | 这类任务下一步怎么做 | Skills, Runbook, 验证流程 |
| 硬门禁 | 哪些风险不能只靠模型自觉 | Hooks, 权限, CI, 审计 |

Skill 自身的三层内部结构：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]
1. **description** — 路由契约：决定 Agent 什么时候想到、什么时候不该用。需说明触发词和不适用场景
2. **SKILL.md** — 主流程：只放 5-8 个会改变行为的步骤，每个带 completion criterion
3. **references/, scripts/, assets/** — 详细材料：Agent 需要时再读

## description 路由设计

description 是 Skill 的第一道路由，最容易忽视的是它同时决定了"什么时候该用"和"什么时候不该用"。^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

**好的 description 示例**：
```yaml
description: Review a code diff for correctness, test coverage, security risk, and scope creep.
              Use when the user asks for code review, PR review, diff review, security review,
              or asks whether a change is safe to merge.
```

**触发方式取舍**：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

| 触发方式 | 适合场景 | 主要成本 |
|----------|----------|----------|
| 自动触发 | 高频、低风险、边界清楚 | 常驻描述占上下文，可能误触发 |
| 手动触发 | 低频、高影响、需要人判断 | 人要记得并主动调用 |
| Router Skill | 手动 Skill 太多记不住 | 需要维护路由表和触发说明 |

## Completion Criterion（完成标准）

Skill 的步骤不能只有建议，必须有 Agent 自己能判断、人能核对的完成标准。^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

**弱示例**（不改变行为）：
```
1. Understand the task.
2. Make a plan.
3. Test your work.
```

**强示例**（改变行为、留下证据）：
```markdown
## Steps
1. 识别变更的模块范围。
   Completion criterion: 每个修改的模块名、文件和测试文件都被列出。
2. 运行测试子集。
   Completion criterion: 命令、退出码、失败用例名都被记录。
3. 检查外部副作用。
   Completion criterion: 不使用生产凭据、真实支付端点或不可逆写入。
4. 交付证据。
   Completion criterion: 最终回复包含变更文件、运行命令、结果和未验证风险。
```

不同任务类型的证据形式不同：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]
- 调研类 — 每个关键断言对应一个来源链接
- 代码审查 — 每个高风险问题给出文件、行号和可复现路径
- 产品规格 — 每个需求映射到用户场景、非目标或验收证据

## Leading Words（语义压缩）

利用模型训练中已存在的稳定概念，用更短的词调动过程先验。^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

**有效词**：red-green-refactor, vertical slice, tracer bullet, maker/checker, single source of truth, Chesterton's Fence

**原则**：
1. **不要硬造** — 团队内部新词模型无先验，需更多解释
2. **避免词汇漂移** — 同一概念不要换来换去（vertical slice ≠ 端到端切片 ≠ 最小闭环 ≠ 薄切片）
3. **词要能改变行为** — "be careful" 太弱，大概率是 no-op

## Skill 维护与删除

Skill 最常见的问题是只加不删。维护纪律比写作技巧更重要：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

- **先删 no-op**：删掉不改变 Agent 行为的规则（"Write clean code", "Be careful", "Think step by step"）
- **Single source of truth**：每个规则只在一个权威位置出现，其他地方只做引用
- **每次失败也删一段**：每次 Skill 导致明显失败，不只加规则，也顺手删一段旧内容 — 防止 Skill 变成"只进不出"的上下文垃圾场

## 安全审计

当 Agent 拥有文件系统、bash、浏览器、网络权限时，外部 Skill 不再是普通文档。用以下检查项审计第三方 Skill：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

| 检查项 | 要看什么 |
|--------|----------|
| 触发范围 | description 是否过宽，会不会误触发高风险流程 |
| 文件访问 | 是否要求读取不必要的目录、密钥、配置 |
| 脚本行为 | scripts/ 是否有网络请求、删除、写入、安装依赖 |
| 外部依赖 | 是否会从 URL 拉取内容当指令执行 |
| 权限边界 | 是否需要生产环境、支付、邮件、工单、数据库写 |
| 证据要求 | 是否有测试、日志、报告、退出条件 |
| 维护状态 | 作者、版本、最近更新、许可证 |

## 实践工作流

新建 Skill 的推荐顺序：^[raw/articles/raw-skill-hell-agent-skill-writing-ruofei.md]

1. 选一个最近两周反复出现的任务，**只做一个 Skill**
2. 写 description，先把触发词和不适用场景写清楚
3. SKILL.md 只保留 **5-8 个主步骤**，每一步带 completion criterion
4. 把模板、术语表、测试卡拆到 references/
5. 能确定执行的检查，尽量写成**脚本**，不让模型靠感觉判断
6. 用 **3 个真实任务试跑**，记录误触发、漏读、提前完成和证据缺失
7. 删掉 no-op 和重复内容，再决定要不要自动触发

## 深度分析

### Skill 为什么会腐烂成 Skill Hell

Skill 的失效模式和编译错误不同：它很少表现为"Agent 完全不会"，而是表现为部分遵守——该查的少查了两个、该停的地方继续往前编、证据只剩一句"已完成"。这种静默失效决定了 Skill Hell 的产生机制不是一次性的坏设计，而是持续累积的熵增：

1. **加与删的不对称**。Agent 犯一次错就加一条"注意"，评审出一次问题就加一条"仔细检查"，每次触发的成本极低；而删除没有天然触发点——没有人因为"少了一条规则"而复盘。于是 sediment 和 no-op 单向沉积，主文件只增不减。
2. **no-op 的不可见性**。"Be careful"、"Think step by step" 这类句子永远不会"出错"，所以不会被失败信号暴露。它们只消耗注意力、摊薄真正会改变行为的规则的权重。判断手段只有一个：删掉之后 Agent 行为是否变差——这是一次成本很低的对照实验，但很少有人真的做。
3. **上下文稀释的隐性代价**。Skill 触发后主文件会成为当前任务的高优先级指令。当流程、背景、示例、历史备注混在一起时，模型无法区分"此刻必须执行"和"仅供参考"，执行抖动由此而来——即使每一行单独看都还活着（sprawl）。

因此 Skill Hell 的本质不是"内容写错了"，而是**过程资产缺乏清理纪律**：它和旧 Wiki、旧 Runbook 腐烂的路径完全同构。[[entities/harness-engineering-deletable-worksite-ruofei|可删除工作区]]把"可删除"当作设计目标，正是对同一种熵增的反制。

### 信息分层即上下文经济学

Progressive Disclosure 通常被理解为"文件目录组织"，但它真正的内核是**上下文的成本核算**。三层结构对应三种截然不同的计费模式：

- **description 是常驻税**。为了让 Agent 能"想起"这个 Skill，描述必须每轮占用上下文。这笔税不是触发时才交，而是乘以全部轮次和全部 Skill 数量——Skill 越多，每个 description 的修剪价值越高。OpenAI 文档提到列表过大时先缩短描述，本质就是这条预算线崩了。
- **SKILL.md 是触发后成本**。只在命中后支付，但支付时权重极高（会被当作当前任务的核心指令）。所以这里的准入标准最严：只放"每次运行都会用到"的 5-8 步，任何"了解背景"类的字都在稀释关键规则的注意力配额。
- **references/ 是按需成本**。只有某些分支需要，可寻址、可延迟。把模板、测试卡、术语表压到这里，不是整洁偏好，而是把低频字节从高权重区挪到低权重区。

由此可以推出一个统一的检验问句——原文那句"这段内容会让 Agent 下一步的行为更稳定吗？"实际上是在问：**这一字节放在哪一层、付哪种成本、换回多少行为收益？** 这与 [[entities/claude-code-why-instructions-ignored-jia-gou-x-2026|CLAUDE.md 入口上下文]]的分工逻辑一致：入口层回答"进仓库少猜错什么"，过程层回答"这类事下一步怎么走"，两者都遵循"内容不缺，缺的是放在正确层级"的原则。

### 完成标准是可评估的契约

Completion criterion 的深层价值在于把 Skill 从"建议列表"升级为**双方可评估的契约**：

- **Agent 侧可自判**：每一步的完成与否不依赖模型自评"我理解了"，而是依赖一个可观测的产物（列出的模块清单、命令与退出码、记录的失败用例名）。这正是 [[entities/agent-skills-comprehensive-survey|Agent Skills 系统综述]]里 exit criterion 与 premature completion 问题的解法——完成标准把"提前宣布完成"的失败模式转化为"记录 blocker 与下一步人工动作"，失败本身也留下了证据。
- **人侧可核对**：接手者不需要重跑 Agent 的推理过程，只需核对证据（命令、退出码、来源链接、行号）。这使 Skill 成为无人值守运行的信任基础——与 [[entities/loop-engineering-addy-osmani-challengehub|Loop 工程]]中"状态、证据、停机条件"是同一条线，只是 Loop 把它压进轮次循环，Skill 把它压进步骤。
- **证据形态按任务类型参数化**：调研要来源链接、代码审查要文件与可复现路径、规格要场景映射——不变式只有一条：**不许用"看起来合理"替代"可以核对"**。

一个附带推论：带 completion criterion 的 Skill 自带轻量评估能力——每次真实任务试跑都在产生 pass/fail 证据，无需额外搭建 eval harness。这是 [[entities/claude-code-skills-superpowers-practice|Claude Code Skills 实践]]"验证比生成更值得写进流程"的结构化版本。

## 实践启示

1. **用"删除对照实验"给 Skill 体检**：对每条规则问"删掉后 Agent 行为会变差吗"，答案不确定的先改成可检查动作，改不出来的先移出主文件——no-op 的默认处置是删除，不是保留。
2. **description 按常驻成本修剪**：触发词靠前、边界写清（何时该用/何时不用各占一句），因为它每轮都在付费；Skill 数量增长时，缩短描述优先于新增 Skill。
3. **主文件执行"5-8 步硬预算"**：每一步必须同时具备分支位置和 completion criterion，加第 9 步之前先问哪一步可以降级到 references/ 或直接删除——用预算倒逼修剪，而不是用意愿。
4. **每次 Skill 导致失败，加规则的同时删一段旧内容**：把"删除"绑进失败复盘的固定动作，打破只进不出的单向沉积。
5. **完成标准写成"失败友好"契约**：明确命令跑不了时记录 blocker 和下一步人工动作，而不是假设成功或隐藏失败；证据形态按任务类型选择（来源链接/行号+复现路径/场景映射）。
6. **词汇纪律先于新词发明**：优先复用模型有先验的工程词（red-green-refactor、vertical slice、single source of truth），同一概念全团队固定一个词，淘汰 "be careful" 类不可核对表述。

## 与 wiki 已有知识的关联

|- [[agent-skills-comprehensive-survey|Agent Skills 系统综述：Skills 和技术债]] — 互补视角，本文更侧重方法论而前篇侧重系统分类
|- [[claude-code-skills-superpowers-practice|Claude Code Skills 实践：验证比生成更值得写进流程]] — 本文的 completion criterion 理念与该篇的"验证优先"方向一致
|- [[claude-code-why-instructions-ignored-jia-gou-x-2026|CLAUDE.md 拆解：Agent 进仓库前的上下文入口]] — Skill 与入口上下文的分工互补
|- [[harness-engineering|Harness Engineering]] / [[loop-engineering-addy-osmani-challengehub|Loop Engineering]] — Skill 是 Harness 中的过程接口层
- [[appstore-activity-harness-engineering-tencent|应用宝活动平台 Harness 工程实践]] — 生产环境 Harness 中的 Skill 角色
- [[code-is-cheap-harness-water-flow-wuyue-aliyun-2026|Code is Cheap：Harness 方法论]] — 过程稳定性设计的思想共鸣

→ [[raw/articles/raw-skill-hell-agent-skill-writing-ruofei|原文存档]]
