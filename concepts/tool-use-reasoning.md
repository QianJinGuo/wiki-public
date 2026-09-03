---
title: "工具使用推理"
created: 2026-06-11
updated: 2026-08-29
type: concept
tags: [tool-use, agent, search, code, data, function-calling, react, toolformer, mcp, tool-selection, tool-composition]
description: "工具使用推理范式：Function Calling、ReAct、Toolformer、工具选择策略"
sources:
  - raw/articles/17-agent-architectures-evolution
  - raw/articles/agent-evolution-four-stages-six-dimensions-aliyun
  - raw/articles/microsoft-agent-framework-tools-overview-provider-matrix
  - raw/articles/aws-bedrock-agentcore-os-level-actions-browser
  - raw/articles/800行代码实现-open-claw-的-tool消息总线子agent管理架构
  - raw/articles/anthropic-12-mcp-production-patterns
---

# 工具使用推理

> 工具使用推理（Tool-Use Reasoning）是 Agent 系统将「决策」与「执行」分离的核心能力——模型负责判断「是否用工具、用什么工具、怎么组合」，框架负责可靠地执行并返回结果。

工具使用推理范式：Function Calling、ReAct、Toolformer、工具选择策略。本概念页汇聚 wiki 中多个相关实体的核心洞察，形成系统化的知识框架。

## 核心定义

### 什么是工具使用推理

工具使用推理（Tool-Use Reasoning）是 LLM Agent 架构中最关键的「世界接口」（World Interface）层，它解决了 LLM 的两个根本性局限：**知识截止日期限制**和**参数化知识边界**。文本交互无法获取实时信息或触发真实世界副作用，工具调用打破了这一瓶颈。 ^[raw/articles/17-agent-architectures-evolution.md]

工具使用推理的本质是**结构化工具打破参数知识边界**——通过让模型输出 JSON 格式的工具调用指令，框架解析后执行，结果注入下一轮输入，形成 LLM ↔ Tool 的闭环回边。

### 工具使用在 Agent 六维度中的位置

根据 Agent 四阶段六维度演化框架（2023-2026），工具维度经历了从 **Function Call → CLI / Script 原生化** 的范式转移。旧模式是针对具体业务场景将系统能力封装成标准 API，注册为模型可调用函数，核心痛点是大量系统没有现成 API、API Schema 管理极其复杂。新范式是 CLI 命令行原生化 + Script 脚本化——模型预训练数据中包含海量 Linux/Unix 命令知识，无需额外定义 API Schema。 ^[raw/articles/agent-evolution-four-stages-six-dimensions-aliyun.md]

这一转变的深层逻辑是：**从「人为适配模型」到「利用模型原生能力」**。早期阶段开发者试图为每个操作编写专用接口、精心调试 System Prompt；现在模型本身具备的预训练知识直接成为工具，工具接口退居「Harness 层」，只负责确定性约束和安全边界。

## 关键维度

### 维度一：Function Calling 协议

#### OpenAI / Anthropic Function Calling

Function Calling 是让 LLM 输出结构化工具调用的标准机制。模型输出 JSON 格式的工具调用请求，包含 `tool_calls` 字段，框架解析后执行并将结果注入下一轮输入。核心 State 字段包括：`user_input`（用户原始请求）、`model_output`（LLM 回复）、`tool_call`（调用的工具及参数）、`tool_return`（工具执行结果）、`next_round`（下一轮输入）。拓扑上形成 **LLM ↔ Tool 的闭环回边**，内部 while 循环只要 model 回复包含 `tool_calls` 就继续执行。 ^[raw/articles/17-agent-architectures-evolution.md]

**常见失败模式**：
- **工具名幻觉**：模型生成不存在的工具名
- **参数类型错误**：JSON 结构与工具 schema 不匹配
- **序列化/反序列化失败**：复杂返回值无法正确传递
- **工具描述不清**：导致模型选了错误工具

#### Anthropic Extended Thinking 与 Tool Use

Anthropic 的 Claude 模型支持 Extended Thinking 模式，在工具调用场景中允许模型在做出决策前进行更深入的内部推理。这一机制提升了模型在复杂工具组合场景下的决策质量。

