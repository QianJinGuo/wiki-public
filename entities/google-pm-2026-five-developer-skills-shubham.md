---

title: "谷歌PM公开：2026开发者五大新技能——问题塑形/上下文设计/审美/编排/判断力"
created: 2026-05-28
updated: 2026-11-24
type: entity
tags: [problem-shaping, context-curation, taste, agent-orchestration, judgment, developer-skills, google, shubham-saboo, 93k-stars]
sources:
  review_value: 7
review_confidence: 7
review_recommendation: worth-reading
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---
# 谷歌PM公开：2026开发者五大新技能——问题塑形/上下文设计/审美/编排/判断力
## 核心命题

谷歌高级 AI 产品经理 Shubham Saboo（93k 星 Awesome LLM Apps 仓库维护者）公开核心洞察：**2026 年最优秀的开发者更像电影导演，而不是程序员**。价值从"实现能力"迁移到"上游能力"——问题塑形、上下文设计、审美、智能体编排、判断力。五种能力决定谁能真正用 AI 做出产品。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

## 背景数据

- 时间窗口：Gemini 3 Pro → Claude Opus 4.6（2025年11月 ~ 2026年2月，不到 3 个月）
- "6 个月前招人的技能标准，现在已毫无价值"
- 实习生把问题定义清晰 + 让 Claude Code 执行 → 交付速度超过 3 天手写代码的资深工程师
- 高效开发者只用 10% 时间 coding，70% 精力用于拆解问题

## 被商品化的四项旧技能

1. **从零开始写代码**：智能体写得更快、Bug 更少 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
2. **样板代码和项目脚手架**：一句提示词直接生成 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
3. **死记硬背语法和 API**：超长上下文窗口已解决 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
4. **把规格说明翻译成代码**：规格本身就是代码 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

> "这些能力之所以曾经重要，是因为实现本身很难。它们支撑了六位数年薪。但实现已经不再是瓶颈。"

## 五种新技能

### 1. 问题塑形（Problem Shaping）

把模糊目标变成可执行的任务。**区分"玩 AI"的人和"用 AI 做产品"的人**的核心能力。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


**核心**：拆解能力。"帮我做一个仪表盘"不是任务，是愿望。问题塑形把它拆成十二个具体、可测试的子任务，每个都有明确的成功标准。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

**示例**：一个人负责问题塑形、十六个智能体负责执行 → 产出 10 万行可运行的 Rust 代码。那个人没有写代码，但把问题拆解到足够精确，智能体仅凭拆解结构就能完成一个编译器。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

**变化**：当"问题定义得好"与"产品真正上线"之间的时间差压缩到几小时，问题定义的质量决定一切。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


### 2. 上下文设计（Context Curation）

智能体产出的质量，与你提供的上下文质量直接成正比。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


**差上下文**： ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

> Build me a customer support agent.

**好上下文**： ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

```
Target user: SaaS customers who are frustrated and considering canceling.
They've already tried the help docs. They're messaging because docs failed them.
Tone: Empathetic but efficient. Don't over-apologize. Don't be robotic.
Here are 3 real tickets that got 5-star ratings: [examples]
Here are 2 that got complaints: [examples]
Edge cases requiring human handoff: Billing disputes over $500, Account security concerns, Legal or compliance questions
Success metric: Resolution without escalation in under 4 messages.
```

**本质**：选择什么信息进入模型的思考空间。给错信息，模型稳定输出错误方向；给对信息，它稳定产生接近产品级结果。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

**实践**：维护 CLAUDE.md、.cursor/rules、GEMINI.md，让智能体一开始就理解产品世界观。第一版输出达到 90%，而不是 50%。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

### 3. 审美（Taste）

**定义**：在东西尚未存在之前，就知道该做什么。是在十个选项摆在面前时，知道其中九个不行。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


**案例**：AI 讨价还价模拟器（双智能体围绕二手车交易对弈）。第一版代码干净没有报错，但界面只是普通聊天窗口，没有人格张力，没有戏剧瞬间。"它作为软件是成立的，作为体验是失败的。" ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

**智能体的局限**：智能体能构建任何你描述的东西，但无法判断什么值得被描述。智能体优化"正确性"，人优化"会不会有人克隆这个项目"。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

**培养方法**：回顾最近五个智能体产出，逐个写下你会改什么以及为什么。那个"为什么"就是审美正在形成。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

### 4. 智能体编排（Agent Orchestration）

知道什么时候用一个智能体，什么时候用多个；什么时候并行，什么时候串行；什么时候加护栏，什么时候放手；什么时候亲自调试，什么时候让智能体自己排错。 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

**三种核心模式**： ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


1. **串行流水线**：A 完成任务后把输出交给 B，适用于步骤之间存在依赖关系 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
   - 例：研究智能体收集数据 → 分析智能体解读 → 写作智能体生成报告

2. **协调者 + 专家团队**：主智能体负责分派任务并整合结果，适用于需要质量控制的复杂任务。协调者会在专家偏离方向时重新提示，并最终合并输出 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

3. **并行执行 + 合并**：多个智能体同时处理独立任务，最后由一个智能体整合。适用于子任务无依赖的场景，市场调研、竞品分析可并行，过去一个下午的串行工作现在只需几分钟 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

**关键判断**：知道何时放手让智能体调试，何时介入。盲目信任和全部手动，代价一样昂贵。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


### 5. 判断力——知道什么时候不用智能体

**核心**：不是每个问题都需要智能体。有些问题只需要一个快速模型和清晰提示。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


**例子**： ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

