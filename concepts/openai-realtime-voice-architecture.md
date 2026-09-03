---
title: "OpenAI Realtime API: relay + transceiver 架构"
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [openai, realtime-api, voice, architecture, webrtc, inference-serving]
sources: ['raw/articles/openai-realtime-api-architecture', 'raw/articles/openai-realtime-api-relay-transceiver']
confidence: high
related:
  - OpenAI
  - [[concepts/managed-agents-architecture|Managed Agents 架构]]
  - [[concepts/inference-optimization|Inference 优化]]
  - [[entities/livekit-agents-voice-ai-framework|LiveKit Agents 语音 AI 框架]]
---
# OpenAI Realtime API: relay + transceiver Architecture
> OpenAI's delivery-low-latency-voice-ai-at-scale technical blog. Core architecture: stateless relay (UDP forwarder) + stateful transceiver (full WebRTC session). Achieves <0.3s first-response latency serving hundreds of millions of weekly active users. Replaces traditional SFU approach.
---
## Architecture Overview
```
Traditional SFU (not used):
Client ↔ SFU (stateful) ↔ AI
  → Problem: unnecessary multi-party infra for 1:1; state everywhere
OpenAI's relay + transceiver:
Client → relay (stateless UDP) → transceiver (full state) → AI inference / TTS / transcription
```
### relay: Stateless UDP Forwarder
**Design constraints**: No decryption, no decoding, no protocol negotiation, no session state.
**What it holds**:
- In-memory forward mapping: `client → transceiver IP:Port`
- A few monitoring counters
- Expiry timers
**What it doesn't hold**: Nothing persistent. If it crashes, the next STUN packet auto-rebuilds routes.
**Routing trick**: ICE ufrag (carried in every packet header) encodes routing hint — **zero external lookup on first packet**.
### transceiver: Stateful Session Endpoint
**What it owns**:
- ICE connectivity checks
- DTLS handshake
- SRTP media decryption
- Complete session lifecycle
**Why it works**: All protocol complexity concentrated here; backend AI services scale as plain services, no WebRTC knowledge required.
---
## Global Relay: Geographic Distribution
**Problem**: Beijing user → US West Coast → 150ms+ one-way latency.
**Solution**: Client packets enter OpenAI network at the geographically nearest relay. Internal backbone carries them to transceiver.
```
User (Beijing) → Global Relay (Beijing/PoP) → Internal backbone → Transceiver cluster
                 ↑ nearest entry point
```
**Benefit**: Lower latency, lower jitter, fewer packet losses, no public internet traversal for AI processing path.
**K8s advantage**: Small fixed public port exposure vs. thousands per session with traditional WebRTC.
---
## Go Optimization for Real-Time Media
### Why Go (unconventional choice)
Industry standard: C/C++ or Rust for real-time media forwarding. Some teams go to kernel bypass. OpenAI chose Go because it was **sufficient for current load**.
### Three Go-level optimizations
| Optimization | What it does | Why it matters |
|-------------|--------------|----------------|
| `SO_REUSEPORT` | Multiple relay processes share UDP port; OS distributes packets across them | No single-process bottleneck |
| `runtime.LockOSThread` | Goroutine reading UDP pinned to fixed thread | Same call's packets → same CPU core → better cache hit rate |
| Pre-allocated buffers | No runtime allocation in hot path | Avoids GC pauses in forwarding path |
---
## Three Design Principles
### 1. Hard state in one place
transceiver owns ICE/DTLS/SRP and session lifecycle. relay only forwards. When something breaks, you check one place.
### 2. Route on information already present
ICE ufrag is a protocol-native identifier. Encoding routing hint in ufrag means first packet routes without hot-path external query.
### 3. Don't upgrade until you have to
Go + a few OS-level optimizations were sufficient. No kernel bypass. "Get it working first, then decide if you need heavier approaches."
---
## Key Technical Decisions
| Decision | Alternative considered | Why OpenAI chose this |
|----------|----------------------|---------------------|
| relay + transceiver | SFU (standard) | SFU's multi-party infra unnecessary for 1:1; adds latency |
| Stateless relay | TURN | TURN requires relay to hold client connection state |
| Go for relay | C/Rust/kernel bypass | Sufficient for current scale; simpler ops |
| Global distributed relays | Single-region deployment | 150ms+ round-trip makes conversation feel broken |
---
## Performance
- **First-response latency**: < 0.3 seconds (vs [[entities/livekit-agents-voice-ai-framework|LiveKit Agents]] at 500-800ms with traditional cascade pipeline)
- **Weekly active users**: Hundreds of millions
- **Relay cluster size**: Relatively small (due to stateless design)
---
## WebRTC 协议栈深度解析

