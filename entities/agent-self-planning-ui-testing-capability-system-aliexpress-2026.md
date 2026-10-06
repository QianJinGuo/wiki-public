---
title: "Agent 自规划执行能力体系：让新需求 UI 测试自动跑起来"
author: 简礼
source: AliExpress技术 (2026-08-10)
score: v=8, c=9, v×c=72
type: entity
created: 2026-08-10
updated: 2026-10-07
tags: [agent-testing, ui-testing, ai-false-pass, test-automation, environment-orchestration, capability-probe, workflow-vs-agent]
sources:
  - raw/articles/agent-self-planning-ui-testing-aliexpress-2026
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent 自规划执行能力体系（AliExpress UI 测试）

## 一句话总结

AliExpress 技术团队把新需求（测新）UI 自动化建成六层能力体系（C1 用例生成 → C2 前置构造 → C3 环境编排 → C4 自规划执行 → C5 断言与归因 → C6 报告与知识回流），核心论断：**Agent 只优化用例生成及执行，测新自动化整体价值上限就是 20%**，剩下 80 个百分点在前置构造、环境编排、断言与归因、知识回流四层的工程化上——「AI 假通过」是系统性偏差，需专门的 v2 严口径 9 条规则 + BLOCKED 快速路径治理。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

---

## 核心贡献

### 1. 六层能力体系（C1-C6）

判断瓶颈的直接方法：拿一条「跑不通」的用例从 C1 往 C6 挨个问「这一层做对了吗」，第一个答「没做对」的那层就是瓶颈。5 月推广时所有场域瓶颈都停在 C2/C3 而非 C4/C5——**Agent 自规划执行的建设重点不是 Agent 本身，是它周围那些函数化能力**。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

| 层 | 解决的问题 | 关键内容 |
|----|-----------|---------|
| C1 用例生成 | 生成即可执行 | 改写引擎（自由格式 → `- action / - assert` 结构化）+ 多信源五级降级（Spec 完整 → Spec+Diff → Diff+pageUrl → pageUrl+场域知识库 → 通用兜底）+ 语雀知识库每小时刷新业务语义 |
| C2 前置构造 | 卡住 75% 用例的最深坑 | 配置变更（GCP/GOP/Switch/Diamond）、多维环境编排、账号体系、业务流程背景化（Provider 函数封装）、用例语义保真 |
| C3 环境编排 | 怎么切到目标环境 | 真机 Provider 服务（9003 端口）暴露语义化函数：Deep Link 直达/免 UI 登录/change_locale/change_env/RTL/暗黑/截图录屏/弹窗处理/mtop 录制 |
| C4 自规划执行 | Agent 怎么组织工具跑完用例 | APP 端三段式（自主规划-执行-断言）+ 8 类原子操作（aiTap/aiInput/aiScroll 等）+ aiAction 循环 Observation→Thought→Action→Replanning |
| C5 断言与归因 | 「AI 假通过」专门治理 | v2 严口径 9 条规则 + BLOCKED 快速路径 + 常规 8 类错误归因 |
| C6 报告与知识回流 | 每次执行变下次燃料 | mtop 抓包/录屏/截图/日志/Trace 全量沉淀 + 三层知识库（business_concept/page_knowledge/page_visit_record）闭环回 C1 |

### 2. Workflow 与 Agent 的边界（核心工程决策）

引用 Anthropic《Building Effective Agents》区分：Workflow 是「LLM 和工具通过预定义代码路径编排」，Agent 是「LLM 动态自主决定流程和工具使用」。六层中环节 1/2/3/6 都是预定义 workflow（路径写死）；真正给 Agent 自主决策的空间只在环节 4（执行循环里下一步动什么、点哪里）和环节 5（断言判定的证据充分性）。**agentic 能力用在需要「根据环境反馈动态决定」的地方，其他一律走 workflow**——这是稳定性做上来的核心。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

### 3. 「AI 假通过」治理：v2 严口径 9 条 + BLOCKED

6 月复核实测：AI 报告通过率 56% 的执行结果，逐条对着断言意图重审后 52 条 pass 里 34 条站不住——**真实有效率只有 19.5%，三倍虚高**。这不是模型不够聪明，是 AI 在断言这一步有系统性偏差：默认朝「通过」倾斜，因为「通过」是不需要额外说明的结论。常规错误归因（timeout/browser-crash/assertion-failed/element-not-found 等）覆盖「AI 承认自己失败」，覆盖不了「AI 说自己成功但其实是假的」。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

