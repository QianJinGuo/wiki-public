---

title: "Agent Harness 可观测性：生产级 AI 项目必须补上的一课"
created: 2026-05-25
updated: 2026-07-31
type: entity
tags: [agent, harness, observability, tracing, evaluation, monitoring]
source: [[raw/articles/agent-harness-observability-production]]
confidence: 0.8
review_value: 5
sources:
  - raw/articles/agent-harness-observability-production
---

# Agent Harness 可观测性：生产级 AI 项目必须补上的一课

## 深度分析

本文来自"叶小钗"，分享开发 Mini-Openclaw Agent 时的可观测性实践。 ^[raw/articles/agent-harness-observability-production.md]

**核心问题**：Agent 执行路径不确定（概率输出），传统三件套（指标/日志/链路追踪）不够用。 ^[raw/articles/agent-harness-observability-production.md]

**AI Min > AI Max**：可观测性只在"能不用 AI 就不用 AI"模式下可行，关键在于知道错在哪、为什么错、怎么改。 ^[raw/articles/agent-harness-observability-production.md]

**Agent 可观测性八大组件**： ^[raw/articles/agent-harness-observability-production.md]
1. **原始数据记录**：model_call/result、tool_call/result、anomaly、evaluation ^[raw/articles/agent-harness-observability-production.md]
2. **指标设计**：工具错误率、token消耗、调用耗时、压缩频率 ^[raw/articles/agent-harness-observability-production.md]
3. **Trace调用树**：model_call_id + tool_call_id + delegation_id 关联父子关系 ^[raw/articles/agent-harness-observability-production.md]
4. **决策归因**：system prompt 规范让模型在 reasoning 输出决策块（目标/候选/选择/原因/结果） ^[raw/articles/agent-harness-observability-production.md]
5. **任务状态机**：pending→planning→running→waiting_child→succeeded/failed/cancelled ^[raw/articles/agent-harness-observability-production.md]
6. **异常检测**：重复失败、空响应循环、迭代超限、压缩频繁 ^[raw/articles/agent-harness-observability-production.md]
7. **评估**：用户反馈 + 启发式评估 + LLM-as-judge ^[raw/articles/agent-harness-observability-production.md]
8. **回放对比**：同 case 新配置重跑，对比两棵 Trace 调用树 ^[raw/articles/agent-harness-observability-production.md]

**闭环**：发现问题 → 定位轨迹 → 修改配置 → 回放对比 ^[raw/articles/agent-harness-observability-production.md]

## 实践启示

1. **Trace > 日志**：时间线日志需要靠猜，Trace 调用树靠 model_call_id/tool_call_id/delegation_id 字段直接关联 ^[raw/articles/agent-harness-observability-production.md]
2. **决策归因的价值**：让模型自己输出选择原因，等同于大模型自我反思，正确率也会变高 ^[raw/articles/agent-harness-observability-production.md]
3. **异常检测核心**：不是工具调用失败本身，而是连续失败不能收敛还在不停执行 ^[raw/articles/agent-harness-observability-production.md]
4. **评估是优化闭环**：否则改完 prompt 只知道"好像顺了"，不知道错误率有没有降 ^[raw/articles/agent-harness-observability-production.md]
5. **回放不是严格复现**：模型有随机性，回放只能做相似条件下的验证，用于判断优化方向 ^[raw/articles/agent-harness-observability-production.md]

## 相关实体
- [[entities/harness-engineered-business-agent-evaluation-aliyun-boyu]]
- [[entities/code-as-agent-harness-survey]]
- [[entities/cong-30-fen-zhong-shou-gu-agent-dao-harness-cheng-wei-xin-hou-duan]]
- [[entities/从-30-分钟手搓-agent到-harness-成为新后端]]
- [[entities/agent-harness-architecture]]

→ [[raw/articles/agent-harness-observability-production|原文存档]] ^[raw/articles/agent-harness-observability-production.md]
- [[entities/perplexity-computer-knowledge-work-empirical-study|perplexity computer empirical study: how ai agents reshape k]]
- [[moc/agent-engineering-guide|MOC]]

