---
title: "ECC Continuous Learning：从工具调用轨迹到本能沉淀的持续学习闭环（homunculus 观察式本能提取）"
author: AliExpress技术
source: AliExpress技术 (2026-08-21)
score: v=8, c=7, v×c=56
type: entity
created: 2026-08-21
updated: 2026-10-03
tags: [agent-skills, continuous-learning, instinct, homunculus, tool-call-observation, self-evolution, hook, muscle-memory, ecc, memory, context-injection]
sources:
  - raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21
confidence: high
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# ECC Continuous Learning：从工具调用轨迹到本能沉淀的持续学习闭环

## 一句话总结

ECC（Everything Claude Code）的 continuous-learning 模块实现了**「homunculus 观察式本能提取」**：后台用 Hook 静默采集 Agent 每次工具调用的真实轨迹，从执行流中自动识别「用户纠正 / 错误修复 / 重复工作流 / 工具偏好」四类反复出现的行为模式，沉淀成带**触发条件 + 置信度 + 作用域**的「本能」文件，再通过聚类进化（evolve）与跨项目提升（promote）注入上下文，让 Agent 在**不改模型权重、不写手写规则**的前提下，仅靠上下文层闭环长出属于自己的「肌肉记忆」。 ^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

## 为什么它是独立维度（与既有自演化/技能进化机制的区别）

与已有的技能自演化机制相比，homunculus 的核心差异化在于**学习信号的来源与粒度**：^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- 既有机制（如 SkillOpt 的文档可训练、skill-self-evolution 的训练式进化）多在**模型/技能文档层面**优化，需要预先定义训练目标或验证集。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- homunculus 则在**执行轨迹层面**做无监督观察——只关心「Agent 实际做了什么」，不要求用户预先声明偏好，从零散的 tool_start/tool_complete 事件流里归纳模式，产出的不是参数或文档，而是**带触发条件和置信度的可注入本能**。 ^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

这一「观察执行轨迹 → 提炼本能 → 注入上下文」的闭环，在全库已有实体中零覆盖，构成不可替代的新维度。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

## 核心机制

### 四段闭环
1. **项目初始化**：Hook 配置（PreToolUse/PostToolUse 全工具匹配）+ 项目身份识别（按项目隔离数据）+ 进程守护（懒启动 + 30 分钟空闲自动回收）^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
2. **行为采集**：行为采集器把工具调用变成结构化观察记录（含敏感信息 `[REDACTED]`），追加写 observations.jsonl，攒够量发 SIGUSR1 信号^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
3. **模式识别**：两种唤醒（信号 + 定时）+ 多层门禁（ANALYZING 锁 / 60s 冷却 / guardian 三层门禁）+ 四类待识别模式 + 非交互式 LLM 子进程（采样 500 条，递归熔断）^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
4. **本能积累与管理**：本能文件落盘 + instinct-cli 生命周期（status/import/export/evolve/promote/projects） ^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

### 四类待识别模式
| 模式 | 触发特征 | 产出本能 |
|------|---------|---------|
| 用户纠正 | 刚 Edit 完紧接着又改成别的样子；报错后重试 | "做 X 时优先用 Y" |
| 错误修复 | tool_complete 报错 → 随后工具改好 → 再次成功 | "遇到错误 X，试试 Y" |
| 重复工作流 | 同一串工具序列反复出现 | "做 X 时按 Y→Z→W 步骤" |
| 工具偏好 | 工具选择规律 | "需要 X 时用工具 Y" |

### 本能文件的工程化设计
- **置信度**：按频次定初值（3-5 次=0.5 / 6-10 次=0.7 / 11+ 次=0.85），随时间 -0.02 每周衰减^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- **作用域**：project vs global，拿不准默认 project 避免污染全局^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- **触发条件 trigger**：本身就是「适用场景」的自然语言描述，可直接向量化做按需检索注入 ^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

