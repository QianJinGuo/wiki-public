---
title: "Claude Agent SDK Skills：可复用的专业知识体系 — Skill ≠ Tool ≠ Memory，渐进加载 + 4 方分工 + 团队 Skill 库设计"
created: 2026-06-18
updated: 2026-11-25
type: entity
tags: [claude-agent-sdk, skills, agent, architecture, anthropic, prompt-engineering]
sources: [raw/articles/claude-agent-sdk-skills-reusable-knowledge]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

Claude Agent SDK 系列第 11 篇，系统阐述 Skills 在 Agent 架构中的定位——将团队经验沉淀为可复用、可审查、可版本管理的「操作手册」，而非一次性 prompt。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 核心边界：Skill ≠ Tool ≠ Memory

文章首先厘清三个容易混淆的概念：^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

| 概念 | 回答的问题 | 例子 |
|---|---|---|
| **Tool** | Agent 能调用什么能力？ | 读文件、跑命令、查数据库、发 Slack |
| **Skill** | Agent 应该怎么完成一类任务？ | 代码审查流程、发布说明模板、事故复盘 SOP |
| **Memory** | 某个项目/用户已经发生过什么事实？ | "这个仓库用 pnpm test" |

类比：**工具是手，Skill 是手册**。Skill 也不是 Memory——Memory 绑定具体项目，Skill 可跨项目复用。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 渐进加载：Skill 的核心工程价值

Skill 最大的工程价值不是"把 prompt 存成文件"，而是**渐进加载**（progressive loading）——避免一启动就把所有规范塞进上下文：^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

1. **第一层**：只暴露 Skill 目录 + 简短描述
2. **第二层**：任务匹配时加载对应 SKILL.md
3. **第三层**：SKILL.md 引用的 references/templates/scripts 按需读取

## Skill 文件结构

一个成熟 Skill 以目录为单位：^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

```
SKILL.md          # 路由 + 核心流程（不要塞所有内容）
references/       # 长规范、代码规范、审查清单
templates/        # 固定输出格式（PR 模板、发布说明）
scripts/          # 可执行辅助脚本
assets/           # 固定素材
```

**关键原则**：SKILL.md 只放路由和核心流程。引用而非堆砌——"做 A 时读取 references/a.md"，把 Skill 从"长 prompt"变成"任务操作系统入口"。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 4 方分工：Skills / Hooks / Subagents / MCP

文章提出四句话判断法：^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

| 维度 | 职责 | 代码审查场景示例 |
|---|---|---|
| **Skills** | 教它怎么做 | 定义审查流程：行为变化→正确性→安全→测试→severity 输出 |
| **Hooks** | 管它什么时候能做 | 拦截：不允许审查 Agent 修改源码、记录工具调用 |
| **Subagents** | 拆给谁去做 | security-reviewer / test-reviewer / docs-reviewer |
| **MCP** | 接入外部系统去做 | GitHub MCP 读 PR / Linear MCP 读需求 / Slack MCP 发结果 |

**核心约束**：Skill 说"应该检查安全问题"只是建议，真正的阻止要靠权限、Hooks 和运行环境。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 何时值得写 Skill：5 条件

1. 任务会重复出现
2. 流程相对稳定
3. 团队希望统一产物质量
4. 流程里有容易忘的细节
5. 内容需要版本管理（变更也要 code review）

一次性提醒（"这次多关注性能"）直接写 prompt 即可。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 可落地的 Skill 模板

```
---
name: [稳定短名]
description: [任务 + 适用场景 + 期望产物]
---
# [Skill 名称]
## When To Use / Inputs / Workflow / Output / Boundaries / References
```

其中 **Boundaries**（边界）很关键——告诉 Agent 不要越界。例如代码审查 Skill：不要直接修改代码、不要评论纯风格偏好、不要把低置信度猜测写成确定结论。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 5 个常见坑

1. **Skill 太长** → 换了个地方堆上下文
2. **description 太泛** → "帮助开发者写代码"什么都触发
3. **把权限写成建议** → "不要删除文件"只是建议，真正不能删除要靠 disallowed_tools/Hooks/沙箱
4. **没有输出格式** → 父 Agent 难消费结果
5. **没有维护机制** → Skill 是知识资产，会过期，需要 review + 版本管理 + 淘汰
## 深度分析与实践启示

### 渐进加载的架构价值：上下文经济学视角

三层加载（目录 → SKILL.md → references/templates/scripts）本质上是在把上下文当作稀缺资源做预算分配。第一层的 name + description 相当于一张路由表：每个 Skill 只占几十 token，却让模型始终"知道自己会什么"；第二层按任务匹配才加载 SKILL.md，意味着与当前任务无关的规范根本不会进入上下文；第三层把长规范、模板、脚本的成本推迟到真正需要的时刻。原文的关键定位是：渐进加载既不是隐藏黑箱（模型始终能看到目录），也不是一次性倾倒（不把全部规范塞满启动上下文）^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]。这与 Hermes Agent 的 `skills_list` / `skill_view` 两级机制完全同构，也印证了 harness 层"上下文窗口是最贵的运行时资源"这一管理思想 [[concepts/harness-context-window-management]]。

