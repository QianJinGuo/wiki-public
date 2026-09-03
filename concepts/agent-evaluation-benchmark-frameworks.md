---
title: "Agent Evaluation & Benchmark Frameworks"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [evaluation, benchmark, testing, agent, pass@k, quality, metrics]
sources:
  - raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation
summary: Agent 评测框架设计，涵盖 pass@k vs pass^k 双指标体系、六种评分器设计、CI/CD 集成、缓存机制、以及从传统测试金字塔到 Agent 评测的范式转变。
---

# Agent Evaluation & Benchmark Frameworks

> "Teams without evals get bogged down in reactive loops — fixing one failure, creating another." — Anthropic

Agent 评测是确保 Agent 系统质量的核心工程实践。与传统软件测试不同，Agent 的非确定性、传播效应和上游波动使得评测设计具有独特的挑战。

## 相关查询

- [[queries/wiki-evolver-skill-evaluation-methods|Wiki Evolver Skill 评测方法]] — 4 任务 benchmark suite、scorecard 评分与 trajectory checklist

## 一、传统测试金字塔的局限性

### 1.1 为什么传统测试不够用

传统测试金字塔（单元测试 → 集成测试 → E2E 测试）覆盖不了 Agent 的核心质量问题：

| 问题 | 描述 | 影响 |
|------|------|------|
| **非确定性** | 同一 prompt 跑两次结果可能不同 | 无法用断言判定 |
| **传播效应** | 改一个词可能导致整个行为链路变化 | 不可预测的连锁反应 |
| **上游波动** | 模型本身升级，Agent 表现可能出现波动 | 持续的质量漂移 |

### 1.2 Agent 评测的特殊性

Agent 评测不是简单的"正确/错误"二分法，而是：
- **能力评估**：Agent 能否完成特定任务
- **可靠性评估**：Agent 完成任务的一致性如何
- **效率评估**：消耗多少 token、调用多少工具
- **安全性评估**：是否产生有害输出

## 二、关键指标：pass@k vs pass^k

### 2.1 双指标体系

| 指标 | 含义 | 衡量什么 |
|------|------|---------|
| **pass@k** | k 次里至少有一次通过的概率 | 能力上限 |
| **pass^k** | k 次全部通过的概率 | 可靠性 |

### 2.2 为什么两者都要看

```
pass@k 高 + pass^k 低 = Agent 有一定概率做对，但不稳定
pass@k 低 + pass^k 高 = Agent 稳定做错，需要改进能力
pass@k 高 + pass^k 高 = 理想的可靠能力
```

> 当 pass@k 很高但 pass^k 很低时，说明 Agent 有一定概率做对，但不稳定。单纯提高 pass@k 阈值没有意义，问题根源在于 Agent 的随机性，而非能力上限。

### 2.3 计算方法

pass@k 计算用对数空间算术，避免组合数在大 n 时的溢出精度问题：

```python
# pass@k 公式
P(pass@k) = 1 - C(n-k, k) / C(n, k)

# 实际实现用对数空间避免数值溢出
log_p = log_comb(n, k) - log_comb(n-k, k)
pass_k_prob = 1 - exp(log_p)
```

## 三、评分器体系

### 3.1 六种评分器类型

| 类型 | 适用场景 | 特点 |
|------|---------|------|
| `exact_match` | 有标准答案的知识问答 | 确定性，可复现 |
| `contains` | 检查关键词是否出现 | 简单，宽松 |
| `regex` | 有格式要求的任务 | 精确匹配 |
| `command` | 能跑测试脚本的场景 | 自动化验证 |
| `llm` | 开放式问答主观评分 | LLM-as-judge |
| `constraint` | 合规检查 | must_match/must_not_match，字数范围 |

### 3.2 评分器组合策略

多评分器可通过 `weight` 控制权重，最终通过条件是 **AND**——两个评分器都得通过才算 trial 通过。

```yaml
graders:
  - type: contains
    weight: 0.3
    keywords: ["error", "failed"]
    must_contain: false  # 检查不能包含
  
  - type: llm
    weight: 0.7
    judge_prompt: "评估回复是否专业、helpful"
```

### 3.3 评分器的分层设计

六种评分器形成从客观到主观的连续光谱：

