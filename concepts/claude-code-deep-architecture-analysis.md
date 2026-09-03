---
title: Claude Code 架构深度分析
created: 2026-05-07
updated: 2026-08-30
type: concept
tags: [claude-code, architecture, agent-harness]
sources: ['raw/articles/claude-code-deep-architecture-analysis']
confidence: high
---
# Claude Code Architecture: Deep Technical Analysis
> Source-level deep dive into Claude Code's architecture: batch tool concurrency with isConcurrencySafe, deferred loading with deferred_tools_delta prefix cache optimization, permission-system-level Plan mode enforcement, multi-layer context compression, git-state injection vs. directory tree injection, and four-framework horizontal comparison.
---
## Architecture Overview
Claude Code's framework design philosophy: **permission-system-level enforcement > prompt-level trust**, and **dynamic git context > static directory snapshots**.
---
## Tool Concurrency: Batch Execution Strategy
**Rule**: Strict order, no cross-batch concurrency.
Model outputs `[Glob, Grep, Read, FileEdit, Glob, Read]`:
- **Batch 1**: [Glob, Grep, Read] — concurrent
- **Batch 2**: [FileEdit] — serial (write operation)
- **Batch 3**: [Glob, Read] — concurrent
**Max concurrency**: Default 10, configurable via `CLAUDE_CODE_MAX_TOOL_USE_CONCURRENCY`.
**Design implication**: Model must issue multiple read-only tool calls in single response to utilize concurrency. Single-tool-per-response patterns get no benefit.
## Deferred Loading: shouldDefer + ToolSearch
### Mechanism
Tools with `shouldDefer: true` (e.g., `EnterPlanMode`) and all MCP tools have **zero schema in initial request** — only a `defer_loading: true` marker with no parameter descriptions.
### ToolSearch Scoring
Each deferred tool declares a `searchHint` field (3-10 word capability description):
| Match Type | Score |
|-----------|-------|
| `searchHint` hit | **4** |
| Tool name hit | **2** |
| Full prompt description hit | **1** |
Results sorted by score. Return format: `tool_reference` blocks (not plain text).
### Prefix Cache: deferred_tools_delta
**Problem**: Discovered tool set grows over session. Inserting dynamic "currently discovered tools list" into message stream changes message sequence → prefix cache misses every time the tool pool changes.
**Old approach** (broken): Insert `<available-deferred-tools>` block before first user message → cache invalidates on every tool discovery.
**Current solution**: Tool discovery info sent via **independent attachment** — does not modify message stream. System prompt and message history stay stable. prefix stays constant, cache keeps hitting.
**Compact boundary**: Framework snapshots current discovered tool set in compact boundary metadata → resume after compression preserves discovery state.
## Tool Result Size Control
Each tool declares `maxResultSizeChars`.调度层 checks result size:
- **Over limit** → result persists to disk, model receives file path reference instead of content
- **Read tool**: set to `Infinity` (would cause read→path→read死循环)
`contentReplacementState` tracks replaced content across turns; can be restored after Compact.
## Permission System
Each tool has `checkPermissions()`. `canUseTool` combines:
- Current permission mode
- alwaysAllow/alwaysDeny rules
- Hooks system
**Three outcomes**: Auto-allow / Ask user / Block.
## Repository Directory Tree: Git State Injection vs. Static Tree
### Framework Comparison
| Framework | Approach | Detail |
|-----------|----------|--------|
| **Claude Code** | Git state injection, on-demand exploration | User context prefix updated every turn; no static tree |
| **Codex** | Auto-generate 2-level directory tree | User prompt, ≤20 entries per directory |
| **OpenCode** | Hardcoded disabled | `&& false` forced skip |
| **Gemini-CLI** | Git workflow guidance injection | System prompt, static |
### Claude Code's Design Rationale
> "Directory tree is a static snapshot. Git state is a dynamic execution context. For programming tasks, 'which files were just changed' is more decision-relevant than 'which files exist in the directory.'"
Cost: Models unfamiliar with on-demand exploration pattern may need extra tool calls in round 1 to establish codebase understanding.
## Plan Mode: Permission-System Enforcement
### Claude Code vs. Other Frameworks
| Aspect | Claude Code | Others |
|--------|-----------|--------|
| Enforcement level | **Permission system** (`toolPermissionContext.mode = 'plan'`) | Prompt instruction ("be read-only") |
| Write operation blocking | Blocked at permission check | Relies on model compliance |
| Tool discovery | Must ToolSearch first | Direct availability |
| Sub-Agent Plan mode | **Forbidden** (throws exception) | N/A |
### Enter/Exit Flow
```
EnterPlanMode (tool, deferred, must ToolSearch)
    ↓
toolPermissionContext.mode = 'plan'
    ↓
Model explores, calls ExitPlanMode
    ↓
Plan written to .claude/plans/<id>.md
    ↓
UI approval dialog → User clicks Approve
    ↓
Restore to original permission mode
    ↓
Execution phase (references approved plan)
```
**Only mandatory user intervention point** in Claude Code's entire flow.
## Context Compression: Multi-Layer Mechanism
Claude Code has the most sophisticated compression system among the four frameworks.
### Dynamic Trigger Threshold
Compression trigger threshold dynamically binds to current model's context window:
```
trigger = context_window - 20K (output reserve) - 13K (buffer)
```
Example: claude-sonnet (200K context) → auto-compress triggers at ~167K.
### Layers
Multiple compression passes run before each LLM call, from light tool result trimming to full summary rewriting.
## 工具并发模型对比：Claude Code vs OpenClaw 的设计哲学分歧
Claude Code 和 OpenClaw（见 [[entities/openclaw-architecture-8-part-summary|OpenClaw 架构解析]]）在工具并发问题上的设计选择代表了两种截然不同的哲学立场，理解这个分歧对于设计 Agent 工具系统有重要意义。

