---
title: Claude Code 源码级生命周期解析
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [claude-code, source-leak, agent-harness, query-loop, context-assembly, permission-system, tool-execution, async-generator]
related:
  - [[entities/opensquilla-launches-open-source-ai-agent-to-cut-token-costs|OpenSquilla]]
  - [[entities/multica-managed-agents-platform|Multica]]
description: Claude Code 源码泄露完整解析——8步核心循环、系统提示词缓存分层、CLAUDE.md四级加载、权限四层机制、isConcurrencySafe并发策略。
sources:
  - raw/articles/claude-code-source-leak-lifecycle
---
# Claude Code 源码级生命周期解析

## 核心定位

基于 Claude Code 源码泄露事件（.map 文件未排除，1900+ TypeScript 文件，51万+ 行代码）的完整解析。从输入消息到交付代码的完整生命周期，揭示 **LLM 调用只是一行代码，真正让 Agent 可用的是围绕它精心设计的 Agent Harness**。

本次源码泄露事件与 [[entities/opensquilla-launches-open-source-ai-agent-to-cut-token-costs|OpenSquilla]] 等开源 AI Agent 项目形成鲜明对比：后者选择主动开源并强调透明性，而 Claude Code 则通过泄露事件才让外界得以窥见其内部实现。

## 8步核心循环

```
用户输入 → 组装上下文 → API调用 → 解析响应 → 权限检查 → 执行工具 → 反馈 → 上下文检查 → 终止
              ↓                                                                              ↓
         组装上下文 ←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←←← (太大则压缩)
```

**核心驱动**：`query.ts` 中 `query()` 异步生成器函数，`queryLoop()` 里跑 `while(true)`，每轮一次完整 API 往返。

## 上下文组装

### 两类内容与 SYSTEM_PROMPT_DYNAMIC_BOUNDARY

| 类型 | 内容 | 缓存策略 |
|------|------|----------|
| **缓存部分** | 角色、系统规则、工具说明、环境信息等 | 跨用户共享 API 全局缓存 |
| **非缓存部分** | MCP 服务器指令（动态变化） | 每轮重新计算 |

### CLAUDE.md 四级加载

```
加载顺序（根 → 当前，优先级从低到高）：
/etc/claude-code/CLAUDE.md           (系统级)
~/.claude/CLAUDE.md                 (用户级)
~/.claude/rules/*.md                 (用户级)
<project>/CLAUDE.md                  (项目级)
<project>/.claude/CLAUDE.md          (项目级)
<project>/.claude/rules/*.md         (项目级)
<project>/CLAUDE.local.md            (本地级，gitignore)
```

### 完整上下文组成

| 内容 | 大小 | 说明 |
|------|------|------|
| 系统提示词 | ~15,000 tokens | 角色、规则、工具、环境 |
| CLAUDE.md | ~2,000 tokens | 项目指令 |
| 记忆 | ~500 tokens | 过往会话的相关记忆 |
| MCP 指令 | ~300 tokens | 已连接服务器文档 |
| 对话历史 | ~5,000 tokens | 当前会话前几轮 |
| 用户消息 | ~10 tokens | 实际输入 |

## State 对象：串联每轮循环

每轮循环的 `State` 对象传递状态：

| 状态 | 影响 |
|------|------|
| `state.messages` | 压缩后更新 |
| `state.maxOutputTokensRecoveryCount` | 重试时增加 |
| `state.toolUseContext` | 工具执行后更新 |

## 权限四层机制

```
拒绝规则 → 允许规则 → Bash分类器(最多2秒) → 交互提示
   ↓           ↓              ↓                ↓
  直接拦截    直接放行       自动判断          用户确认
```

**分类器判断**：
- 只读操作（`git status`、`ls`）→ 直接放行
- 有风险操作（`rm`、`git push --force`）→ 用户确认
- 2秒无法判断 → 进入交互提示

## 工具执行：isConcurrencySafe 机制

```typescript
type Tool = {
  execute(input, context): Promise<ToolUseResult>
  isConcurrencySafe(input): boolean  // 关键
  isReadOnly(input): boolean
}
```

**执行策略**：
- 只读操作：并发执行（最多10个并行）
- 写操作：串行执行，避免冲突

## 43个内置工具

| 类别 | 工具 |
|------|------|
| 文件 I/O | FileReadTool, FileEditTool, FileWriteTool, NotebookEditTool |
| 搜索 | GlobTool, GrepTool |
| 执行 | BashTool |
| Web | WebFetchTool, WebSearchTool |
| Agents | AgentTool, SendMessageTool |
| 任务 | TaskCreateTool, TaskGetTool, TaskUpdateTool, TaskListTool |
| 规划 | EnterPlanModeTool, ExitPlanModeTool |
| 技能 | SkillTool |

## 技能预算控制

| 限制项 | 值 |
|--------|-----|
| 所有技能描述占上下文 | ≤ 1%（200K 模型约 2000 tokens）|
| 单个技能描述 | ≤ 250 字符 |
| 超出预算时 | 按需截断，详细定义按需获取 |

## 源码泄露揭示的工程哲学

### 哲学一：缓存是工程问题，不是模型问题