- 重新格式化 JSON → 丢给 Gemini 3 Flash
- 十个文件里的文案替换 → 轻量模型几秒搞定
- 一个你已经完全理解的 Bug → 自己改比向智能体解释更快

**框架**： ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

- 问题模糊、多步骤、需要探索解空间 → 用智能体
- 问题简单、定义清晰、答案已知 → 用快速模型
- 问题显而易见 → 用自己的手

**最高效的人并不是凡事都用智能体，他们在合适层级调用合适工具。** ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


## 培养方法

1. **培养审美**：回顾最近五个智能体输出，写下会改什么以及为什么 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
2. **优化上下文**：为当前项目写 CLAUDE.md，哪怕只花 30 分钟 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
3. **练习问题塑形**：面对模糊需求，在提示之前先拆成 10 个子任务 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
4. **练习编排能力**：把串行工作流拿出来，看哪些步骤可以并行 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]
5. **校准工具判断**：连续一周记录哪些任务用了智能体而简单提示就够 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]

## 结论

> "过去二十年里重要的技能之所以重要，是因为实现本身很难。而现在，实现已经不再困难。真正留下来的，是所有更上游的能力。"

打开你最近一个项目问问自己：你花更多时间在写代码，还是在塑造问题？如果答案是写代码，那你仍在用旧时代的技能结构。**新的方式，从一份上下文文档和清晰的问题定义开始。代码，会自己出现。** ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md]

## 深度分析

### 1. 价值迁移的经济学：实现成本归零后，稀缺性整体上移

这条判断的本质是经济学命题：当智能体把"实现"的边际成本压到近零——写代码、脚手架、语法记忆、规格转译四项旧技能被商品化——开发者的稀缺租金就整体迁移到上游五项能力 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:25-30]。实习生反超资深工程师的例子常被误读为"AI 更快"，真正的信息是瓶颈已从打字速度移到问题定义质量：把问题定义得足够清晰、让 Claude Code 收尾的一方赢得交付竞赛，与工龄和资历无关 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:32]。"电影导演而非程序员"因此应读作分工结构变化——人收缩到创意与决策层，智能体承担执行层，两层的技能清单完全不同 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:21-23]。

### 2. 问题塑形与上下文设计：模型外与模型内的同一套功课

两者作用于不同边界，常被混为一谈。问题塑形在模型之外工作：把"做一个仪表盘"式的愿望拆成十二个可测试的子任务，使执行者仅凭拆解结构就能工作——一人塑形、十六个智能体执行、产出十万行可运行 Rust 编译器的案例，证明拆解精度可以整体替代个人编码能力 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:36-42]。上下文设计在模型之内工作：决定哪些信息进入模型的思考空间，差上下文是一句愿望，好上下文则包含用户画像、语气约束、正反例与人工升级边界 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:44-62]。维护 CLAUDE.md 一类常驻上下文文件之所以值得，在于它把第一版输出从 50% 抬到 90%——在多轮智能体工作流里首版质量是复利基数，后续每一轮修正的成本都以此为底 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:65-67]。

### 3. 审美：目标函数错位使它无法委托给智能体

五项能力中唯有审美不可外包，根源是目标函数错位：智能体优化"正确性"（代码干净、无报错），而产品成败往往取决于"是否值得被描述"。讨价还价模拟器第一版软件成立、体验失败，说明正确性指标捕捉不到体验缺陷 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:73-75]。"十个选项里知道九个不行"揭示审美的作用方式是排除而非生成——这正是智能体架构的盲区：它们擅长枚举可行解，无法预判哪个解没有市场 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:71]。回顾五个产出、写下会改什么以及为什么的培养法，实质是把隐性偏好显性化的刻意练习，让排除标准可积累、可复用 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:77]。

### 4. 编排与判断力：把智能体当作需要预算管理的资源

编排回答"如何组合"，判断力回答"是否启用"，合起来是一套资源分配框架。三种编排模式的选择变量是任务间依赖结构：有依赖走串行流水线，复杂任务用协调者加专家团队，独立子任务走并行加合并——并行化的收益量级是把一个下午压缩到几分钟 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:79-91]。判断力则是成本校准的三级阶梯：模糊、多步、需要探索解空间的问题才配得上智能体；定义清晰的简单问题用快速模型；显而易见的问题用自己的手 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:93-105]。这本质是验证成本经济学——只为不确定性付溢价；"盲目信任与全部手动代价一样昂贵"意味着最优点在两个极端之间且随任务移动，放手与介入的配比本身就是要持续校准的参数 ^[raw/articles/google-pm-2026-five-developer-skills-shubham.md:91]。

## 实践启示

### 1. 投资系统设计能力而非 AI 工具熟练度
工具会变，系统思维不变——学习如何设计 AI-native 系统比学习特定 AI 工具更有长期价值。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


### 2. 审查和验证是 AI 时代的核心技能
AI 产出量大但需要验证——审查、测试、验证 AI 产出的能力比生成能力更重要。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


### 3. 跨领域沟通能力价值上升
AI 可以写代码但无法理解业务需求——开发者作为技术和业务的翻译者，价值上升。 ^[https://mp.weixin.qq.com/s/L_uAB0NjfgIuvlDIrP4TYQ]


## 相关主题
- [[entities/agent-orchestration]] — Agent 编排模式（AWS Step Functions 控制平面）
- [[entities/claude-code-governance-soft-rules]] — Claude Code 可控性设计
- [[entities/openai-skills-shell-compaction-agent-primitives]] — OpenAI Skills/Shell/Compaction 三位一体

→ [[raw/articles/google-pm-2026-five-developer-skills-shubham|原文存档]]
