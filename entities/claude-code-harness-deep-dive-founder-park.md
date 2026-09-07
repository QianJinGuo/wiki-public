---

title: "Claude Code Harness 深度分析"
type: entity
tags: [agent, claude, context, harness, llm, prompt]
created: 2026-05-21
updated: 2026-09-07
review_value: 7
review_confidence: 7
sources: [raw/articles/claude-code-harness-deep-dive-founder-park]
reviewed: 2026-09-07
review_verdict: hub-retained
review_category: dup
review_note: "judged dup-0.75: harness分析重复版; retained as hub (in-links>=20); MOC rewrite candidate"
---

## TAOR Loop：Orchestrator 越笨越稳定

Claude Code 的核心是 Think-Act-Observe-Repeat（TAOR）循环：模型控制循环决策，运行时只是执行器。Orchestrator 仅约 50 行核心代码，暴露 4 种工具原语：Read/Write/Execute/Connect。Bash 是通用适配器。脚手架应随模型变强而变薄——这是 Claude Code 架构的核心理念 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## Context 管理

Claude Code 实现 Auto-compaction 在 50% 上下文水位时自动触发，由 LLM 摘要替代原始对话而非简单截断。Sub-Agent 使用独立 Context 预算实现隔离。Prompt Cache 经济学涉及 14 种 cache-break 向量。Session Continuity 实现了类似 git branch 的 checkpoint/rollback/fork 机制 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## 六层记忆系统

1. Managed Policy（组织策略） ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]
2. Project CLAUDE.md（项目配置） ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]
3. User Preferences（用户偏好） ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]
4. Auto-Memory（自动学习用户模式） ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]
5. Session（会话上下文） ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]
6. Sub-Agent Memory（子 Agent 专项记忆） ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

记忆是索引不是存储，可自我编辑和去重 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## 五档权限光谱

权限系统从宽松到严格分为五档：plan → default → acceptedEdits → dontAsk → bypassPermissions。底层有 bashSecurity.ts 实现的 23 项安全检查，包括 Unicode 零宽字符注入、IFS null-byte、Zsh equals expansion 等防御 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## 多 Agent 编排

Sub-Agent 使用独立进程/TAOR/Context，提供 3 种预设（Explore/Haiku/Plan/General-purpose）。Agent Teams 是独立实例通过共享文件系统协调（实验性）。KAIROS（未发布）定位为 Always-On Agent，支持 nightly memory distillation、GitHub webhook 和 Cron 5min 调度 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## Anti-Distillation & Undercover

Claude Code 实现了 fake_tool_injection 防御以污染训练数据，connector-text 摘要机制生成 API 推理链摘要+加密签名，Undercover Mode 作为单向门不提及内部代号 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## 深度分析

**1. Thin Harness 作为架构哲学的落地**：Claude Code 的 50 行 Orchestrator + 4 种工具原语是"Thin Harness"理念的极致实践。与 [[entities/thin-harness-fat-skills]] 描述的"~200行轻量框架"一脉相承——脚手架不承载业务逻辑，模型越强框架越薄 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

**2. 六层记忆系统是 Context 工程的完整范式**：Claude Code 的记忆层次覆盖从组织策略到会话上下文的全光谱，且记忆是"索引而非存储"的设计让系统可以主动编辑和去重。这是 [[entities/agentmemory-coding-agent-local-memory]] 讨论的本地记忆系统在产品级实现中的完整形态 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

**3. 五档权限光谱是 Agent 信任分级的基础设施**：从 plan（仅规划）到 bypassPermissions（完全放权），配合 23 项安全检查，是 Agent 安全架构的完整实践。这种权限光谱设计解决了"Agent 应该有多少自主权"这个核心问题 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

**4. Session Continuity 的 git 类比揭示了 Agent 状态管理本质**：checkpoint/rollback/fork 机制将 Agent 的不确定性执行变成了可版本化的确定性状态机。这是解决 Agent 不可靠性（hallucination、drift）的产品级方案，而非简单地在模型层打补丁 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

**5. Anti-Distillation 体现了闭源模型公司的防御性工程**：fake_tool_injection 污染训练数据、connector-text 摘要+加密签名、Undercover Mode 单向门——这些不是功能，而是对竞争性蒸馏的主动防御。这揭示了顶级 AI 公司已将"防止模型被蒸馏"作为产品工程的核心优先级 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## 实践启示

1. **用 TAOR 循环设计你的 Agent 核心**：保持 Orchestrator 足够薄（<100行），将业务逻辑下沉到 Skill/工具层。模型的判断能力应该控制循环，而非被循环淹没 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

2. **在 50% 上下文水位触发压缩而非等到耗尽**：Claude Code 的 auto-compaction 阈值（50%）是工程经验值。主动压缩比被动截断保留更多有效信息，适用于任何长上下文 LLM 应用 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

3. **建立权限分级的显式评估标准**：根据任务风险等级选择对应权限档位——plan 模式用于高风险操作，bypassPermissions 仅用于经过充分测试的确定性任务。不存在"全用最高权限"的合理场景 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

4. **为 Sub-Agent 设计独立 Context 预算**：当多 Agent 协作时，强制每个 Agent 使用独立上下文预算，避免单一 Agent 的上下文膨胀拖垮整个系统。这是多 Agent 系统的必备隔离机制 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

5. **在模型 API 层面防御蒸馏**：如果你的产品输出包含高价值推理过程，考虑实现 connector-text 摘要+签名机制，使外部调用无法获取完整推理链。这在 [[entities/agent-harness-context-management-working-set]] 的安全设计中有类似体现 。 ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

→ [[raw/articles/claude-code-harness-deep-dive-founder-park|原文存档]] ^[raw/articles/claude-code-harness-deep-dive-founder-park.md]

## 相关实体

- [[moc/prompt-engineering-guide|MOC]]
