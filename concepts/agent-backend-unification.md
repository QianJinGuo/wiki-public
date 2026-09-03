---
title: Agent 与后端统一架构
created: 2026-04-30
updated: 2026-08-01
type: concept
tags: [architecture, agent, backend, infrastructure]
related:
  - [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
  - [[entities/how-ai-agent-memory-works|AI Agent 记忆系统架构]]
  - [[entities/openclaw-multi-4|OpenClaw多租户迁移: Phase 2&3部署]]
sources: ['raw/articles/iii-dev-worker-trigger-function']
confidence: medium
---
# Agent 与后端统一架构
## 核心问题
当前大多数 Agentic 架构将系统分为两层：
| 层次 | 包含组件 |
|------|---------|
| **Harness 层** | 编排循环、工具调用（MCP/A2A）、记忆、上下文管理、错误处理 |
| **后端层** | 队列、状态、HTTP 路由、服务端渲染、可观测性 |
这种分离导致复杂度呈 `agents² × services` 扩张：1 个 Agent + 5 个后端服务 = 5 条调试路径；4 个 Agent + 5 个后端服务 = 80 条调试路径。
## 解决思路：Worker / Trigger / Function 原语
| 原语 | 职责 |
|------|------|
| **Function** | 真正执行工作的单元，带稳定标识符（如 `orders::validate`），可存在于任何进程、任何语言 |
| **Trigger** | 触发 function 运行的条件：HTTP、cron、队列订阅、状态变化、流事件 |
| **Worker** | 连接 engine 并注册 functions 和 triggers 的进程 |
## 核心主张
> **当 Agent 也是 Worker 时，Harness 就是后端的一部分。**
- Agent 和 API 服务、ML pipeline、微服务使用同一组 primitives
- **实时发现**：Worker 连接时收到完整 function 目录，断开时自动注销
- **运行时扩展**：无需重新部署或重启
- **统一可观测性**：OpenTelemetry traces 跨 Worker、跨语言、跨 Agent 与后端边界
## Agent 的"特殊组件"重新映射
| 传统 Agent 组件 | 在 Worker/Trigger/Function 体系中的对应 |
|----------------|--------------------------------------|
| 工具 | Function |
| 记忆 | `state::set` / `state::get` |
| 编排 | Trigger + Function 组合 |
| Harness 厚度 | 注册多少 functions、如何组合它们 |
## 与传统 Harness 框架的对比
| 框架 | Harness 厚度 | 代表 |
|------|-------------|------|
| Anthropic | 薄 | 强信任模型，弱编码逻辑 |
| OpenAI | 中 | 指令栈、编排模式、显式交接 |
| CrewAI | 中 | 多路并行，Flows 处理路由 |
| LangGraph | 厚 | 每个决策是节点，每次转移是定义好的边 |
iii.dev 的主张：薄与厚只是注册多少 functions、如何组合它们的问题，不是架构设计空间本身的问题。
## Paradigm Shift 的本质
软件范式转移的关键不在添加功能，而在**坍缩类别**：
- "Everything is a file" 让 Unix 变得可组合
- Components 视为 functions 让 React 心智模型成立
- "添加一个 worker" 成为 iii 里所有问题的答案
当 primitives 设计正确，类别坍缩，复杂度大幅简化——Harness 与后端之间、云与边缘之间、基础设施与应用之间的边界均消解为同一三个原语。
---
## Worker/Trigger/Function 具体实现架构图解
### 架构分层
iii.dev 的 Worker/Trigger/Function 体系并非单一层次，而是由四层逻辑构成：
```
┌─────────────────────────────────────────────────────┐
│                   Engine Layer                      │
│  (消息路由、状态管理、function 发现、身份认证)          │
├─────────────────────────────────────────────────────┤
│                   Worker Layer                       │
│  (连接到 Engine，注册 functions/triggers，维持心跳)    │
├──────────────────┬──────────────────────────────────┤
│  Agent Worker    │    Service Worker                 │
│  (Agent 专用逻辑) │    (传统后端服务)                  │
│  - 记忆管理       │    - HTTP Handler                 │
│  - 编排循环       │    - 数据库操作                   │
│  - 工具调用       │    - 消息队列消费                  │
├──────────────────┴──────────────────────────────────┤
│                  Function Layer                      │
│  (具体业务逻辑，可使用任何语言实现)                    │
│  - orders::validate                                  │
│  - users::authenticate                               │
│  - data::transform                                   │
└─────────────────────────────────────────────────────┘
```
### Function 的注册与发现机制
Function 不是预先注册的静态入口，而是通过 Worker 动态注册到 Engine：
```
Worker 启动时:
  1. Worker 连接到 Engine (gRPC/HTTP)
  2. Worker 发送注册请求: { functions: ["orders::validate", "orders::create"], triggers: [...] }
  3. Engine 更新内部注册表，实时广播到所有连接的 Workers
  4. 其他 Worker 可以立即发现并调用这些 functions

Worker 断开时:
  1. Engine 检测到心跳超时（如 30 秒无响应）
  2. 自动从注册表中移除该 Worker 的所有 functions
  3. 正在执行的 function 不受影响（Engine 记录了调用关系）
  4. 下一次调用该 function 时，Engine 返回 "function unavailable"
```
### Trigger 的类型与配置
Trigger 是"什么条件触发 function 执行"的抽象。以下是 Trigger 的完整类型体系：
| Trigger 类型 | 配置参数 | 典型场景 |
|-------------|---------|---------|
| **HTTP** | Method, Path, Headers | 外部 API 调用、Webhook |
| **Cron** | Cron Expression, Timezone | 定时任务、数据同步 |
| **Queue** | Queue Name, Concurrency, DLQ | 异步消息处理、事件驱动 |
| **State** | Entity, Condition, Transition | 状态机驱动的工作流 |
| **Stream** | Stream Name, Partition, Offset | 实时数据处理、日志分析 |
| **Manual** | Trigger ID, Payload | 人工触发的审批流程 |
Trigger 可以组合——一个 function 可以同时由 HTTP 和 Queue 触发，Engine 自动处理竞合（相同触发条件时，后到的 trigger 被忽略或排队）。
### Worker 的生命周期管理
```
┌─────────┐    connect()     ┌─────────┐
│ Worker  │ ─────────────────→ │ Engine  │
│ (启动)  │                   │         │
└─────────┘                   └─────────┘
     │                              │
     │ 注册 functions               │
     │ & triggers                    │
     ↓                              │
┌─────────┐                   ┌─────────┐
│ Running │ ←── heartbeat ───→ │ Engine  │
│ (运行中) │                   │         │
└─────────┘                   └─────────┘
     │                              │
     │ 检测到异常/主动退出            │
     ↓                              │
┌─────────┐                   ┌─────────┐
│ Worker  │ ─── graceful ───→  │ Engine  │
│ (退出)  │     shutdown       │ 清理注册 │
└─────────┘                   └─────────┘
```
Engine 维护每个 Worker 的健康状态：每 30 秒心跳，超时 3 次则标记为 Dead。Dead Worker 的 functions 被标记为 Unavailable，但正在运行的调用会继续完成（Engine 内部追踪 in-flight 调用）。
### 与现有后端服务的集成方式
iii.dev 的 Worker/Trigger/Function 模型不要求重写既有服务，而是通过**适配层（Adapter）**接入：
**场景 1：REST API 服务接入**
```python
# 不需要修改原有服务代码
class RESTAdapter(Worker):
    def __init__(self, api_base_url: str):
        self.engine = connect_to_engine()
        # 将原有 HTTP 接口映射为 Function
        self.register_function("legacy::orders_get", self.orders_get)
        self.register_function("legacy::orders_create", self.orders_create)
    
    async def orders_get(self, order_id: str):
        # 调用原有 REST API
        return await http.get(f"{self.api_base_url}/orders/{order_id}")
```
**场景 2：消息队列消费者接入**
```python
class MQAdapter(Worker):
    def __init__(self, queue_name: str):
        self.engine = connect_to_engine()
        self.register_trigger("queue::order_created", self.on_order_created)
    
    async def on_order_created(self, message: dict):
        # 消息处理逻辑（原有消费者代码）
        await process_order(message)
```
**场景 3：数据库变更驱动**
```python
class DBAdapter(Worker):
    def __init__(self, dsn: str):
        self.engine = connect_to_engine()
        self.register_trigger("db::orders.status_changed", self.on_status_change)
    
    async def on_status_change(self, before: dict, after: dict):
        # 监听 orders 表 status 字段变化
        if after['status'] == 'shipped':
            await self.call_function("notifications::send_shipping", after)
```
---
## 与现有中间件/队列系统的集成路径
### 集成架构全景
Worker/Trigger/Function 模型需要与现有基础设施共存，而非替代。以下是典型互联网公司的集成路径：
```
┌──────────────────────────────────────────────────────────────┐
│                     Existing Infrastructure                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   Kafka     │  │   Redis     │  │   PostgreSQL        │  │
│  │  (消息队列)   │  │  (缓存/状态)  │  │  (持久化存储)         │  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
└─────────┼────────────────┼────────────────────┼──────────────┘
          │                │                    │
          ↓                ↓                    ↓
┌──────────────────────────────────────────────────────────────┐
│                   iii.dev Engine (New)                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Function Registry │ Trigger Router │ State Manager │    │
│  └─────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────┘
          │                │                    │
          ↓                ↓                    ↓
┌──────────────────────────────────────────────────────────────┐
│              Worker Layer (Gateway/Agent/Service)           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ MQ       │  │ Redis    │  │ DB       │  │ Agent    │    │
│  │ Worker   │  │ Worker   │  │ Worker   │  │ Worker   │    │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │
└──────────────────────────────────────────────────────────────┘
```
### Kafka 集成：Queue Trigger 的实现
Kafka 是互联网公司最常用的消息队列，iii.dev 的 Queue Trigger 本质上是一个 Kafka Consumer Worker：
```python
class KafkaTrigger(Worker):
    """
    将 Kafka Topic 映射为 Trigger：
    - Topic 的每条消息 = 一次 Trigger 触发
    - Consumer Group = Worker Group，跨实例负载均衡
    - 死信队列（DLQ）= Trigger 的失败重试机制
    """
    def __init__(self, brokers: list[str], topic: str, group_id: str):
        self.engine = connect_to_engine()
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=brokers,
            group_id=group_id,
            enable_auto_commit=False  # Engine 手动控制 offset
        )
        # 注册为 Trigger，而非 Function
        self.register_trigger(f"kafka::{topic}", self.on_message)
    
    async def on_message(self, message: dict):
        # message 是 Kafka 消息的 value（已 JSON deserialize）
        # Engine 负责路由到目标 Function
        await self.engine.route_trigger(
            trigger=f"kafka::{self.topic}",
            payload=message,
            metadata={"offset": message.offset, "partition": message.partition}
        )
```
### Redis 集成：State Trigger 与分布式锁
Redis 在 iii.dev 体系中承担两类角色：**状态存储**和**分布式协调**。
```python
# State Trigger：监听 Redis Key 变化
class RedisStateTrigger(Worker):
    def __init__(self, redis_url: str, watch_keys: list[str]):
        self.pubsub = aioredis.pubsub()
        await self.pubsub.subscribe(*[f"__keyspace@0__:{k}" for k in watch_keys])
        self.register_trigger("redis::key_changed", self.on_key_change)
    
    async def on_key_change(self, channel: str, key: str):
        # Redis Keyspace Notification: key 变化事件
        await self.engine.route_trigger(
            trigger="redis::key_changed",
            payload={"key": key, "channel": channel}
        )

# 分布式锁：Function 调用的并发控制
class DistributedLock:
    """
    某些 Function 需要全局唯一执行（如幂等性要求高的操作）。
    通过 Redis 实现分布式锁，确保同一时间只有一个 Worker 在执行。
    """
    async def acquire(self, lock_key: str, ttl: int = 30) -> bool:
        return await redis.set(lock_key, self.worker_id, nx=True, ex=ttl)
    
    async def release(self, lock_key: str):
        # Lua script 保证原子性：只有持有锁的 worker 才能释放
        await redis.eval(RELEASE_LOCK_SCRIPT, lock_key, self.worker_id)
```
### OpenTelemetry 集成：统一可观测性
iii.dev 的核心主张之一是"统一可观测性"——Agent 和后端服务使用同一套 tracing 基础设施。
```python
class OTelIntegration:
    """
    每个 Worker 启动时自动注入 OpenTelemetry tracer。
    Function 调用自动生成 span，跨 Worker 边界自动 trace continuation。
    """
    def __init__(self, service_name: str, otlp_endpoint: str):
        self.tracer = trace.get_tracer(service_name)
        # 设置 OTLP exporter（可对接 Jaeger/Zipkin/CloudTrace）
        exporter = OTLPSpanExporter(endpoint=otlp_endpoint)
        span_processor = BatchSpanProcessor(exporter)
        self.tracer_provider.add_span_processor(span_processor)
    
    def trace_function(self, func_name: str):
        """Decorator: 自动为 Function 添加 tracing"""
        def decorator(func):
            @functools.wraps(func)
            async def wrapper(*args, **kwargs):
                with self.tracer.start_as_current_span(f"function::{func_name}") as span:
                    span.set_attribute("function.name", func_name)
                    span.set_attribute("worker.id", self.worker_id)
                    try:
                        result = await func(*args, **kwargs)
                        span.set_status(Status(OK))
                        return result
                    except Exception as e:
                        span.record_exception(e)
                        span.set_status(Status(ERROR))
                        raise
            return wrapper
        return decorator
```
**Trace 关联示例**：一个完整的 Agent 调用链路：
```
trace_id: abc123
├── span: trigger::http_post_orders (Gateway Worker)
│   └── span: function::orders::validate (Business Logic Worker)
│       └── span: function::inventory::check (Inventory Worker)
│           └── span: redis::get_stock (Redis Worker)
└── span: agent::plan_next_action (Agent Worker)
```
所有 span 共享同一个 trace_id，可以在单个 Dashboard 中看到完整的请求生命周期。
### 迁移策略：渐进式而非 Big Bang
将既有系统迁移到 Worker/Trigger/Function 模型，不需要一次性重写。建议采用**渐进式迁移策略**：
| 阶段 | 行动 | 风险 |
|------|------|------|
| **Phase 1: 双运行** | 新功能用 iii.dev，原有系统保持不变，通过 Adapter 共用 Engine | 低风险，允许团队学习新模型 |
| **Phase 2: 桥接** | 逐步将原有服务暴露为 Functions，通过 Adapter 注册到 Engine | 中风险，需要改造部分调用链路 |
| **Phase 3: 迁移** | 将高价值、高变更频率的 Agent 迁移到 iii.dev Worker 模型 | 中风险，需要重写 Agent 编排逻辑 |
| **Phase 4: 替换** | 确认 iii.dev 可完全替代原有基础设施后，逐步下线遗留系统 | 低风险（已验证），但需要完整并行验证 |
---
## 相关概念
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进|从多智能体编排到AI自主决策：资损防控体系的架构演进]]
- [[entities/how-ai-agent-memory-works|AI Agent 记忆系统架构]]
- [[entities/openclaw-multi-4|OpenClaw多租户迁移: Phase 2&3部署]]
- [[entities/runtime-deploy-apache-doris-mcp-server-quick-suite-ai-analytics|AgentCore Runtime部署Apache Doris MCP Server]]
- [[entities/openclaw-multi-1|OpenClaw多租户迁移: 背景与架构概览]]
- [[entities/agent-principle-architecture-engineering-practice|你不知道的 Agent 原理架构与工程实践]]
- [[entities/openclaw-multi-3|OpenClaw多租户迁移: Phase 1 基础设施部署]]
- [[entities/agent-era-architect-skills-guide|Agent 时代架构师技能指南]]
- [[entities/amazon-bedrock-model-inference-serverless-architecture-case-study|Amazon Bedrock模型推理的Serverless异步架构]]
- [[concepts/agent-memory-system-design|Agent Memory System Design]]
- [[entities/claude-code-architecture|Claude Code 架构解析]]
- [[entities/hermes-agent-memory-system-vs-openclaw|Hermes Agent 记忆系统深度拆解]]
- [[entities/design-patterns-for-ai-agents-2026|Design Patterns for AI Agents 2026]]
[[entities/iii-dev]] — Worker/Trigger/Function 原语框架的实现者  
[[concepts/harness-engineering-framework]] — Harness Engineering 六层框架  
[[entities/thin-harness-fat-skills]] — YC/Garry Tan 的 Fat Skills + Thin Harness 思路  
[[entities/agentcore-harness]] — AWS AgentCore Managed Harness  
[[concepts/openclaw-architecture]] — OpenClaw 薄抽象 + 显式控制流探索
- [[entities/agent-harness-architecture|Agent Harness 架构]]
- [[entities/claude-code-source-architecture|Claude Code 源码拆解：从启动到多 Agent 扩展层]]
- [[queries/agent-memory-system-design|Agent Memory System 设计指南]]
- [[queries/sales-team-agent-knowledge-base-architecture|200人销售团队企业级 Agent 知识库问答系统架构设计]]
- [[concepts/agent-memory-systematic-framework|Agent Memory 系统性框架]]
- [[entities/agent-architecture-harness-new-backend|Agent架构关键变化：Harness正在成为新后端]]

## 关联实体

**上游依赖**:
- [[entities/从多智能体编排到ai自主决策资损防控体系的架构演进]] — 提供基础理论/方法
- [[entities/how-ai-agent-memory-works]] — 提供基础理论/方法
- [[entities/openclaw-multi-4]] — 提供基础理论/方法

**下游应用**:
- [[entities/openclaw-multi-1]] — 具体应用场景
- [[entities/agent-principle-architecture-engineering-practice]] — 具体应用场景
- [[entities/openclaw-multi-3]] — 具体应用场景

**平行协作**:
- [[entities/design-patterns-for-ai-agents-2026]] — 替代/补充方案
- [[entities/iii-dev]] — 替代/补充方案
- [[entities/thin-harness-fat-skills]] — 替代/补充方案

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