#### OpenClaw 的薄抽象 Tool 实现

OpenClaw 直接基于 Anthropic SDK 构建 Tool 系统，采用极简四要素抽象：一个工具由 `name`、`description`、`input_schema`、`execute` 组成，`input_schema` 的类型直接取自 `@anthropic-ai/sdk` 的 `Tool` 类型定义，零中间层转换。 ^[raw/articles/800行代码实现-open-claw-的-tool消息总线子agent管理架构.md]

ToolRegistry 是统一的注册表实现，提供 `register()`、`execute()`、`getToolDefinition()` 三个核心方法，以及 `exclude()` 方法为子 Agent 生成受限工具集。`exclude()` 是能力隔离的核心——子 Agent 不应该持有 `spawn`（避免递归创建子 Agent）、`message`（避免直接向用户发消息）等工具，通过排除清单向 LLM 和开发者明确传达能力边界。

### 维度二：ReAct 模式——推理与行动的交织

#### ReAct 的核心机制

ReAct（Reasoning + Acting）解决了 Tool Use 的单步调用无法处理「根据观察结果决定下一步」的场景。现实任务往往是动态推理而非一次性工具调用，需要「观察环境→推理下一步→执行行动→再次观察」的循环。 ^[raw/articles/17-agent-architectures-evolution.md]

ReAct 在 Tool Use 基础上新增三个 State 字段：
- `thought`：当前推理步骤的思考
- `action`：决定执行的动作
- `observation`：动作执行后的观察结果

拓扑上形成 **Thought → Action → Observation → Thought...** 的闭环。Router 工作方式是 Observation 结果作为下一轮 Thought 的输入——**上一步观察决定下一步行动**，而非预设固定执行路径。这是最关键的结构性改进，Tool Use 的 tool→model 回边是整个架构演化的转折点。

**失败模式**：
- **局部贪心**：每步选择当前最优，但整体非最优（类似 RL 中的 greedy 策略）
- **观察噪声放大**：错误观察导致后续推理全错
- **循环终止判断**：可能无限循环或过早终止

#### ReAct vs 纯 Tool Use 的选择

当单工具调用不足以完成任务、需要多步推理链时，应从 Tool Use 升级到 ReAct。ReAct 的适用场景：搜索→读取→分析→输出的多步骤流程、需要根据中间结果动态调整策略的任务、长程任务中需要自我校验的场景。 ^[raw/articles/17-agent-architectures-evolution.md]

### 维度三：Toolformer——自监督工具学习

Toolformer 是让模型学会自行决定何时、如何使用工具的自监督学习框架。与手工设计工具调用不同，Toolformer 通过大规模预训练让模型自己发现工具使用的模式，然后通过微调将这种能力固化。

核心思路：
1. **工具发现**：模型在预训练过程中发现某些任务通过调用工具比直接生成文本更有效
2. **API 对齐**：将外部工具的调用格式与模型内在的推理链对齐
3. **自洽性校验**：确保模型在相同输入下对同一工具的调用保持一致

### 维度四：工具选择策略

#### 基于规则的 vs LLM 自主决策

**规则匹配**（Rule-Based）：适用于工具集固定、调用模式可枚举的场景。优点是确定性高、可审计；缺点是无法处理新场景、需要人工维护规则。

**LLM 决策**（LLM-Decided）：适用于工具集动态、调用模式不可枚举的场景。模型根据工具描述自主判断最合适的工具，优点是灵活性高、泛化能力强；缺点是不可预测性、难以审计。

#### Microsoft Agent Framework 的 Provider 矩阵

Microsoft Agent Framework 按「谁执行、谁托管」将工具分为四类：

**1. Function Tools（应用代码）**：开发者用 `@tool` / `FunctionTool` 暴露的本地函数，由框架在应用进程内调度。可移植性最好、最适合接业务逻辑与细粒度权限。^[raw/articles/microsoft-agent-framework-tools-overview-provider-matrix.md]

