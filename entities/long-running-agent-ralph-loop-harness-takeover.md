---

title: "长周期 Agent 详解：从 Ralph Loop 到可接管 Harness"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, architecture, code, harness-engineering, memory, open-source, search, security]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/long-running-agent-ralph-loop-harness-takeover
---

# 长周期 Agent 详解：从 Ralph Loop 到可接管 Harness

→ [[raw/articles/long-running-agent-ralph-loop-harness-takeover|原文存档]] ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]

## 深度分析

长周期 Agent 详解：从 Ralph Loop 到可接管 Harness ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]
### 核心观点
1. # 长周期 Agent 详解：从 Ralph Loop 到可接管 Harness ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]
> 来源：架构师（JiaGouX） | 作者：若飞 | 2026-05-10
## 太长不看
- Codex `/goal` 很重要，但它解决的主要是"能不能一直干下去"，不等于把长任务的正确性也一起解决了。
2. - 朴素 Ralph Loop 的问题不在循环次数，而在每一轮都在悄悄积累目标漂移、上下文漂移和质量漂移。 ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]
3. - 长周期 Agent 比起"半途而废"，更怕"勤奋地跑偏"。 ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]
4. - 前置 Spec 的价值，是把错误的决策分叉提前剪掉，避免后面的 token 在错路上越跑越远。 ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]
5. - 外部状态文件比聊天记录靠谱。 ^[raw/articles/long-running-agent-ralph-loop-harness-takeover.md]

### 关联实体

- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道]]
- [[entities/两万字详解claude-code源码核心机制]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/构建基于多智能体架构的深度思考交易系统-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]