## 上游发现 + 下游固化的分工

持续学习在上游负责**发现**（从行为数据捞无意识经验），记忆系统在下游负责**固化与加载**（借 Agent 原生静态记忆/自动记忆机制把验证过的本能注入上下文），形成「观察 → 提炼 → 沉淀 → 注入 → 产生新行为 → 再观察」的全自动闭环。这一「发现-固化」分工视角是对 Agent 记忆系统架构的补充。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

## 应用

- **沉淀个人隐形工作习惯**：固化无意识的探索顺序、试错路径与工具偏好^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- **上下文成本优化**：量化「哪些弯路最烧 token」，把最贵的弯路固化成本能后测算 token 消耗与步数下降^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- **业务规则浮现**：从反复调用序列提炼业务约束^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]
- **good case 评测基线**：高置信本能成功轨迹作 good case、反之 bad case，构建回归评测基线^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md]

## 深度分析

### 观察与执行的分离：学习信号源的重新定位

homunculus 最激进的设计选择，是把「学习」从对话层彻底移到执行层。传统 Agent 自我改进依赖用户显式反馈（"你做错了，应该这样"）或手写规则，而 ECC 的采集器只读取 PreToolUse/PostToolUse 的结构化事件——tool_name、tool_input、tool_response、session_id——完全不解析自然语言对话。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:34-49] 这意味着学习信号的粒度是「动作对」而非「语义」：一次 Edit 紧跟着另一次 Edit 修改同一处，就是一个用户纠正信号；tool_complete 报错后几个工具内再次成功，就是一个错误修复信号。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:64-71] 执行轨迹之所以比对话更可信，是因为用户"说"的偏好常常滞后或失真，而"做"的轨迹是无损的——这正是原文核心命题「真正决定效率的习惯从不出现在描述里，却清清楚楚记录在每一次执行轨迹中」的工程化落地。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:18-22]

这个分离还带来一个常被忽视的副作用：由于 Hook 异步执行、不阻塞主流程，学习成本被摊薄到近零——Agent 不需要为学习做任何额外动作，采集是纯旁路。对比需要专门训练回合或验证集的自演化机制（如 [[entities/skillopt|SkillOpt]] 在文档层面优化），这是「零打扰学习」与「回合式学习」的本质分野。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:25-31]

### 本能提取：从原始事件到可注入知识的两次抽象

原始 tool_start/tool_complete 事件流离「可用的知识」很远，ECC 用了两级抽象。第一级是**模式归类**：LLM 子进程采样最近 500 条观察记录，在四类预定义模式（用户纠正 / 错误修复 / 重复工作流 / 工具偏好）的约束下归纳，且同一模式须出现 3 次以上才成立——这里的关键不是让 LLM 自由发挥，而是用固定模式类别约束了输出空间，保证产出的本能格式统一、可比较、可合并。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:64-74] 第二级是**工程化包装**：每条本能带 id、trigger（自然语言适用场景描述）、confidence、scope、证据链条，置信度按频次定初值（3-5 次=0.5，11+ 次=0.85）并每周 -0.02 衰减。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:76-79]

置信度衰减是这个设计里最容易被低估的细节：它隐含承认了「本能会过期」——项目重构后旧的错误修复经验可能失效，工具链更换后旧偏好不再适用。让本能自带时效性，等于把记忆的遗忘曲线内置进了知识管理层，而不是依赖人工清理。作用域默认 project 而非 global，则是用保守默认值隔离污染风险：单项目的怪癖不该成为全局教条。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:79]

### 上游发现与下游固化：与记忆系统的分工而非替代

