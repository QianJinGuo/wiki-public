---
title: "Amazon Bedrock AgentCore AG-UI 协议：为 AI Agent 构建生成式 UI"
description: "AWS 推出 AG-UI 协议，让 Agent 在 AgentCore Runtime 上动态生成交互式 UI，实现从纯文本对话到富交互界面的 Agent 体验升级。"
created: 2026-07-01
updated: 2026-09-05
type: entity
tags: [aws, bedrock, agentcore, agent, generative-ui, mcp]
sources: [raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor]
confidence: 0.85
review_value: 8
review_confidence: 9
review_stars: 4
review_recommendation: strong
---

# Amazon Bedrock AgentCore AG-UI 协议：为 AI Agent 构建生成式 UI

> 原文存档：[[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor|原文存档]]

## 摘要

AG-UI（Agent-User Interaction Protocol）是一个开放的 Agent-用户交互协议，它使 AI Agent 能够超越纯文本对话，动态渲染交互式图表、实时共享画布，并在执行过程中暂停以获取用户确认。Amazon Bedrock AgentCore 通过支持 AG-UI 协议，为开发者提供了构建生产级交互式 Agent 应用的完整解决方案。^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:1-20]

## 核心概念

### 什么是 AG-UI 协议

AG-UI 定义了 Agent 后端与前端之间的标准化事件通信协议。它支持：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]


- **生成式 UI（Generative UI）**：Agent 可在对话中渲染交互式图表和组件^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:180-190]
- **共享状态（Shared State）**：前后端双向同步状态，如实时更新的待办画布^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:203-213]
- **人机协同（Human-in-the-loop）**：Agent 可在执行中暂停，等待用户输入后继续^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:215-231]

AG-UI 与多种 Agent 框架（Strands Agents、LangGraph、CrewAI）和前端库（React、Angular、Vue）兼容，实现前后端解耦。^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]

### Amazon Bedrock AgentCore 的三协议架构

AgentCore Runtime 支持三种互补协议：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]


| 协议 | 用途 | 连接对象 |
|------|------|----------|
| **MCP** | 工具连接 | Agent ↔ 工具/数据源 |
| **A2A** | Agent 间通信 | Agent ↔ Agent |
| **AG-UI** | 用户交互 | Agent ↔ 用户界面 |

^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:21-22]

当部署带有 AG-UI 协议标志的 Agent 容器时，AgentCore 充当透明代理，处理认证（SigV4 或 OAuth 2.0）、会话隔离、自动扩缩容和可观测性。^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:21-22]

## 技术架构

### FAST 全栈解决方案模板

FAST（Fullstack AgentCore Solution Template）是一个开箱即用的 starter 项目，包含：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]


- **AgentCore Runtime**：无服务器 Agent 托管环境^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]
- **AgentCore Gateway**：MCP 工具连接网关^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]
- **AgentCore Identity**：Amazon Cognito 认证^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]
- **AgentCore Memory**：持久化对话历史^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]
- **Code Interpreter**：安全代码执行环境^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]
- **React 前端**：预构建的交互界面^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]

FAST v0.4.1 新增两种 AG-UI 模式：`agui-strands-agent` 和 `agui-langgraph-agent`，共享同一个前端解析器。^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:23-24]

### AG-UI 事件流协议

AG-UI 使用 Server-Sent Events (SSE) 传输类型化事件流。单个工具调用产生的事件序列：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]


```
data: {"type": "RUN_STARTED", "threadId": "t1", "runId": "r1"}
data: {"type": "TEXT_MESSAGE_START", "messageId": "m1", "role": "assistant"}
data: {"type": "TEXT_MESSAGE_CONTENT", "messageId": "m1", "delta": "Let me check..."}
data: {"type": "TOOL_CALL_START", "toolCallId": "tc1", "toolCallName": "get_weather"}
data: {"type": "TOOL_CALL_ARGS", "toolCallId": "tc1", "delta": "{\"location\": \"Seattle\"}"}
data: {"type": "TOOL_CALL_END", "toolCallId": "tc1"}
data: {"type": "TOOL_CALL_RESULT", "toolCallId": "tc1", "content": "{\"temp\": 55}"}
data: {"type": "RUN_FINISHED", "threadId": "t1", "runId": "r1"}
```

^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:131-143]

### 双模式后端实现

**Strands Agent 模式**：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]

