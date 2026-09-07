---
title: "QQ 浏览器团队 Agent 经验管理 — 从个人记忆到团队语境"
created: 2026-08-13
updated: 2026-09-07
type: entity
tags: [tencent, agent, experience-management, team-context, memory, codebuddy, review, dedup]
sources: [raw/articles/ai-coding的下一站不是更会写代码而是更懂团队]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# QQ 浏览器团队 Agent 经验管理 — 从个人记忆到团队语境

> 腾讯程序员 2026-07-30（linjangyang、lypeerluo、kehawang、yansycao）。AI Coding 团队规模化后暴露短板：每次 session 踩坑纠偏产出的有效经验，session 结束即归零。业界多聚焦"个人记忆"，本文建设**团队经验沉淀系统**，让 AI 带项目语境进场。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]

## 关键实践

- 从初版 **90% 候选经验为废料**的起点，打磨出 Review/Dedup/Merge 三层治理链路^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
- 案例：stop hook 同步上报缺陷 → 必须异步否则阻塞进程^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
- 链路收敛为「对话上报 → 主题分组 → 经验抽取 → Review → Dedup → Merge → 入库 → 召回统计」^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
- 效果：垃圾率 90% → 约 5%、平均 80% 入库率；6 仓库、50+ 研发、1,236 次对话，入库 789 条经验^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]

与 [[entities/agent-memory-architecture|Agent 记忆架构]] 主题互补——个人记忆 vs 团队经验库；工程实现上属于 [[entities/harness-skill-engineering-alibaba-practice|Harness/Skill 工程]] 的团队层扩展。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md] 评测见 [[entities/agent-memory-evaluation-landscape-taobao-survey|Agent 记忆评测]]，姊妹篇 [[entities/team-experience-management-ai-coding-tencent-qqbrowser|腾讯团队经验管理]]。

## 深度分析

### 为什么个人记忆不够：团队语境是缺失的一层

stop hook 案例是全文缩影：AI 在自己的 session 里被纠正过一次——"必须异步，否则阻塞进程"，但纠正只存在于发起那轮 session 的开发者脑中，一周后另一同事触发同样场景，AI 再次给出同步实现，直到测试环境才暴露。团队规模化后总结出四类系统性问题：经验即抛、重复踩坑（隐性约束写在 git blame 里但从不进文档）、知识碎片化、AI 永远是"新同事"。业界方案（AutoDream、Hermes、Mem0）都聚焦"个人记忆"、追求"记住更多"；本文反其道——聚焦"团队经验质量"，追求"留下更少但更可信"。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]

### 90% 废料信号：过程录像不等于经验

初版方案极简：对话上报 → 自动提取 → 入库，Prompt 只要求"提取你认为有价值的内容"，结果约 90% 是废经验，且"废"以"召回后 Agent 行为是否正向改变"衡量。废料是四层结构性问题叠加：噪声极高（"多试几次就好了"是碰运气）、上下文脱离（"stop hook 需要异步"脱离限定场景会误导所有 hook）、错误引导、价值难定义。核心教训：**不加过滤的自动提取是垃圾放大器**。由此重定义"经验"：判定标准只有一个——**被召回后 Agent 能否产生正向行为变更**；并拆出三类"不容易直接发现"的经验：黑话镜头（如"D 站"=盗版站点）、索引镜头（如直达页面在 xhome 模块 FastCutXXX）、逻辑镜头（如 stop hook 并发阻塞）——"能不能自己发现"是可判定的客观维度。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]

### 三层治理：每层目标、指标与默认方向都不同

链路可靠来自三层治理，统一驱动为"错例分析 → 规则抽象 → 评测验证"。**Review** 默认保留、定向过滤——"漏掉好经验"代价大于"多放几条边缘经验"；纯文本无法验证技术事实，故引入源码探索：一条声称"FastScrollBar 应使用 attachToQBListView()"的经验看似自洽，检索类定义发现该方法根本不存在（正确为 FastScrollBarCompat.attachToQBRecyclerView()），被事实性拦截。**Dedup** 核心风险是误去重——边界污染永久不可逆，故宁严勿宽，最关键的是**禁止桥接式合并**（A≈B、B≈C 推不出 A≈C）；260 条经验 F1 71.79%，严格重复场景 Recall 91.67%。**Merge** 执行 create/update/skip/contradict：唯一目标判定前置、无唯一目标默认回退 create；update 必须自检（"积极合并"倾向突出，Recall 96.30% 但 Precision 78.79%）；contradict 不自动决断，走人工裁决——冲突是团队新旧认知不一致的信号；六轮实验 F1 94.27%。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]

### 经验分发：召回不是终点，采纳才是

治理通过的经验写入 IWIKI，Knot 自动索引，Agent 在 session 内经 Knot MCP 检索注入——复用已有基建而非造轮子。125 次检索：召回率 68.8%、no_hit 4.8%、平均注入 2.4 条、1,299ms。但命中不等于采纳：缺乏观测 Agent 是否依据经验改变行为的途径；长期未命中的经验无淘汰机制，库静默膨胀；项目演进时旧经验可能集体失效。生产决定基线 → 使用暴露问题 → 维护反哺生产，任一环不闭合，经验系统就从资产退回噪声——其本质是**团队认知能力的工程化管道**。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md] 这正是 [[concepts/agent-memory-lifecycle-philosophies|记忆生命周期]]、[[concepts/ai-team-knowledge-harness|团队知识沉淀]] 的闭环。

## 实践启示

1. **先定义资产边界，再做自动化沉淀**。不回答"什么算经验、什么不算、判断标准是什么"就启动自动提取，系统会把对话摘要、通用建议、一次性 case 全塞进库。关键不是"多提取"，而是"高置信地沉淀"。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
2. **别用一个 prompt 解决所有问题**。Review 拦垃圾、Dedup 候选集去重、Merge 历史库治理——目标、指标、prompt 各自独立，揉在一起互相牵制。多个针对性模型各做一件事，比一个大模型一次性搞定更可靠。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
3. **默认策略按业务风险分别设计，没有统一答案**。Review 默认保留、Dedup 宁严勿宽、Merge 保护历史边界——"宁可错杀"与"宁可放过"在各层方向完全相反。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
4. **先研究垃圾，再反推好经验**。九类典型垃圾（事实性错误、通用常识、对话摘要、一次性 case、粒度混用、不可执行、证据不足等）转成标注规则与 Reviewer 标准；可审计路径要求模型输出"为什么是经验/命中哪些排除规则/对话证据"三维解释，让判断脱离黑盒。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
5. **Prompt 迭代从错例中抽象规则，不是凭感觉改**。固定流程：收集错例 → 识别误判类型 → 抽象规则 → 实验验证 → 判断收益，固定评测集防过拟合；Recall/Precision/Garbage Rate 三者合看。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]
6. **给经验配"适用场景"而非只存结论**。stop hook 必须异步、超时别急着调参数，脱离 When/Why 就是误导。主题分组对照实验更揭示反直觉细节：给片段额外附 session 大纲反而产出更多垃圾，分组本身的粒度就够了。^[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队.md]

→ [[raw/articles/ai-coding的下一站不是更会写代码而是更懂团队|原文存档]]
