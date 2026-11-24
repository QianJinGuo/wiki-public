---
title: "How we built an MCP bridge: AgentCore-hosted AI agent 访问本地 MCP 工具"
created: 2026-08-06
updated: 2026-11-21
type: entity
tags: [mcp, mcp-bridge, websocket, native-messaging, agentcore, agent-architecture, remote-local]
sources: [raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools]
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# How we built an MCP bridge: 云上 Agent 访问本地 MCP 工具的桥接架构

## 问题：远程 MCP Client ↔ 本地 MCP Server 的协议缺口

MCP（Model Context Protocol）支持两种传输机制：**stdio**（同机本地进程间通信）和 **streamable HTTP**（远程 server/client 通信）。但存在一个缺失场景：**MCP server 在本地、MCP client 在远端**——云上部署的 AI Agent 需要访问用户笔记本电脑上的本地文件/工具（Excel 表格等）。这正是 Claude Cowork（云 Agent 调用本地工具）的产品模式，但本文给出的是完全自托管在 AWS 上的实现，用自己的模型 + 自定义工具 server。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md]

## 桥接方案：WebSocket 隧道 + Native Messaging

实现方式是把 MCP 消息通过 **WebSocket 隧道 + 浏览器 native messaging** 桥接：Agent 部署在 Amazon Bedrock AgentCore 上，MCP server 跑在用户本地机器，二者之间用隧道转发 MCP 消息。云上 Agent 可以直接读取用户机器上的文件（演示场景：汇总本地 Excel 工作簿）。作者团队内部已构建了生产级金融 AI 助手，一年内处理 41,000+ 对话。完整源码开源在 GitHub（aws-samples/sample-mcp-bridge-agentcore）。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md]

## 生产硬化

文章在 What's Next 部分讨论额外的生产加固措施（认证、隧道安全、断线恢复等）。这是"MCP 传输层缺口"的工程解法——MCP 标准本身没有定义 remote-client → local-server 方向的传输，需要应用层自行桥接。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md]

## 深度分析

### 四组件拓扑与"逐跳剥封"的消息流

整体架构由四个组件构成：AgentCore runtime 在云端托管 Strands agent（充当 MCP client）；浏览器扩展提供聊天界面并作为双向中继；MCP Bridge 是跑在用户本地机器上的 FastMCP proxy，负责在 native messaging 信封格式与原生 MCP JSON-RPC 之间做翻译；MCP server 则作为标准本地进程与 bridge 同机部署，走 stdio 传输。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:23-33]

这个设计的精髓在于：扩展和 bridge 都是"不理解 MCP 语义"的纯中继层——工具调用经 WebSocket → native messaging → stdio 三段链路，每跳只剥一层信封，JSON-RPC payload 全程透传。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:34-45]

### 云端侧：presigned WebSocket 与并发请求关联

浏览器扩展不持有任何 AWS 凭证：启动时经 native messaging 向本地 bridge 请求预签名，bridge 用本机凭证 + bedrock-agentcore SDK 生成 SigV4 签名的 `wss://` URL（限定 runtime ARN，5 分钟有效），凭证不出用户机器、不进浏览器。掉线时 side panel 2 秒后自动换新 URL 重连，过期窗口对用户不可感知。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:46-51]

两个工程决策：其一，agent 每条消息都重新 `tools/list` 并把 schema 动态包装成 Strands AgentTool，新增工具下次请求即自动可用、agent 零改动；其二，每条出站请求以 `(session_id, jsonrpc_id)` 为键注册 `asyncio.Future`，响应按 ID 匹配唤醒，多个工具调用可并发在途。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:52-54]

### 本地侧：native messaging 的约束与双循环 bridge

选 native messaging 是为了绕开两个硬约束：扩展无网络权限访问 localhost，逐消息弹窗不可接受。浏览器在固定路径查找 manifest（`allowed_origins` 白名单限定可启动的扩展 ID），进程生命周期与连接等长，消息为 UTF-8 JSON 加 32 位长度前缀。注意消息尺寸是非对称的——native host 发往浏览器的消息上限 1 MB（防恶意本地程序），浏览器发往 native host 的上限是 64 MiB。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:56-80]

