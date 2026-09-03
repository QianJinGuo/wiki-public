---
title: "Skill Framework & Writing Patterns"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [skill, agent, framework, writing, patterns, governance]
sources:
  - raw/articles/anthropic-12-mcp-production-patterns
  - raw/articles/从-anthropic-到-googleagent-skills-正在进入设计模式阶段
summary: Skill 框架与编写模式，涵盖 Skill 的定义、结构、生命周期，以及与 MCP 的关系，包括 14 种 Skill 编写模式和治理最佳实践。
---

# Skill Framework & Writing Patterns

> Skill 是 Agent 执行特定任务的能力单元，与 MCP 工具形成互补，共同构成 Agent 的完整能力体系。

## 一、Skill 的定义

### 1.1 Skill vs Tools

| 维度 | MCP Tool | Skill |
|------|---------|-------|
| **粒度** | 原子操作 | 完整任务流程 |
| **知识** | 执行能力 | 执行能力 + 领域知识 |
| **使用** | 被调用 | 指导 Agent 如何使用工具 |
| **内容** | API 封装 | Playbook、操作手册 |

### 1.2 Skill 的价值

> "MCP Server 可以告诉 Agent 有哪些工具，但不告诉 Agent 在复杂业务里应该怎样安全、有效地组合这些工具。"

Skill 填补了这个空白：
- **Playbook**：什么时候用什么工具
- **操作手册**：如何正确执行复杂任务
- **最佳实践**：避免常见错误
- **边界规则**：安全使用的约束条件

## 二、Skill 的结构

### 2.1 基本组成

```yaml
skill:
  name: "线上错误排查"
  trigger: 
    - "线上出问题了"
    - "服务不可用"
    - "error in production"
  
  steps:
    - name: "查看错误概要"
      tools: ["sentry.search"]
      args:
        query: "${user_input}"
      
    - name: "分析影响范围"
      tools: ["sentry.fetch_trace"]
      args:
        issue_id: "${steps[0].result.issue_id}"
      
    - name: "检查相关部署"
      tools: ["deploy.check"]
      args:
        time_range: "recent"
      
    - name: "生成诊断报告"
      tools: ["sentry.report"]
      output: "markdown"
  
  validation:
    - "是否找到根因"
    - "是否有解决方案"
```

### 2.2 Skill 触发条件

```python
class SkillTrigger:
    def __init__(self):
        self.patterns = [
            # 精确匹配
            ("exact", "帮我排查线上错误"),
            
            # 模糊匹配
            ("fuzzy", "服务挂了"),
            
            # 正则匹配
            ("regex", "(error|exception|failed).*production"),
            
            # 语义匹配
            ("semantic", "生产环境出现异常"),
        ]
```

## 三、Skill 与 MCP 的关系

### 3.1 Server-Distributed Skills

**趋势**：未来的 MCP Server 不只分发能力，还会分发使用能力的方法。

```
MCP Server 连接
       ↓
    获得工具 + 获得 Skill/Playbook
       ↓
    Agent 知道：
    - 有哪些工具可用
    - 何时使用、如何组合
```

### 3.2 Skill 增强工具价值

| 工具只有 | 工具 + Skill |
|---------|-------------|
| API 能力 | API 能力 |
| - | 何时使用 |
| - | 如何组合 |
| - | 常见错误避免 |
| - | 安全边界 |

### 3.3 Plugin Bundle 模式

```yaml
plugin:
  name: "Sentry Monitoring"
  skills:
    - sentry_error_diagnosis
    - sentry_performance_analysis
    - sentry_alert_management
  mcp_servers:
    - sentry-mcp
  hooks:
    - on_error_auto_diagnosis
```

## 四、Skill 编写模式

### 4.1 基础模式

| 模式 | 描述 | 适用场景 |
|------|------|---------|
| **Sequential** | 步骤顺序执行 | 固定流程任务 |
| **Conditional** | 条件分支执行 | 多场景支持 |
| **Parallel** | 并行执行 | 独立子任务 |
| **Loop** | 循环执行 | 批量处理 |

### 4.2 高级模式

| 模式 | 描述 | 适用场景 |
|------|------|---------|
| **Fallback** | 主方案失败时切换 | 容错设计 |
| **Rollback** | 失败时回滚 | 事务性任务 |
| **Escalation** | 升级处理 | Agent 无法完成 |
| **Merge** | 合并多分支结果 | 综合分析 |

