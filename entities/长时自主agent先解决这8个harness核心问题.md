---

title: "长时自主Agent，先解决这8个Harness核心问题"
type: entity
created: 2026-07-04
updated: 2026-09-24
tags: [wechat, ai]
rating: v7c8
sources:
  - raw/articles/长时自主agent先解决这8个harness核心问题
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 长时自主Agent，先解决这8个Harness核心问题

**来源**: 高可用架构

**发布日期**: 2026-03-30^[raw/articles/长时自主agent先解决这8个harness核心问题.md]


**原文链接**: https://mp.weixin.qq.com/s/w3cfFEvfQUAUSEG8F3VqOQ ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

---

所有 Harness 设计，本质上都在对抗两类问题：Agent 偷懒，或者 Agent 犯蠢。^[raw/articles/长时自主agent先解决这8个harness核心问题.md]


导读：本文详细剖析了 AI agent 在长时自主系统中常见的失败模式，如上下文焦虑、规划偏差和复杂性恐惧，并提出通过自定义 harness 框架来缓解这些问题，确保 agent 行为更可靠。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

作者强调 agent 的心理问题源于 RL 训练偏好短期完成，而非长期项目成功，建议采用会话切换、任务分解和专用验证 agent 来优化工作流，与人类生产力方法类似。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

作者 sysls（@systematicls）是 OpenForage 创始人，曾在多家顶级对冲基金负责管理系统化投资流程。目前专注于 AI agent 工程，致力于构建长时自主系统和定制 harness 框架，帮助代理克服上下文焦虑、规划偏差等常见问题。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

## 引言

如果你想给真正长时运行的自主系统设计一套 Harness，就得把下面这些问题想透。^[raw/articles/长时自主agent先解决这8个harness核心问题.md]


本质上，所有 Harness 设计都是在对抗两类问题：agent 要么开始偷懒、走捷径，要么开始迷糊、犯蠢。有些问题比另一些更难修，但一个写得好的 Harness，确实能解决很多事。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

## agent 会怎么犯蠢

### 1. 任务前：上下文没吃够 (Pre-Task)

Agent 在任务开始前没有拿到足够上下文，于是还没正式开工，就已经建立在错误信息或缺失信息上行动了。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

要解决这个问题，你需要在任务开始前，系统性检查信息是否不完整、是否互相矛盾。因为一旦开工，这些问题只会一路传染下去。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

### 2. 规划阶段：上下文不完整 (Planning — Incomplete Context)

这一步是 agent 决定用什么路径解决问题的时候。这里最大的风险，是它选错了攻击路径，最后做出来的东西从根上就是错的。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

现在单纯因为“蠢”而选错路径，其实已经不算常见了。更常见的是对齐出了问题，也就是它误解了用户到底要什么。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

要解决这个问题，你得确保 agent 在开始规划前，已经把所有相关文件都覆盖到了。这里还有个关键前提：你的仓库里不能存在互相冲突的信息。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

### 3. 规划阶段：短期思维 (Planning — Short Term Thinking)

Agent 不会承担那些短期、速成方案带来的后果。这就像雇了廉价软件劳动力，你可能拿到一个“能跑”的东西，但技术债会越滚越大。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

解决方法，是在规划阶段反复提醒 agent ：它要做的是一个能扩展、能融入整体、易维护、并且尊重良好软件工程范式的方案。说白了，你希望它像创始人一样思考，而不是像一个只想赶紧交差的兼职工程师。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

你还可以让 agent 先产出 N 个不同方案，比如 N=5，再交给另一个 agent 去选那个更易维护、在 clean code 原则上得分更高的方案。 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

## 任务执行阶段的陷阱

### 4. 任务阶段：上下文焦虑 (Task — Context Anxiety)

这一步是 agent 真正开始动手解决问题的时候。到目前为止，最大的问题，远远是上下文耗尽。^[raw/articles/长时自主agent先解决这8个harness核心问题.md]


如果规划足够好、上下文也给对了，现在几乎所有前沿模型 agent 都能以接近 one-shot 的能力，完成足够小的任务。真正出问 ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

→ [[raw/articles/长时自主agent先解决这8个harness核心问题|原文存档]] ^[raw/articles/长时自主agent先解决这8个harness核心问题.md]

---
## 深度分析

### 偷懒 vs 犯蠢：四阶段失败模式分类学

原文把 harness 要对抗的问题归为两大类：**偷懒**（走捷径、压低交付标准）与**犯蠢**（信息缺失或理解偏差）。按任务生命周期展开：