Claude Code 的批量并发模型（同一 batch 内并行执行 read-only 工具，严格按 batch 顺序串行写操作）是一种**保守确定性策略**：它不追求最大并行度，而是确保工具执行的先后顺序与 LLM 输出顺序一致，从而避免因并发写导致的非确定性结果和状态不一致。batch 间的顺序保证使得整个执行历史是线性的、可重现的。

OpenClaw 的 `ToolRegistry.execute(name, args)` 则是**原子命令模式**：每个工具调用是独立原子的，工具之间的执行顺序完全由 LLM 的输出顺序决定（通过 `AgentLoop.run()` 逐个执行 tool_calls 数组）。这种模型的优势是实现极简，劣势是当 LLM 输出乱序或交叉的多个写操作时，文件系统状态可能处于中间态而非最终态。

从 [[entities/ai-agent-tool-count-trap|AI Agent 工具数量陷阱]] 的角度来看，两种设计都面临同样的根本问题：随着工具数量增加，LLM 正确选择工具的概率下降。Claude Code 的解决方案是 deferred loading + ToolSearch（让模型按需搜索工具而非预知所有工具），OpenClaw 的解决方案是通过 `exclude()` 限制子 Agent 的工具集（缩小选择范围）。前者是动态发现，后者是静态裁剪——前者更灵活但依赖模型的 searchHint 理解能力，后者更安全但需要主 Agent 精确编排子 Agent 的能力边界。
## 权限系统作为 Agent 安全的核心防线：从 Claude Code 到更广泛的设计
Claude Code 的 Plan Mode 权限系统级实施（`toolPermissionContext.mode = 'plan'`）是将安全边界从软件工程层面提升到 Agent 系统层面的典型案例。这个设计值得与 [[entities/harness-engineering|Harness Engineering]] 中的 guardrails 概念进行对比分析。

传统软件的安全模型是"先验证后执行"：任何有副作用的操作（文件系统写入、网络调用、进程执行）在执行前必须经过权限检查。但在纯 prompt-based Agent 中，这个检查链被简化为"模型是否会遵从 system prompt 中的禁止指令"——这是一个概率性保证而非确定性保证。Claude Code 的权限系统将这个保证重新工程化：在 Plan Mode 下，文件写入工具的 `checkPermissions()` 直接返回 block，而非依赖模型的"自律"。

