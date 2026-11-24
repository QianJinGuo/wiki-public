---
title: "Harness 实践 将任何文字编辑成精美的文章 code秘密花园"
created: 2026-08-01
updated: 2026-11-24
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Harness 实践 将任何文字编辑成精美的文章 code秘密花园

> -> [[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md|原文存档]]

## 摘要

作者 ConardLi 在此前"Agent 自动制作知识讲解视频"的实践基础上验证"好的 Harness 是可以迁移的"，开源了 beautiful-article Skill，用几乎同一套骨架把任意素材（URL、PDF、DOCX、文本）变成排版精美的网页长文 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。Skill 把流程切成 8 个 Phase 并卡 3 个强制人工检查点（Plan、First Spread、Delivery），底层是 Reacticle 组件协议——一套受约束的语义化 React 组件契约加 Raw 逃生舱，配 11 套编辑级主题，让 AI 只负责组合组件、不操心版式，保证"稳定、不崩版" ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。实战采用 Claude Code + MiniMax M3（1M 上下文、原生多模态）组合，终审由 Editorial/Visual/Technical 三个独立 SubAgent 分别质检，修复采用最小切片策略 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。文章最后把视频 Skill 与文章 Skill 逐层对照，指出上下文管理、工具系统、执行编排、状态记忆、评估观测、约束恢复六个 Harness 部件设计几乎一致，且质检修复日志落盘可以反过来驱动 Skill 自进化 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md]。

## 关键要点

- Reacticle = React + Article：提供 Article/Hero/Lead/Section/Table/Quote/Formula/CodeBlock 等语义组件平替 Markdown 语法；Raw 自由层可塞任意 HTML/SVG/CSS，但所有样式必须消费约束好的主题 token
- 8 个 Phase：Intake → Source → Editorial Planning → Plan Checkpoint（★）→ First Spread（★）→ Full Article Build → Final Review → Repair → Delivery（★）；检查点禁止替用户做主，每个决策项必须独立等用户答复
- 内置 9 种文章类型，信息保留比例从约 25%（interactive-explainer）到 100%（longform）不等；还有 narrow/regular/wide/full 版式与 none/user-assets/placeholders/ai-generated 配图模式
- 一节一文件是多 Agent 并行的铁律：每个并行 SubAgent 只负责一个 section 文件，大型 Raw 放独立的 raw-blocks/，主 Agent 只负责组装
- 实战环境：Claude Code + MiniMax M3 + CC Switch 切换供应商；产出 CSS+JS 全内联的单文件 HTML（断网可打开）与 PDF 两种交付物
## 深度分析

### 8 Phase 流程与三个强制人工检查点：把决策冻结在错误最便宜的位置

Skill 把"素材→精美文章"切成 Intake、Source→Markdown、Editorial Planning、Plan Checkpoint、First Spread、Full Build、Final Review、Repair、Delivery 等环节，并在 Plan、First Spread、Delivery 三处设"必须停"的人工检查点 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:88-110]。分段的真正作用是提前冻结决策面：Phase 2 先产出 plan.md，用 Brief/Outline/Theme/Assets 四段写死目标读者、章节结构、主题与配图策略，成为后续所有开发的文件化边界；Phase 3 再逐项确认文章类型、信息保留比例、主题、版式宽度、配图模式——Agent 只给推荐、不替用户做主，因为读者目标不同，生成的文章形态完全不同 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:300-342]。生成顺序也不是一口气写完：先只做封面+首屏+第一节+一个代表性视觉块来定调，人工确认后再放量，因为首屏气质一旦偏了，后面写得越多返工成本越高 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:362-392]。

### Reacticle 组件协议与 Raw 逃生舱：约束下的自由

