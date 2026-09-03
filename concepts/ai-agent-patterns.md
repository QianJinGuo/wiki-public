---
title: "AI Agent Patterns"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [agent, ai-agent, patterns, architecture, autonomous, goal-driven]
sources:
  - raw/articles/你不知道的-agent原理架构与工程实践
  - raw/articles/karpathy-vibe-coding-agentic-engineering-v3
summary: AI Agent 核心模式，涵盖从 Task-Driven 到 Goal-Driven 的演进、Agent 的核心组件（规划、记忆、工具使用）、以及自主 Agent 的设计模式。
---

# AI Agent Patterns

> AI Agent 是能够自主决策、执行任务、与环境交互的智能系统。本文梳理 Agent 的核心架构模式和工程实践。

## 一、Agent 的定义与本质

### 1.1 什么是 AI Agent

AI Agent 是：
- **自主性**：能够自主决策，无需人工干预
- **目标导向**：围绕目标而非固定流程工作
- **环境交互**：能够感知环境并采取行动
- **持续性**：能够在较长时间跨度内工作
- **学习适应**：能够从经验中学习和改进

### 1.2 Task-Driven vs Goal-Driven

| 维度 | Task-Driven | Goal-Driven |
|------|-------------|-------------|
| **工作方式** | 执行预设任务 | 追求目标达成 |
| **灵活性** | 低 | 高 |
| **适应性** | 固定输入输出 | 动态调整策略 |
| **复杂性** | 适合简单任务 | 适合复杂任务 |

## 二、Agent 的核心组件

### 2.1 经典架构

```
┌─────────────────────────────────────────────────────┐
│                    Agent                            │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │
│  │   Planning  │  │   Memory    │  │   Tools   │  │
│  │   (规划)    │  │   (记忆)    │  │   (工具)  │  │
│  └─────────────┘  └─────────────┘  └───────────┘  │
├─────────────────────────────────────────────────────┤
│                   Perception                        │
│                   (感知层)                          │
└─────────────────────────────────────────────────────┘
```

### 2.2 规划模块（Planning）

| 能力 | 描述 | 技术 |
|------|------|------|
| **任务分解** | 将复杂目标拆分为子任务 | CoT, ToT |
| **计划生成** | 创建执行计划 | ReAct, Plan-Execute |
| **自我反思** | 评估和改进计划 | Reflection |
| **适应性** | 根据结果调整计划 | 反馈循环 |

### 2.3 记忆模块（Memory）

| 类型 | 时效性 | 容量 | 用途 |
|------|--------|------|------|
| **感官记忆** | 极短 | 高 | 原始感知输入 |
| **工作记忆** | 短 | 低 | 当前任务上下文 |
| **情景记忆** | 中 | 中 | 会话历史 |
| **长期记忆** | 长 | 大 | 知识和经验 |

### 2.4 工具模块（Tools）

Agent 使用工具的能力：
- **API 调用**：与外部系统交互
- **代码执行**：计算和处理数据
- **搜索能力**：获取外部知识
- **文件操作**：读写文件

## 三、Agent 架构模式

### 3.1 ReAct 模式

Reasoning + Acting：

```python
class ReActAgent:
    def run(self, task):
        observation = None
        thought = None
        for step in range(max_steps):
            # 1. 推理
            thought = self.reason(observation, task)
            
            # 2. 决定行动
            action = self.plan(thought)
            
            # 3. 执行
            result = self.execute(action)
            
            # 4. 观察
            observation = self.observe(result)
            
            # 5. 检查终止
            if self.is_complete(observation):
                return observation
```

### 3.2 Plan-Execute 模式

```python
class PlanExecuteAgent:
    def run(self, goal):
        # 1. 规划阶段
        plan = self.planner.create_plan(goal)
        
        # 2. 执行阶段
        for step in plan.steps:
            result = self.executor.execute(step)
            
            # 3. 验证结果
            if not self.validator.validate(result, step.expected):
                # 4. 重新规划
                plan = self.planner.replan(goal, result)
        
        return result
```

### 3.3 Reflection 模式

```python
class ReflectionAgent:
    def run(self, task):
        # 1. 生成
        output = self.generate(task)
        
        # 2. 反思
        critique = self.reflect(output, task)
        
        # 3. 改进
        improved = self.improve(output, critique)
        
        # 4. 迭代直到满意
        for _ in range(max_iterations):
            if self.is_satisfactory(improved):
                return improved
            critique = self.reflect(improved, task)
            improved = self.improve(improved, critique)
```

### 3.4 Hierarchical Agent

多层 Agent 形成层级：

```
Level N: 战略规划
    ↓
Level N-1: 任务管理
    ↓
...
    ↓
Level 0: 具体执行
```

