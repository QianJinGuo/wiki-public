---
title: Hermes Agent Skill
created: 2026-04-28
updated: 2026-08-01
type: concept
tags: [hermes, skill, agent, self-evolution, capability, skill-system]
related:
  - [[entities/hermes-agent|Hermes Agent]]
  - [[entities/agent-skill-writing|Agent Skill 编写指南]]
  - [[concepts/hermes-agent|Hermes-Agent]]
  - [[concepts/skill-formal-theory-survey|Skill 形式化理论]]
  - [[concepts/harness-engineering-framework|Harness Engineering]]
sources: ['raw/articles/agent-tools-research']
confidence: medium
---
# Hermes Agent Skill

> Skill 是 Hermes 自我进化机制的核心载体——让 Agent 把每次成功经验转化为可复用的能力沉淀。

## 相关查询

- [[queries/wechat-inbox-equilibrium-explainer|微信公众号 RSS 订阅最佳实践]] — wechat-inbox 动态平衡机制、RSS 窗口限制与质量过滤策略

## 核心定位

Hermes 的 Skill 系统是连接 **"临时执行"** 与 **"持久能力"** 的桥梁。当 Agent 成功完成一个任务时，Skill 生成机制将其决策路径和执行步骤固化为一个结构化文件，下次遇到同类任务时无需重新学习即可复用。

与单纯的 Tool 不同，Skill 更接近 **"经验模式"** 而非"原子操作"：

| 维度 | Tool | Skill |
|------|------|-------|
| 本质 | 原子功能调用 | 经验模式 + 执行策略 |
| 生命周期 | 静态注册 | 动态生成 + 持续迭代 |
| 复用方式 | 固定参数调用 | 上下文自适应 |
| 存储形式 | 代码/SDK | Markdown 文件 |
| 版本化 | 需独立版本管理 | 内置版本演进 |

## Skill 六元组结构

每个 Skill 文件包含六个核心维度（与 [[concepts/skill-formal-theory-survey|Skill 形式化理论]] 的六元组定义一致）：

```yaml
name: <skill-name>           # 唯一标识，文件名前缀
description: <intent>         # 意图描述，用于触发识别
trigger: <condition>          # 触发条件（任务模式/关键词/参数约束）
action: <sequence>            # 执行序列（工具调用链）
feedback: <evaluation>         # 结果评估（成功标准/回退策略）
metadata: <version, author>   # 版本、来源、质量评分
```

### 触发条件（Trigger）

Skill 的触发发生在两个层次：

1. **显式触发** — 用户用 Skill 名称或同义词直接请求
2. **隐式触发** — 记忆系统检测到当前任务与某 Skill 的 `trigger` 模式匹配

隐式触发的准确性依赖记忆系统的索引质量，也是 Hermes 的核心能力边界之一。

### 执行序列（Action）

Action 不是简单的工具调用链，而是**带条件分支的决策图**：

```yaml
action:
  - step: 分析任务类型
    condition: IF domain == "code" THEN goto: 分析代码结构
    condition: IF domain == "research" THEN goto: 搜索相关资料
  - step: 分析代码结构
    tools: [read_file, grep, search_files]
    fallback: 降级为粗粒度文件遍历
```

### 结果评估（Feedback）

Feedback 决定 Skill 是否值得保留和复用：

```yaml
feedback:
  - metric: 任务完成率
    threshold: > 80%
    action: IF 低于阈值，记录失败案例用于 Skill 迭代
  - metric: 执行时间
    threshold: < 预期 × 1.5
    action: IF 超过阈值，尝试优化执行路径
```

## Skill 生成管道

当 Hermes 执行任务时，Skill 生成管道会经历：

```
任务执行 → 轨迹记录 → 模式挖掘 → Skill 合成 → 质量验证 → 激活
```

### 轨迹记录

每次任务执行都会生成一个 **轨迹**（trajectory），包含：

- 输入（任务描述 + 上下文）
- 决策点（每个分支选择的原因）
- 工具调用序列
- 输出（成功/失败 + 结果质量）
- 执行时间、token 消耗