**2. Hosted Tools（Provider 托管）**：由 OpenAI / Azure OpenAI Responses 或 Foundry 等运行时托管执行，包括 Code Interpreter（沙箱内执行代码）、File Search（向量库/文件检索）、Web Search（联网检索）、Image Generation（托管文生图）。关键特点是不经过你的 Python 函数体，计费、配额、数据驻留都跟随 Provider。

**3. MCP Tools（Model Context Protocol）**：MCP 适合把已有工具生态（数据库、SaaS、内部平台）以标准协议接到 Agent，不必每个集成写一个薄封装。只要底层 client 支持 function calling，Local MCP 通常可与其他 Function Tools 混用。

**4. Foundry 扩展工具（项目级连接）**：通过 FoundryChatClient 挂载，包括 Foundry Toolboxes（命名/版本化的托管工具包）、Bing Grounding/AI Search、SharePoint/Memory Search、Computer Use/Browser Automation、Agent-to-Agent（A2A）。

#### Provider × Tool 二维矩阵

同一工具类型在不同 Client 上可用性不同，选型时必须按「Client 类型 × 工具类型」二维矩阵核对：

| Tool Type | Responses | Chat Completion | Foundry | Anthropic | Ollama |
|---|---|---|---|---|---|
| **Function Tools** | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| **Code Interpreter** | ✅ | ❌ | ✅ | ✅ | ❌ |
| **File Search** | ✅ | ❌ | ✅ | ❌ | ❌ |
| **Web Search** | ✅ | ✅ | ✅ | ✅ | ❌ |
| **Hosted MCP** | ✅ | ❌ | ✅ | ✅ | ❌ |
| **Local MCP** | ✅ | ✅ | ✅ | ✅ | ✅ |

**选型原则**：先定 Client 类型（Responses vs Chat Completion vs Foundry），再选工具清单。需要 Code Interpreter + File Search 时，优先 Responses 或 Foundry，不要假设 Chat Completion 全能。

### 维度五：工具组合模式

#### 顺序组合（Sequential Composition）

工具按逻辑顺序串联执行，一个工具的输出直接作为下一个工具的输入。典型模式：搜索工具获取 URL → 抓取工具读取内容 → 分析工具处理文本。适合线性依赖链清晰的场景。

#### 并行组合（Parallel Composition）

多个工具同时执行，结果聚合后统一处理。典型模式：同时对多个数据源发起查询、并发爬取多个页面。适合独立性高、无数据依赖的子任务。

#### 条件组合（Conditional Composition）

根据中间结果动态决定下一步调用哪个工具。典型模式：搜索结果数量 < 阈值时补充搜索，质量评分 < 阈值时回退到备用方案。适合需要自适应策略的复杂场景。

#### 循环组合（Loop Composition）

工具被反复调用直到满足终止条件。典型模式：反复抓取页面 → 解析 → 发现新链接 → 继续抓取。适合遍历和迭代类任务，需要设置最大迭代次数防止无限循环。

### 维度六：MCP 作为工具标准

#### MCP 的三层架构

MCP（Model Context Protocol）是 Anthropic 提出的 Agent 连接外部系统的标准协议，解决 API 差异、工具粒度、上下文成本、认证安全四大核心问题。架构上分为 Agent（Claude等）、MCP Client、MCP Server（Remote-First Server Pattern）、External Systems（API、DB等）四层。

#### MCP 工具交互面设计模式

**模式一：按意图组织工具（Intent-Grouped Tools Pattern）**

不要把每个 API endpoint 1:1 包装成 tool（Agent 需要自己拼接多个底层动作）。正确做法是按用户意图封装——例如 `create_issue_from_thread` 内部处理编排、ID归一化、附件关联、错误重试。适合 API 面不算太大、用户任务相对明确的系统。

**模式二：薄交互面模式（Thin Surface Pattern）**

当底层 API 面太大（几百~几千个操作）时，只暴露少量高能力工具：search（让 Agent 搜索可用 API）和 execute（让 Agent 写短脚本由服务端在沙箱执行）。典型案例：Cloudflare MCP Server，2 个工具覆盖约 2500 个 endpoint，工具定义只需约 1000 tokens。

**模式三：按需加载工具（On-Demand Tool Loading Pattern）**

