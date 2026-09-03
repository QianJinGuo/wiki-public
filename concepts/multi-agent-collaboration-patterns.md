---
title: "Multi-Agent Collaboration Patterns"
created: 2026-05-21
updated: 2026-08-29
type: concept
tags: [multi-agent, collaboration, orchestration, agent, coordination, swarm]
sources:
  - raw/articles/openclaw-multi-agent-team-practice
  - raw/articles/17-agent-architectures-evolution
summary: 多智能体协作模式，涵盖 Orchestrator-Worker、Hierarchical、Debate、Swarm 等协作范式，以及任务分解、角色定义、通信协议等工程实践。
---

# Multi-Agent Collaboration Patterns

> 多智能体协作是超越单智能体能力边界的关键范式，通过专业分工和协调实现复杂任务。

## 一、为什么需要多智能体

### 1.1 单智能体的局限

| 局限 | 描述 |
|------|------|
| **能力边界** | 单一模型难以精通所有领域 |
| **上下文限制** | 单 agent 的上下文窗口有限 |
| **任务复杂度** | 复杂任务需要多步骤、多领域能力 |
| **并行处理** | 串行执行效率低 |

### 1.2 多智能体的优势

- **专业分工**：每个 agent 专注于特定领域
- **并行执行**：多个 agent 同时工作
- **能力组合**：通过协作实现复杂任务
- **可扩展性**：增加 agent 数量提升能力

## 二、协作架构模式

### 2.1 Orchestrator-Worker 模式

```
        ┌─────────────────────────────────────┐
        │           Orchestrator              │
        │   (任务分解、分配、协调)            │
        └─────────────────────────────────────┘
                       ↓ ↑
    ┌──────────┬──────────┬──────────┐
    ↓          ↓          ↓          ↓
┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐
│Worker1│  │Worker2│  │Worker3│  │Worker4│
│代码   │  │搜索   │  │文件   │  │测试   │
└──────┘  └──────┘  └──────┘  └──────┘
```

**适用场景**：任务可分解、worker 专业化的场景

### 2.2 Hierarchical 模式

多层 agent 形成层级结构：
- **Level 0**：执行具体任务
- **Level 1**：管理一类 worker
- **Level 2**：战略规划和资源分配

### 2.3 Debate 模式

```
    ┌──────────────┐
    │   Pro Agent  │ ← 论证 A 观点
    └──────────────┘
           ↕
    ┌──────────────┐
    │   Mediator   │ ← 综合分析
    └──────────────┘
           ↕
    ┌──────────────┐
    │   Con Agent  │ ← 提出反驳
    └──────────────┘
```

**适用场景**：需要多角度分析、平衡决策

### 2.4 Swarm 模式

```
    Agent A ←→ Agent B ←→ Agent C
      ↑                       ↓
      └───────────────────────┘
           (动态协作网络)
```

**特点**：
- 去中心化
- 动态角色分配
- 涌现行为

## 三、任务分解策略

### 3.1 如何分解任务

| 策略 | 描述 | 示例 |
|------|------|------|
| **功能分解** | 按能力领域分解 | 搜索、编码、测试 |
| **阶段分解** | 按工作流程分解 | 计划、执行、验证 |
| **层次分解** | 按抽象级别分解 | 高层目标 → 具体步骤 |

### 3.2 粒度控制

```
过粗：难以并行、责任不清
     ↓
合适：独立可执行、清晰边界
     ↓
过细：协调成本高、碎片化
```

## 四、通信协议

### 4.1 通信模式

| 模式 | 描述 | 适用 |
|------|------|------|
| **同步** | 等待响应 | 强依赖任务 |
| **异步** | 消息队列 | 弱依赖、可并行 |
| **广播** | 一对多 | 状态通知 |

### 4.2 消息格式