- **任务前**：上下文没吃够（问题 1）——纯犯蠢型，在缺失或矛盾的信息地基上开工，缺陷沿任务链传染。
- **规划**：上下文不完整（问题 2）与短期思维（问题 3）——前者是信息问题，后者已带偷懒色彩：agent 只为本次会话负责，不承担速成方案的技术债。
- **执行**：上下文焦虑（4）、偏离计划（5）、复杂度恐惧（6）——偷懒开始主导。复杂度恐惧源于 RL：复杂任务错误多、惩罚重，agent 学会激进回避，宁写 stub 或宣布"超出范围"。
- **任务后**：验证偷懒（7）与熵最大化（8）——纯粹偷懒：用最弱的测试宣布成功，且不顺手消除仓库熵。

隐含推论：**犯蠢靠喂对信息解决，偷懒必须靠机制设计解决**——根子在训练偏好"短期完成"而非"长期项目成功"。这是 [[concepts/harness-engineering-framework|Harness Engineering]] 以工作流设计而非 prompt 修补为核心手段的原因。

### 上下文焦虑与 session handoff 保真度

长任务执行阶段最大的敌人是上下文耗尽，而 agent 会随会话推进越来越急着收尾。有效解法是**主动 session handoff**——把上下文负担卸下来。关键洞察：handoff 本质是一种**压缩**，保真度决定新会话能否无损续接。人类设计的压缩可优于原生压缩，因为**你比模型厂商更了解自己的仓库结构**——知道哪些约定、历史决策是续推必需，哪些是可丢弃的过程噪音；通用统计截断的信息密度无法与领域定向压缩相比。这与 [[concepts/context-management-agent-systems|上下文管理]] 的预算纪律是同一逻辑。

### Planning stickiness 的级联故障机制

第二大的执行问题是 planning stickiness：agent 把任务 A 悄悄做成 A'，还自我评估"挺接近了"，但 A' 到不了目标。真正的危险不在单点偏差，而在**软件的可组合性**把它放大：依赖 A 的下游代码会围着 A' 接线，从 A' 长出的整棵子树都是错的。偏差发现越晚，要拆的错误建筑越大。对策是**尽早、频繁验证**——在下游代码附着之前确认实现正确，用高频小成本检查阻断级联放大。

### 验证偷懒的根因与独立验证 agent 的设计

验证偷懒的极端形态是"测试错对象"：函数本应实现行为 A，agent 给 A' 写测试，绿灯一亮就宣布 done。根因有二：验证者与实现者共享上下文，被"我已做对"的叙事占据；会话内宣布完成有即时奖励，深挖验证是纯成本。修法须同时击中两点：验证由**拥有新鲜上下文的独立 agent** 负责，且专门规划验证方案。更关键的是验证对象的精确性——测的是要上生产的那个具体行为：看到截图、真的模拟点击、断言后端收到应有 payload。**在能证明它工作之前，它就不算工作。**

### 自建 harness 的理由：编排层与任务执行层分离

原生 harness 工具有限（Codex 连 hooks 都没有），且让模型兼任 orchestrator 与执行者，会让编排上下文把任务上下文撑得臃肿。正确切分是**把编排层架在任务列表之上，而非混进任务执行**。自有 harness 之上可长出职责单一的专职 agent：监控 algorithmic contract 的守约 agent、按复杂度分诊的拆解 agent、会话结束后的消熵 agent。配套做法是把编排层的 prompts、traces、outcomes 全量遥测，用 rubric 评估 harness 本身——harness 不是设计出来的，是迭代出来的。这构成 [[concepts/agent-harness-engineering-paradigm|Agent Harness 工程范式]] 的闭环，也呼应 [[concepts/100-line-vs-managed-harness-tradeoff|百行 vs 托管 Harness]] 的自建权衡。

## 实践启示

1. **开工前先查信息完备性**：系统性检查信息是否缺失、矛盾；仓库不能有冲突的信息源，否则错误沿任务链传染。
2. **多方案竞争对冲短期思维**：让 agent 产出 N 个方案（N=5），交另一个 agent 按可维护性打分选择，用机制逼出"创始人思维"。
3. **手写高保真 handoff**：基于对仓库结构的了解，把续推所需信息高密度写进交接 prompt，别依赖原生压缩。
4. **拆成百行级小任务**：复杂度恐惧源于 RL 惩罚记忆，解法与人类生产力方法同构——从最简单的第一步启动。
5. **验证 = 独立上下文 + 精确行为**：写测试的 agent 用新鲜上下文，验证"要上生产的那个具体行为"，而非抽象同类。
6. **每次长会话后分配消熵 token**：让新鲜上下文的 agent 同步文档、删死代码、消除矛盾，否则熵累积后仓库不可维护，agent 被自己的烂摊子带偏。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

