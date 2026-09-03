---
title: "Context Management in Agent Systems"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [context, memory, agent, working-set, context-management, token]
sources:
  - raw/articles/agent-harness-context-management-working-set
  - raw/articles/agent-context-management-architecture-patterns
summary: Agent 系统中的上下文管理，涵盖工作集管理、上下文窗口、上下文压缩、选择性记忆等核心模式，以及如何避免上下文膨胀和上下文污染。
---

# Context Management in Agent Systems

> 上下文管理是 Agent 系统中最核心的工程挑战之一，直接影响 Agent 的能力上限和系统成本。

## 一、上下文管理的本质

### 1.1 什么是上下文

Agent 的上下文（Context）包括：
- **系统提示**（System Prompt）
- **对话历史**（Conversation History）
- **工具定义**（Tool Definitions）
- **工作内存**（Working Memory）
- **外部知识引用**（External Knowledge）

### 1.2 上下文管理的挑战

| 挑战 | 描述 |
|------|------|
| **窗口限制** | LLM 有固定的上下文窗口（如 128K tokens） |
| **成本** | 上下文越长，token 成本越高 |
| **干扰** | 无关上下文可能干扰正确决策 |
| **衰减** | 早期上下文可能被"遗忘" |

## 二、工作集管理（Working Set Management）

### 2.1 工作集定义

工作集（Working Set）是 Agent 在当前任务中实际需要的上下文子集：

```
总上下文 → 筛选 → 工作集 → Agent 决策
```

### 2.2 工作集筛选原则

| 原则 | 描述 | 示例 |
|------|------|------|
| **相关性** | 选择与当前任务最相关的 | 最近 N 轮对话 |
| **时效性** | 优先新信息 | 最新文档版本 |
| **重要性** | 保留关键事实 | 用户偏好、约束条件 |

### 2.3 动态工作集

```python
class DynamicWorkingSet:
    def __init__(self, max_size=50_000):
        self.max_size = max_size
        self.current = []
    
    def add(self, item, priority=None):
        # 根据优先级和空间动态管理
        if self.size() + item.size > self.max_size:
            self.evict_least_relevant()
        self.current.append((item, priority))
    
    def evict_least_relevant(self):
        # 淘汰策略：LRU、重要性、相关性综合评分
        pass
```

## 三、上下文压缩

### 3.1 压缩策略分类

| 策略 | 方法 | 适用场景 |
|------|------|---------|
| **摘要压缩** | 生成摘要替代原文 | 长对话、历史记录 |
| **结构化提取** | 提取关键实体和关系 | 文档、知识库 |
| **选择性丢弃** | 丢弃低价值内容 | 噪声对话 |
| **分层索引** | 构建层级结构 | 大型文档集 |

### 3.2 摘要压缩实现

```python
def compress_history(messages, max_tokens=10_000):
    # 1. 识别关键信息
    key_info = extract_key_facts(messages)
    
    # 2. 生成分组摘要
    summaries = []
    for group in group_consecutive(messages):
        summary = summarize(group)
        summaries.append(summary)
    
    # 3. 限制总长度
    return truncate_to_token_limit(summaries, max_tokens)
```

### 3.3 RAG 与上下文的结合

```
┌──────────────────────────────────────┐
│           User Query                  │
└──────────────────────────────────────┘
                   ↓
┌──────────────────────────────────────┐
│     相关性检索 → 知识片段            │
│     工作集 + 检索结果 → 上下文      │
└──────────────────────────────────────┘
                   ↓
┌──────────────────────────────────────┐
│           LLM 生成                    │
└──────────────────────────────────────┘
```

## 四、上下文污染与防护

### 4.1 污染类型

| 类型 | 描述 | 影响 |
|------|------|------|
| **任务污染** | 不同任务的上下文混合 | 决策混淆 |
| **角色污染** | 系统提示被用户输入覆盖 | 行为异常 |
| **工具污染** | 工具返回值干扰主任务 | 错误调用 |
| **时间污染** | 过时信息导致错误判断 | 决策过时 |

