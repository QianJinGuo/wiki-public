---
title: "Production Agent Engineering"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [production, agent, engineering, reliability, monitoring, deployment]
sources:
  - raw/articles/anthropic-12-mcp-production-patterns
summary: 生产级 Agent 工程，涵盖从 POC 到生产的完整工程路径，包括可靠性保障、监控告警、部署策略、以及成本控制。
---

# Production Agent Engineering

> 生产级 Agent 工程是将 Agent 从实验环境落地到生产环境所需的完整工程实践。

## 一、POC vs 生产

### 1.1 关键差异

| 维度 | POC | 生产 |
|------|-----|------|
| **可靠性** | 70-80% 可以接受 | 99%+ 必需 |
| **延迟** | 不敏感 | 毫秒级要求 |
| **成本** | 不关注 | 严格控制 |
| **监控** | 简单日志 | 完整可观测性 |
| **安全** | 可忽略 | 必须优先 |
| **扩展** | 单实例 | 多实例/分布式 |

### 1.2 生产门槛

```
┌─────────────────────────────────────────────┐
│           生产级 Agent 必须具备              │
├─────────────────────────────────────────────┤
│  ✓ 错误处理和恢复机制                       │
│  ✓ 完整的监控和告警                         │
│  ✓ 成本控制和优化                           │
│  ✓ 安全边界和权限控制                        │
│  ✓ 审计日志                                 │
│  ✓ 水平扩展能力                             │
│  ✓ 优雅降级                                 │
└─────────────────────────────────────────────┘
```

## 二、可靠性工程

### 2.1 错误分类

| 错误类型 | 描述 | 处理策略 |
|---------|------|---------|
| **可重试** | 临时性故障 | 指数退避重试 |
| **不可重试** | 永久性失败 | 快速失败，报告 |
| **超时** | 操作超时 | 超时检测和取消 |
| **资源耗尽** | 内存/令牌耗尽 | 资源管理，降级 |

### 2.2 重试机制

```python
class RetryMechanism:
    def __init__(self):
        self.max_retries = 3
        self.backoff_strategies = {
            'exponential': exponential_backoff,
            'linear': linear_backoff,
            'fixed': fixed_backoff,
        }
    
    async def execute_with_retry(self, operation, error_handler=None):
        for attempt in range(self.max_retries):
            try:
                result = await operation()
                return result
            except RetryableError as e:
                if attempt == self.max_retries - 1:
                    raise
                
                delay = self.calculate_backoff(attempt, e)
                await asyncio.sleep(delay)
                
                if error_handler:
                    error_handler(e, attempt)
            
            except NonRetryableError as e:
                raise
```

### 2.3 优雅降级

```python
class GracefulDegradation:
    def __init__(self):
        self.fallbacks = {
            'search': self.search_fallback,
            'analysis': self.analysis_fallback,
            'generation': self.generation_fallback,
        }
    
    async def execute(self, capability, primary_fn, **kwargs):
        try:
            return await primary_fn(**kwargs)
        except Exception as e:
            if capability in self.fallbacks:
                logger.warning(f"Primary {capability} failed: {e}")
                return await self.fallbacks[capability](**kwargs)
            else:
                raise
```

## 三、监控与可观测性

### 3.1 指标体系

| 指标类型 | 示例 | 告警阈值 |
|---------|------|---------|
| **延迟** | p50/p95/p99 | p99 > 5s |
| **成功率** | 任务完成率 | < 95% |
| **错误率** | 错误占比 | > 5% |
| **成本** | Token/请求 | 超预算 |
| **资源** | 内存/CPU | > 80% |

### 3.2 日志结构

```python
class StructuredLogger:
    def log_agent_action(self, action):
        return {
            'timestamp': now(),
            'level': 'INFO',
            'event': 'agent_action',
            'agent_id': self.agent_id,
            'action_type': action.type,
            'input_tokens': action.input_tokens,
            'output_tokens': action.output_tokens,
            'duration_ms': action.duration_ms,
            'success': action.success,
            'error': action.error,
            'metadata': action.metadata,
        }
```

### 3.3 追踪

```python
class AgentTracer:
    def __init__(self):
        self.trace_id = None
        self.spans = []
    
    async def trace(self, operation_name, fn, *args, **kwargs):
        with start_span(operation_name) as span:
            span.set_attribute('agent_id', self.agent_id)
            span.set_attribute('operation', operation_name)
            
            try:
                result = await fn(*args, **kwargs)
                span.set_status(Status.OK)
                return result
            except Exception as e:
                span.set_status(Status.ERROR)
                span.record_exception(e)
                raise
```