Agent 启动时连接多个 MCP Server 可能拥有几百个工具，全量加载 tool definitions 消耗大量上下文预算。解决方法是延迟加载：Agent 先通过搜索工具查找可能相关的工具，只把命中的工具定义加载进上下文。Anthropic 官方测试显示可减少 **85%+ 的工具定义 token 消耗**，同时保持较高的工具选择准确率。

#### MCP 的五组工具交互面模式

根据 Anthropic 12 个 MCP 生产模式，工具交互面设计分为五组12个模式：

1. **工具交互面**（连接形态）：Remote-First Server Pattern、Intent-Grouped Tools Pattern、Thin Surface Pattern
2. **交互语义**（用户体验）：Inline UI Pattern（工具返回可交互界面）、Elicited Input Pattern（Form Mode 处理结构化输入）、External Handoff Pattern（URL Mode 处理敏感流程）
3. **认证与凭证**（安全边界）：Discoverable Auth Pattern（CIMD 发现认证方式）、Vault-Held Credentials Pattern（凭证生命周期上移到平台层）
4. **上下文经济**（成本效率）：On-Demand Tool Loading Pattern、Programmatic Tool Calling Pattern（在沙箱中处理工具结果再给模型）
5. **打包分发**（交付形态）：Plugin Bundle Pattern（Skills + MCP servers + hooks 统一分发）、Server-Distributed Skills Pattern（MCP Server 直接分发使用工具的 playbook）

### 维度七：OS-Level Actions 与浏览器自动化

#### AgentCore Browser OS级操作

Amazon Bedrock AgentCore 引入 OS-level Actions，允许 Agent 直接操控 GUI 界面——通过 **Action-Screenshot-Reaction 闭环**实现浏览器自动化。8个原子操作覆盖鼠标、键盘、截图等 OS 层交互，Agent 通过视觉反馈（截图）感知环境状态并决定下一步操作。 ^[raw/articles/aws-bedrock-agentcore-os-level-actions-browser.md]

**OS 层与 Web 层的能力边界**：早期基于 Playwright 和 CDP（Chrome DevTools Protocol）构建，擅长操作 DOM 元素——页面导航、表单填写、元素点击、内容提取。但 Web 层存在硬边界：任何操作系统渲染的 UI（原生对话框、安全提示、证书选择器、右键菜单、浏览器设置页）均位于 DOM 之外，CDP 无法触及，Playwright 无法交互。OS Level Actions 通过 `InvokeBrowser` API 突破这一边界。

**8个原子操作分类**：

| 类别 | 操作 | 核心能力 |
|------|------|---------|
| 鼠标 | mouseClick / mouseMove / mouseDrag / mouseScroll | 全范围指针交互，覆盖点击、定位、拖拽、滚动 |
| 键盘 | keyType / keyPress / keyShortcut | 字符输入、重复按键、组合键（ctrl+a等） |
| 视觉 | screenshot | 全桌面捕获，返回 base64 PNG |

核心设计细节：`mouseClick` 坐标可省略（继承当前光标位置）；`keyShortcut` 最多 5 键组合；`screenshot` 是唯一返回数据的操作；`mouseScroll` 的 `deltaY` 负值=向下滚动。

**Action-Screenshot-Reaction 闭环的工程意义**：本质上是**感知即观测**架构。Agent 不依赖结构化 API 返回状态，而是通过视觉截图直接观测屏幕像素。这与人类操作计算机的方式完全一致——看见对话框 → 理解内容 → 点击按钮。全桌面截图可捕获：原生对话框、OS 模态框、浏览器 Chrome 界面、甚至多显示器环境的跨屏内容。

### 维度八：Tool Approval 与安全闸门

#### Dry-Run Harness——副作用闸门

当任务需要系统化安全闸门时，从 Mental Loop（反事实执行）升级到 Dry-Run Harness。核心思路是：工具调用必须先 dry_run，审批通过后才能执行真实操作。^[raw/articles/17-agent-architectures-evolution.md]

State 新增字段：`dry_run_result`（干跑结果）、`approval_status`（pending/approved/rejected）。拓扑是 Workflow 显式插入人工审批 Step。Router 工作方式是干跑通过 → 进入审批队列 → 审批通过 → 执行真实操作。任何一步失败则终止。