### 4.3 治理模式

```yaml
skill:
  name: "删除生产数据"
  requires:
    - human_approval
    - backup_verified
    - audit_log_enabled
    
  constraints:
    - "不能删除超过1000条"
    - "必须在维护窗口执行"
    - "需要二次确认"
```

## 五、Skill 生命周期

### 5.1 阶段

```
设计 → 编写 → 测试 → 部署 → 监控 → 迭代
```

### 5.2 版本管理

```yaml
skill_versions:
  "sentry_diagnosis":
    v1.0: "初始版本"
    v1.1: "增加 trace 分析"
    v2.0: "重构步骤，加入 LLM 辅助"
    current: v2.0
```

### 5.3 质量保证

| 检查项 | 方法 |
|--------|------|
| **触发准确性** | 测试各种输入是否正确触发 |
| **步骤正确性** | 验证每步执行结果 |
| **边界处理** | 测试异常输入 |
| **性能** | 测量执行时间和 token 消耗 |

## 六、Skill 评估与迭代

### 6.1 评估指标

| 指标 | 描述 | 测量方法 |
|------|------|---------|
| **触发准确率** | 正确触发的比例 | 测试集 |
| **执行成功率** | 成功完成的比例 | 生产日志 |
| **用户满意度** | 用户反馈 | 评分/反馈 |
| **Token 效率** | 平均 token 消耗 | 监控 |

### 6.2 持续改进

```python
class SkillImprover:
    def analyze_failures(self, skill_name):
        failures = self.get_failures(skill_name)
        
        # 1. 识别失败模式
        patterns = self.cluster_failures(failures)
        
        # 2. 提出改进建议
        for pattern in patterns:
            suggestion = self.suggest_fix(pattern)
            yield suggestion
    
    def auto_optimize(self, skill_name):
        # 基于反馈自动优化
        pass
```

## 七、最佳实践

1. **粒度适当**：Skill 应该覆盖完整任务，但不要太泛
2. **明确触发**：触发条件要清晰、可测试
3. **步骤可验证**：每步执行结果要可验证
4. **错误处理**：要有容错和回退机制
5. **文档完整**：包含使用说明和边界条件
6. **版本控制**：有清晰的版本管理和升级策略

## 相关实体

- [[entities/agent-skill-writing]]
- [[entities/agent-skill-writing-advanced]]
- [[entities/agent-skill-writing-evaluation]]
- [[entities/tencent-skill-writing-complete-playbook-jackjchou]]
- [[entities/agent-reliability-engineering-skillify-continuous-improvement]]

## 八、关联实体

- [[entities/anthropic-12-mcp-production-patterns]] — MCP 与 Skill 结合
- Qoder Skills（已删除） — Qoder Skills 指南
- [[concepts/model-context-protocol-mcp]] — MCP 协议
- [[concepts/agent-evaluation-benchmark-frameworks]] — Skill 评估
- [[comparisons/agent-skill-evaluation-methods|Agent/Skill 评估方法对比]] — 三 Skill 统一评估语言参考表

## 关联实体

**上游依赖**:
- [[entities/agent-skill-writing]] — 提供基础理论/方法
- [[entities/agent-skill-writing-advanced]] — 提供基础理论/方法
- [[entities/agent-skill-writing-evaluation]] — 提供基础理论/方法

**下游应用**:
- [[entities/tencent-skill-writing-complete-playbook-jackjchou]] — 具体应用场景
- [[entities/agent-reliability-engineering-skillify-continuous-improvement]] — 具体应用场景
- [[entities/anthropic-12-mcp-production-patterns]] — 具体应用场景

**平行协作**:
- [[entities/baidu-netdisk-three-layer-agent-architecture]] — 替代/补充方案
- [[entities/pi-agent-framework-event-bus-design]] — 替代/补充方案
- [[entities/skill-development-guide-linyi]] — 替代/补充方案


→ [[raw/articles/anthropic-12-mcp-production-patterns|原文存档]]

## 新增关联实体
- [[entities/baidu-netdisk-three-layer-agent-architecture]]
- [[entities/pi-agent-framework-event-bus-design]]
- [[entities/skill-development-guide-linyi]]
- [[entities/weread-official-skill-huashu-critical-gap]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