这与 [[entities/openclaw-security-and-feature-enhancement-practices|OpenClaw 安全实践]] 中描述的安全模型形成有趣的互补：OpenClaw 面临的安全挑战是 Agent 拥有执行权限后的自主行为控制（防 prompt injection、防文件系统滥用），而 Claude Code 的 Plan Mode 解决的是"用户明确要求 AI 不要写文件时如何强制执行"。两者从不同方向逼近同一个问题：如何确保 Agent 的实际行为与用户意图一致。

从工程实现看，Claude Code 的权限系统有以下几个值得借鉴的设计原则：
1. **权限模式与执行上下文绑定**：`toolPermissionContext.mode` 是线程局部的状态，切换模式不改变工具代码本身，只改变权限判断结果
2. **权限模式是互斥的**：Plan Mode 和普通执行模式之间是严格的二阶段状态机，不存在混合状态
3. **写操作的权限检查是强制性的**：文件编辑类工具即使在 Plan Mode 之外，如果 permission mode 不是 `full`，仍可能被 block

这些原则对于设计企业级 Agent 控制系统具有直接参考价值：当 Agent 需要在生产环境中操作关键业务系统时，"信任但验证"（trust but verify）的权限分层比单纯依赖 system prompt 的"请只读"要可靠得多。
## Context Compression: Multi-Layer Mechanism
Claude Code has the most sophisticated compression system among the four frameworks.^[raw/articles/claude-code-deep-architecture-analysis.md:503-600]
### Dynamic Trigger Threshold
Compression trigger threshold dynamically binds to current model's context window: `context window - 20K (output reserve) - 13K (buffer)`. Example: claude-sonnet (200K context) → auto-compress triggers at ~167K.^[raw/articles/claude-code-deep-architecture-analysis.md:510-518]
### Five-Layer Compression
1. **Tool Result Budget (applyToolResultBudget)**: Lightest, runs every turn. Over-limit results written to disk, replaced with file path reference.^[raw/articles/claude-code-deep-architecture-analysis.md:525-529]
2. **History Truncation (snipCompact)**: No LLM call, scores messages by rules. Low-score messages marked "deletable". Returns `tokensFreed` for subsequent threshold calculation.^[raw/articles/claude-code-deep-architecture-analysis.md:531-536]
3. **Micro-Compression (microCompact)**: Uses Anthropic API's `cache_edits` parameter to delete old tool results **server-side**. Two modes: time-triggered (60min cache TTL) and hot-cache mode (attention mask屏蔽). Local messages stay unchanged.^[raw/articles/claude-code-deep-architecture-analysis.md:538-570]
4. **Context Collapse**: Folds old conversations into summaries but **preserves recent raw granularity**. Trigger: ~90% context preparation, 95% blocking. Mutually exclusive with autoCompact.^[raw/articles/claude-code-deep-architecture-analysis.md:572-586]
5. **Full Summary Compression (autoCompact)**: Heaviest, fork sub-agent to generate full conversation summary via LLM. PreCompact hooks → preprocess → fork agent → replace with boundary + summary + preserved tail + re-injected attachments → PostCompact hooks.^[raw/articles/claude-code-deep-architecture-analysis.md:588-598]
## Sub-Agent System: Seven Execution Modes
Claude Code's sub-agent system uses single entry point `AgentTool` for unified management — **most complete and versatile** among the four frameworks.^[raw/articles/claude-code-deep-architecture-analysis.md:629-636]
| Mode | Trigger | Characteristics |
|------|---------|----------------|
| Sync Foreground | `run_in_background: false` (default) | Blocks until result |
| Async Background | `run_in_background: true` | Returns agent ID immediately, poll via `TaskOutput` |
| Auto-Background | Runtime > 120 seconds | Auto-switch, notify user |
| Worktree Isolation | `isolation: 'worktree'` | Creates temporary git worktree, isolated file operations |
| Remote Execution | `isolation: 'remote'` | Cloud environment, always background |
| Fork Mode | `subagent_type` omitted (experimental) | Inherits parent's complete conversation history and system prompt |
| Teammate Mode | `agent swarms` enabled | Independent tmux session, bidirectional via `SendMessage` |^[raw/articles/claude-code-deep-architecture-analysis.md:639-671]
### Built-in Agent Types
| Type | Tool Set | Use Case |
|------|----------|----------|
| general-purpose (default) | All tools (except AgentTool) | General complex tasks |
| Explore | Read-only (Read/Grep/Glob/WebSearch) | Codebase exploration |
| Plan | Read-only + ExitPlanMode | Planning phase |
| claude-code-guide | Read/Grep/WebSearch | Answering Claude Code usage questions |
| Custom | YAML file definition | Project-specific scenarios |^[raw/articles/claude-code-deep-architecture-analysis.md:673-696]
### Parent-Child Context Sharing
Regular sub-agent clones from parent: file read cache, tool result budget state, permission context, MCP connection info. Fork mode additionally inherits **complete conversation history** and **byte-exact system prompt copy** (ensuring prompt cache hit).^[raw/articles/claude-code-deep-architecture-analysis.md:698-710]
## Failure Handling
Claude Code's failure handling is **lax by default**: tool execution failures don't interrupt flow — error info goes directly to model for self-decision. No tool failure count budget.^[raw/articles/claude-code-deep-architecture-analysis.md:739-740]
### Tool Execution Errors
Single tool failure doesn't affect other tools in same batch — failures and successes both returned to model. Model decides whether to retry.^[raw/articles/claude-code-deep-architecture-analysis.md:742-747]
### API Error Recovery
- **Output token limit**: Auto-retry up to 3 times ^[raw/articles/claude-code-deep-architecture-analysis.md:751-753]
- **Request too long (prompt_too_long)**: Trigger autoCompact first, then retry; if compact request itself exceeds limit, strip oldest messages from head, up to 3 rounds ^[raw/articles/claude-code-deep-architecture-analysis.md:755-757]
- **Network/API failures**: `withRetry()` with exponential backoff ^[raw/articles/claude-code-deep-architecture-analysis.md:759-763]
### Permission Denial Progressive Escalation
Unique to Claude Code: When **3 consecutive denials** or **20 cumulative denials** occur, `shouldFallbackToPrompting()` returns true, switching from "auto-deny" to "ask user" mode. Async sub-agents can't show UI, so denial state maintained separately (`localDenialTracking`) — can only fail continuously, cannot self-escalate.^[raw/articles/claude-code-deep-architecture-analysis.md:765-780]
### Loop Detection
Claude Code **has no built-in loop detection**. Difference from OpenCode (same tool+params 3x → ask user) and Gemini-CLI (inject recovery prompt → 60s → terminate). Only safeguards: autoCompact when context near limit + user manual ESC interrupt.^[raw/articles/claude-code-deep-architecture-analysis.md:782-787]
## Hooks System (Claude Code Unique)
Hooks is one of Claude Code's most distinctive capabilities — key design for positioning as "extensible platform" over pure CLI tool. Opens external intervention interfaces at critical lifecycle points.^[raw/articles/claude-code-deep-architecture-analysis.md:816-817]
### Capabilities
Hook is shell script/callback registered on specific events. Receives event context (tool name, args, results), returns JSON decision via stdout:^[raw/articles/claude-code-deep-architecture-analysis.md:819-866]
- **Block tool call**: `PreToolUse` returns `decision: 'block'`
- **Modify tool input**: `PreToolUse` returns `updatedInput`
- **Inject context**: Returns `additionalContext` to model
- **Modify tool output**: `PostToolUse` returns `updatedMCPToolOutput`
- **Replace initial message**: `SessionStart` returns `initialUserMessage`
- **Terminate session**: Any hook returns `continue: false`
- **Automated permission decisions**: `PermissionRequest` hook returns allow/deny directly
### 24 Hook Events
| Category | Events |
|----------|--------|
| Session | SessionStart / SessionEnd / Setup / Stop / StopFailure |
| Tool | PreToolUse / PostToolUse / PostToolUseFailure |
| Permission | PermissionRequest / PermissionDenied |
| Sub-Agent | SubagentStart / SubagentStop / TeammateIdle |
| User Interaction | UserPromptSubmit / Notification |
| Compression | PreCompact / PostCompact |
| Task | TaskCreated / TaskCompleted |
| System | ConfigChange / CwdChanged / FileChanged / InstructionsLoaded |
| MCP | Elicitation / ElicitationResult |^[raw/articles/claude-code-deep-architecture-analysis.md:868-888]
### Configuration & Priority
Three levels: Enterprise Managed Policy > User-level (`~/.claude/settings.json`) > Project-level (`.claude/settings.json`). `--no-hooks` flag disables all hooks.^[raw/articles/claude-code-deep-architecture-analysis.md:890-899]
### Impact on Benchmarking
Hooks are potential confounders for benchmarking fairness: PreToolUse can modify tool inputs, PostToolUse can modify MCP tool outputs, UserPromptSubmit can silently inject additional context. Use `--no-hooks` mode when benchmarking Claude Code.^[raw/articles/claude-code-deep-architecture-analysis.md:901-910]
## CLAUDE.md Memory System
### Mechanism
CLAUDE.md is Claude Code's persistent memory carrier. On session start, automatically scans and loads in priority order:^[raw/articles/claude-code-deep-architecture-analysis.md:913-932]
1. `~/.claude/CLAUDE.md` — Global user-level, cross-project
2. Project root `CLAUDE.md` — Project-level
3. Current working directory `CLAUDE.md` — Directory-level
4. All parent directories from current to project root — Recursive upward