- 使用 `ag-ui-strands` 库的 `StrandsAgent` 包装器
- 自动将 Strands 流事件转换为 AG-UI SSE^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:55-65]
- 每请求创建新 Agent，自动绑定 Gateway MCP 工具^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:55-65]
- AgentCore Memory 通过 session-manager provider 附加，可选启用^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:57-62]

**LangGraph Agent 模式**：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]

- 使用 `copilotkit` 库的 `LangGraphAGUIAgent`^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:96-114]
- 每请求构建新的编译图，获得 MCP 工具^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:103-114]
- 支持 `CopilotKitMiddleware` 中间件扩展^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:103-114]

关键优势：两种模式产生完全相同的 AG-UI 事件，前端无需感知后端框架差异。^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:166-167]

## 生成式 UI 实战

### CopilotKit + FAST 增强方案

CopilotKit 是一个专为生成式 UI、共享状态和人机协同设计的 React 库。与 FAST 集成的示例展示了三种核心能力：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]


#### 1. 内联组件渲染

前端注册可被 Agent 调用的 React 组件：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]


```typescript
useComponent({
  name: "pieChart",
  description: "Displays data as a pie chart.",
  parameters: PieChartPropsSchema,
  render: PieChart,
});
```

^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:192-199]

Agent 调用 `pieChart` 工具时，CopilotKit 拦截 `TOOL_CALL_START` 和 `TOOL_CALL_ARGS` 事件，直接在对话中渲染 `PieChart` 组件。^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:200-202]

#### 2. 双向状态同步

待办画布示例展示前后端状态实时同步：
- 用户通过自然语言指令添加任务："Add three tasks: design the API, write tests..."^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:205-206]
- Agent 调用 `manage_todos` 工具，通过 AG-UI `STATE_SNAPSHOT` 事件更新画布^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:205-206]
- 用户直接在 UI 编辑待办，Agent 下次轮询时通过系统提示注入看到更新^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:207-213]

#### 3. 人机协同中断

会议调度器演示 Agent 暂停执行等待用户输入：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]

- Agent 发出 `TOOL_CALL_START` 调用 `scheduleTime`^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:219-229]
- CopilotKit 渲染时间选择器而非执行后端工具^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:226-229]
- 用户选择时间后，响应作为 `TOOL_CALL_RESULT` 返回，Agent 继续执行^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:231-231]

### 架构组件

CopilotKit + FAST 的完整部署包含：^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md]

- Amazon Cognito 用户池（认证）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]
- Amazon ECR 仓库（容器镜像）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]
- AgentCore Runtime（Agent 托管）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]
- AgentCore Gateway（MCP 工具连接）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]
- AgentCore Memory（对话持久化）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]
- CopilotKit Runtime Lambda + API Gateway（前后端桥接）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]
- AWS Amplify（前端托管）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:244-244]

## 实践启示

### 适用场景

**AG-UI 协议最适合**：
- 需要动态渲染图表、表格等交互组件的对话式应用^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:180-202]
- 需要用户中途干预的复杂工作流（审批、确认、选择）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:215-231]
- 前后端状态需要实时同步的协作场景^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:203-213]
- 希望后端框架（Strands/LangGraph）与前端解耦的团队^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:166-167]

**生成式 UI 的控制光谱**：
- 高前端控制（当前方案）：前端预定义组件，Agent 选择渲染并填充数据^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:184-185]
- 中等控制：Agent 返回声明式 UI 描述，前端渲染^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:184-185]
- 高 Agent 自由：Agent 返回完整 UI 表面，前端嵌入（需沙箱和输入验证）^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:184-185]

### 部署建议

1. **快速启动**：使用 FAST 模板，选择 `agui-strands-agent` 或 `agui-langgraph-agent` 模式^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:168-179]
2. **增强交互**：集成 CopilotKit 示例，体验生成式 UI、共享状态、人机协同^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:180-249]
3. **生产优化**：利用 AgentCore 的内置认证、扩缩容、可观测性能力^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:21-22]

### 与相关技术的关系

- **MCP**：AG-UI 处理用户交互，MCP 处理工具调用，两者在 AgentCore 中并行工作^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:21-22]
- **A2A**：AG-UI 连接 Agent 与用户，A2A 连接 Agent 与 Agent^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:21-22]
- **传统 REST API**：AG-UI 的流式 SSE 替代了轮询，更适合实时对话场景^[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor.md:131-143]

→ [[raw/articles/build-generative-ui-for-ai-agents-on-amazon-bedrock-agentcor|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