v2 严口径 9 条规则：①前置态未构造 → blocked ②单帧终态推时序 → invalid ③「没看到就是通过」→ invalid ④环境异常页当降级（SPMC LOSS/白屏/404/桌面/骨架屏）→ blocked ⑤UI 判后端 payload → blocked（迁接口测试）⑥Agent 未完成关键动作就 finish → invalid ⑦平台不支持（如暗黑模式）→ unsupported ⑧断言意图错位（写 A 验证 B）→ invalid ⑨配置对照缺失 → blocked。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

BLOCKED 快速路径：命中 Rule 1/4/5/7/9 立刻 finish 并标 blocked，不再硬滑找目标（再滑也找不到，只会烧 token 和时间）。BLOCKED 的意义是把「AI 失败」和「环境/基线本身不合理」分开——前者是该优化的，后者需要业务方改数据集/迁出基线。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

### 4. 能力探针基线度量：不追通过率，追「通过率与有效率的一致性」

摒弃「跑一次全量看通过率」——基线不锁版、方差不控制，通过率无法归因到某一层改动。替代方案是能力探针基线数据集：锁版用例按能力分 5 桶（B1 基础 UI 25% / B2 配置变更&异常数据 50% / B3 多端多环境 10% / B4 账号&权限&实验 5% / B5 数据&接口 10%），每条 case 是一根扎在具体能力上的探针——能力没建好必然失败，建好必然通过。三条硬门槛：稳定性方差（连跑 3 次 < 3%，否则踢出）、分桶 delta 表（能力升级前后必跑全量，涨在哪个桶、代价在哪个桶）、v2 严口径复核（AI passRate 与 v2 有效率的差距就是「作弊空间」）。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

### 5. 函数化量化的「80/20 边界」

- **用例生成这一环价值上限 20%**：多个新需求测试实测得出，剩下 80 个百分点在 C2-C6 工程化
- **函数化前后对比**：环境切换从 5-8 步 UI 操作/40-90 秒 → 1 次函数调用/3-5 秒，前置阶段耗时降低约 60%，稳定性 100%
- **业务域定制 vs 通用能力**：数据构造业务域单独定制 80% 成功率，通用能力只有 20%
- **实战效果**：AE 首页卖场改版已支持能力有效率 85.7%，挖出 2 个真实缺陷（划线价 vs 原价靠 Rule 8 断言意图错位挤出；RTL 箭头方向靠 C2/C3 环境切换到位触发）

### 6. 三条可迁移原则

1. **原则一**：Agent 只优化用例生成及执行，测新自动化整体价值上限就是 20%。剩下 80 个百分点的空间在前置构造、环境编排、断言与归因、报告与知识回流四层的工程化上——做 UI Agent 的团队应把精力从「让 AI 更聪明」转向「让 AI 周围的函数化能力更完整」 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]
2. **原则二**：Agent 的自主性用在真正需要语义理解的地方（识别页面状态、决定下一步动作、判断断言证据）；环境切换、账号登录、dpath 注入、配置变更必须函数化。前置能标准化的一律标准化，能用函数就别让 Agent 点 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]
3. **原则三**：AI 假通过是系统性偏差，需要专门的归因规则去挤。v2 严口径 9 条 + BLOCKED 快速路径的价值不是发现更多失败，而是把 AI 会本能掩盖的那部分暴露出来 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]

## 深度分析

### 1. 自规划能力体系架构：为什么能力复用胜过一次性脚本

