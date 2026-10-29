---
title: "前端如何消费 Agent 的 SSE 流 — Agent 前端工程实践"
created: 2026-07-24
updated: 2026-10-29
type: entity
tags: [agent, sse, frontend, streaming, vue, agent-loop, event-driven]
confidence: 0.8
provenance_state: extracted
sources: [raw/articles/前端如何消费agent-sse流, raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 前端如何消费 Agent 的 SSE 流 — Agent 前端工程实践

## 摘要

在 mini-openclaw 框架实践中，前端通过 SSE（Server-Sent Events）消费 Agent 运行过程中的事件流，实现实时 Agent 聊天页面。Agent 只负责产出事件，外部消费者决定如何展示——CLI 打印到终端，Web 页面将事件转 SSE 后由前端逐帧渲染。前端通过 `fetch + ReadableStream` 代替原生 EventSource，自行实现流式解析、缓冲拼接和多类型事件分发。^[raw/articles/前端如何消费agent-sse流.md]

## 核心要点

- **事件驱动架构**：Agent 运行过程中不断 yield 事件（文本 token、推理片段、工具调用、工具结果），前端通过 SSE 流实时消费，而非等待完整响应^[raw/articles/前端如何消费agent-sse流.md]
- **fetch + ReadableStream 替代 EventSource**：因为聊天接口需要使用 POST 方法携带用户输入和 Agent ID，原生 EventSource 仅支持 GET，故采用 `fetch + response.body.getReader()` 自行实现流式解析^[raw/articles/前端如何消费agent-sse流.md]
- **流式解码与缓冲拼接**：使用 `TextDecoder` 的 `{ stream: true }` 模式处理多字节字符跨 chunk 截断问题，维护 buffer 变量实现跨 chunk 事件完整性^[raw/articles/前端如何消费agent-sse流.md]
- **自定义事件协议**：SSE 的 `data:` 字段使用前缀编码事件类型（`__tool_start__`, `__tool_result__`, `__reasoning__`, `__error__`, `__done__` 等），后端只需 yield 字符串，前端按前缀分发^[raw/articles/前端如何消费agent-sse流.md]
- **事件时间线建模**：前端 Store 将历史事件（historicalEvents）和实时事件（liveEvents）合并为一条事件时间线，用户消息、模型输出、工具调用、工具结果统一建模为 RenderableConversationEvent^[raw/articles/前端如何消费agent-sse流.md]

## 关键技术实现

### SSE 流解析

后端路由返回 `StreamingResponse`，其中 `stream_chat()` 是一个 generator，不断 yield SSE 格式文本：^[raw/articles/前端如何消费agent-sse流.md]


```
data: 我先看一下文件。

data: __tool_start__:{"tool_call_id":"call_1","name":"read_file","args":{"path":"a.txt"}}

data: __tool_result__:{"tool_call_id":"call_1","name":"read_file","output":"...","is_error":false}

data: __done__
```

每个事件由 `data:` 行组成，事件之间用空行 (`\n\n`) 分隔——这是前端解析时最重要的边界。^[raw/articles/前端如何消费agent-sse流.md]

### 跨 chunk 拼接

由于网络数据以不定长 chunk 传输，事件边界与 chunk 边界可能不对齐，前端维护一个 buffer：^[raw/articles/前端如何消费agent-sse流.md]


```javascript
buffer += decoder.decode(value, { stream: true }).replace(/\r\n/g, '\n')
const blocks = buffer.split('\n\n')
buffer = blocks.pop() || ''
```

这段逻辑确保：新读到的文本追加到 buffer、用空行切分完整 block、不完整的最后一个 block 留在 buffer 中等待下一个 chunk。^[raw/articles/前端如何消费agent-sse流.md]

### 多行 data 拼接

SSE 允许多个 `data:` 行组成一个事件。后端发普通文本时按行拆分为多条 `data:`，因此前端必须将多条 `data:` 行 join 回来：^[raw/articles/前端如何消费agent-sse流.md]


```javascript
const dataLines = block
  .split('\n')
  .filter((line) => line.startsWith('data:'))
  .map((line) => (line.startsWith('data: ') ? line.slice(6) : line.slice(5)))
const data = dataLines.join('\n')
```

拿到 data 后，按前缀分发到不同回调函数。^[raw/articles/前端如何消费agent-sse流.md]

### Store 中的事件时间线

`stores/chat.ts` 管理几类核心状态：sessions（会话列表）、currentSessionId（当前会话）、sending（发送状态）、historicalEvents（已落盘的历史事件）、liveEvents（本次请求的临时事件）。页面渲染的数据源是 `currentTimelineEvents` computed 属性：^[raw/articles/前端如何消费agent-sse流.md]


```javascript
const currentTimelineEvents = computed(() => {
  const fallbackEvents = createFallbackEvents(currentSession.value?.messages ?? [])
  const baseEvents = historicalEvents.value.length > 0 ? historicalEvents.value : fallbackEvents
  return [...baseEvents, ...liveEvents.value]
})
```

这样页面可以同时显示已落盘的旧会话和本次请求中正在发生的 token、工具调用和工具结果。^[raw/articles/前端如何消费agent-sse流.md]

### 事件协议表

| SSE data 内容 | 前端回调 | 页面含义 |
|---|---|---|
| 普通文本 | `onToken(token)` | assistant 正文继续追加 |
| `__reasoning__:{...}` | `onReasoning(info)` | reasoning 片段继续追加 |
| `__tool_start__:{...}` | `onToolStart(info)` | 出现 pending 的工具调用 |
| `__tool_result__:{...}` | `onToolResult(info)` | 工具调用结束，显示输出和错误状态 |
| `__anomaly__:{...}` | `onAnomaly(info)` | 异常检测事件 |
| `__error__:...` | `onError(message)` | 本次请求失败 |
| `__done__` | `onDone()` | 本次流结束 |

## 深度分析

### 事件驱动架构在 Agent UI 中的必然性

Agent 执行过程天然是事件驱动的——模型调用、工具执行、结果处理交替进行，不存在单一的"完成时刻"。传统 REST API 的请求-响应模式无法表达这种动态过程，而 SSE 流提供了一种**单向实时通道**，让后端在事件发生时立即推送给前端。这种架构选择不是技术偏好，而是 Agent 交互范式的必然要求。^[raw/articles/前端如何消费agent-sse流.md]


将事件编码在 `data:` 内容前缀而非标准 `event:` 字段，是一个务实的工程决策。后端不需要处理 SSE 的事件类型映射，只需 yield 带前缀的字符串；前端也只需简单的字符串匹配即可分发。这种"约定优于配置"的方式降低了实现复杂度，同时保持了扩展性——新增事件类型只需增加一个前缀分支判断。^[raw/articles/前端如何消费agent-sse流.md]


### 流式缓冲作为 Agent UI 的基础设施

前端工程最容易被低估的复杂度在于**字节流与事件流之间的映射层**。网络运输层没有事件边界概念，HTML5 Streams API 返回的是随机的字节片段，可能截断 UTF-8 字符的中间字节。TextDecoder 的流式模式、buffer 拼接、空行分割——这几行代码实际上构成了 Agent 实时 UI 的基础设施层。任何不处理跨 chunk 边界的实现，在多字节字符或高并发场景下都会出现 UI 渲染错乱，而这种错误往往难以复现和定位。^[raw/articles/前端如何消费agent-sse流.md]


### 事件统一建模的设计价值

前端将 token、reasoning、tool call、tool result 都统一建模为 `RenderableConversationEvent`，这一抽象将 Agent 聊天页面的本质从"文本流展示"重新定义为**事件时间线渲染**。页面维护的不是一个大字符串，而是一条由用户消息、模型输出、工具调用、工具结果组成的时序事件列表。这种建模方式使得：^[raw/articles/前端如何消费agent-sse流.md]

- 历史会话重放变得自然——直接回放事件列表
- UI 组件可以针对不同事件类型做差异化渲染（如 tool call 用卡片、文本用气泡）
- 工具链可以无缝集成——后续的 evaluation、trace 标记等新事件类型只需新增一个回调

### 与 AI Agent 生态的对接模式

mini-openclaw 的 SSE 流设计遵循了 Agent Loop 的核心思想：Agent 只产生事件，不关心消费方式。这种"关注点分离"（Separation of Concerns）使得同一个 Agent 后端可以同时服务于 CLI、Web、移动端等多种前端，只需覆盖不同的事件渲染逻辑。这与 MCP 协议中"工具提供能力，客户端决定编排"的设计哲学一脉相承。^[raw/articles/前端如何消费agent-sse流.md]


## 实践启示

1. **始终使用 `fetch + ReadableStream` 而非 `EventSource`**：Agent 聊天接口需要 POST 方法传输用户输入和 Agent 配置，原生 EventSource 仅支持 GET。自行实现流式解析虽然多几行代码，但提供了完全的控制力。

2. **流式解码的三个关键细节**：(1) `TextDecoder` 的 `{ stream: true }` 处理多字节字符跨 chunk；(2) `\r\n` 统一替换为 `\n` 保持换行稳定；(3) buffer 拼接后取最后一个 block 留作不完整段，而非丢弃。

3. **用前缀编码代替 SSE event 字段**：将事件类型编码在 `data:` 内容前缀里（如 `__tool_start__:`），比使用标准 event 字段更简单。新增事件类型只需在分发函数中增加一个 `if (data.startsWith(...))` 分支。

4. **确保未知前缀不被当成正文显示**：在调用 `onToken(data)` 之前先匹配所有前缀分支，否则未知前缀的前缀文本会被错误地追加到 assistant 正文中。

5. **Store 层维护事件时间线而非纯文本**：Agent 聊天页面展示的不是单纯的文本流，而是一次任务执行过程。前端应当维护事件列表而非拼接字符串，这样 history 回放、live streaming、异常处理等场景都能统一处理。

## 相关实体

- [[entities/cli-agent-patterns-mcp-shell-agents|CLI Agent 模式与 MCP Shell Agent]]
- [[entities/agent-orchestration-multi-agent-systems|多 Agent 编排系统]]
- [[entities/spec-driven-development-harness|Spec-Driven Development]]
- [[entities/skill-orchestration-6-dependencies|Skill 编排与依赖管理]]

→ [[raw/articles/前端如何消费agent-sse流|原文存档]]

---

## 第 2 来源：携程机票 APP 的 SSE 全栈落地（2026-10-29，SUPP）

前文讲的是"前端如何消费 Agent 的 SSE 流"；携程机票这篇把视角扩到**从 Web 到 APP（Native + RN）的全栈落地与性能优化**，补齐了三块前文没有的工程细节：全链路代理、端侧实现分层、以及预热/缓存/压缩这组优化。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]

