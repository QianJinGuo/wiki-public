---

title: "Hermes Agent Skills 源码级拆解：3级渐进加载 × 6步调度 × 5维安全扫描"
created: 2026-06-10
updated: 2026-08-29
tags: [agent, aws, code, data, memory, open-source, prompt, rl, search, security, skill, tool-use, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/hermes-agent-skills-source-code-analysis-shuge
---

# Hermes Agent Skills 源码级拆解：3级渐进加载 × 6步调度 × 5维安全扫描


## 相关实体

- [[entities/hermes-skill-system-deep-dive|hermes新顶流agent skills闭环系统深度解析]]
→ [[raw/articles/hermes-agent-skills-source-code-analysis-shuge|原文存档]] ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]

- [[moc/data-infrastructure|MOC]]
## 深度分析

Hermes Agent Skills 源码级拆解：3级渐进加载 × 6步调度 × 5维安全扫描 涉及agent领域的核心技术议题。 ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
### 核心观点
1. # Hermes Agent Skills 源码级拆解：3级渐进加载 × 6步调度 × 5维安全扫描 ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
> 源码分析版（vs [[entities/hermes-skill-system|Hermes Agent Skill 系统深度解析]] winty版）
## 核心定位
Hermes 两套记忆机制： ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
- **通用记忆**（MEMORY.
2. md）：存储"知道什么"——用户偏好、项目信息 ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
- **Skills**：过程性记忆（Procedural Memory），存储"怎么做"——工作流、最佳实践
Skills 遵循 **agentskills. ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
3. io 开放标准**，非私有格式。 ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
4. ## 渐进式披露（Progressive Disclosure） ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
三个加载层级： ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
| Level | 调用 | 内容 | Token | ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
|---|---|---|---| ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
| 0 | `skills_list()` | `[{name, description, category}, . ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
5. ]` | ~3k | ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
| 1 | `skill_view(name)` | Full content + metadata | varies | ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
| 2 | `skill_view(name, path)` | Specific reference file | varies | ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]
懒加载思路：Agent 先扫 Level 0 列表，判断相关 Skill，再按需加载完整内容。 ^[raw/articles/hermes-agent-skills-source-code-analysis-shuge.md]

### 关联实体

- [[entities/两万字详解claude-code源码核心机制]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr]]
- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/构建基于多智能体架构的深度思考交易系统-v2]]

