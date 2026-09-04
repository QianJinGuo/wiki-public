---

title: "基于钉钉机器人的 Qoder CLI / Claude Code 双引擎 AI 助手实践"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, architecture, claude, code, fine-tuning, k8s, llm, memory, mlops, prompt, rl, robotics, search, security, tool-use]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant
---

# 基于钉钉机器人的 Qoder CLI / Claude Code 双引擎 AI 助手实践

→ [[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant|原文存档]] ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]

## 深度分析

基于钉钉机器人的 Qoder CLI / Claude Code 双引擎 AI 助手实践 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
### 核心观点
1. # 基于钉钉机器人的 Qoder CLI / Claude Code 双引擎 AI 助手实践 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
> 闪购搜索团队 久梦 @阿里云开发者
> 原文：https://mp.
2. com/s/UdQ7xhM25Er6Eyk0xs577w ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
## 一、背景与问题
在闪购搜索团队的日常工作中，我们需要频繁地进行搜索问题排查、性能分析、实验管理等操作。 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
3. 这些操作分散在多个平台（SLS日志、TPP实验平台、代码仓库等），效率低下。 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
4. **目标**：在钉钉群里直接对话一个AI助手，它能代替人去查日志、看实验、分析性能、甚至部署代码。 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
5. ## 四、Docker 部署方案 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]
两个引擎部署在同一个 Docker 容器内，共享工作目录和 MCP 配置。 ^[raw/articles/dingtalk-qoder-claudecode-dual-engine-ai-assistant.md]

### 关联实体

- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/两万字详解claude-code源码核心机制]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/深入理解-claude-code-源码中的-agent-harness-构建之道]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]

## 相关实体

- [[moc/prompt-engineering-guide|MOC]]