OpenAI 选择 relay + transceiver 架构而非传统 SFU，核心原因在于对 WebRTC 协议栈的深度理解。以下是各层协议的详细解析。

### WebRTC 层级架构

WebRTC 是一个复杂的协议栈，从上到下包括：

- **应用层**：媒体协商（SDP）、会话描述
- **传输层**：SRTP（安全实时传输）、SCTP（数据通道）
- **信令层**：DTLS（数据报传输层安全）、ICE（交互式连接建立）
- **网络层**：STUN、TURN、ICE candidate 候选

传统 SFU 将这些复杂性分散在服务端，而 OpenAI 的做法是将完整 WebRTC 状态集中在 transceiver，relay 只做无状态的 UDP 转发。

### ICE 协议与 ufrag 路由设计

ICE（Interactive Connectivity Establishment）是 WebRTC 建立连接的核心协议，其核心思想是：**双方交换尽可能多的候选地址对，通过连通性检查找到最优路径**。

ICE candidate 包含：
- **类型**：host（本地网卡）、srflx（公网反射）、relay（TURN 中继）
- **协议**：UDP/TCP
- **优先级**：用于候选排序
- **ufrag**：ICE 用户片段，存在于每个 STUN/RTP 包头部

OpenAI 的关键优化是**将 ufrag 作为路由 hint 编码在每个包的头部**，使得 relay 可以在不查询会话表的情况下直接转发到对应的 transceiver。这个设计的精妙之处在于：

1. ufrag 是 ICE 协议原生字段，不需要额外的协议扩展
2. 每个 STUN/RTP 包都携带 ufrag， relay 可以从包本身提取路由信息
3. 即使 relay 重启，只要客户端发送下一个包，ufrag 可以重建路由

### DTLS 握手机制

DTLS（Datagram TLS）是 WebRTC 加密的核心协议，其设计考虑到了 UDP 的不可靠特性：

- **重传机制**：DTLS handshake 包需要重传，因为 UDP 可能丢包
- **序列号**：每个包有序列号，防止重放攻击
- **Certificate Verify**：验证端点身份

OpenAI 将完整 DTLS 握手放在 transceiver 的理由是：**DTLS 需要维护会话状态**（握手进度、证书链、加密密钥），这与 relay 的无状态设计矛盾。

### SRTP 媒体加密

SRTP（Secure RTP）是 WebRTC 音视频加密的标准协议：

- **加密**：AES-CM（AES Counter Mode）
- **认证**：HMAC-SHA1
- **密钥交换**：SDES（Session Description Protocol Security Descriptions）或 DTLS-SRTP

transceiver 在处理 SRTP 时的职责：
1. 维护 SRTP 会话密钥
2. 解密收到的媒体包
3. 加密要发送的媒体包
4. 处理 SRTP 重放保护（维护重放窗口）

### 与传统 SFU 的协议开销对比

传统 SFU（如 mediasoup、Janus）需要在服务端维护：
- 每个参与的媒体流的状态
- SFU 与每个客户端的 DTLS/SRTP 会话
- 媒体路由表（谁订阅谁）

这导致 SFU 的内存使用与参与会话数成正比。而 OpenAI 的架构将状态限制在 transceiver（仍然是与会话数相关），但 relay 完全无状态，可以无限水平扩展。

## OpenAI relay + transceiver 与 LiveKit Agents 的深度对比

[[entities/livekit-agents-voice-ai-framework|LiveKit Agents]] 是另一个重要的实时语音 AI 框架，其架构设计与 OpenAI 有显著差异，理解两者的区别有助于在不同场景下做出技术选型。

### 架构哲学的根本差异

**OpenAI**：将复杂性从基础设施转移到 AI 推理层
- relay 负责无状态 UDP 转发，transceiver 负责完整 WebRTC 会话
- AI 推理服务作为"纯后端"，不需要理解 WebRTC 协议
- 延迟优化核心在网络层（relay 分布）

**LiveKit**：将 AI 推理深度嵌入实时媒体管线
- VAD → STT → LLM → TTS 四层流式级联
- AI 组件是媒体管线的一部分，而非独立服务
- 延迟优化核心在 AI 推理层（流式输出）

### 延迟对比与根因分析

| 指标 | OpenAI | LiveKit Agents |
|------|--------|----------------|
| 端到端延迟 | < 300ms | 500-800ms（ cascade pipeline） |
| 延迟来源 | 网络传输 | STT 完整转写 + LLM 生成 + TTS 合成 |
| 延迟优化策略 | 地理分布 relay | 流式级联管线 |

OpenAI 的 <300ms 延迟建立在两个前提上：
1. **端到端模式**：AI 推理直接处理音频流，而非先 STT 再 LLM
2. **极简信令**：relay 的首包路由零查询