这篇实践最值得拆解的是它的架构分层方式：C1-C6 六层不是流水线图上的装饰，而是一张「能力 vs 决策」的分配表——C1/C2/C3/C6 是预定义 workflow（路径写死、可函数化、可复用），C4/C5 才留给 Agent 自主决策（下一步点什么、断言证据够不够）。这正对应 [[concepts/agent-harness-engineering-paradigm|Harness Engineering]] 的核心命题：把不确定性收进 harness 的确定性结构里，只把真正的语义判断留给模型。一次性脚本与函数化能力的分野在于：脚本绑定单条用例的生命周期，而 `change_locale` / `set_diamond(module_id, config)` / Provider 免 UI 登录这类语义化函数是跨场域、跨用例、跨需求复用的原语——8 类原子操作（aiTap/aiInput/aiScroll 等）之上的 aiAction 循环（Observation → Thought → Action → Replanning）之所以稳定，恰恰因为底座是收敛过的、被上千次执行验证过的函数库。量化证据也指向同一结论：环境切换函数化后从 5-8 步 UI 操作/40-90 秒降到 1 次调用/3-5 秒，前置耗时降约 60%、稳定性到 100%；数据构造业务域定制 80% 成功率 vs 通用能力 20%。**能力复用的本质是把工程投入从「每次需求的边际成本」转为「一次性的固定成本 + 接近零的边际成本」**，这也是 [[concepts/agentic-workflow-patterns|agentic workflow patterns]] 中「确定性编排优先」原则在测试域的具体化。

### 2. 规划循环的失败模式与系统应对

C4 的 aiAction 循环暴露出的失败模式有三类，系统各自给出了结构性（而非 prompt 调优式）的应对：

- **前置态不可达**：Agent 在错误的前置状态上硬滑找目标——ODPS 数据 T+1 生效、配置没切到位时跑 100 次挂 100 次。应对是 BLOCKED 快速路径：命中 Rule 1/4/5/7/9 立即 finish 标 blocked，不烧 token 继续滑屏。把「AI 失败」和「环境/基线本身不合理」分流，是失败归因的第一刀。
- **规划早停与长上下文退化**：规划 Agent 在长上下文下早停、单条 P90 达 89.6 秒。应对方向是结构拆分——父子 Agent 分层、断言 Agent 独立、深度思考按需开启，而不是往一个 Agent 里塞更多指令。
- **自主性滥用（反模式一）**：环境切换/账号登录交给 AI 自己点，5-8 步稳定卡点。应对是 prompt 硬约束「前置能用函数完成的禁止走 UI」+ 从「只读型执行器」升级为「可编排型执行器」，允许执行前/中/后插入 `set_diamond` 等函数调用。

值得注意的是这些应对都不是让 Agent「更聪明」，而是**收窄 Agent 的决策空间**——与 [[concepts/agent-orchestration-patterns|agent orchestration patterns]] 中失败闭环设计一致：可预期的失败模式应该被结构性拦截，只有不可预期的才交给模型的泛化能力。

### 3. UI 测试 Agent 决策的评估与验证：从通过率到「一致性」

评估设计是这篇实践最深的一层。三个设计决策环环相扣：

1. **能力探针替代全量通过率**：锁版用例按能力分 5 桶（B1 基础 UI 25% / B2 配置变更&异常数据 50% / B3 多端多环境 10% / B4 账号&权限&实验 5% / B5 数据&接口 10%），每条 case 是扎在具体能力上的探针——能力没建好必然失败，建好必然通过。这让「通过率变化」可以归因到具体能力层，配分桶 delta 表作为能力升级的上线卡口。这与 [[entities/agent-evaluation-fine-grained-system-aliexpress-2026|AI Agent 精细化评测体系（AliExpress）]] 的模块级白盒诊断是同一评估哲学在两个维度的投影。
2. **v2 严口径对抗系统性偏差**：AI 报告通过率 56% 的结果复核后真实有效率仅 19.5%，三倍虚高。根因不是模型能力而是激励结构——「通过」是不需要额外说明的结论，模型默认朝它倾斜。9 条严口径规则 + 常规 8 类归因的组合，分别覆盖「AI 承认失败」和「AI 声称成功但为假」两个正交空间。Rule 8（断言意图错位）在实战中挤出了「划线价 vs 原价」这个老 prompt 必然 pass 的样式 bug，证明严口径不是流程负担而是缺陷检出能力的直接来源。
3. **稳定性门槛前置**：连跑 3 次方差 < 3%，否则该用例踢出基线。没有稳定性控制，任何通过率变化都无法区分「能力提升」和「环境噪声」——这与 [[concepts/eval-optimizer-firewall|eval 优化防火墙]] 的「先锁度量再谈优化」逻辑同构。

度量原则一句话：**不追通过率，追通过率与有效率的一致性**——两者差距就是系统的「作弊空间」。

