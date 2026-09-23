---

title: "多 Agent 不是虚拟公司：从 Anthropic 五种模式看信息流怎么设计"
type: entity
created: 2026-07-04
updated: 2026-09-23
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 多 Agent 不是虚拟公司：从 Anthropic 五种模式看信息流怎么设计

**来源**: 架构师

**发布日期**: 2026-04-18^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


**原文链接**: https://mp.weixin.qq.com/s/fMPSK00Lxb0uv90sun_BYQ ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

---


架构师（JiaGouX）

我们都是架构师！

架构未来，你来不来？

这几天 Anthropic 发了一篇文章，专门讲多 Agent 协调模式。^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


标题看起来很朴素：《Multi-agent coordination patterns: Five approaches and when to use them》。五种模式、适用场景、什么时候该从一种演进到另一种。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

如果只把它当成一篇“多 Agent 五种架构总结”，其实有点可惜。^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


我读完更强烈的感觉是：Anthropic 真正想提醒的，是另一件更工程化的事：^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


多 Agent 的难点不在于给模型分配几个角色，而在于把任务拆到合适的上下文边界里，再让信息流、验证和停止机制能稳稳接住。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

这和最近社区里很流行的一类说法正好相反。^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


很多多 Agent 教程喜欢把系统画成一个 虚拟公司 ：一个 Agent 当产品经理，一个当架构师，一个当开发，一个当测试。图很好看，故事也很好懂。CrewAI、MetaGPT 这类框架之所以积累了大量用户，很大程度上就是因为这个直觉太好解释了。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

但有意思的是，Anthropic、OpenAI、Google 三家在构建自己的生产级 Agent 系统时，没有一家采用“虚拟公司”模式。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

Anthropic 用的是 orchestrator-worker 并行探索，OpenAI Codex 靠的是 spec 文件 + skills + compaction，Google Gemini CLI 走的是 Conductor 扩展 + 持久化 Markdown 文件。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

三家的做法虽然不一样，但都没有出现“PM Agent 交给 Dev Agent 再交给 QA Agent”这种流水线。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

这或许不是巧合。

LLM 系统真正卡住的地方，往往不是“岗位不够齐”，而是上下文丢失、信息传递失真、验证标准模糊、循环不收敛。 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

所以今天不打算把 Anthropic 的文章只是翻译一下。我更想借它来理清一个思路：^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


多 Agent 架构不是组织架构，首先是信息架构。^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]


## 太长不看版

- • Anthropic 总结了五种多 Agent 协调模式：生成-验证、编排-子 Agent、Agent 团队、消息总线、共享状态。

- • 这五种模式表面是在讲“Agent 怎么协作”，底层其实是在回答三个问题：上下文边界怎么切，信息怎么流，系统什么时候停。

- • 大多数团队未必需要一上来做复杂的“虚拟公司式 Agent 团队”。从单 Agent 或编排-子 Agent 开始，通常更稳。

- • Generator-Verifier 的关键不是“两个 Agent”，而是可检查的验收标准。没有标准，验证 Agent 只是一个更贵的橡皮图章。

- • Orchestrator-Subagent 适合短任务、独立探索和清晰边界。Claude Code 的 subagent 就是典型例子。但它会遇到信息瓶颈。

- • Agent Teams 适合长期、独立、可分区的任务，比如大代码库迁移。前提是任务边界足够硬，否则多个 worker 会互相打架。

- • Message Bus 适合事件驱动流水线。

^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

→ [[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计|原文存档]] ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]

---

## 深度分析

### 五种模式各自解决什么问题

Anthropic 的五种模式表面是协作方式分类，实质是一条按瓶颈演进的复杂度路径，每一级都在解决上一级暴露的信息流缺陷 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

- **生成-验证（Generator-Verifier）**：解决"输出错一次代价很高"的问题。它的核心不是两个 Agent，而是可执行的验收标准（rubric、测试集、迭代上限）——没有标准的 verifier 只是更贵的橡皮图章，且需要最大迭代次数和兜底策略防止循环卡死 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。
- **编排-子 Agent（Orchestrator-Subagent）**：解决"支线探索污染主线上下文"的问题。Claude Code 的 subagent 就是典型：子 Agent 在独立上下文窗口里做搜索和调查，只把压缩后的结论带回来，主 Agent 保持对整体目标的连续掌控。瓶颈是信息必须经过编排器，子 Agent 之间的中间发现要传几轮后细节会被摘要掉 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。
- **Agent 团队（Agent Teams）**：解决"一次性 subagent 无法积累领域上下文"的问题。worker 持久存在、跨多轮任务沉淀上下文（如大代码库迁移中每个 worker 负责一个服务）。前提极硬：任务必须能稳定分区，且需要独立分支/worktree、清晰 ownership、合并前冲突检测等机制，否则多个 worker 会互相打架 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。
- **消息总线（Message Bus）**：解决"编排器 if-else 膨胀"的问题。事件驱动的发布-订阅让新 Agent 只需订阅 topic 即可加入，适合来源多、工作流不固定的场景（如安全运营分诊）。代价是可追踪性：事件级联难以调试，静默失败（错分路由不报错）比崩溃更危险 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。
- **共享状态（Shared State）**：解决"中心协调者成为吞吐瓶颈"的问题。多 Agent 读写同一持久化存储，发现实时流通，适合协作研究。风险从工程层（并发写入、版本覆盖）升级到行为层：Agent 互相响应形成不收敛的循环——这不是加锁能解决的，必须预先设计终止条件（时间/token 预算、N 轮无新增即停、judge Agent） ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