LiveKit 的 500-800ms 延迟是其架构的固有瓶颈：
- STT 需要积累一定音频才能开始转写（通常 200-500ms）
- LLM 生成第一个 token 前需要完整理解输入
- TTS 合成第一个音频帧需要完整句子

### 扩展性对比

**OpenAI 的扩展性优势**：
- relay 无状态，可任意水平扩展
- transceiver 与 AI 推理解耦，可以独立扩展
- 数百万人同时使用，但 relay 集群相对较小

**LiveKit 的扩展性特点**：
- 每个 Agent 连接占用一个 WebSocket 连接
- Agent 实例需要维护媒体会话状态
- MCP 工具调用可能引入额外延迟

### 功能丰富度对比

| 功能 | OpenAI | LiveKit |
|------|--------|---------|
| 多 Agent 交接 | 不支持 | 原生支持 |
| MCP 工具集成 | 不支持 | 原生支持 |
| SIP 电话 | 不支持 | 原生支持 |
| 自托管 | 不支持 | Apache 2.0 开源 |
| 供应商锁定 | 有（OpenAI） | 无（自托管） |

## 工程实践：从 OpenAI 架构学到的设计原则

### 状态极简化原则

OpenAI relay + transceiver 架构的核心洞见是：**硬状态只放一处**。在设计分布式系统时，这个原则有广泛的适用性：

1. **识别系统中的状态持有者**：哪些组件必须维护会话状态？
2. **将状态推到边缘**：将状态集中在与用户直接交互的节点，减少状态在网络中的流动
3. **无状态路径水平扩展**：所有无状态组件可以无限制地水平扩展

### 路由信息自包含原则

将路由信息编码在协议自身字段中（如 ICE ufrag），使得中间节点可以在不查询外部数据的情况下完成路由。这个原则的应用场景：

- HTTP 的 Cookie 自包含路由信息
- gRPC 的 metadata 传递调用上下文
- 消息队列的消息属性自包含路由 hint

### 延迟预算的跨层优化

OpenAI 实现 <300ms 延迟的关键不是单个层的优化，而是**从网络层到 AI 推理层的协同优化**：

- 网络层：地理分布 relay 减少首包路由延迟
- 传输层：UDP 直连减少握手延迟
- 应用层：端到端 AI 处理减少级联延迟

这种跨层优化思维在设计其他实时系统时同样适用。

## 延伸：与 Agent Harness 架构的对比

relay + transceiver 的设计哲学与 [[concepts/managed-agents-architecture|Managed Agents 架构]] 中的 brain-hand 分离高度共鸣，同时也可与 [[entities/build-live-translation-apps-with-gpt-realtime-translate|gpt-realtime-translate]] 的实时翻译管线做技术对照。

| 设计维度 | OpenAI relay/transceiver | Hermes Managed Agents |
|---------|--------------------------|----------------------|
| 状态极简点 | relay（无状态转发） | Agent process（业务逻辑） |
| 状态集中点 | transceiver（完整会话） | Harness（上下文/记忆/工具） |
| 扩展策略 | 无状态组件水平扩展 | 有状态组件独立迭代 |
| 故障隔离 | relay 崩溃可自愈 | Agent 重启不丢 Harness 上下文 |

核心共同原则：**硬状态只放一处，无状态路径可随意水平扩展。**

这也呼应了 [[concepts/inference-optimization|Inference 优化]] 中的 PD 分离（Prefill/Decode 分离）：分离越干净，各自优化的空间越大。

## Related Concepts
- [[concepts/managed-agents-architecture]] — Session/Harness/Sandbox abstraction separation; parallels with relay (forwarding) vs. transceiver (stateful processing)
- [[concepts/harness-engineering-framework]] — Six-layer architecture with separation of concerns; compare with relay (forwarding) vs. transceiver (stateful processing)
- [[entities/livekit-agents-voice-ai-framework|LiveKit Agents]] — 开源语音 AI 框架，四层流式级联管线，与 OpenAI 端到端模式深度对比

## 新增关联实体
- [[entities/openai发布新一代实时语音模型能够像人说话一样进行推理翻译和转录]]

## 关联实体

**上游依赖**:
- [[entities/livekit-agents-voice-ai-framework]] — 提供基础理论/方法
- [[entities/livekit-agents-voice-ai-framework]] — 提供基础理论/方法

**下游应用**:
- [[entities/livekit-agents-voice-ai-framework]] — 具体应用场景
- [[entities/build-live-translation-apps-with-gpt-realtime-translate]] — 具体应用场景

**平行协作**:
- [[entities/livekit-agents-voice-ai-framework]] — 替代/补充方案
- [[entities/openai发布新一代实时语音模型能够像人说话一样进行推理翻译和转录]] — 替代/补充方案

## 所属 MOC

- [[moc/mlops-training-inference|Mlops Training Inference]]