Deeper directories injected later, overriding shallower rules on conflict. `InstructionsLoaded` hook triggers on each CLAUDE.md load. After autoCompact, CLAUDE.md content re-injected as attachment.^[raw/articles/claude-code-deep-architecture-analysis.md:933-935]
### Typical Uses
- **Project constraints**: Code style rules, forbidden directories
- **Common commands**: build/test/lint commands
- **Architecture notes**: Module responsibilities, dependencies
- **Team norms**: Commit message format, PR process
- **Cross-session memory**: Model actively writes CLAUDE.md to record discoveries for next session ^[raw/articles/claude-code-deep-architecture-analysis.md:937-958]
## State Persistence & Session Recovery
### Session Storage
Claude Code persists complete session transcript as JSONL to `~/.claude/projects/<project_hash>/`. Record types include:^[raw/articles/claude-code-deep-architecture-analysis.md:1106-1146]
- `user/assistant` messages (with thinking blocks and tool calls)
- `system` (subtype: `compact_boundary`) compression boundary marker
- `system` (subtype: `microcompact_boundary`) micro-compression boundary
- `summary` autoCompact generated summary
- `content-replacement` tool result disk replacement records
- `file-history-snapshot` file modification history
- `worktree-state` worktree state records

**System prompt not stored in transcript** — dynamic assembly parts regenerated on resume.
### Resume Process (`/resume`)
1. Determine read range: Find last `compact_boundary`, load only messages after boundary
2. Rebuild conversation chain: Build message DAG via `parentUuid`
3. Restore application state: content-replacement, contextCollapse archive, file-history-snapshot, TodoWrite, worktree-state
4. Re-inject dynamic content: System prompt reassembled, attachments re-injected ^[raw/articles/claude-code-deep-architecture-analysis.md:1172-1213]
## MCP Protocol Integration
MCP (Model Context Protocol) is Anthropic's open protocol. Claude Code is **the only framework among the four with complete native MCP implementation**.^[raw/articles/claude-code-deep-architecture-analysis.md:1215-1220]
### Dynamic Tool Extension
MCP server tools appear as `mcp__<serverName>__<toolName>`, sharing same mechanisms as built-in tools: concurrency scheduling, permission checks, result size limits. MCP tools default to deferred loading; can mark `alwaysLoad = true` for immediate availability.^[raw/articles/claude-code-deep-architecture-analysis.md:1222-1232]
### Resource Access
MCP servers also provide **resources** (files, database records, API responses). `ListMcpResources` lists available resources, `ReadMcpResource` reads content.^[raw/articles/claude-code-deep-architecture-analysis.md:1234-1243]
### Auth & Interaction
`McpAuth` handles OAuth flows. MCP Elicitation protocol allows servers to request user input during execution, routed via `Elicitation`/`ElicitationResult` hooks.^[raw/articles/claude-code-deep-architecture-analysis.md:1245-1251]
## Budget Management
Claude Code splits budget management into **four independent control dimensions**.^[raw/articles/claude-code-deep-architecture-analysis.md:1262-1267]
| Dimension | Description |
|-----------|-------------|
| Token Budget | `output_config.task_budget` sets total token budget for entire agentic turn |
| Cost Budget | `maxBudgetUsd` sets maximum USD cost per session |
| Tool Result Budget | Each tool declares `maxResultSizeChars`; over-limit stored to disk, replaced with path reference |
| Max Turns | `maxTurns` limits maximum Agent loop iterations |
## Framework Comparison Summary
| Feature | Claude Code | Codex | OpenCode | Gemini-CLI |
|---------|------------|-------|----------|-----------|
| Tool concurrency | Auto (isConcurrencySafe batched) | No (manual batch, 1-25) | Auto | Auto |
| Deferred loading | ✅ (shouldDefer + ToolSearch) | ❌ | ❌ | ❌ |
| Tool result size limit | ✅ (disk persist + path ref) | Truncate (head+tail) | ❌ | ❌ |
| LSP tool | ✅ | ❌ | ✅ | ❌ |
| Semantic code search | ❌ | ❌ | ✅ (Exa Code) | ❌ |
| Plan mode enforcement | Permission system | Prompt | Reasoning budget switch | Tool whitelist |
| Sub-Agent Plan mode | Forbidden | N/A | N/A | N/A |
| Git state injection | ✅ (every turn) | ❌ | ❌ | ✅ (static) |
## Related Concepts
- [[concepts/claude-code-source-leak-lifecycle]] — Source code lifecycle analysis (8-step queryLoop / CLAUDE.md 4-level loading)
- [[concepts/claude-code-tool-design-evolution]] — Tool design evolution (AskUserQuestion / TodoWrite→Task)
- [[concepts/managed-agents-architecture]] — Anthropic's managed Agents architecture for comparison
- [[entities/claude-code-architecture|Claude Code 架构解析]]
- [[entities/claude-code-architecture-analysis|Claude Code 设计原则与对照分析]]
- [[entities/claude-code-source-architecture|Claude Code 源码拆解：从启动到多 Agent 扩展层]]
- [[entities/ai-agent-tool-count-trap|AI Agent 工具数量陷阱]] — 工具并发模型与数量规模对 LLM 决策质量的影响
- [[entities/openclaw-security-and-feature-enhancement-practices|OpenClaw 安全与功能增强实践]] — Agent 安全模型对照
- [[entities/openclaw-architecture-8-part-summary|OpenClaw 架构解析]] — 薄抽象 + 显式控制流的设计哲学

## 关联实体

**上游依赖**:
- [[entities/openclaw-architecture-8-part-summary]] — 提供基础理论/方法
- [[entities/ai-agent-tool-count-trap]] — 提供基础理论/方法
- [[entities/harness-engineering]] — 提供基础理论/方法

**下游应用**:
- [[entities/openclaw-security-and-feature-enhancement-practices]] — 具体应用场景
- [[entities/claude-code-architecture]] — 具体应用场景
- [[entities/claude-code-architecture-analysis]] — 具体应用场景

**平行协作**:
- [[entities/claude-code-source-architecture]] — 替代/补充方案
- [[entities/ai-agent-tool-count-trap]] — 替代/补充方案
- [[entities/openclaw-security-and-feature-enhancement-practices]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