#### Tool Approval 框架级统一闸门

Microsoft Agent Framework 的 Tool Approval 不是某个云厂商的独占能力，而是 **function-invoking chat client 上的横切能力**——Function Tools、部分 Hosted 调用、MCP tool call 均可走同一套「先暂停、等人确认、再继续」流水线。^[raw/articles/microsoft-agent-framework-tools-overview-provider-matrix.md]

**Python 单函数声明示例**：
```python
@tool(approval_mode="always_require")
def transfer_funds(account: str, amount: float) -> str:
    return f"Transferred {amount} to {account}"
```

生产环境中，Tool Approval 配置应该**按工具风险等级建立清单**：
- **高风险工具**（`always_require`）：写数据库、写文件到共享路径、执行 Shell 命令、转账/支付
- **中风险工具**（`user_input_required`）：文件删除、外部链接访问、长时运行任务
- **低风险工具**（`never_require`）：只读查询、公开信息检索

#### Metacognitive Agent——自我边界

真正高级的 Agent 不是更敢做事，而是更知道什么时候不该做。Metacognitive Agent 维护 `AGENT_SELF_MODEL`——包含知识域（会什么）、工具列表（能用什么）、置信度阈值（什么情况下应该拒绝）、高风险话题（需要升级处理）。拓扑上是决策分支（reason_directly / use_tool / escalate），自我边界建模使 Router 能做出「能力判断」而非「任务匹配」。^[raw/articles/17-agent-architectures-evolution.md]

### 维度九：错误处理与优雅降级

#### 工具调用的常见错误处理策略

| 问题 | 原因 | 解决 |
|------|------|------|
| 工具选择错误 | 描述不清晰 | 改进描述、增加示例 |
| 参数构造错误 | schema 复杂 | 简化参数结构 |
| 结果理解错误 | 返回格式不明确 | 结构化输出、提供格式说明 |
| 循环调用 | 缺少终止条件 | 明确成功/失败标准 |
| 工具执行失败 | 网络/权限/参数错误 | 重试机制、降级策略 |

#### OpenClaw ExecTool 的三层防护

ExecTool 展示了一个完整的安全设计样本——三层独立的防护：^[raw/articles/800行代码实现-open-claw-的-tool消息总线子agent管理架构.md]

**第一层**：危险命令正则黑名单，覆盖 `rm -rf /`、fork bomb、写裸设备等高危模式（但正则黑名单只是最低防线，不能替代沙箱隔离）

**第二层**：资源限制，默认 30 秒超时、2MB maxBuffer，超时后进程被 kill

**第三层**：输出截断，超过 10,000 字符时取首尾各 5,000 字符（保留首尾而非只取前 N 字符，因为命令输出的末尾通常包含最有价值的信息：错误信息、统计摘要）

#### 重试与回退机制

- **重试策略**：指数退避重试（处理临时性网络错误）
- **降级策略**：搜索失败时降级到本地索引、API 不可用时降级到缓存数据
- **优雅失败**：工具不可用时返回结构化错误信息而非直接崩溃，包含错误原因和建议操作

## Agent-as-Tool：嵌套组合

### as_tool() 模式

Microsoft Agent Framework 提供 Agent → Function Tool 的桥接：内层 Agent 自带工具与指令，外层 Agent 把它当作一个可调用工具，本质上是**分层委派（hierarchical delegation）**。^[raw/articles/microsoft-agent-framework-tools-overview-provider-matrix.md]

```python
weather_agent = OpenAIChatCompletionClient(...).as_agent(
    name="WeatherAgent",
    description="Answers weather questions for a location.",
    instructions="You answer weather questions only.",
    tools=get_weather,
)
weather_tool = weather_agent.as_tool(
    name="WeatherLookup",
    description="Look up weather for any location",
    arg_name="query",
)
main_agent = OpenAIChatCompletionClient(...).as_agent(
    instructions="Delegate weather to WeatherLookup.",
    tools=[weather_tool],
)
```

**适用 as_tool()**：子域边界清晰、接口稳定、允许模型自主决定何时调用、调用结果为最终答案或结构化数据。