轨迹是 Skill 生成的原材料，参见 [[entities/trace2skill-trajectory-distillation-agent-skills|Trace2Skill 轨迹蒸馏]]。

### 模式挖掘

轨迹记录积累后，模式挖掘从多个轨迹中识别：

- **共性路径** — 同一类型任务的通用执行步骤
- **关键决策点** — 导致成功 vs 失败的分叉点
- **参数模式** — 有效参数的分布和约束条件

### Skill 合成

模式挖掘的输出通过 LLM 合成 Skill 文件：

```python
def synthesize_skill(trajectories: list[Trajectory]) -> Skill:
    # 1. 从共性路径生成 action 序列
    action = extract_common_path(trajectories)
    # 2. 从关键决策点生成 trigger 条件
    trigger = generalize_trigger(trajectories)
    # 3. 从成功案例提取 feedback 标准
    feedback = derive_quality_metrics(trajectories)
    return Skill(action=action, trigger=trigger, feedback=feedback)
```

## Skill 的版本演进

Skill 不是一次性生成的，而是随使用持续迭代：

| 版本阶段 | 触发条件 | 主要变化 |
|---------|---------|---------|
| v0.1 (初始版) | 首次成功执行 | 基础路径，窄触发条件 |
| v0.2 (验证版) | 3+ 次成功执行 | 拓宽触发条件，增加 fallback |
| v1.0 (稳定版) | 20+ 次执行，>90% 成功率 | 完整 error handling |
| v1.x (迭代版) | 每次执行后更新 | 微调参数、优化路径 |

## Skill 与 Harness 的关系

Skill 位于 [[concepts/harness-engineering-framework|Harness Engineering]] 的**工具层**，是 Harness 的可替换组件：

```
Harness
├── 上下文管理 (Context Engineering)
├── 反馈回路 (Feedback Loop)
├── 人类监督 (Human in Loop)
└── 工具层
    ├── Tool (原子操作)
    └── Skill (经验模式) ← Hermes 的核心竞争力
```

Skill 的替换成本远低于 Tool：Skill 是 Markdown 文件，可以由 Agent 自我修改；Tool 需要代码变更和测试。

这也呼应了 **"Thin Harness, Fat Skills"** 原则：Harness 本身应该尽量轻薄，把业务逻辑封装在 Skill 层。

## 常见 Skill 反模式

### 过度泛化

Skill 的 trigger 条件过于宽松，导致在不该触发的场景被激活：

```yaml
# 反模式：trigger 太宽
trigger: 任何包含 "代码" 的任务

# 正确：trigger 应该精确
trigger: |
  domain == "code" AND
  task_type IN ["refactor", "test_generation", "bug_fix"] AND
  context_length > 500 tokens
```

### 缺乏 Feedback

没有 feedback 定义的 Skill 无法自我改进，最终会积累错误：

每个 Skill 必须定义至少一个可量化的 feedback metric，否则会被标记为"低质量 Skill"并降级。

### 硬编码路径

Action 中的工具调用过于硬编码，不适应不同环境：

```yaml
# 反模式：硬编码绝对路径
action:
  - run: /usr/local/bin/lint --config /home/user/.eslintrc

# 正确：使用环境无关的描述
action:
  - run: 代码质量检查
    tool: linter
    auto_discover: true
```

## 相关实体

- [[entities/hermes-agent|Hermes Agent]] — Skill 系统运行所在的 Agent 平台
- [[entities/agent-skill-writing|Agent Skill 编写指南]] — Skill 通用编写规范
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2|Qoder Skills 完全指南]] — Skill 编写最佳实践
- [[entities/thin-harness-fat-skills|Thin Harness Fat Skills]] — Harness/Skill 职责划分原则
- [[entities/trace2skill-trajectory-distillation-agent-skills|Trace2Skill 轨迹蒸馏]] — 轨迹到 Skill 的自动化转化
- [[entities/perplexity-internal-skill-design-guide|Perplexity 内部 Skill 设计指南]] — 四维 Skill 体系
- [[entities/skillclaw|SkillClaw]] — Skill 框架对比
- [[entities/hermes-skill-system-winty|Skill 系统：Agent 经验沉淀]] — Hermes Skill 系统深度解析
- [[entities/skill-development-guide-aliyun-2026|阿里云 Skill 开发指南]] — 一站式 Skill 开发与发布