ECC 明确把自己定位在记忆架构的**上游**：持续学习只负责从行为数据里「发现」无意识经验，注入上下文的「固化与加载」交给 Agent 原生静态记忆/自动记忆机制完成，两者构成「观察 → 提炼 → 沉淀 → 注入 → 产生新行为 → 再观察」的全自动闭环。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:88-93] 这个分工视角补充了 [[concepts/agent-memory-architecture|Agent 记忆系统架构]] 常见框架里缺的一环：多数记忆系统设计讨论「存什么、怎么取」，却很少回答「记忆从哪来」——homunculus 的答案是：记忆不必由人写入，可以从系统自己的执行轨迹里长出来。

instinct-cli 的 promote 机制是这条闭环里「经验晋升」的具象化：同一 ID 的本能在 2+ 个项目出现且平均置信度 ≥0.8 才写入用户级目录，evolve --generate 则按触发条件聚类（2+ 成 Skill、3+ 且平均置信度 ≥0.75 成 Agent）。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:81-86] 这实际上定义了一条经验的政治晋升通道：项目级观察 → 跨项目验证 → 用户级本能 → 聚类成 Skill/Agent，每一步都有量化门槛，避免了「一次巧合被当成普适规律」。

### 注入策略：从常驻上下文到按需检索的演进空间

三种本能注入方式的排序本身就是一条演进路线：聚类成 Skill 是重封装（一次性迁移成本高但产物稳定），会话启动全量注入高置信本能是粗粒度兜底（简单但烧预算），而按场景检索动态注入是最理想的终态——因为 trigger 字段本身就是「适用场景」的自然语言描述，天然可向量化，每轮对话前按当前任务做相似度检索，只注入命中的少数几条。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:95-100] 这把本能从「常驻上下文」变成「按需加载」，与 [[concepts/context-window-economics|上下文窗口经济学]] 的预算逻辑一致：知识的价值不在于随时在场，而在于恰好被需要的时刻出现。

值得注意的工程护栏还有递归熔断：分析子进程携带特殊环境变量，被它触发的 Hook 立即退出，避免「分析行为本身又被观察、又触发分析」的无限递归——任何观察系统观测自己时都需要这一层自我排除。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:73-74]

## 实践启示

1. **别问用户要规则，去看执行轨迹**：用户说得清的偏好只是冰山一角，反复出现的工具序列、报错后的修复路径才是真实的工作习惯。给 Agent 加学习能力的第一个动作是装 Hook 采轨迹，不是写 CLAUDE.md。
2. **学习信号必须结构化到「动作对」粒度**：只记录「哪个工具、什么输入、什么结果」就足够识别四类模式，不需要理解对话语义。粒度越细、越结构化，后续模式识别越便宜。
3. **给知识设衰减和作用域，默认保守**：置信度按频次定初值、每周衰减，作用域拿不准默认 project——让经验自带时效性和边界，比事后人工清理便宜得多。
4. **经验晋升要有量化门槛**：单项目观察不等于跨项目规律，用「2+ 项目出现且平均置信度 ≥0.8」这类硬门槛控制 promote，用聚类阈值（2+ 成 Skill / 3+ 成 Agent）控制封装粒度，防止巧合升格为教条。
5. **注入方式按预算演进**：先用会话启动全量注入兜底跑通闭环，再向 trigger 向量化 + 按需检索迁移——把常驻知识变成按需加载，是上下文成本优化的主杠杆。
6. **观察系统必须自我排除**：任何「观察 Agent 行为」的子系统都会被自己的 Hook 捕获，递归熔断（环境变量标记 + Hook 立即退出）是这类系统的必备护栏，不是可选项。^[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21.md:73-74]

## 相关概念
- [[concepts/skill-engineering-principles|Skill 工程原则]]
- [[entities/skill-self-evolution-three-approaches|技能自演化三种路径]]
- [[entities/memento-skills-agent-self-evolving|Memento 自演化 Agent]]
- [[entities/skillopt|SkillOpt 技能文档训练]]
- [[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21|原文存档]]

→ [[raw/articles/ecc-continuous-learning-homunculus-observation-instinct-aliexpress-2026-08-21|原文存档]]
