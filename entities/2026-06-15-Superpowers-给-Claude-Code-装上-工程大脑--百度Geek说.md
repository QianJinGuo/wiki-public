---
title: "Test-Driven Development Skill"
created: 2026-08-01
updated: 2026-11-13
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

> -> [[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md|原文存档]]

## 摘要

这是百度 Geek说公众号对 Claude Code 插件 Superpowers 的万字深度解析，一句话定位是"Superpowers 不是让 Claude 变聪明，而是让 Claude 变守纪律"——它通过 14 个内置技能强制 AI 走"澄清→设计→规划→执行→验证"的工程流程。作者（奔跑的脆皮肠）从其 querit.ai 订阅支付前端项目的真实翻车经历（4 天 10+ 轮交互、状态机混乱、mock 数据与后端对不上）出发，把裸跑 AI 的三大原罪归结为回答随机性、直觉快思考与注意力稀释，并以认知负荷理论、《清单革命》等框架解释 Superpowers 的分步流程与验证清单为何有效。文章剖析了 Jesse Vincent（obra，30 年开源老兵、RTorrent/Request Tracker 作者）的设计动机——"AI 编码代理缺少的不是能力，而是纪律"，并给出项目数据：2025.10.09 发布，2026.05 达 170,000+ Stars，Anthropic 官方插件市场安装量近 30 万次、居第三方第一。技术层面拆解了 brainstorming Skill 的源码（强制触发、一次一问、2-3 方案、YAGNI、200-300 字分段设计确认、设计文档落盘 docs/plans/）与 TDD Skill（RED/GREEN/REFACTOR），提出"概率操控"解释（MUST 强制词汇、数字锚点、状态锁定、链式 Skill 调用锁定采样路径），并以 chardet v7 重写（5 天、性能提升最高 48 倍、与旧版仅 1.29% 代码重叠、LGPL 转 0BSD）为标杆案例，同时诚实列出八项负向收益（简单任务流程开销、创意约束、上下文占用、过度工程化等）。^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md]

## 关键要点

- 三大原罪：回答随机性（概率预测器每次采样"掷骰子"）、直觉快思考（倾向一步到位直接给代码）、注意力稀释（长上下文记忆串台）；对应解法是外部化认知负荷管理（认知负荷理论）与"术前清单"式验证清单（《清单革命》：术后并发症死亡率下降 47%）
- 作者与起源：Jesse Vincent 2025-10-09 因 Anthropic 发布 Claude Code 官方插件系统而公开积累的工作流；其方法论源自 2000 年代初通过 IRC 远程指挥 MIT 实习生的管理经验——管理 AI 代理与管理初级程序员是同一问题；还运用了 Cialdini《影响力》的说服原则（MUST 权威词汇、承诺一致性）
- 增长与认可：2025.11 达 27,000 Stars 登顶 GitHub Trending #1，2026.01.15 被 Anthropic 官方市场收录，2026.05 超 170,000 Stars、官方市场安装量近 30 万次（第三方第一，仅次于 Anthropic 自家 frontend-design）
- brainstorming Skill 设计：描述用"You MUST use this before any creative work"强制宽触发；一次只问一个问题（降低概率分布复杂度）；强制提出 2-3 个带权衡的方案；YAGNI ruthlessly；设计以 200-300 字分段逐段确认；产出 docs/plans/YYYY-MM-DD-topic-design.md 并链式调用 git worktrees 与 writing-plans 形成显式状态机
- 标杆案例 chardet v7：维护者 Dan Blanchard 用 Claude Code + Superpowers 重写，brainstorming 产出设计文档、全新仓库不访问旧源码，5 天完成，性能最高提升 48 倍，与旧版代码重叠仅 1.29%，许可证 LGPL 改为 0BSD
- 负向收益（八项）：简单任务流程开销大于收益（驼峰转下划线也被问四个问题）、约束扼杀创意、Skills 注入占用上下文窗口、流程本身制造过度工程化（要个 checkbox 得到完整测试套件）、学习曲线、团队协作摩擦、第三方维护风险、"安全感陷阱"——流程规范不等于结果正确，Superpowers 提高下限但不保证上限

## 深度分析

### 概率操控：提示词工程如何重塑采样路径

文章最具原创性的部分是把 Superpowers 的作用机制还原为一场**概率操控**。LLM 本质上是概率预测器，"直接写代码"往往是采样时概率最高的续写路径，却不一定是最优路径^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md:369-373]。14 个 Skill 没有给模型注入任何新知识，而是用四种手段系统性重塑概率分布：**强制词汇**（"You MUST use this" 把 brainstorming 触发概率从约 20% 推到约 80%）、**数字锚点**（"one at a time"、"2-3 approaches"、"200-300 words" 让输出结构稳定）、**状态锁定**（把已确认设计写入 `docs/plans/` 落盘，将对话状态持久化到文件系统，避免长上下文中的概率漂移）、**链式调用**（显式指定下一个 Skill，形成确定性状态机转换而非让模型自由选择）^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md:628-652]。配合自回归特性——每一步输出都成为下一步的强条件——模型被"锁定"在正确路径上。这与 [[concepts/prompt-engineering-fundamentals]] 的结构化约束思路一脉相承，也解释了 [[concepts/context-engineering]] 为何强调约束优先于能力。

### 外部化纪律：认知科学与管理学的三重印证