## 四、成本控制

### 4.1 成本组成

```
总成本 = Σ(输入Token × 单价) + Σ(输出Token × 单价) + API调用费
```

### 4.2 优化策略

| 策略 | 方法 | 节省比例 |
|------|------|---------|
| **上下文压缩** | 减少输入 token | 20-50% |
| **缓存** | 缓存相似请求 | 30-70% |
| **模型选择** | 简单任务用小模型 | 50-80% |
| **批处理** | 合并请求 | 20-40% |

### 4.3 预算管理

```python
class CostBudget:
    def __init__(self, daily_limit):
        self.daily_limit = daily_limit
        self.today_spent = 0
        self.today_date = date.today()
    
    def check_budget(self):
        if date.today() != self.today_date:
            self.today_spent = 0
            self.today_date = date.today()
        
        if self.today_spent >= self.daily_limit:
            raise BudgetExceededError(f"Daily budget {self.daily_limit} exceeded")
    
    def record_usage(self, cost):
        self.today_spent += cost
```

## 五、部署架构

### 5.1 单体 vs 微服务

| 架构 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **单体** | 小规模、简单任务 | 简单、低延迟 | 扩展性差 |
| **微服务** | 大规模、复杂系统 | 灵活扩展 | 复杂度高 |

### 5.2 部署模式

```yaml
# Kubernetes deployment for Agent
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent-service
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: agent
          image: agent:latest
          resources:
            limits:
              memory: "2Gi"
              cpu: "1"
          env:
            - name: MAX_CONCURRENT_TASKS
              value: "10"
            - name: RETRY_MAX_ATTEMPTS
              value: "3"
```

### 5.3 灰度发布

```python
class CanaryDeployment:
    def __init__(self):
        self.traffic_split = 0.1  # 10% 流量到新版本
    
    def route(self, request):
        if self.is_canary_user(request.user_id):
            return self.new_version
        return self.current_version
```

## 相关实体

- [[entities/agent-harness-architecture-deep-dive-aksahy]]
- [[entities/agent-production-harness-engineering]]
- [[entities/agent-harness-engineering-survey-2026]]
- [[entities/claude-code-open-source-model-enterprise-practice]]
- [[entities/aws-bedrock-ops-alert]]

## 六、关联实体

- [[entities/anthropic-12-mcp-production-patterns]] — MCP 生产模式
- [[concepts/agent-security-architecture]] — 安全架构
- [[concepts/agent-evaluation-benchmark-frameworks]] — 评测框架
- [[concepts/inference-optimization]] — 推理优化
- [[concepts/open-source-ai-ecosystem|Open Source AI Ecosystem]] — 开源 AI 生态系统提供生产级 Agent 框架和工具生态
- [[queries/claude-code-scalable-deployment-failure-modes|Claude Code 规模化部署失败模式]] — 生产环境部署常见问题与缓解策略

## 关联实体

**上游依赖**:
- [[entities/agent-harness-architecture-deep-dive-aksahy]] — 提供基础理论/方法
- [[entities/agent-production-harness-engineering]] — 提供基础理论/方法
- [[entities/agent-harness-engineering-survey-2026]] — 提供基础理论/方法

**下游应用**:
- [[entities/claude-code-open-source-model-enterprise-practice]] — 具体应用场景
- [[entities/aws-bedrock-ops-alert]] — 具体应用场景
- [[entities/anthropic-12-mcp-production-patterns]] — 具体应用场景

**平行协作**:
- [[entities/complexity-ratchet-garry-tan]] — 替代/补充方案
- [[entities/fastapi上线实战认证限流零停机一套代码搞定]] — 替代/补充方案
- [[entities/一次构建随处复用python-中的泛型仓库模式]] — 替代/补充方案


→ [[raw/articles/anthropic-12-mcp-production-patterns|原文存档]]

## 新增关联实体
- [[entities/complexity-ratchet-garry-tan]]
- [[entities/fastapi上线实战认证限流零停机一套代码搞定]]
- [[entities/一次构建随处复用python-中的泛型仓库模式]]

## 所属 MOC

- [[moc/layer-5-production-security|Layer 5 Production Security]]