### 全链路：SSE 卡在 7 层代理上

业务的流量链路不是客户端—服务端直达，完全落地 SSE 需要全链路支持：7 层加速节点（akamai/aws）、流量接入层（slb）、中间转发节点（虫洞）、sidecar（envoy/nginx）。分块传输本质是传数据碎片，**中间层各自实现并不遵循统一标准**——nginx 默认会缓存 response。处理办法：链路上所有 nginx 主动关闭报文缓存并透传 `proxy_pass_header X-Accel-Buffering`，同时开启数据压缩。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]

### 端侧三层：云端 / Native / RN Plugin

APP 侧基于 `text/event-stream` 响应体持续推送、通过 Event 处理数据，分三层实现（云端、native 层、RN 业务层），并在携程公共框架团队的 SSE 基础能力之上给业务提供接口；RN 侧另加 Plugin，分别处理 `onOpen`（建连）、`onMessage`（批次数据）、`onClosed`（完成）、`onError`（报错）。选型理由三条：移动平台可走原生网络库、同一套 SSE 服务同时支持 APP 与 H5（避免多套实时方案）、**单向推送的连接维护成本低于 WebSocket**，弱网/网络切换时重连逻辑更灵活。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]

### 四项优化：预建连、预请求、请求缓存、报文压缩