### 为什么"虚拟公司"类比误导

"虚拟公司"类比把多 Agent 架构讲成组织架构，但人类的岗位分工源于人的限制（注意力有限、专业切换成本高、需要接口和会议协作），LLM 的限制完全是另一组：关键上下文没带进来、中间推理被压缩成结论后失真、任务目标在传递中漂移、验证标准太抽象、Agent 互相响应烧 token 但不收敛 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

更具体地说，"虚拟公司"模式按角色接力传递的是**结论而非推理过程**：Agent A 产出文档交给 B，B 重新理解、重建上下文，原始意图衰减、隐含假设丢失，每次传递累积误差，工作流越长输出越"局部正确但整体漂移"。人类组织里跨边界的发现（写代码时发现需求有问题、测试时发现架构不稳）靠经验和非正式沟通补回来，而按角色接力的 Agent 系统恰恰把这些发现压掉了 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

佐证是三家厂商的生产级系统都没有采用这个模式：Anthropic 用 orchestrator-worker 并行探索，OpenAI Codex 靠 spec 文件 + skills + compaction，Google Gemini CLI 走 Conductor + 持久化 Markdown——没有任何一家出现"PM Agent 交给 Dev Agent 再交给 QA Agent"的流水线 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

### 上下文窗口经济学

Anthropic 在自己的 Research 系统里验证过一个关键结论：**多 Agent 带来的性能提升，80% 可以用 token 消耗量来解释**。也就是说多 Agent 的价值主要不是"分工更合理"，而是用更多 token 覆盖了更大的搜索空间——它更适合广度优先的并行探索，而不是模拟人类组织的职能接力 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

这个视角也解释了为什么窗口变大不等于什么都该塞进去：subagent 本质是受控的上下文隔离工具，问题不是"这段支线信息能不能装下"，而是"它该不该污染主线" ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

### 过度通信 vs 通信不足

多 Agent 信息流设计的两极失败模式值得对照：

- **通信不足**：按角色接力的系统传递失真的结论，跨边界的关键发现被压掉；编排模式里子 Agent 的中间发现需要绕回主 Agent 再转发，几轮摘要后细节丢失，依赖关系被切断 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。
- **过度通信**：共享状态模式下 Agent 互相响应彼此的补充，双方都觉得自己在推进，系统却不收敛——表面都在工作，实际只是持续烧 token；消息总线的事件级联则让"什么都没处理"的静默失败藏在日志之下 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

真正的选型问题因此不是"我要几个 Agent"，而是"这个任务的信息依赖结构是什么"：上下文边界怎么切、信息要经过谁、系统什么时候停 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

## 实践启示

**按任务形状选模式，而不是按"高级程度"选**。五种模式是积木不是等级表，越往右不代表越高级；正确的路径是先用最简单能跑通的模式，看它在哪里撑不住，再按瓶颈往上走。观察到的信号→模式的映射：输出错一次代价高且标准可写清→生成-验证；子任务短、边界清楚、最终要统一综合→编排-子 Agent；子任务长期独立、需要积累领域上下文→Agent 团队；工作流由事件触发且类型不断增加→消息总线；Agent 需要实时共享中间发现→共享状态 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

**编排-子 Agent 模式下限制 token 扇出**。由于性能提升 80% 由 token 量解释，orchestrator-worker 设计必须主动管控扇出：明确每个子任务的输出契约（只交回压缩后的结论而非过程），限制并行子 Agent 数量和每轮迭代上限，把"子 Agent 只需交回最终结论"作为保持编排模式的前提——一旦子 Agent 需要边做边互相影响，就已经在接近需要换模式的信号 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

**大多数场景下，单 Agent + 工具就够了**。设计前先回答 7 个问题：任务是否真的超出单 Agent 的上下文或搜索能力？子任务是独立还是强依赖？每个 Agent 的上下文边界是什么？中间发现只回主 Agent 还是实时共享？什么算完成、能不能写成可检查标准？循环怎么停？失败时回滚、重试、降级还是交给人？这些问题回答清楚后，多数团队会发现：一个单 Agent 加好的状态文件、测试、权限、日志和任务边界，就能解决大部分问题。三家的实践（claude-progress.txt、Codex runbook、Conductor 的持久化 Markdown）共同验证了同一原则：推理链的关键节点必须外化到持久存储，而不是依赖 context window "记住" ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

**把角色拆回能力节点**。"测试工程师 Agent"应该拆成一条可复用的 test-triage 工作流：找测试入口、区分失败类型、缩小复现范围、映射回变更文件、给出修复建议。Codex 官方 skills 的命名（test-triage、render-debug、packaging-notarization）指向的不是"一个人"而是"一类可反复处理的问题"——persona 只管语气和立场，真正托住结果的是能力节点、协调模式和运行时约束 ^[raw/articles/多-agent-不是虚拟公司从-anthropic-五种模式看信息流怎么设计.md]。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