```
确定性端                                          不确定性端
   ↓                                                 ↓
exact_match → regex → contains → command → constraint → llm
   |                                                   |
可复现、无歧义                              开放式任务，但引入评分不稳定性
```

## 四、CI/CD 集成

### 4.1 分层测试策略

```bash
# PR 合并前只跑核心用例
agent-eval run -c eval.yaml --tags core --fail-under 0.9

# 日常回归跑全量
agent-eval run -c eval.yaml --fail-under 0.8

# 安全审查单独跑
agent-eval run -c eval.yaml --tags safety --fail-under 1.0
```

`--fail-under` 通过率低于阈值返回退出码 1，阻断流水线。

### 4.2 评测结果消费

`results/summary.json` 可被后续流水线步骤消费：
- 趋势分析 Dashboard
- 回归检测
- 性能预警

## 五、缓存机制

### 5.1 缓存设计

缓存 key = SHA256(Agent 类型 + Agent 配置 + 任务输入)

**缓存命中时**：
- 不调用 Agent API
- metadata 加 `cache_hit: true` 标记
- 评分规则变更后重跑评测，不会重复调用 Agent API

### 5.2 缓存的战略价值

Agent 评测的成本（API 调用 + 计算资源）远高于传统单元测试。缓存机制使得"改评分规则 → 重跑评测"的迭代成本从 **O(n·API_calls)** 降低到 **O(n)**。

## 六、A/B 对比

```bash
agent-eval compare <run_id_a> <run_id_b>  # 支持 ID 前缀匹配
```

输出标注每个任务是回归了还是改进了。

## 七、扩展接口

### 7.1 Agent 接口

```go
type Agent interface {
    Execute(ctx context.Context, input model.TaskInput) (*model.AgentOutput, error)
    Close() error
}

// 在 init() 里注册
Register("internal_rpc", func(config) (Agent, error) { ... })
```

### 7.2 Grader 接口

```go
type Grader interface {
    Grade(ctx context.Context, input GradeInput) (*model.GradeResult, error)
    Type() string
}
```

**极简设计**（各两个方法）降低了插件开发的门槛。`init()` 注册机制允许在运行时动态加载插件。

## 八、深度分析

### 8.1 评测先行原则

在没有量化指标的情况下优化 Agent，就像在没有测试的情况下重构——你不知道改进了什么，也可能破坏了原本正常的功能。

**建议**：在引入 Agent 的第一天就搭建评测框架。

### 8.2 评分器选择策略

| 场景 | 推荐评分器 |
|------|-----------|
| 上线前的自动化测试 | exact_match/regex/command——可复现 |
| 线上监控 | 加 llm 评分器捕捉开放式任务质量问题 |
| 敏感场景（安全/合规） | constraint 评分器必须作为守门员，fail-under=1.0 |

### 8.3 版本控制配合

当 Agent 版本升级时，缓存失效是正常的。但如果评测结果出现大幅波动，要区分是模型升级导致的真实改进/退化，还是缓存失效导致的评测偏差。

## 九、关联实体

- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation-framework]] — AgentEval 框架
- [[entities/harness-generator-evaluator-anthropic]] — Anthropic Generator-Evaluator Harness
- [[entities/lbs-intentbench]] — LBS-IntentBench 评测基准
- [[entities/ai-skill-metrics-system]] — AI Skill 评测指标体系
- [[concepts/inference-optimization]] — 推理优化与评测关系
- [[queries/agent-evaluation-harness-best-practices|Agent Evaluation Harness 最佳实践]] — 评测框架设计与常见偏见

## 关联实体

**上游依赖**:
- [[entities/agent-eval-wallezhang-yaml-driven-agent-evaluation-framework]] — 提供基础理论/方法

**下游应用**:
- [[entities/harness-generator-evaluator-anthropic]] — 具体应用场景
- [[entities/lbs-intentbench]] — 具体应用场景

**平行协作**:
- [[entities/ai-skill-metrics-system]] — 替代/补充方案
- [[entities/browser-request-recording-ai-code-generation-e2e-api-testing]] — 替代/补充方案


→ [[raw/articles/agent-eval-wallezhang-yaml-driven-agent-evaluation|原文存档]]

## 新增关联实体
- [[entities/browser-request-recording-ai-code-generation-e2e-api-testing]]

## 所属 MOC

- [[moc/layer-4-ecosystem|Layer 4 Ecosystem]]
