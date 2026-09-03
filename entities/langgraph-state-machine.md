---

title: "LangGraph 状态机执行引擎深度解析"
created: 2026-04-27
updated: 2026-08-29
type: entity
tags: [langgraph, state-machine, agent-framework, langchain, react, execution-engine, fan-out-fan-in, checkpoint]
sources: [raw/articles/langgraph-state-machine-under-the-hood]
review_value: 8
review_confidence: 7
---

## Overview
LangGraph 通过**状态机**机制让 LLM 实现「想一步、停一步、判断一步」的能力。核心是将 Agent 行为建模为 State + Node + Edge 的图结构，由调度器驱动循环执行。   ^[raw/articles/langgraph-state-machine-under-the-hood.md]
与 LangChain 早期线性 Chain 的根本区别：LangChain 控制流固定，LangGraph 支持**带状态的循环控制流**，能根据运行时状态动态决策下一步。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]

## 核心概念解析
### StateGraph 三要素
| 要素 | 作用 | LangGraph 表示 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
|------|------|---------------| ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| **State** | 存数据：对话历史 + 工具结果 + 中间变量 | `Annotation.Root({...})` 定义的不可变快照 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| **Node** | 执行动作：LLM 调用 / 工具执行 / 业务逻辑 | `addNode("name", function)` | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| **Edge** | 决定走向：普通跳转 / 条件分支 | `addEdge()` / `addConditionalEdges()` | ^[raw/articles/langgraph-state-machine-under-the-hood.md]

### StateGraph 内部核心数据结构
``` ^[raw/articles/langgraph-state-machine-under-the-hood.md]
nodes: Map<名称, 函数> ^[raw/articles/langgraph-state-machine-under-the-hood.md]
edges: Map<from, to[]> ^[raw/articles/langgraph-state-machine-under-the-hood.md]
conditional_edges: Map<from, (state)=>节点名> ^[raw/articles/langgraph-state-machine-under-the-hood.md]
channels/schema: 每个状态字段的 Reducer ^[raw/articles/langgraph-state-machine-under-the-hood.md]
``` ^[raw/articles/langgraph-state-machine-under-the-hood.md]

### compile() 的意义
`compile()` 把声明式的图描述变成可执行的调度引擎： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
1. 验证图结构（孤立节点？到 END 的路径？） ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
2. 构建邻接表（预计算每个节点的后继） ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
3. 初始化 Channels（每个状态字段注册 Reducer） ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
4. 返回 CompiledGraph（可 invoke/stream） ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

## Reducer 机制
**核心：节点返回的是「更新片段」而非「完整新状态」。** ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
三种常见 Reducer 模式： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
| 模式 | 写法 | 用途 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
|------|------|------| ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| **追加** | `(prev, next) => [...prev, ...next]` | messages 消息追加 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| **覆盖** | `(_, next) => next` | step、count 等单值覆盖 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| **累加** | `(prev, next) => prev + next` | callCount 等计数场景 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
**State 是不可变的**，每次节点执行产生新快照，旧快照被保留——这是 Checkpoint 实现的基础。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]

## 调度器：事件循环
调度器本质是事件循环： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
```python ^[raw/articles/langgraph-state-machine-under-the-hood.md]
while 当前节点 != END: ^[raw/articles/langgraph-state-machine-under-the-hood.md]
    1. 执行当前节点 node(state) → partial_update ^[raw/articles/langgraph-state-machine-under-the-hood.md]
    2. 用 Reducer 合并状态 → new_state ^[raw/articles/langgraph-state-machine-under-the-hood.md]
    3. 查询出边（普通边直达，条件边调用路由函数） ^[raw/articles/langgraph-state-machine-under-the-hood.md]
    4. 下一个节点入队 ^[raw/articles/langgraph-state-machine-under-the-hood.md]
    5. 取队列头重复 ^[raw/articles/langgraph-state-machine-under-the-hood.md]
``` ^[raw/articles/langgraph-state-machine-under-the-hood.md]

### 条件边（Conditional Edge）
路由函数根据运行时状态决定下一个节点——**这是 ReAct Agent 的本质：一个带条件边的循环图。** ^[raw/articles/langgraph-state-machine-under-the-hood.md]
```typescript ^[raw/articles/langgraph-state-machine-under-the-hood.md]
function routeAfterLLM(state): string { ^[raw/articles/langgraph-state-machine-under-the-hood.md]
  if (lastMessage.hasToolCalls) return "tools"; ^[raw/articles/langgraph-state-machine-under-the-hood.md]
  return END; ^[raw/articles/langgraph-state-machine-under-the-hood.md]
} ^[raw/articles/langgraph-state-machine-under-the-hood.md]
``` ^[raw/articles/langgraph-state-machine-under-the-hood.md]

