---
title: "Polaris — 浙大 ZJU-REAL 开源端到端科研智能体"
created: 2026-08-15
updated: 2026-09-07
type: entity
tags: [agent, research-agent, ai-scientist, llm-wiki, open-source, zju, arxiv, skill-system, mcp]
sources: [raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究]
confidence: 0.75
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Polaris — 浙大 ZJU-REAL 开源端到端科研智能体

Polaris（北极星）是浙江大学 ZJU-REAL 团队开源的端到端科研智能体，把「文献调研 → 想法生成 → 想法评审 → 实验 → 论文写作 → 论文评审」六个阶段串成一条完整流水线，AI 自主推进、人只在关键节点做决定。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]

## 六阶段流水线

1. **文献调研**：每日自动抓取 arXiv 新提交，AI 通读后为每篇生成一句话结论 + 中文导读（作者机构、概念标签），研究者筛选后收入文献库。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
2. **想法生成**：从概念空白、论文局限、研究趋势等信号挖掘候选想法，按新颖性/可行性等维度四维评分。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
3. **想法评审**：多位立场各异的 AI 评审员围绕每对想法正反辩论，按辩论结果做 Elo 排名（对局数、胜场记录在案），晋级与否由研究者决定。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
4. **实验**：连接实验室 GPU 服务器，AI 设计实验、写代码、启动训练、分析结果并迭代，生成图文实验报告。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
5. **论文写作**：在线 LaTeX 工作台（多文件工程/实时编译/PDF 预览），AI 逐节起草。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
6. **论文评审**：AI 同行评审逐条核验引用真伪、数字与实验记录对账，发现编造引用直接打回。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]

## 关键设计

- **实验智能体循环**：先规划（实验目标拆成带验收标准的步骤清单）→ 执行（写代码/部署/训练）→ 每步校验（退出码/产物/指标核对）→ 一轮跑完 AI 自析，指标不涨换思路补步骤继续。计划是「随实验证据生长的任务板」，不是写死的剧本。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
- **人在环上（human-in-the-loop）**：想法晋级、GPU 预算动用、论文投稿都停下来等人点头；实验到取舍节点 AI 暂停提问（如「换更难数据集还是先补度量」），答复被采纳进下一轮。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
- **LLM Wiki 文献库**：参考 Karpathy LLM Wiki 思路，每篇论文编译成一页深度解读（动机/方法/可借鉴处 + 插图说明），概念互相链接 + 时间维度主题演化，支持语义检索与 Obsidian 导出。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
- **技能系统**：评审标准、辩论人设、实验代码规范都可做成技能挂到对应环节；内置文献综述速写、实验设计规范、Rebuttal 起草等技能，支持自定义 + 技能市场。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
- **PolarisBuddy 全局助手**：常驻工作区，Command J 唤出，了解整个工作区上下文，可直接交付目标自行规划执行。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
- **开放能力**：文献管理、实验资产通过 MCP + 技能体系对外开放，可在 Claude Code、Codex、DeepSeek Harness 中直接调用；桌面客户端覆盖 macOS/Windows/Linux。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]
- **多人实验室平台**：方向文献库团队共建、一篇论文全平台只解析一次，AI 用量/环节消耗在实验室工作台可视化。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]

## 定位对比

Polaris 是「完整科研流水线编排」路线（端到端六阶段 + 人机分工），区别于[[entities/claude-science-开源平替-open-science-2026|Claude Science 开源平替]]与 [[entities/anthropic推出claude-science-科研界的claude-code|Anthropic Claude Science]] 的「研究 AI 工作台」路线，也与 [[entities/4300万论文30亿三元组科研agent实现多视角创新评估|科研 Agent 多视角创新评估]]（文献图谱驱动的评估视角）不同。三者互补：Polaris 强在流程闭环与实验室多人协作，Claude Science 强在模型能力底座与产品化，文献图谱路线强在创新评估的全局视野。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md]

## 相关概念与实体

- [[concepts/llm-wiki-paradigm|LLM Wiki 范式]] — Polaris 文献库直接采用 Karpathy LLM Wiki 思路
- [[concepts/scientific-method-ai-research|科研方法 × AI]]
- Agent 架构
- MCP 生态
- [[entities/skillclaw|SkillClaw]] — 技能系统生态参照
- [[entities/deepseek-code-harness|DeepSeek Harness]] — Polaris 能力开放的对接目标之一

→ [[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究|原文存档]]