## Skill 生态横向对比

不同平台的 Skill 系统设计哲学：

| 平台 | Skill 本质 | 生成方式 | 版本策略 | 与 Tool 的关系 |
|------|-----------|---------|---------|--------------|
| Hermes | 经验模式（Markdown） | 自动轨迹蒸馏 | 持续迭代 | Skill ⊃ Tool（Tool 是原子 Skill） |
| Claude Code | 工具定义（TS） | 手工编写 | 版本化 | Tool 是独立实体 |
| OpenClaw | Skill 文件（Markdown） | 半自动生成 | 静态 | Tool + Skill 并列 |
| Qoder | 操作规程（YAML） | 手工编写 | 标签化 | Tool + Skill 融合 |

Hermes 的独特优势：**Skill 生成和迭代完全自动化**，不需要人工干预。而 Claude Code 的工具需要 TypeScript 编写、测试、发布的完整工程流程。

## Skill 与记忆系统的协同

Skill 的生成和激活都依赖记忆系统，这是 Hermes 的核心设计：

```
记忆系统（向量检索 + 层级衰减）
    ↓                    ↑
Skill 触发           Skill 生成
    ↓                    ↑
任务执行 ←————————————轨迹记录
```

### Skill 触发依赖记忆索引

当用户发送任务时，记忆系统的检索流程：

1. **语义检索** — 从向量数据库检索与当前任务语义相似的历史任务
2. **Skill 候选** — 如果某 Skill 的 trigger 与检索结果匹配，激活该 Skill
3. **上下文注入** — Skill 的 action 序列和参数 Schema 被注入到 Agent 上下文

如果记忆索引质量差（向量嵌入不准确、历史数据不足），Skill 触发的准确率会显著下降。

### Skill 生成依赖轨迹积累

Skill 的生成需要同类型任务的多个轨迹作为输入：

- **轨迹数量不足**（< 3 次成功执行）→ 无法提炼共性模式
- **轨迹多样性不足**（参数分布太窄）→ Skill 的 trigger 过于具体
- **轨迹质量不一**（有些失败）→ Skill 的 feedback 需要更细致的阈值定义

这也是为什么 Hermes 的价值需要**时间积累**——新部署的 Hermes 没有足够的轨迹来生成有价值的 Skill。

### 冷启动问题

新部署 Hermes 的 Skill 数量为 0，需要一个"暖场期"：

| 阶段 | 时间 | Skill 数量 | 主要来源 |
|------|------|-----------|---------|
| 冷启动 | 第 1 周 | 0-3 | 内置默认 Skill |
| 暖场期 | 第 2-4 周 | 3-10 | 手工编写 + 简单模式生成 |
| 成长期 | 第 2-3 月 | 10-50 | 自动轨迹蒸馏 |
| 成熟期 | 3+ 月 | 50+ | Skill 互相调用、Skill 版本迭代 |

冷启动阶段建议：手工编写高频场景的 Seed Skill（如"代码审查"、"会议纪要"），让 Hermes 快速形成基础能力圈。

## 关联实体

**上游依赖**:
- [[entities/hermes-agent]] — 提供基础理论/方法
- [[entities/agent-skill-writing]] — 提供基础理论/方法
- [[entities/trace2skill-trajectory-distillation-agent-skills]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-skill-writing]] — 具体应用场景
- [[entities/qoder-skills-完全指南从零开始让-ai-按你的标准执行-v2]] — 具体应用场景
- [[entities/thin-harness-fat-skills]] — 具体应用场景

**平行协作**:
- [[entities/perplexity-internal-skill-design-guide]] — 替代/补充方案
- [[entities/skillclaw]] — 替代/补充方案
- [[entities/hermes-skill-system-winty]] — 替代/补充方案

## 所属 MOC

- [[moc/ai-skill-design|Ai Skill Design]]