## 四、自主性设计

### 4.1 自主等级

| 等级 | 描述 | 人类参与 |
|------|------|---------|
| **L1** | 辅助决策 | 全程参与 |
| **L2** | 建议+批准 | 关键节点确认 |
| **L3** | 自动执行+报告 | 事后审查 |
| **L4** | 自主执行+干预 | 必要时干预 |
| **L5** | 完全自主 | 无需参与 |

### 4.2 边界定义

```python
class AutonomyBoundary:
    def __init__(self):
        self.ops_rules = {
            # (操作类型, 阈值) -> 需要的审批级别
            ('write', 'any'): HUMAN_APPROVAL,
            ('delete', 'critical'): MULTI_APPROVAL,
            ('read', 'public'): AUTO,
        }
    
    def requires_approval(self, operation):
        op_type, severity = operation
        return self.ops_rules.get((op_type, severity), DEFAULT_RULE)
```

### 4.3 监控与干预

```python
class MonitoringSystem:
    def monitor(self, agent_action):
        # 1. 记录行动
        log(agent_action)
        
        # 2. 风险评估
        risk = self.assess_risk(agent_action)
        
        # 3. 高风险行动上报
        if risk > HIGH_THRESHOLD:
            notify_human(agent_action)
        
        # 4. 异常检测
        if self.is_anomalous(agent_action):
            self.trigger_review()
```

## 五、Agent 可靠性

### 5.1 常见失败模式

| 失败模式 | 描述 | 对策 |
|---------|------|------|
| **幻觉** | 生成错误信息 | 验证机制 |
| **循环** | 重复相同行动 | 状态跟踪 |
| **偏离** | 偏离原始目标 | 目标检查 |
| **崩溃** | 陷入死循环 | 超时/次数限制 |

### 5.2 错误恢复

```python
class ErrorRecovery:
    def handle_error(self, error, context):
        if is_retryable(error):
            # 1. 重试有限次数
            return retry_with_backoff(error)
        elif is_decomposable(error):
            # 2. 分解问题
            return self.decompose_and_retry(error)
        elif requires_human(error):
            # 3. 升级
            return escalate(error)
        else:
            # 4. 优雅失败
            return graceful_failure(error)
```

## 六、最佳实践

1. **明确边界**：定义 Agent 能做什么、不能做什么
2. **最小干预**：设计自主等级，避免过度控制
3. **可观测性**：全面日志记录，便于追踪和调试
4. **容错设计**：假设任何操作都可能失败
5. **持续验证**：验证中间结果，而非只看最终结果

## 相关实体

- [[entities/agent-harness-architecture-deep-dive-aksahy]]
- [[entities/guide-ai-agents-models-apps-harnesses-mollick]]
- [[entities/co-existence-paradigm-shift-agentic-ai-mollick-2026]]
- [[entities/agent-development-crawl-walk-run-crewai-iterative]]
- [[entities/17-agent-architectures-evolution]]

## 七、关联实体

- [[entities/你不知道的-agent原理架构与工程实践]] — Agent 原理架构
- [[entities/karpathy-vibe-coding-agentic-engineering-v3]] — Agentic Engineering
- [[concepts/multi-agent-collaboration-patterns]] — 多智能体协作
- [[concepts/agentic-workflow-patterns]] — 工作流模式
- [[concepts/coding-agent-architecture|Coding Agent Architecture]] — 编程 Agent 是自主 Agent 的重要子类

## 关联实体

**上游依赖**:
- [[entities/agent-harness-architecture-deep-dive-aksahy]] — 提供基础理论/方法
- [[entities/guide-ai-agents-models-apps-harnesses-mollick]] — 提供基础理论/方法
- [[entities/co-existence-paradigm-shift-agentic-ai-mollick-2026]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-development-crawl-walk-run-crewai-iterative]] — 具体应用场景
- [[entities/17-agent-architectures-evolution]] — 具体应用场景
- [[entities/你不知道的-agent原理架构与工程实践]] — 具体应用场景

**平行协作**:
- [[entities/karpathy-vibe-coding-agentic-engineering-v3]] — 替代/补充方案
- [[entities/pi-main-agent-engineering-17-dimensions]] — 替代/补充方案
- [[entities/taobao-product-domain-agent-architecture]] — 替代/补充方案


→ [[raw/articles/你不知道的-agent原理架构与工程实践|原文存档]]

## 新增关联实体
- [[entities/pi-main-agent-engineering-17-dimensions]]
- [[entities/taobao-product-domain-agent-architecture]]
- [[entities/token级精准控制生成长度3b模型击败gpt-54claude]]

## 所属 MOC

- [[moc/layer-3-agent-engineering|Layer 3 Agent Engineering]]