Agent 写的不是裸 HTML，而是 Reacticle（React+Article）组件契约：AI 只负责组合 Article/Hero/Lead/Section/Table 等语义组件，用什么标签、什么层级、间距多少全被组件封装，换来"稳定、不崩版" ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:111-139]。自由度靠 Raw 逃生舱补齐——任意 HTML/SVG/CSS/React 都能塞进去画自定义时序图或交互控件，但硬约束是 Raw 里所有样式必须消费约束好的主题 token，自由与稳定各归其位 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:141-155]。11 套编辑级主题同时是 CSS token 包和给 AI 读的 Markdown profile（Tufte 主题会明确告诉 AI"不要渐变阴影、数据墨水优先"）；Reacticle 是运行时协议、管输出表面，Skill 是拼装说明书、管生产过程，职责分离 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:157-195]。

### 三 SubAgent 终审质检与最小切片修复：评估层的分工与止损

所有 Section 写完后开三个独立 SubAgent 分维度终审：Editorial 管文章性/信息取舍/结构，Visual 管主题/Raw/配图/移动端，Technical 管构建/控制台/代码公式/可访问性，检出问题进入修复流程 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:408-422]。修复坚持最小切片：某节信息薄只补那节、某个 Raw 太花只改那个 Raw、首屏气质不对只调 Hero/Lead/Summary，保护已经正确的部分不被一次局部反馈搞坏 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:593-609]。这套 [[concepts/subagent-spawning-pattern]] 能跑起来的前提是一节一文件：每个并行 SubAgent 只写一个 section 文件、大型 Raw 放独立 raw-blocks/、Article.tsx 组装权只归主 Agent，互不覆盖 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:505-519]。

### 与视频 Skill 的六部件 Harness 对照：可迁移的骨架

作者把成熟 Harness 拆成上下文管理、工具系统、执行编排、状态与记忆、评估与观测、约束与恢复六个部件（参见 [[entities/agent-harness-12-components-7-decisions]] 的部件化拆法），逐一验证"好的 Harness 可以迁移"：上下文管理靠渐进加载，Phase 1 只读素材抽取文档、Phase 2 才读文章类型与主题、Phase 4/5 进入开发才读组件协议与主题 profile，避免一次性灌入稀释注意力 [[concepts/context-management-agent-systems]] ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:467-487]；执行编排上视频 4 阶段 2 检查点、文章 8 Phase 3 检查点，阶段数不同但骨架一样；状态与记忆把关键决策落盘成文件，写到后面直接回读而非凭印象 [[concepts/working-set-vs-long-term-memory]] ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:541-553]；评估观测按产出分级设质检点，Plan 阶段上下文还热由主 Agent 自查、关键节点换独立 SubAgent，且先修复再汇报；自进化则让 review/repair 日志反过来指出最易出错的环节，驱动规则与默认策略迭代 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:611-627]。

## 实践启示

1. 设计 Skill 的第一刀是切阶段并在关键处强制停下让人拍板；检查点必须逐项独立列出决策，"已经替你选了 X，不对再告诉我"等于偷渡默认值 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:314-342]。
2. 让 AI 产出结构化内容时不要裸写最终格式：先定义受约束的语义组件协议管稳定性，再用 Raw 逃生舱 + 主题 token 约束承接真正的自由需求 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:111-155]。
3. 长任务记忆靠文件不靠上下文：source.md/plan.md/extraction-notes.md 各有固定职责，Agent 每写一节前回读文件把自己拉回正轨，防止已确认方向写着写着漂移 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:541-553]。
4. 并行开工前先做物理隔离：一节一文件、Raw 独立目录、组装权独占主 Agent，才能多个 SubAgent 同时工作而不互相覆盖。
5. 质检与修复日志不是给人看的，是给 Agent 看的自进化燃料：落盘后定期回看哪类环节最常出错，回头改规则、检查清单和默认策略 [[concepts/agent-self-improvement-loops]] ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:611-627]。
6. 不必从零搭 Agent：把一个垂直任务用 Skill 做稳，凑齐状态、流程、检查点和交付标准，本身就是在做 [[concepts/harness-engineering]]；同一骨架可迁移到周报、播客 Shownotes、课程讲义、技术文档等任何足够复杂的任务 ^[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md:651-669]。

## 来源

- 原文：[[raw/articles/2026-06-18-Harness-实践-将任何文字编辑成精美的文章-code秘密花园.md|Harness 实践 将任何文字编辑成精美的文章 code秘密花园]]
