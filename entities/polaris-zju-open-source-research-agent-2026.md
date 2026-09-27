---
title: "Polaris — 浙大 ZJU-REAL 开源端到端科研智能体"
created: 2026-08-15
updated: 2026-09-27
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

## 深度分析

### 从「零碎工具」到「流程闭环」：编排才是稀缺能力

过去两年大模型已能读论文、写代码、改文章，但大多以问答工具形态散落在各环节，节省的只是零碎时间。Polaris 的关键不在单点能力，而在把六个阶段串成流水线，让上一阶段成果自然流入下一阶段：文献变成想法，想法变成实验，实验变成经过评审的论文。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:16-18] 端到端编排的价值在于中间产物被结构化——一页深度解读、四维评分、实验报告——从而能被下游阶段直接消费，这是散装工具形态做不到的。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:30-48]

### 辩论 + Elo：把主观判断变成可审计的排序

「这个想法值不值得做」难以客观度量，Polaris 用多位立场各异的 AI 评审员围绕每对想法正反辩论，再按辩论结果做 Elo 排名，对局数和胜场记录在案。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:46-48] 这一设计把主观评估转换为可复现、可审计的过程数据，比单一模型打分更抗噪声，同时把晋级决定权保留给研究者，避免自动筛选放大模型偏好。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:46-48]

### 计划是「随证据生长的任务板」，不是写死的剧本

实验智能体循环的关键在于：规划产出带验收标准的步骤清单，每步校验退出码、产物、指标，一轮结束后 AI 自析结果，指标不涨就换思路并把新步骤补进清单。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:58] 加上取舍节点的暂停提问机制——AI 停下来向研究者提问，答复被采纳进下一轮——计划成为随实验证据不断生长的任务板，与一次写死的静态 plan-and-execute 形成鲜明对比。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:58-60]

### 硬约束对账：AI 写论文的信任边界

论文环节设有一条硬性约束：每个数字只能来自真实实验记录，每条引用只能指向真实存在的文献；成稿后 AI 同行评审逐条核验引用真伪、把数字与实验记录对账，发现一条编造引用稿件直接打回。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:70-72] 这等于把幻觉问题转化为可机检的一致性校验，是用 LLM 从事高风险学术写作的务实解法。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:70-72]

### 技能系统：把实验室的品味变成可挂载资产

不同实验室有不同的品味和规矩，Polaris 用技能系统把评审标准、辩论人设、实验代码规范沉淀为可挂载到对应环节的资产，还支持自定义和从技能市场安装。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:80] 这把隐性组织知识显性化、可复用化；配合 MCP 对外开放能力（Claude Code、Codex、DeepSeek Harness 可直接调用），平台形成了向内沉淀、向外扩展的双向接口。^[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究.md.md:80, 104]

## 实践启示

1. 构建 AI 工作流时优先设计「阶段产出可被下一阶段结构化消费」的流水线，而非孤立的单点工具——中间产物的结构化程度决定整条链的自动化上限。
2. 对主观评估类任务（想法筛选、方案评审），用多立场辩论 + Elo 排名代替一次性打分，把判断变成带记录的可审计过程，最终决策权留给人。
3. 实验/长任务 agent 的计划不要一次写死：拆成带验收标准的步骤清单，逐步校验退出码、产物、指标，并允许 AI 依据证据改写后续步骤。
4. 用 LLM 生成任何带事实主张的文本（论文、报告）时，加一条硬性对账约束：每个数字可溯源到原始记录，每条引用可验证存在，验证不过直接打回。
5. 把团队的隐性规范（评审标准、代码风格、写作流程）沉淀为可挂载、可分享的技能文件，而不是散落在口口相传里。
6. 人机分工的合理切分是「AI 推进到取舍节点即停」：用一份随进度更新的待办清单（AI 完成的打勾、待决策的亮出）管理暂停点，而不是事事请示或一路跑到底。

## 相关概念与实体

- [[concepts/llm-wiki-paradigm|LLM Wiki 范式]] — Polaris 文献库直接采用 Karpathy LLM Wiki 思路
- [[concepts/scientific-method-ai-research|科研方法 × AI]]
- Agent 架构
- MCP 生态
- [[entities/skillclaw|SkillClaw]] — 技能系统生态参照
- [[entities/deepseek-code-harness|DeepSeek Harness]] — Polaris 能力开放的对接目标之一

→ [[raw/articles/浙大团队开源ai科研智能体polaris让ai与你一起做研究|原文存档]]