### 4.2 隔离策略

```python
class ContextIsolator:
    def isolate_task(self, task_id, context):
        # 1. 任务专属系统提示
        task_prompt = self.get_task_prompt(task_id)
        
        # 2. 隔离的记忆存储
        task_memory = self.get_memory_store(task_id)
        
        # 3. 独立工具配置
        task_tools = self.get_tool_config(task_id)
        
        return {
            'system': task_prompt,
            'memory': task_memory,
            'tools': task_tools,
            'history': context.history
        }
```

### 4.3 上下文边界管理

| 技术 | 作用 |
|------|------|
| **对话轮次标记** | 明确区分不同轮次 |
| **任务标识符** | 防止跨任务干扰 |
| **版本控制** | 追踪上下文变更 |
| **快照/恢复** | 支持任务切换 |

## 五、MCP 上下文经济

### 5.1 工具定义的上下文成本

MCP Server 提供的工具定义会消耗上下文预算：

```
40 个工具 × 10-15KB schema = 400-600KB/turn
```

### 5.2 On-Demand Tool Loading

```python
# 延迟加载工具定义
class OnDemandToolLoader:
    def __init__(self, all_tools):
        self.all_tools = all_tools
        self.loaded = set()
    
    def search_and_load(self, query):
        # 1. 搜索相关工具
        relevant = self.search(query, self.all_tools)
        
        # 2. 只加载命中的
        for tool in relevant:
            if tool.id not in self.loaded:
                self.loaded.add(tool.id)
                self.context.add(tool.definition)
        
        # 3. 85%+ 的工具定义被跳过
```

### 5.3 程序化工具调用

减少工具结果进入上下文：

```python
# 在沙箱中处理工具结果
def process_result_in_sandbox(result):
    # 1. 循环调用、过滤、聚合
    # 2. 只把必要结果放入上下文
    processed = aggregate(result)
    return processed
```

## 六、最佳实践

1. **先测量，再优化**：了解 token 消耗分布是优化的前提
2. **工作集要动态**：根据任务阶段调整上下文内容
3. **隔离要彻底**：不同任务、不同用户的上下文要严格隔离
4. **压缩要智能**：使用 LLM 生成摘要而非简单截断
5. **工具要按需加载**：避免不必要的工具定义消耗

## 相关实体

- [[entities/agent-context-management-architecture-patterns]]
- [[entities/agent-harness-context-management-working-set]]
- [[entities/别再把上下文当聊天记录]]
- [[entities/how-we-built-cognitive-memory-for-agentic-systems]]
- [[entities/05-11-the-great-memory-panic-of-2026]]

## 七、关联实体

- [[entities/agent-harness-context-management-working-set]] — 工作集视角
- [[entities/agent-context-management-architecture-patterns]] — 架构模式
- [[concepts/agent-memory-system-design]] — Agent 内存系统设计
- [[concepts/inference-optimization]] — 推理优化与上下文关系
- [[concepts/coding-agent-architecture|Coding Agent Architecture]] — 编程 Agent 的上下文管理是其核心子系统

## 关联实体

**上游依赖**:
- [[entities/agent-context-management-architecture-patterns]] — 提供基础理论/方法
- [[entities/agent-harness-context-management-working-set]] — 提供基础理论/方法

**下游应用**:
- [[entities/别再把上下文当聊天记录]] — 具体应用场景
- [[entities/how-we-built-cognitive-memory-for-agentic-systems]] — 具体应用场景
- [[entities/05-11-the-great-memory-panic-of-2026]] — 具体应用场景

**平行协作**:
- [[entities/agent-harness-context-management-working-set]] — 替代/补充方案
- [[entities/agent-context-management-architecture-patterns]] — 替代/补充方案
- [[entities/agent-开发范式演进从环境工程出发]] — 替代/补充方案


→ [[raw/articles/agent-harness-context-management-working-set|原文存档]]

## 新增关联实体
- [[entities/agent-开发范式演进从环境工程出发]]

## 所属 MOC

- [[moc/layer-2-interaction|Layer 2 Interaction]]