### 4 方分工如何映射到真实团队工作流

Skills/Hooks/Subagents/MCP 的四句判断法可以直接映射为一次代码审查的组织结构：Skill 定义审查顺序与输出格式，Hooks 承担"不能改源码、不能碰生产库、记录所有工具调用"的硬约束，Subagent 把安全/测试/文档拆成可并行的专家角色，MCP 把 GitHub/Linear/Slack 等外部系统接入流程。**核心规则是职责不可互相替代**：Skill 写"应该检查安全问题"只是建议性知识，真正的阻断必须落在权限、Hooks 与运行环境层——"说"与"能"分属两个层面，混层是 Agent 产品最常见的事故来源^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]。原文的企业平台视角进一步说明这不是纯技术分类——财务、研发、客服、内容团队各挂不同 Skill 集，权限 preset 随角色同步切换，可观测系统记录使用情况。Skill 由此从 prompt 文件升级为 Agent 产品里的"专业能力包"，四方分工因此成为产品化的分水岭^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]。这也解释了为什么 Skill 库值得单独做治理：它既是知识层又是触发层，任何一层的腐化都会直接表现为 Agent 行为退化。

### 团队 Skill 库与 5 个坑的失败经济学

原文的团队库经验值得单独强调：上来就建 engineering-best-practices 这类大而全 Skill，结局是"什么都沾一点、什么都触发、什么都不精确"^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]。按任务拆分（code-review / test-generation / release-notes / incident-review 等）之后，每个 Skill 只需回答三个问题——什么时候用、具体怎么做、输出什么格式；共享规范不复制，而是放 references/ 让相关 Skill 引用，改一处全体受益。失败经济学的关键在触发层：description 是给模型判断"何时用"的线索而非给人看的简介，粒度错误会被系统性放大——太泛则什么都触发、太窄则要用时想不起来，这正是"Skill 太长"与"description 太泛"两个坑的共同根源 [[entities/regression-tax-skills-hurt-llm-agents]]。而"把权限写成建议"则是激励与执行混层：文本约束在模型侧只影响倾向，硬边界必须下沉到 disallowed_tools/Hooks/沙箱。需要指出，文章提供了清晰框架但缺落地度量：无触发准确率数据、无渐进 vs 全量加载的 token 对比、无团队库 breaking changes 的版本管理实践——这些空白恰好对应后续可观测性主题应补的运行时证据。

### 实践启示

1. **description 按"任务类型 + 适用对象 + 期望产物"写**，并显式排除无关场景——它是触发器，不是简介。
2. **SKILL.md 只放路由和核心流程**，长内容下沉到 references/templates/scripts，用"做 A 时读取 references/a.md"的引用式写法，而非堆砌——SKILL.md 的加载成本是每次触发的固定开销，应与触发频率成反比。
3. **Boundaries 是一等公民**：显式写"不直接修改代码、不评论纯风格偏好、不把低置信度猜测写成确定结论"，边界能显著降低输出噪音。
4. **安全约束永远不要只写在 Skill 里**——disallowed_tools、Hooks、沙箱才是执行层，文本只是建议；判断标准是"如果模型忽略这句话，坏事是否仍然不可能发生"。
5. **用五条件过滤是否值得建 Skill**（重复出现 / 流程稳定 / 统一质量 / 易忘细节 / 需要版本管理）；不满足就直接写一次性 prompt，不要为临时提醒建文件——Skill 库的每一项都摊销触发层的噪音成本。
6. **把 Skill 当知识资产管理**：变更进 code review、定期 review 内容、建立淘汰机制——知识资产会过期，写完即忘是最贵的坑。 [[hermes-agent-skills-source-code-analysis-shuge]]

## 系列上下文

本文是 Claude Agent SDK 系列第 11 篇。系列前 10 篇覆盖：工具（Tool）、权限（Permission）、Session/Compaction、Subagent、MCP。下一篇预告：可观测性（Observability）。^[raw/articles/claude-agent-sdk-skills-reusable-knowledge.md]

## 相关实体

- [[entities/hermes-agent-skills-source-code-analysis-shuge|Hermes Agent Skills 源码分析]]
- [[entities/harness-engineering-core-patterns|Harness Engineering Core Patterns]]
- [[entities/claude-code-skills-superpowers-practice|Claude Code + Superpowers 实践]]
- → [[raw/articles/claude-agent-sdk-skills-reusable-knowledge|原文存档]]
- [[moc/prompt-engineering-guide|MOC]]
