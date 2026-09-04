---

title: "手把手：用 Hermes Skills + Karpathy 的 LLM Wiki 让 AI 越用越懂你"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, aws, code, data, database, k8s, knowledge-mgmt, llm, memory, mlops, observability, open-source, search, skill, tool-use, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system
---

# 手把手：用 Hermes Skills + Karpathy 的 LLM Wiki 让 AI 越用越懂你

→ [[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system|原文存档]] ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]

## 深度分析

手把手：用 Hermes Skills + Karpathy 的 LLM Wiki 让 AI 越用越懂你 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
### 核心观点
1. # 手把手：用 Hermes Skills + Karpathy 的 LLM Wiki 让 AI 越用越懂你 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
## 整体结构：三层互相喂养
- **Memory**：记住你是谁（事实类）
- **Skills**：记住怎么干活（方法类）
- **Wiki**：目录把零散知识组织起来（空间+时间维度）
三者互相喂养，越用越厚。 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
2. ## 第一步：确认 Skills 目录存在 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
ls ~/. ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
3. hermes/skills/ ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
# 如果不存在：
mkdir -p ~/. ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
4. hermes/skills/ ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
## 第二步：理解 SKILL.
5. md 的结构 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
name: writing-pr-descriptions ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
description: "按团队规范写 PR 描述" ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
version: 1 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]
## When to Use
当完成功能开发，准备提交 PR 的时候。 ^[raw/articles/hermes-skills-llm-wiki-self-improving-knowledge-system.md]

### 关联实体

- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/scale-robot-reinforcement-learning-with-nvidia-isaac-lab-on-]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/nvidia-isaac-lab-sagemaker-robot-rl-humanoid]]

## 相关实体

- [[moc/data-infrastructure|MOC]]
