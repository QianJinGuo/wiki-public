---
title: "peerd: 浏览器原生的 AI Agent Harness"
type: entity
tags: [agent-harness, browser, security, sandboxing, peer-to-peer, chrome-extension]
created: 2026-06-25
updated: 2026-08-29
sources: [raw/articles/peerd-browser-native-agent-harness]
review_value: 9
review_confidence: 8
review_stars: 5
related: [entities/gemini-35-flash-computer-use-agent-harness]
---

# peerd: 浏览器原生的 AI Agent Harness

peerd 是第一个完全运行在浏览器中的 [[concepts/harness-engineering-framework|AI Agent Harness]]——一个 Chrome/Firefox 扩展，在你现有的浏览器中运行完整的 agent loop，驱动你的标签页，启动沙箱化计算环境（JS Notebook、WASM Linux VM、客户端应用），并通过 WebRTC 网络实现 peer-to-peer 的 agent 间通信。 ^[raw/articles/peerd-browser-native-agent-harness.md]

## 架构创新

peerd 的核心设计决策是**将浏览器本身作为运行时和安全模型**，而非在浏览器之上构建额外的隔离层：^[raw/articles/peerd-browser-native-agent-harness.md]


- **V8 isolates** 提供沙箱隔离
- **WebCrypto** 管理密钥保险库
- **WebAuthn passkeys** 解锁访问
- **Opaque-origin iframes** 实现内容隔离
- **Subresource Integrity** 验证代码完整性

peerd **不自行编写任何加密或进程隔离代码**——完全复用浏览器平台数十年积累的安全基础设施。^[raw/articles/peerd-browser-native-agent-harness.md]


## 安全模型：双角色架构

这是 peerd 最独特的设计贡献： ^[raw/articles/peerd-browser-native-agent-harness.md]

1. **Key-holder agent**：持有 API 密钥，负责推理和决策，但**永远不接触原始页面内容**
2. **Disposable runner**：无密钥、无网络权限的临时执行器，负责读取页面内容
3. **Fenced output**：runner 的输出被标记为不可信（fenced as untrusted） ^[raw/articles/peerd-browser-native-agent-harness.md]
4. **动作验证**：每个 agent 驱动的操作都必须在实际页面上验证后才算完成

这种分离确保了即使 runner 被注入恶意内容（prompt injection），key-holder 也不会泄露密钥。^[raw/articles/peerd-browser-native-agent-harness.md]


## 计算沙箱

peerd 提供三层沙箱化计算环境： ^[raw/articles/peerd-browser-native-agent-harness.md]

- **JS Notebooks**：轻量级代码执行
- **WASM Linux VMs**：完整的 Linux 环境编译为 WebAssembly
- **Client-side apps**：个人应用的本地运行

所有计算都在浏览器沙箱内完成，无后端依赖。^[raw/articles/peerd-browser-native-agent-harness.md]


## Peer-to-Peer Agent 通信（Preview）

peerd 的 preview channel 支持通过 WebRTC 进行 agent-to-peer 通信：^[raw/articles/peerd-browser-native-agent-harness.md]


- **无中心服务器**：agent 之间直接通信
- **WebRTC 数据通道**：低延迟、端到端加密
- **DWeb 协议**：研究级别，仍在演进中

## BYOK 模式

Bring Your Own Key——用户可以连接任何模型提供商（OpenAI、Anthropic、本地模型等），peerd 不强制绑定特定模型。 ^[raw/articles/peerd-browser-native-agent-harness.md]

## 与其他 Agent Harness 的差异化

| 维度 | peerd | Claude Computer Use | OpenAI Operator | 传统 Browser Agent |
|------|-------|--------------------|-----------------|-------------------|
| 运行位置 | 浏览器扩展 | 云端/API | 云端 | 云端/本地 | ^[raw/articles/peerd-browser-native-agent-harness.md]
| 后端依赖 | 无 | 有 | 有 | 通常有 |
| 安全模型 | 浏览器原生 + 双角色 | API 层 | 云端沙箱 | 自定义 |
| Agent 间通信 | P2P WebRTC | 无 | 无 | 无 | ^[raw/articles/peerd-browser-native-agent-harness.md]
| 密钥管理 | WebCrypto vault | API 传参 | 云端 | 环境变量 |
| 沙箱 | V8 isolates + WASM | 无 | 云端 | 自定义 |

## 当前状态

0.x 实验性 beta。初始功能已完成集成，但 API 表面仍在变动，breaking changes 可能发生，存储格式可能变化。

## 三个独有贡献（不应合并到现有 entity）

1. **双角色安全架构**：key-holder agent 与 disposable runner 的分离，确保密钥持有者永远不接触原始页面内容
2. **浏览器原生安全复用**：完全依赖 V8 isolates/WebCrypto/WebAuthn，不自建加密或隔离代码
3. **P2P Agent 通信**：WebRTC 基础的去中心化 agent 间通信，无中心服务器依赖

## 相关实体

- [[concepts/harness-engineering-framework|Agent Harness 工程框架]]
- Computer Use Agent
- [[entities/gemini-35-flash-computer-use-agent-harness|Gemini 3.5 Flash Computer Use Agent Harness]]
- [[concepts/agent-security-architecture|Agent 安全架构]]