- **预建连**（host 维度）：真正请求前提前建好连接；同 host 多请求在协议一致且连接空闲时自动复用，时机可在业务入口初始化触发。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]
- **预请求**（request 维度）：在前序流程里提前请求数据，做单一 request 的链路复用。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]
- **请求缓存**：命中时复用缓存数据或复用请求链路；用父子任务管理真实请求与在途——父任务管真实 network 请求，相同 cachekey 的在途请求作为子任务挂载，同步接收实时数据，并按状态标识复用情况。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]
- **报文压缩**：P90 上报文 size 与链路耗时显著正相关，必须开压缩。切换到 SSE 时压缩算法从 ZSTD 变成 Gzip，导致报文 size 显著上升（首批数据耗时出现少量负向影响），因此**把编码从 Gzip 换成 Brotli**。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]

### 数据与坑

**数据**：列表页多批次加载完成的总耗时**平均提升 11.8%**；首批数据耗时无明显差异；链路侧减少 RN 与 Native 的通信（数十毫秒）与一次请求，并避免"服务端数据已 ready、还要等前端请求到达才下发"。压缩侧：Brotli 压缩率与 ZSTD 基本一致，平均报文比 Gzip 小约 20%、超长报文小约 30%，SSE-Br 比 SSE-Gzip 快约 20ms；服务端内存/CPU 无明显差异、平均报文压缩耗时下降 30%（基数小，主要用来证明无负向收益），且 Brotli 是标准 HTTP 编码、框架接入成本低。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]

**三个坑**：①**SSL Pinning**——接 SSE 前用 TCP、之后改 HTTP，部分使用 VPN 等网络环境的用户会无法访问；②**链路一致性**——上云/海外访问存在远距离传输，需确认所有地区都支持 SSE，否则出现部分地区分批失效（表现为多批数据同一时间到达），要降级处理并向服务商报障；③**重连风暴**——原生支持自动重连但暂未启用，若启用需防范极端场景大量用户重连压垮服务端。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]

### 异构分批：顺手解决 API 粒度与视图粒度的矛盾

接口粒度常以资源为导向，而前端视图需求与后端资源划分不一致，导致前端要处理多个接口的请求时序、渲染时序与逻辑 PK 关系。把不同功能的接口包装成**异构多批数据聚合在同一个 SSE 请求里分批下发**，可把逻辑下沉到服务端（是否分批、如何分批由服务聚合控制）、简化时序边缘场景与多端复用，也显著降低预请求与缓存的实现复杂度；文中给出该场景下与 BFF 聚合、GraphQL 的横向对比。作者的最后一句判断值得记住：**SSE 的价值不是比 WebSocket 更快，而是用标准 HTTP 的最小代价同时获得服务端主动推送、全链路复用与原生重连异常处理**。^[raw/articles/ctrip-flight-sse-full-stack-streaming-app-2026-10-29.md]