```python
class AgentMessage:
    sender: str      # 发送者 ID
    receiver: str    # 接收者 ID（可为空=广播）
    type: str        # REQUEST/RESPONSE/NOTIFICATION
    content: dict    # 消息内容
    context: dict   # 上下文信息
    timestamp: float
    
# 示例
{
    "sender": "orchestrator",
    "type": "REQUEST",
    "content": {
        "task": "search_code",
        "query": "login function"
    }
}
```

### 4.3 状态共享

```python
class SharedState:
    def __init__(self):
        self.blackboard = {}  # 共享黑板
        self.locks = {}       # 锁机制
    
    def write(self, key, value, agent_id):
        # 检查权限、记录版本
        self.blackboard[key] = value
    
    def read(self, key):
        return self.blackboard.get(key)
```

## 五、角色定义

### 5.1 常见角色

| 角色 | 职责 |
|------|------|
| **Orchestrator** | 任务分解、分配、协调 |
| **Specialist** | 特定领域专家 |
| **Critic** | 审查、提出问题 |
| **Synthesizer** | 综合结果、生成最终输出 |

### 5.2 角色动态分配

```python
class DynamicRoleAssigner:
    def assign(self, task, available_agents):
        # 1. 分析任务需求
        required_skills = analyze_task(task)
        
        # 2. 评估 agent 能力
        agent_capabilities = [a.skills for a in available_agents]
        
        # 3. 匹配分配
        assignments = match(required_skills, agent_capabilities)
        
        # 4. 处理冲突
        return resolve_conflicts(assignments)
```

## 六、错误处理与恢复

### 6.1 失败类型

| 类型 | 原因 | 处理 |
|------|------|------|
| **Agent 失败** | agent 无响应 | 重试、替换 |
| **通信失败** | 消息丢失 | 重发、确认机制 |
| **任务失败** | 子任务无法完成 | 重新分解、跳过 |
| **死锁** | 循环依赖 | 超时检测、干预 |

### 6.2 恢复策略

```python
class MultiAgentRecovery:
    def handle_failure(self, failed_agent, task):
        # 1. 识别失败
        if is_agent_failure(failed_agent):
            return reassign_task(task, another_agent)
        
        # 2. 简化任务
        simplified = simplify_task(task)
        return retry(simplified)
        
        # 3. 上报
        escalate_to_human(task)
```

## 七、最佳实践

1. **明确角色边界**：每个 agent 有清晰职责，避免重复工作
2. **最小化通信**：设计高效协议，减少消息开销
3. **容错设计**：假设任何 agent 都可能失败
4. **可观测性**：记录协作过程，便于调试
5. **动态调整**：根据任务特点调整 agent 组合

## 相关实体

- [[entities/agent-development-crawl-walk-run-crewai-iterative]]
- [[entities/agent-orchestration]]
- [[entities/agent-room-emergent-collaboration-multi-agent-decision]]
- "AWS Bedrock 多智能体协作指南"
- [[entities/agentic-design-system-from-chatbot-to-orchestration]]

## 八、关联实体

- [[entities/openclaw-multi-agent-team-practice]] — OpenClaw 多智能体实践
- [[entities/17-agent-architectures-evolution]] — 17 种 Agent 架构演进
- [[concepts/multi-agent-systems]] — 多智能体系统
- [[concepts/agentic-workflow-patterns]] — 工作流模式

## 关联实体

**上游依赖**:
- [[entities/agent-development-crawl-walk-run-crewai-iterative]] — 提供基础理论/方法
- [[entities/agent-orchestration]] — 提供基础理论/方法

**下游应用**:
- [[entities/agent-room-emergent-collaboration-multi-agent-decision]] — 具体应用场景
- "AWS Bedrock 多智能体协作指南" — 具体应用场景

**平行协作**:
- [[entities/agentic-design-system-from-chatbot-to-orchestration]] — 替代/补充方案
- [[entities/openclaw-multi-agent-team-practice]] — 替代/补充方案
- [[entities/17-agent-architectures-evolution]] — 替代/补充方案


→ [[raw/articles/openclaw-multi-agent-team-practice|原文存档]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