bridge 内部跑两个并发循环、经 FastMCP proxy 衔接：主循环从浏览器读消息、解信封、放入输入队列；proxy 常驻，取消息写入 MCP server 子进程 stdin，把 stdout 响应放进输出队列；第二个后台循环取响应、重新封装写回浏览器。双循环 + 队列把浏览器请求节奏与 server 处理速度解耦，慢工具执行时 bridge 不阻塞、仍能接收下一请求。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:84-99] MCP server 本身是 bridge 启动时按 `mcp.json` 拉起的常驻子进程，无逐请求进程开销，新增一个 server 只改一行配置。

### 安全模型：三层现有防线与四层生产缺口

演示版只保留三层措施：native messaging 的扩展 ID 来源校验、SigV4 预签名 URL 的 5 分钟过期、扩展/bridge/MCP server 三者各自独立进程无共享内存。文章明确指出架构**独有的暴露面就是 bridge 本身**——它接受云端 agent 指令、以用户文件系统权限在本地执行；核心原则：agent 的访问不得超过用户明确授予的范围，且用户始终能看到调用了哪些工具、参数是什么。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:204-219]

生产化的四层加固各有明确针对性：JWT 握手（WebSocket 第一帧）防预签名 URL 泄露后的未授权使用；Ed25519 逐消息签名（私钥在 agent、公钥在 bridge）防云到本机传输途中被篡改；目录白名单（越界路径直接拒绝）防 prompt injection 诱导 agent 读取 SSH 密钥等敏感文件；本地审计日志（工具名、参数、时间戳、结果状态）提供事后可追溯性。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:220-223] 这一分层与 [[concepts/prompt-injection-defense|prompt injection 防御]]中"权限边界 + 全量审计"的思路一致。

## 实践启示

1. **传输缺口要在应用层自桥接，且中继层保持协议无关。** MCP 标准只定义了 stdio 与 streamable HTTP 两种传输，remote-client → local-server 方向是空白；解法是让扩展和 bridge 做纯信封中继（每跳剥一层包装），JSON-RPC payload 原样透传——这样任何 stdio MCP server 零改造接入，也不被未来 MCP 协议演进锁死。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:34-45]
2. **凭证不出本机：用本地签发的 presigned URL 做云端连接。** 浏览器永不接触 AWS 凭证，presigned URL 由本地 bridge 签发、5 分钟短时效 + 自动重连兜底；生产环境再叠加 JWT 首帧握手与 Ed25519 载荷签名，形成"凭证不泄露 ≠ 通道不被滥用"的双保险。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:46-51]
3. **用队列解耦中继与慢工具。** 双循环 + FastMCP proxy 让 bridge 不等上一个慢工具返回就接收下一请求——适用于一切消息中继/代理场景（网关、隧道、sidecar）：接收与执行路径分离。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:84-99]
4. **动态工具发现优于静态注册。** 每条消息重新 `tools/list`，新增 MCP server 只是 `mcp.json` 一行配置、无需重新部署 agent——对比硬编码工具注册表，这让本地工具生态可以独立于 agent 发版节奏演进。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:52-54]
5. **文件系统白名单是这类架构的第一道必须项。** bridge 以用户权限执行云端指令，最大的攻击面是 prompt injection 诱导越界读文件（SSH 密钥、凭证）；目录 allowlist + 逐调用审计日志应作为上线门槛而非可选项。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:220-223]
6. **面向终端用户分发时打包成单二进制。** 生产版用 PyInstaller 把运行时、依赖、配置打进单可执行文件，manifest 直接指向它——用户无需装 Python 或管虚拟环境，首次启动扩展即可用。这条路径已被作者团队内部一年 41,000+ 对话的金融 AI 助手验证。^[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools.md:249-265]

## 与 Wiki 现有知识的关联

- 与 [[entities/smartsheet-remote-mcp-server-aws-architecture|Smartsheet 远程 MCP Server]] 互补：Smartsheet 是 local-agent → remote-server，本文是 remote-agent → local-server，构成 MCP 双向桥接全景
- Claude Cowork 生态：[[entities/claude-managed-agents-self-hosted-sandbox-mcp-tunnels-enterprise|Claude Managed Agents MCP Tunnels]] 是企业版托管隧道，本文是自托管实现
- MCP 协议背景见 MCP 协议生态

→ [[raw/articles/how-we-built-an-mcp-bridge-to-give-our-agentcore-hosted-ai-agent-access-to-local-mcp-tools|原文存档]]