Claude Code 的 `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 揭示了一个核心区分：**模型不知道自己何时应该被缓存，工程系统必须主动管理缓存边界。**

缓存部分（角色、系统规则、工具说明）是跨用户共享的，不依赖具体对话状态。这部分内容可以预计算并全局复用，节省大量 token 成本。非缓存部分（MCP 指令）每轮重新计算，因为它们的值取决于当前运行时环境。

这个设计的工程复杂度在于：开发者必须**预判哪些内容会跨请求复用、哪些内容是请求特定的**。Claude Code 在这里的选择是把"角色+规则+工具"放进缓存池，把"MCP+动态上下文"留作请求级计算。这是一个在 token 成本和工程复杂度之间的务实权衡。

### 哲学二：权限是 UX 问题，不是安全问题

Claude Code 的权限四层机制（拒绝→允许→分类器→交互）说明了一个关键认知：**在个人开发工具场景下，权限系统的主要目标是用户体验，而非安全边界。**

拒绝/允许规则是声明式的，适合明确的全局策略。但现实中的开发操作大多数介于"明确安全"和"明确危险"之间——2秒分类器的设计就是为了处理这类灰色地带：如果操作是只读的就放行，如果可能破坏性的就问用户，如果分类器超时就读用户的意图。

这个设计的巧妙之处在于**把用户变成了分类器的 backstop**，而非把用户变成规则的编写者。大多数普通用户不会写规则，但他们能回答"你刚才是在做 X 吗？"

### 哲学三：并发安全是工具契约，不是全局策略

`isConcurrencySafe` 机制的巧妙之处在于把并发决策权下放给每个工具自己。工具实现者最清楚自己的操作是否线程安全——读文件可能是安全的，但写文件并发执行会导致文件损坏。

这意味着**工具设计者必须显式声明并发安全性**，而不是让框架假设所有工具都遵循同一并发模型。串行执行的工具（如 BashTool 写操作）通过 `isConcurrencySafe: false` 告知框架；读操作工具则声明 `isConcurrencySafe: true` 以启用并行加速。

### 哲学四：Context 是预算，不是存储

CLAUDE.md 的四级加载机制和 token 预算控制揭示了一个核心认知：**Context 是有限预算，不是无限存储。** 技能描述限制在 250 字符、总技能上下文 ≤ 1%，这些都是因为 Claude Code 把 Context 视为需要被管理的资源。

这个哲学导致的设计决策是：不要把大量信息塞进 Context，而是**让工具按需获取信息**。Claude Code 选择给模型搜索工具（Grep）而不是预加载所有代码，就是这个哲学的直接体现。

## 开源 vs. 闭源：两种透明性路径

Claude Code 源码泄露事件与 OpenSquilla、DeepSeek V4 等开源项目形成了鲜明对比，揭示了 AI Agent 领域的两种透明性路径：

| 维度 | Claude Code（泄露式透明） | OpenSquilla/DeepSeek（主动开源） |
|------|--------------------------|----------------------------------|
| 透明度来源 | 非自愿泄露 | 主动公开 |
| 完整性 | 1900+ TypeScript 文件泄露，完整性高 | 权重/代码主动开源，可能有选择性 |
| 可审计性 | 外部可审计内部实现 | 外部可自由使用和修改 |
| 信任建立 | 通过泄露事件"被信任" | 通过开源许可主动建立信任 |
| 商业保护 | 代码不开源，通过泄露才可见 | 部分或全部开源，放弃商业保护 |
| 社区反馈 | 无法与官方直接互动 | 社区可提交 PR 和 Issue |

Claude Code 的泄露式透明有一个独特的不对称性：**外部可以看到内部实现，但无法贡献改进**。而 OpenSquilla 等项目选择开源则允许社区参与改进，但也放弃了部分商业独占性。

DeepSeek V4 的完全开源策略（开源权重 + 训练账单 + 数据配方）与 Claude Code 的闭源核心形成极端对比。前者追求的是"完全可复现"的信任，后者追求的是"产品竞争力"的商业价值。

[[entities/multica-managed-agents-platform|Multica]] 等托管 Agent 平台代表第三种路径：**开放架构但闭源实现**，允许用户使用和部署，但不允许查看和修改核心实现。
## 关键要点

1. **LLM 调用只是一行代码**——真正让 Agent 可用的是 Agent Harness
2. **上下文 = 消息 + 项目规则 + 偏好 + 记忆 + 历史**（不是简单的"你的消息"）
3. **SYSTEM_PROMPT_DYNAMIC_BOUNDARY** 分隔缓存/非缓存内容
4. **CLAUDE.md 四级加载**确保企业/用户/项目/本地指令层层叠加
5. **State 对象串联每轮循环**——每轮决策影响下一轮
6. **权限四层设计**——规则优先（最快），分类器最多等2秒
7. **isConcurrencySafe** 决定只读并发、写入串行
8. **缓存是工程问题**——框架必须主动管理缓存边界，不是模型自己决定
9. **权限是 UX 问题**——四层机制服务于用户体验，而非安全边界
10. **Context 是预算**——不是无限存储，是需要被管理的资源

## 关联

- [[concepts/managed-agents-architecture]] — Anthropic Managed Agents 架构
- [[concepts/harness-engineering-framework]] — Harness Engineering 框架
- [[entities/claude-code-agentic-harness-design-patterns]] — Claude Code 12个可复用 Harness 设计模式

## 关联实体

**上游依赖**:
- [[entities/opensquilla-launches-open-source-ai-agent-to-cut-token-costs]] — 提供基础理论/方法
-  — 提供基础理论/方法

**下游应用**:
- [[entities/multica-managed-agents-platform]] — 具体应用场景
- [[entities/opensquilla-launches-open-source-ai-agent-to-cut-token-costs]] — 具体应用场景

**平行协作**:
- [[entities/multica-managed-agents-platform]] — 替代/补充方案
- [[entities/claude-code-agentic-harness-design-patterns]] — 替代/补充方案

## 所属 MOC

- [[moc/claude-code-complete-guide|Claude Code Complete Guide]]