## Fan-out / Fan-in 并行模式
- **Fan-out**：一个节点同时触发多个后继节点（异步并发）
- **Fan-in**：等待所有前驱节点完成，merge 节点才入队执行
这是**并行搜索、多 Agent 协作**的基础模式。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

## CompiledGraph 四大接口
| 方法 | 作用 | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
|------|------| ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| `invoke()` | 同步执行，返回最终 State | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| `stream()` | 流式执行，每个节点完成 yield 一次（打字机效果） | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| `getGraph()` | 可视化图结构（调试用） | ^[raw/articles/langgraph-state-machine-under-the-hood.md]
| `getState()` | 获取当前状态（需要 Checkpointer） | ^[raw/articles/langgraph-state-machine-under-the-hood.md]

## 与 [[entities/agent-skill-writing]] 的关系
LangGraph 是构建复杂 Agent Skill 的底层运行时框架。当一个 Skill 需要： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

- 多步骤工作流（有条件分支、循环）
- 状态持久化（对话历史、工具结果）
- 并行执行（多工具同时调用）
→ 底层由 LangGraph 的 StateGraph 调度器驱动。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

## 深度分析
### 状态机模型对 Agent 架构的根本性影响
LangGraph 通过状态机模型解决了 LangChain 早期架构的致命弱点：控制流固定、无法动态决策。传统 Chain 本质上是「流水线」，而 StateGraph 是「可编程的图灵机」——这意味着 LLM Agent 第一次有了真正意义上的控制流编程能力。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]

### Reducer 机制的设计意图
Reducer 不是简单的状态合并工具，它体现了**不可变数据结构**的核心思想：每次节点执行都产生新快照而非修改原状态。这带来三个关键优势： ^[raw/articles/langgraph-state-machine-under-the-hood.md]
1. **时间旅行调试**：任意历史状态可回溯 ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
2. **Checkpoint 基础**：状态快照即存档点 ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]
3. **并发安全**：无锁并行执行，冲突只发生在 merge 阶段 ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

### 条件边是 ReAct 的本质
看似复杂的 ReAct Agent（ Thought → Action → Observation 循环），在 LangGraph 中只是一个带条件边的循环图。路由函数 `routeAfterLLM` 取代了手写的 if-else 控制逻辑，LLM 的「决策」被转化为图的边选择——这是 Agent 控制流从硬编码走向声明式的关键一步。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]

### Fan-out/Fan-in 的工程意义
并行模式不只是性能优化，它改变了 Agent 的**交互拓扑**： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

- Fan-out：一个问题同时探索多条解决路径（搜索 vs 知识库 vs 计算器）
- Fan-in：多个专业能力合并为统一决策（多 Agent 协作的核心）
这种模式让 Agent 从「单兵作战」进化到「团队协作」。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

## 实践启示
### 何时该用 LangGraph
适用场景： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

- 多步骤工作流，需要根据中间结果动态决策
- Agent 需要「思考-行动-观察」循环（ReAct 模式）
- 需要状态持久化或断点恢复（Checkpointer）
- 多 Agent 并行协作
不适用场景： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

- 简单线性 Chain 能解决的单次 LLM 调用
- 对延迟极度敏感的实时交互（调度器有额外开销）

### 状态设计的最佳实践
``` ^[raw/articles/langgraph-state-machine-under-the-hood.md]
状态字段设计原则： ^[raw/articles/langgraph-state-machine-under-the-hood.md]

- 消息类 → 用追加 Reducer（如 messages）
- 计数器/步骤 → 用覆盖 Reducer（如 step）
- 需要汇总的 → 用累加 Reducer（如 callCount）
- 避免把「中间推理过程」放进 State，保持 State 精简
``` ^[raw/articles/langgraph-state-machine-under-the-hood.md]

### 条件边的路由函数规范
路由函数必须**无副作用、纯同步、只依赖 state 参数**。常见陷阱： ^[raw/articles/langgraph-state-machine-under-the-hood.md]^[raw/articles/langgraph-state-machine-under-the-hood.md]

- 在路由函数里调用外部 API（拖慢调度、破坏幂等性）
- 在路由函数里修改 state（应只读取）
- 路由函数返回的节点名拼写错误（静默跳转到不存在的节点）

### 并行执行的血泪教训
Fan-out 触发时，确保并行节点之间**无状态依赖**。如果 `search_web` 和 `search_db` 共享某个 state 字段且使用了非交换的 Reducer（如覆盖型），执行顺序会导致结果不确定。 ^[raw/articles/langgraph-state-machine-under-the-hood.md]

## 相关主题
- [[entities/anthropic-mcp-revisited]] — MCP 是云端 Agent 标准化接入层，与 LangGraph 的工具调用层有协同关系
-  — Skill 编写规范，LangGraph 可作为 Skill 的执行引擎
- [[raw/articles/langgraph-state-machine-under-the-hood.md|原文存档]]