**不适用 as_tool() → 应改用 Workflow**：强执行顺序（第二步依赖第一步输出）、事务性要求（必须 checkpoint 回滚）、多 Agent 必须同步协作。

### 何时用嵌套 Agent vs Workflow

**第一反应不应该是把所有子任务都变成嵌套 Agent**。应该先问「这些子任务之间是否有强数据流依赖或事务要求」——如果有，Workflow 更合适；如果子任务足够独立且接口稳定，as_tool() 可以显著简化架构。

在决定是否使用 as_tool() 构建嵌套 Agent 前，建议做**独立性测试**：
- 子 Agent 是否有清晰的单一职责（不依赖其他子 Agent 协同完成）？
- 子 Agent 的输入输出是否可以被自然语言或简单结构化数据表达？
- 子 Agent 是否允许模型自主决定是否调用（而非每次 run 必须调用）？

## 工具选择的评估框架

### 控制流三要素

每看到一个新「工具架构名词」，先问三个问题：它新增了什么 **state**？它新增了什么 **router**？它新增了什么 **evaluator**？答不出来就不是新架构，只是旧架构换了个名字。 ^[raw/articles/17-agent-architectures-evolution.md]

### 工具系统的演进判断

从 17 种架构的演进视角看工具系统的发展：

| 阶段 | 核心能力 | 代表架构 |
|------|---------|---------|
| 最小质量闭环 | critique pass | Reflection |
| 世界接口 | 结构化工具打破参数知识边界 | Tool Use |
| 观察行动循环 | Thought→Action→Observation 循环 | ReAct |
| 显式规划 | 先生成步骤清单再执行 | Planning |
| 验证驱动 | Plan→Execute→Verify，失败回重规划 | PEV |
| 副作用闸门 | 先预演再执行 | Dry-Run |

### 选型决策树

```
任务复杂度？
├── 单步查询 → 直接 Function Call
├── 多步推理（需要观察结果决定下一步）→ ReAct
├── 复杂长程任务 → Planning + ReAct
└── 需要自我修正 → PEV

稳定性要求？
├── 高确定性下限 → Workflow 硬编码
├── 可接受上限波动 → ReAct + Harness
└── 需要人工审批 → Dry-Run + Tool Approval

工具数量？
├── 少量固定工具 → 直接注册
├── 大量工具 → Intent-Grouped / Thin Surface / On-Demand Loading
└── 跨系统工具 → MCP
```

## 相关实体

### 架构演进与控制流
- [[entities/17-agent-architectures-evolution|17种Agent架构演进：控制流设计的完整演化史]] — 17种架构的工具使用模式对比
- [[entities/agent-evolution-four-stages-six-dimensions-aliyun|Agent 四阶段演化与六维度技术变化]] — 工具维度的 CLI/Script 原生化趋势

### 框架与工具系统
- [[entities/microsoft-agent-framework-tools-overview-provider-matrix|Microsoft Agent Framework Tools 总览]] — 4类工具 + Provider矩阵 + Tool Approval
- [[entities/800行代码实现-open-claw-的-tool消息总线子agent管理架构|OpenClaw Tool 架构]] — 薄抽象 ToolRegistry + exclude() 能力隔离
- [[entities/aws-bedrock-agentcore-os-level-actions-browser|AgentCore Browser OS级操作]] — Action-Screenshot-Reaction 闭环

### 协议与标准
- [[concepts/model-context-protocol-mcp|MCP（Model Context Protocol）]] — Anthropic 提出的 Agent 连接外部系统的标准协议
- [[concepts/tool-use-patterns-ai-agents|Tool Use Patterns in AI Agents]] — 工具设计原则、粒度决策、MCP 交互面模式

### 循环与编排
- Agent 循环设计 — ReAct Loop、Plan-Execute、Reflexion、迭代优化循环

## 实践启示

### 1. 工具描述是产品界面，不是 API 文档

工具的 `description` 字段是模型理解工具用途的核心依据，好的描述应该同时满足人和机器：包含功能名称、输入参数说明、输出结果说明、适用场景、不适用场景。描述要同时面向 Agent 的推理链和人类工程师的维护需求。 ^[raw/articles/anthropic-12-mcp-production-patterns.md]

