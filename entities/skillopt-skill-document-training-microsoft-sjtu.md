---

title: "SkillOpt：把 Agent 技能文档变成可训练对象"
created: 2026-06-10
updated: 2026-09-05
tags: [agent, code, evaluation, fine-tuning, llm, memory, microsoft, mlops, prompt, skill, tool-use, vision, workflow]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/skillopt-skill-document-training-microsoft-sjtu
---

# SkillOpt：把 Agent 技能文档变成可训练对象

→ [[raw/articles/skillopt-skill-document-training-microsoft-sjtu|原文存档]] ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]

## 深度分析

SkillOpt：把 Agent 技能文档变成可训练对象 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
### 核心观点
1. # SkillOpt：把 Agent 技能文档变成可训练对象 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
> 整理自 VibeCoder 团队对 SkillOpt 论文的中文报道
> 原文：https://mp.
2. com/s/l5ZtF-TPtttCtjyLiiGYUQ ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
> 论文：Microsoft × 上海交大 × 同济 × 复旦
> 推特点评：Rohan Paul「像训练小程序一样训练 agent 技能」
## 一句话定位
**SkillOpt = 冻结模型参数，把 agent 外部技能文档当作可训练对象，用验证集门控每一次编辑。 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
3. ** 部署阶段零额外模型调用（optimizer 只在训练阶段参与）。 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
4. > 类比：LoRA 冻结模型主体、只训练一个小参数适配层；**SkillOpt 冻结全部模型参数、只训练一份外挂 skill 文件** —— 社区直接称"LoRA for skills"。 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
5. ## 解决的工程盲区 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]
三种主流 skill 生产方式，同一个问题：**没有验证机制**。 ^[raw/articles/skillopt-skill-document-training-microsoft-sjtu.md]

### 关联实体

- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/你不知道的-agent原理架构与工程实践-v2]]
- [[entities/龙虾装上了可以用来干啥分享下我的-openclaw-多智能体团队搭建经验-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]