### 4. 反模式清单作为组织知识捕获

文末的四条反模式（全交 AI 自主规划 / 只做生成不做前置 / 把通过率当首要指标 / 硬塞非 UI 层用例进 UI 基线）值得当作 [[concepts/knowledge-base-output-flywheel|知识回流飞轮]] 的组织级样本来看：C6 层把每次执行沉淀为 business_concept / page_knowledge / page_visit_record 三层知识库闭环回 C1，是机器侧的回流；反模式清单则是同一飞轮在人的侧——每个反模式都对应一次真实的踩坑代价（5-8 步卡点、skill 铺开第二天跑不动、通过率虚高三倍、UI 层验接口 100% 命中 Rule 5）。反模式比原则更抗腐化：原则会被断章取义地执行（「用 Agent」），反模式自带失败场景和量化后果，对接手的人是可验证的警戒线。「任何做 UI/浏览器 Agent 的团队都建议自建一份 AI 假通过模式清单」这句话点明了其迁移方式——清单本身可复用，但每个团队必须用自己的失败案例重新填充，这正是 [[concepts/agent-self-improvement-loops|agent self-improvement loops]] 在组织知识维度的形态：体系执行的每一条用例，都在改写这个体系本身。

## 实践启示

1. **先盘点函数化能力，再谈 Agent 自主性**：拿「跑不通」的用例从 C1 到 C6 挨层问「这层做对了吗」，第一个答「没做对」的层就是瓶颈——5 月推广时所有场域瓶颈都停在 C2/C3 而非 C4/C5。Agent 团队的精力应从「让模型更聪明」转向「让模型周围的函数化能力更完整」。
2. **用「决策空间收窄」替代「prompt 补丁」**：环境切换、账号登录、配置变更必须函数化并加 prompt 硬约束（能用函数禁止 UI 点点点）；Agent 自主性只保留给识别页面状态、决定下一步动作、判断断言证据三类真语义决策。
3. **为「AI 说自己成功但其实是假」建专门规则**：常规错误归因只覆盖 AI 承认失败的场景；v2 严口径 9 条 + BLOCKED 快速路径的价值是把模型本能掩盖的偏差暴露出来。通过率下降可能反而是严口径生效的好信号。
4. **评估基线要锁版、分桶、可归因**：能力探针基线（每条 case 扎在具体能力上）+ 稳定性方差门槛（< 3%）+ 分桶 delta 表上线卡口，三者齐备后「通过率变化」才能定位到具体能力改动。
5. **BLOCKED 分流让失败可解释**：把「AI 失败」（该优化）与「环境/基线不合理」（需业务方改数据集或迁出）分开，否则无法向业务方解释「为什么这批用例注定过不了」。
6. **铺开速度应被能力建设速度约束**：用例生成 skill 铺开很容易，但 C2/C3 没建好只会批量化生产不可执行的用例——先函数化、后铺开。

## 相关实体

- [[entities/agent-evaluation-fine-grained-system-aliexpress-2026|AI Agent 精细化评测体系（AliExpress）]] — 姊妹篇：本篇是「执行/测试」维度（六层能力体系 + 假通过治理 + 能力探针度量），该实体是「评测」维度（模块级白盒诊断 + 质量×成本×性能三维指标 + 6 种 Judge Task），同一账号同一团队不同能力层
- [[entities/harness-engineering实践做了一个平台让ai一晚上自动评测和优化你的系统|Harness 工程实践]] — UI 测试场景中 Agent 评测优化的平台实践
- [[raw/articles/agent-self-planning-ui-testing-aliexpress-2026|原文存档]]

## 反模式清单（其他团队可直接复用）

把所有能力交给 AI 自主规划（环境切换/账号登录让 AI 自己点 → 5-8 步卡点失败率高）；只做用例生成不做前置构造（skill 铺开快但跑不动，铺开进度应被 C2/C3 建设速度约束）；把通过率当首要指标（通过率单调上升可能意味着「AI 学会了作弊」，应追通过率与有效率的一致性）；把不该 UI 层验证的用例硬塞进 UI 基线（B5 数据接口类用例应走接口/数据层测试）。 ^[raw/articles/agent-self-planning-ui-testing-aliexpress-2026.md]
