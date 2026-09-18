---

title: "竞赛编程Agent进入全球前十！南大、清华新模型CF rating超3500"
created: 2026-07-08
updated: 2026-09-19
type: entity
tags: [agent, coding-agent]
sources: [raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500, raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500-2026-07-08]
confidence: 0.64
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 竞赛编程Agent进入全球前十！南大、清华新模型CF rating超3500

## 摘要

Solvita 是南京大学、清华大学等机构提出的竞赛编程 **Agentic Evolution** 框架：不微调底层大模型，而是在 Planner、Solver、Oracle、Hacker 四类 Agent 外部构建可训练的图结构知识网络，让「解题—认证—攻击—修复」形成持续积累经验的闭环。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

在 CodeContests、APPS、AetherCode 及 12 场 post-cutoff Codeforces 真实比赛（76 题）上，15 个 backbone-benchmark 组合中 14 个取得最高 pass@1；GPT-5.4 / Claude Opus 4.6 / DeepSeek V4 Pro 版本的 Solvita 均进入 **Legendary Grandmaster** 区间（CF rating 超 3500）。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500-2026-07-08.md]

## 核心要点

- **四角色闭环**：Planner 抽象题目并选策略，Solver 生成代码与局部修复，Oracle 构造可靠内部测试，Hacker 主动攻击候选程序。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
- **不微调、只长经验**：底层 LLM 冻结，增益全部来自外挂 agentic loop 与可训练知识网络，可与任意 backbone 叠加。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
- **三层可训练知识网络（Q/M/S）**：Q 存题目与元信息，M 存解法分解与失败对比，S 存可复用算法技能与 C++ 模板；新题沿 Q→M→S 两跳激活技能，边权按成功/失败更新——成功强化，失败削弱或派生 contrastive 节点。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
- **Patch-based repair**：失败后产出 SEARCH/REPLACE 局部补丁而非整份重写，同预算下通过率更高、迭代更少、token 更省。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
- **Oracle 与 Hacker 互补**：Oracle 保守（reference solver / validator / checker），Hacker 激进（semantic / stress / antihash），二者互补。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
- **收益随数据量增长**：无训练闭环已显著优于 single-pass，叠加知识网络后继续提升（1.5k→4.5k 题收益递增），token 消耗仍与开源 framework 相当。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

## 深度分析

### 四角色闭环：求解者必须有一个攻击自己的对手

竞赛编程不只是「把题面翻译成代码」，一个正确解法要同时完成题面理解、算法选择、复杂度估计、代码实现与边界排查。Solvita 把这些环节显式拆给四个角色：Planner 将原题面转成形式化描述，抽取变量、约束、目标与输入输出结构，并预测算法标签与复杂度；Solver 依策略生成 C++ 程序并在样例与 Oracle 测试上验证；Oracle 不写最终答案，只构造「可信监督」；Hacker 像对拍高手，分析漏洞并选择攻击路线。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

让闭环真正闭合的不是「多了几个 Agent」，而是失败证据的回流：被 hack 到的 bug 不只用于当前题修复，还作为失败经验传播给四个角色各自的知识网络；某条攻击路线失败时，系统沿 fallback chain 继续尝试。相比 AlphaCodium、MapCoder 等多阶段固定流程，它补上的正是随经验更新的长期记忆与路由机制。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500-2026-07-08.md]

### 可训练图结构知识网络：把记忆从检索变成路由

每个 Agent 都配有可训练的 graph-structured knowledge network，边权由 pass/fail verdict、测试认证质量、adversarial vulnerability 等反馈信号更新。它与传统 RAG 有本质差异：RAG 更像「找到相似文本塞进 prompt」，Solvita 学的是「什么题目结构应路由到什么算法技能」；把失败编码成节点与边权之后，经验才成为可累积资产。对比 [[concepts/agent-memory-architecture|Agent 记忆架构]]，重点不在存储容量，而在记忆本体的**可训练性**：它不再是静态上下文，而是一层可被信号优化的策略组件。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

### 为什么竞赛编程是 LLM 的照妖镜

竞赛编程能暴露问题，是因为正确性判定被外部化了：模型的自我报告不算数，隐藏测试说了算。LLM 的失败因而高度集中——同一类问题在不同约束下可能对应完全不同的算法，仅按表面相似度检索样例容易选到「看起来像但本质不对」的套路；样例测试远远不够，边界条件、复杂度极限、多答案 checker 都很难靠自测覆盖。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

因此在算法任务上，**验证能力**与生成能力同等稀缺。Oracle 与 Hacker 的对偶正为此而设：Oracle 先构造 reference solver、generator、validator 与 checker，要求 reference solver 复现公开样例后才批量认证测试；Hacker 则寻找边界输入、复杂度极限、结构性反例与哈希冲突。二者结合后在错误解法检测、正确解保留与 stronger-test confirmation 上取得更好平衡，即**假阳性与假阴性控制的双向夹逼**，也与 [[concepts/verifier-driven-development|验证器驱动开发]] 和 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 的可验证信号路线呼应。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

### 证据：14/15 最优 pass@1、CF rating 超 3500 与局部修复

真实比赛评测选取 12 场 post-cutoff Codeforces rounds 共 76 题，在官方时间限制内完成且不允许赛后修改。论文另对比 Solver 的 full regeneration 与 patch repair：相同最大迭代预算下，patch repair 通过率更高、迭代更少、token 更省——长链路解题中「推倒重来」并不总是好策略，候选解法往往大部分逻辑已正确，缺的是精准修补。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

其意义在于把性能来源从「更大的模型」部分转移到「更好的经验组织方式」：backbone 不变，靠 agentic loop、知识网络与对抗验证拿到跨区间跃升。对 AI for Code 而言，这是一条从一次性代码生成走向持续进化代码智能体的路径。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500-2026-07-08.md]

## 实践启示

1. **验证挪到模型外部并设门槛**：reference solver 能复现公开样例、认证条件满足才接受测试。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
2. **配一个专职攻击者**：让独立角色构造边界与压力样例，比求解者自评可靠，并避免单条路线失败即停。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
3. **失败后局部修补**：用 SEARCH/REPLACE 补丁保留已满足的约束，同预算下提高通过率并省 token。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
4. **记忆要可训练**：以「题目结构 → 解法分解 → 可复用技能」分层图结构，用 pass/fail 与认证质量更新边权。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]
5. **先建闭环再谈训练**：无训练闭环已显著超越 single-pass，知识网络是可叠加的第二层增益且随数据量增长。^[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500.md]

## 相关实体

- [[concepts/coding-agent-architecture|Coding Agent 架构]]
- [[concepts/agent-role-specialization|Agent 角色专门化]]
- [[concepts/agent-self-improvement-loops|Agent 自我改进回路]]
- [[concepts/agent-memory-architecture|Agent 记忆架构]]
- [[concepts/verifier-driven-development|验证器驱动开发]]
- [[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528|Agent Lightning]]

→ [[raw/articles/竞赛编程agent进入全球前十南大清华新模型cf-rating超3500|原文存档]]