### 2. 优先 CLI 原生，其次 Script，最后 Function Call

根据 Agent 四阶段演化，工具选型优先级应该是：^[raw/articles/agent-evolution-four-stages-six-dimensions-aliyun.md]
1. **CLI 命令**：模型预训练知识覆盖的标准 Linux/Unix 命令（grep、cat、vim等），零额外成本
2. **第三方 CLI**：遵循 `--help` 规范的工具，模型可运行时自主理解
3. **Script 脚本**：本地与远程统一，协议黑盒化，通过 Skill 包装后赋予 Agent 复杂能力
4. **Function Call / API**：仅在现成 API 满足场景需求时使用，维护成本高

### 3. 工具粒度决策：先「意图探测」

让 Agent 看工具列表，看它能否在 5 分钟内说清楚能用这些工具完成什么任务。如果说不清楚，说明工具描述有问题或者粒度设计不当。这是 Anthropic 推荐的工具设计验证方法。 ^[raw/articles/anthropic-12-mcp-production-patterns.md]

### 4. 凭证托管是生产级集成的门槛

token 不应该放在工具参数、环境变量、临时配置里——生产系统必须把凭证生命周期上移到平台层。MCP Vault-Held Credentials Pattern 是标准做法：创建 session 时引用 vault ID，平台负责把合适的凭证注入到连接中，并处理刷新、撤销和生命周期管理。 ^[raw/articles/anthropic-12-mcp-production-patterns.md]

### 5. 工具结果要处理，不要直接给 LLM

程序化工具调用（在沙箱中处理工具结果）是减少 token 消耗的有效手段：过滤数据、聚合字段、做中间计算，只把必要结果放进模型上下文。Anthropic 官方测试显示可减少约 **37% 的 token 使用**。

### 6. 子 Agent 工具集必须显式隔离

使用 `exclude()` 或类似机制为每个子 Agent 创建受限工具集。排除清单应该包括：spawn（防止递归）、message（防止绕过主 Agent 通信）、edit_file（限制写入能力）、cron（避免调度能力越界）。这是 Agent 系统的「最小权限原则」。

### 7. OS-Level Actions 的正确使用姿势

OS Actions 适合：触发系统打印对话框、需要键盘快捷键（ctrl+s）、右键上下文菜单、系统隐私对话框、证书选择器。CDP/Playwright 适合：页面导航、表单填充、DOM 元素点击、内容提取。两者构成互补层——CDP 处理 Web 层可预测任务，OS Actions 处理「最后一公里」的生产环境边缘情况。

### 8. 工具设计要同时考虑失败模式和降级策略

每个工具调用路径都应该有明确的失败模式和降级策略：重试机制（指数退避）、降级路径（搜索失败降级到本地索引）、优雅失败（结构化错误信息而非崩溃）。

## 核心洞察

**工具使用推理的本质是「决策」与「执行」的解耦**。LLM 负责判断「是否用工具、用什么工具、怎么组合」，这是推理层面的不确定性；框架负责可靠地执行并返回结果，这是执行层面的确定性。这种分离是整个 Agent 架构的核心价值——用模型的「动态判断」替代人类的「预先设计」，同时通过 Harness 层保证工程上的可靠性。

**从「人为适配模型」到「利用模型原生能力」** 是工具维度的根本转变。CLI 命令行原生化意味着模型预训练知识直接成为工具，工具接口退居 Harness 层。这条主线的终点是：模型成为信息世界的「原住民」，Agent 框架退居「驾驭工程」，只负责确定性约束和安全边界。 ^[raw/articles/agent-evolution-four-stages-six-dimensions-aliyun.md]

**Tool Approval + Metacognitive 的组合让 Agent 从「能做」升级到「知道不该做」**。Dry-Run（副作用闸门）+ 自我边界认知的组合，是工具使用推理走向成熟的关键标志。架构演化到此，工具使用已经从「能不能调用」进化到「该不该调用」的层次。 ^[raw/articles/17-agent-architectures-evolution.md]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
