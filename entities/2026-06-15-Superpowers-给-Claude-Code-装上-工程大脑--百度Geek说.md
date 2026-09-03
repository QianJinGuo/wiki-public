---
title: "Test-Driven Development Skill"
created: 2026-08-01
updated: 2026-08-29
type: entity
tags: ['raw', 'article']
sources: [raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说]
provenance_state: extracted
---

> -> [[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md|原文存档]]

sha256: 47518294cba699dee622bef9fa70fcf2a43801a85bee893b6e5747c291d07637 ^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md]

## 摘要

这是百度 Geek说公众号对 Claude Code 插件 Superpowers 的万字深度解析，一句话定位是"Superpowers 不是让 Claude 变聪明，而是让 Claude 变守纪律"——它通过 14 个内置技能强制 AI 走"澄清→设计→规划→执行→验证"的工程流程。作者（奔跑的脆皮肠）从其 querit.ai 订阅支付前端项目的真实翻车经历（4 天 10+ 轮交互、状态机混乱、mock 数据与后端对不上）出发，把裸跑 AI 的三大原罪归结为回答随机性、直觉快思考与注意力稀释，并以认知负荷理论、《清单革命》等框架解释 Superpowers 的分步流程与验证清单为何有效。文章剖析了 Jesse Vincent（obra，30 年开源老兵、RTorrent/Request Tracker 作者）的设计动机——"AI 编码代理缺少的不是能力，而是纪律"，并给出项目数据：2025.10.09 发布，2026.05 达 170,000+ Stars，Anthropic 官方插件市场安装量近 30 万次、居第三方第一。技术层面拆解了 brainstorming Skill 的源码（强制触发、一次一问、2-3 方案、YAGNI、200-300 字分段设计确认、设计文档落盘 docs/plans/）与 TDD Skill（RED/GREEN/REFACTOR），提出"概率操控"解释（MUST 强制词汇、数字锚点、状态锁定、链式 Skill 调用锁定采样路径），并以 chardet v7 重写（5 天、性能提升最高 48 倍、与旧版仅 1.29% 代码重叠、LGPL 转 0BSD）为标杆案例，同时诚实列出八项负向收益（简单任务流程开销、创意约束、上下文占用、过度工程化等）。^[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md]

## 关键要点

- 三大原罪：回答随机性（概率预测器每次采样"掷骰子"）、直觉快思考（倾向一步到位直接给代码）、注意力稀释（长上下文记忆串台）；对应解法是外部化认知负荷管理（认知负荷理论）与"术前清单"式验证清单（《清单革命》：术后并发症死亡率下降 47%）
- 作者与起源：Jesse Vincent 2025-10-09 因 Anthropic 发布 Claude Code 官方插件系统而公开积累的工作流；其方法论源自 2000 年代初通过 IRC 远程指挥 MIT 实习生的管理经验——管理 AI 代理与管理初级程序员是同一问题；还运用了 Cialdini《影响力》的说服原则（MUST 权威词汇、承诺一致性）
- 增长与认可：2025.11 达 27,000 Stars 登顶 GitHub Trending #1，2026.01.15 被 Anthropic 官方市场收录，2026.05 超 170,000 Stars、官方市场安装量近 30 万次（第三方第一，仅次于 Anthropic 自家 frontend-design）
- brainstorming Skill 设计：描述用"You MUST use this before any creative work"强制宽触发；一次只问一个问题（降低概率分布复杂度）；强制提出 2-3 个带权衡的方案；YAGNI ruthlessly；设计以 200-300 字分段逐段确认；产出 docs/plans/YYYY-MM-DD-topic-design.md 并链式调用 git worktrees 与 writing-plans 形成显式状态机
- 标杆案例 chardet v7：维护者 Dan Blanchard 用 Claude Code + Superpowers 重写，brainstorming 产出设计文档、全新仓库不访问旧源码，5 天完成，性能最高提升 48 倍，与旧版代码重叠仅 1.29%，许可证 LGPL 改为 0BSD
- 负向收益（八项）：简单任务流程开销大于收益（驼峰转下划线也被问四个问题）、约束扼杀创意、Skills 注入占用上下文窗口、流程本身制造过度工程化（要个 checkbox 得到完整测试套件）、学习曲线、团队协作摩擦、第三方维护风险、"安全感陷阱"——流程规范不等于结果正确，Superpowers 提高下限但不保证上限

## 来源

- 原文: [[raw/articles/2026-06-15-Superpowers-给-Claude-Code-装上-工程大脑--百度Geek说.md|Test-Driven Development Skill]]