文章为"纪律比聪明重要"给出了跨学科支撑。**认知负荷理论**：工作记忆容量有限，超载则决策质量骤降；分步流程本质上是把"一次性想清楚所有事"外部化为"每次只想一件事"，"一次只问一个问题"的设计正源于此。**《清单革命》**：术前清单让术后并发症死亡率下降 47%，验证清单就是为 AI 手术台设计的"术前清单"——不提升医生技术，只确保关键步骤不被跳过。**管理学迁移**：Jesse Vincent 自述其 "management hacks" 是 2000 年代初通过 IRC 远程指挥 MIT 实习生那套方法的复刻——管理 AI 代理与管理初级程序员本质上是同一个问题；连 Cialdini《影响力》的说服原则（MUST 权威词汇、承诺一致性）也被证实对 LLM 同样有效^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md:189-195]。这与 [[concepts/coding-harness-engineering]] 的核心主张一致：harness 的价值在于行为约束而非能力增强。

### 与 The Bitter Lesson 的分层兼容

针对"给模型加约束会不会限制其能力"的质疑（Bitter Lesson 说通用方法终将碾压人造规则），文章的化解方式是**分层论证**：Bitter Lesson 讨论的是"如何让模型本身更聪明"，Superpowers 讨论的是"如何让已经聪明的模型在当前任务中稳定表现出你需要的行为"，两者不在同一层面竞争。文章同时诚实承认了 Bitter Lesson 的最终预言：当模型通过大规模强化学习学会自主规划、强制测试、自我审查的"元能力"后，这类手写外挂终将被淘汰^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md:400-414]。Superpowers 的定位因此是**真空期杠杆**——在模型内生能力补足缺口之前的过渡工具。这个判断对 harness 类工程具有普遍意义：一切流程约束层都面临"被下一代模型吸收"的时间压力，工程价值在于精准匹配当下模型的能力缺口。

### 负向收益的诚实边界：流程不等于结果

文章最有价值的部分之一是对八项负向收益的坦诚列举，其中两项尤其深刻。**过度工程化的讽刺**：Superpowers 在 Skill 源码中硬编码了 YAGNI 原则（移除不必要的功能），但其流程本身却在制造不必要的复杂度——用户要一个"记住我" checkbox，得到的却是可配置 cookie 策略、审计日志和完整测试套件。**安全感陷阱**：走了 brainstorming 不等于需求没问题，走了 TDD 不等于边界覆盖到位，AI 审查 AI 还可能遗漏同类错误——流程规范 ≠ 结果正确^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md:1028-1035]。文章的定位很精确：Superpowers 提高**下限**（不会太差），但不保证**上限**（不一定很好）。这类流程工具是风险管理工具而非质量创造工具，理性用法是风险驱动开发——用"后悔成本"而非任务大小来决定流程重量。

### 从写代码到当主管：开发者角色的质变

文章把 Superpowers 的宏观意义归结为开发者角色的重新定义：从"亲自撸代码的操作工"升级为"思考、判断、把关"的技术主管，分工被概括为"大模型 = 能力，Superpowers = 纪律，你 = 方向"。这一转变与精益开发的"延迟决策"原则契合：AI 不替你做关键决策，而是在每个检查点停下来等你确认，你的角色是精益生产中的"安灯拉绳者"——发现异常就拉绳叫停。关键认知是：你仍然为每一行进入主分支的代码承担最终责任，只是精力从"怎么写对"转向"怎么保证写得对"。这与 [[concepts/agent-role-specialization]] 描述的人机分工趋势相互印证，也呼应了 [[concepts/agentic-workflow-patterns]] 中"人类作为检查点审查者"的编排模式。

## 实践启示

1. **用"后悔成本"而非任务大小决定流程重量**：驼峰转下划线不值得走五阶段流程，支付状态机值得。判断标准是"做错了要付多大代价修复"，而非"任务看起来多简单"——高风险任务穿防弹衣，低风险任务直接给代码。

2. **在项目级 CLAUDE.md 里预置降级策略**：在 `.claude/CLAUDE.md` 写明默认最简实现、不主动引入新依赖、仅用户明确要求时启动完整流程、子代理仅在任务涉及 >3 个独立模块时开启——让调度器自动降级，避免每次手动开关^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md:888-900]。

3. **强制把设计共识落盘到文件**：brainstorming 把已确认设计写入 `docs/plans/YYYY-MM-DD-<topic>-design.md` 是对抗长上下文概率漂移的关键手段。任何多轮 AI 协作都应效仿：对话中达成的共识若不持久化，后续轮次大概率丢失。

4. **前端类任务先问清状态范围和数据格式再写码**：querit.ai 案例的核心教训是 AI 只写"快乐路径"、mock 数据与后端字段对不上、状态机设计不专业。写码前强制澄清：要覆盖哪些 UI 状态分支（loading/error/empty）、mock 是否基于后端类型定义生成、状态机与组件是否分离。

5. **检查点是为你设计的，别无脑点"继续"**：主代理在检查点审查子代理产出（发现缺类型定义、审核失败不可重试等问题并要求修正）是整套流程的质量来源。把检查点当作免费的人工 code review 窗口，发现问题立即打断，而非事后返工。

6. **警惕流程规范带来的虚假安全感**：走了全流程 ≠ 结果正确。brainstorming 的问题可能不够深入，TDD 的测试可能漏掉边界，AI 审 AI 可能犯同类错误。保持独立判断——Superpowers 提高下限，上限仍取决于你对需求的洞察深度。

## 来源

- 原文: [[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md|Test-Driven Development Skill]]
