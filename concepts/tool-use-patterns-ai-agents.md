---
title: "Tool Use Patterns in AI Agents"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [tool, agent, mcp, function-calling, api, integration]
sources:
  - raw/articles/anthropic-12-mcp-production-patterns
summary: AI Agent 工具使用模式，涵盖工具设计原则、粒度决策、MCP 工具交互面模式、函数调用范式，以及从 API 封装到 Agent 可用工具的转化策略。
---

# Tool Use Patterns in AI Agents

> "好的 MCP Server，不是 API 的翻译层，而是 Agent 面向任务的产品接口。"

## 一、工具使用的本质

### 1.1 什么是工具

工具（Tool）是 Agent 与外部系统交互的接口：
- **API 调用**：查询数据、执行操作
- **文件操作**：读写文件、执行命令
- **搜索能力**：网络搜索、数据库查询
- **计算能力**：代码执行、数学计算

### 1.2 工具 vs API

| 维度 | 传统 API | Agent 工具 |
|------|---------|-----------|
| **调用者** | 程序 | LLM（需要理解） |
| **参数设计** | 开发者友好 | Agent 友好 |
| **错误处理** | 明确代码 | 需要自然语言解释 |
| **组合方式** | 程序控制 | Agent 决策 |

## 二、工具设计原则

### 2.1 按意图封装（Intent-Grouped Tools）

**错误做法**：把每个 API endpoint 1:1 包装成 tool

```python
# 错误：暴露底层 API
get_thread, parse_messages, create_issue, link_attachment
# Agent 需要自己拼接4个底层动作
```

**正确做法**：按用户意图封装

```python
# 正确：按用户意图封装
create_issue_from_thread
# 内部处理：编排、ID归一化、附件关联、错误重试
```

### 2.2 薄交互面模式（Thin Surface Pattern）

当 API 面太大（几百~几千个操作）时：

```python
# 只暴露少量高能力工具
tools = [
    search,   # 让 Agent 搜索可用 API
    execute   # 让 Agent 写短脚本在沙箱执行
]

# Cloudflare 案例：2 个工具覆盖约 2500 个 endpoint
```

### 2.3 工具描述是产品界面

```python
# 工具描述要同时满足人和机器
{
    "name": "create_issue_from_thread",
    "description": """
    Create a Linear issue from an email thread.
    
    Args:
        thread_id: The email thread ID
        assignee: Team member to assign (optional)
        priority: P0/P1/P2/P3 (optional)
    
    Returns:
        Created issue URL and ID
    
    Use when:
        - User wants to track a thread as a task
        - Email contains action items
        - Follow-up needed
    
    Don't use when:
        - Thread is informational only
        - Issue already exists
    """,
    "parameters": {...}
}
```

## 三、函数调用范式

### 3.1 ReAct 模式

```
Thought → Action → Observation → Thought → ...
```

```python
class ReActAgent:
    def step(self, observation):
        thought = self.reason(observation)
        action = self.select_tool(thought)
        result = self.execute(action)
        return result
```

### 3.2 Tool Use 的常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 工具选择错误 | 描述不清晰 | 改进描述、增加示例 |
| 参数构造错误 | schema 复杂 | 简化参数结构 |
| 结果理解错误 | 返回格式不明确 | 结构化输出、提供格式说明 |
| 循环调用 | 缺少终止条件 | 明确成功/失败标准 |

### 3.3 程序化工具调用

减少工具结果进入上下文：

```python
# 在沙箱中处理工具结果
def process_code_results(output):
    # 1. 过滤数据
    # 2. 聚合字段
    # 3. 做中间计算
    # 4. 只把必要结果放进上下文
    return aggregated_summary
```

## 四、MCP 工具交互面

### 4.1 远程 vs 本地 Server

| 形态 | 适用场景 |
|------|---------|
| 本地 MCP Server（stdio） | 桌面应用、IDE Agent、Claude Code |
| 远程 MCP Server | 生产环境（浏览器/移动端/云端） |

### 4.2 按需加载工具（On-Demand Tool Loading）

```
Agent 启动 → Tool Search → 搜索相关工具 → 只加载命中的 → 执行
```

**价值**：减少 85%+ 的工具定义 token 消耗

### 4.3 工具组合模式

```python
# 搜索 + 执行 = 能力组合
def handle_large_api_surface(server):
    # Agent 先搜索需要的工具
    relevant_tools = server.search("database query")
    
    # Agent 写脚本执行
    script = """
        for table in tables:
            result = query(table)
            aggregate(result)
    """
    return server.execute_in_sandbox(script)
```

## 五、安全与权限

### 5.1 凭证管理

**问题**：token 放在工具参数、环境变量里 → 生产系统危险

**方案**：凭证托管到 Vault

```python
# 不推荐
tool.call(api_key="sk-xxx")

# 推荐
session = create_session(vault_id="xxx")
tool.call_with_vault(session)
```

### 5.2 权限边界

```python
class ToolPermission:
    def check(self, tool, operation, context):
        # 1. 检查用户权限
        # 2. 检查操作风险级别
        # 3. 记录审计日志
        # 4. 高风险操作需要确认
        pass
```

### 5.3 沙箱执行

```python
# 工具在沙箱中执行
class SandboxedTool:
    def execute(self, code, limits):
        # 资源限制
        # 网络隔离
        # 超时控制
        # 结果过滤
        return result
```

## 六、实践启示

1. **工具描述是产品设计**：要同时考虑 Agent 和人的理解
2. **按意图封装工具**：不要暴露底层 API，端到端任务作为工具
3. **大 API 面用搜索**：不要试图封装所有，用搜索+执行代替
4. **工具结果要处理**：不要直接把原始输出给 LLM，先聚合过滤
5. **凭证要托管**：不要在参数或环境变量里放敏感信息

## 相关实体

- [[entities/agent-harness-architecture-deep-dive-aksahy]]
- [[entities/aws-sagemaker-sft-dpo-tool-calling]]
- [[entities/guide-ai-agents-models-apps-harnesses-mollick]]
- [[entities/agent-context-management-architecture-patterns]]
- [[entities/how-we-built-cognitive-memory-for-agentic-systems]]

## 七、关联实体

- [[entities/anthropic-12-mcp-production-patterns]] — MCP 12 模式
- [[concepts/model-context-protocol-mcp]] — MCP 协议
- [[concepts/agentic-workflow-patterns]] — 工作流模式
- [[concepts/context-management-agent-systems]] — 上下文管理

## 关联实体

**上游依赖**:
- [[entities/agent-harness-architecture-deep-dive-aksahy]] — 提供基础理论/方法
- [[entities/aws-sagemaker-sft-dpo-tool-calling]] — 提供基础理论/方法

**下游应用**:
- [[entities/guide-ai-agents-models-apps-harnesses-mollick]] — 具体应用场景
- [[entities/agent-context-management-architecture-patterns]] — 具体应用场景

**平行协作**:
- [[entities/how-we-built-cognitive-memory-for-agentic-systems]] — 替代/补充方案
- [[entities/anthropic-12-mcp-production-patterns]] — 替代/补充方案
- [[entities/crawler-vs-opencli-doubao]] — 替代/补充方案


→ [[raw/articles/anthropic-12-mcp-production-patterns|原文存档]]

## 新增关联实体
- [[entities/crawler-vs-opencli-doubao]]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
