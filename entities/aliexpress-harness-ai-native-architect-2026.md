---
title: "从 Architect 视角看 AI-Native 落地：AliExpress Harness 生成能力建设"
type: entity
created: "2026-08-03"
updated: 2026-10-09
tags: [wechat, harness, ai-native, code-generation, d2c, spec-driven, evaluation, self-evolution]
rating: v8c9
confidence: 0.85
provenance_state: extracted
sources:
  - raw/articles/aliexpress-harness-ai-native-architect-2026-08-03
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 从 Architect 视角看 AI-Native 落地：AliExpress Harness 生成能力建设

**来源**: AliExpress技术（喜橙）

**发布日期**: 2026-08-03

**原文链接**: https://mp.weixin.qq.com/s/UbF04NBK6jccBpAeLr5InQ

## 摘要

AliExpress 团队在 Harness Engineering / Context Engineering / Loop Engineering 范式升级背景下，将 H5 B/C 端 Harness 生成能力从 Spec Coding + LangGraph 升级为「七层工程基础设施 + 四层确定性模型」双层架构，并在 Harness 之上重点建设**标准化、评测、自进化**三个能力。核心洞见：评测的断言依据、最优路径判定、标准的延续性与可解释性，是 Harness 落地绕不开的问题。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

## Harness 架构

**/ait 生码 command 管线**：init（扫描工作区/docs/环境校验）→ clarify（PRD 结构化 + 页面 spec + 组件 spec 并行 + 执行计划）→ generate（plan.yaml 驱动脚手架+模块+页面代码）→ docs（知识库同步）→ evaluation → verify（发布卡口）→ publish → evolve（session 归因 + harness 工程优化 PR）。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

**双层能力**：

- **七层工程基础设施**（Agent "能跑起来"的基座）：规则、记忆、上下文、执行、循环、反馈、恢复
- **四层确定性模型**（Agent "做得对"的业务层）：针对生码场景编排的确定性约束 ^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

## 三大核心能力

### 1. 标准化（领域适配层决定产出是否"对"）

标准模块 = **UI 契约 × 数据契约 × 功能契约**三位一体的最小封装单元。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

- **UI 标准化**：设计-代码协同组件契约体系。组件库两层（基础组件契约层进映射表 / 业务组装组件模板层不进映射表）；双轨制（存量直出 componentKey 硬绑定 / 自定义生成评估后沉淀回组件库）
- **数据标准化**：接口标准化（命名即架构归属）+ 场景数据协议（表格固定 list/total/pageSize/current 等预设字段）+ 字段标准化（三类独立注册/推导规则）+ **13 个一级领域治理体系**（Owner 负责注册审批、业务能力聚合原则、领域确定后归属自动推导）
- **功能标准化**：原子级功能组件（埋点/国际化/加购/收藏封装 SDK）+ 复合功能三层 Spec 结构化澄清（PRD Spec"要做什么"/Tech Spec"怎么做"/Data Spec"数据怎么流"，内置填空结构防自由发挥）^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

### 2. 评测（生成的代码怎么保证是对的）

**5 个评测维度**：结构正确性（tsc --noEmit + DevServer + 无 console.error）、数据完整性（AST 扫描字段引用 vs Data Spec diff）、逻辑正确性（逐条对照 PRD 功能点+条件分支覆盖）、UI 保真度（Chrome DevTools MCP 截图 vs Figma AI 视觉对比 + DOM 嵌套 + Figma JSON 间距校验）、E2E 功能（MCP 自然语言驱动测试 + Observation-First AI 视觉断言）。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

**客观性保障机制**：

- 检测与修复分离（Phase 1 SubAgent 只报告不修复，报告写独立文件）
- 多 Agent 交叉验证（logic-checker 与 data-checker 问题交叉印证）
- 不可篡改检测结果（evaluation/{dimension}/report.json 写入后不可改）
- 人工兜底（verify 阶段系统给 verdict，人做最终决策）

**4-Gate 自修复流程**：Gate1 能跑（编译+启动+无 crash，≤3轮）→ Gate2 数据足（字段完整+接口对齐，≤2轮）→ Gate3 用法对（Props/Hooks 合规，≤2轮）→ Gate4 功能通（E2E 全过，≤2轮）。核心约束：**前置 Gate 未通过时后置 Gate 结果无效**——先解决结构性问题再处理语义性问题。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

### 3. Evolve 自进化（修过的 bug 下次不再出现）

三阶段：发现问题（Python：修复记录→重复模式→跨生成聚合→噪声过滤）→ 定位根因并修复（LLM：分析根因→判断修哪里 Skill/Spec/Rules→生成补丁）→ 审查合入（Python+LLM+人：格式校验→LLM 评审 ≥85 分→MR 人工确认）。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

关键设计决策：

- **修复方向优先级**：skill > spec > hooks > rules > knowledge（修 Skill 执行步骤比加 rule 更彻底）
- **永不降级**：证据不足时暂缓（deferred），不写没人用的 knowledge
- **消费保障**：每条新知识必须有明确消费者（某个 Skill 会读它），无消费者 = 拒绝合入 ^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

## 效果数据

- **C 端 D2C 导购**：商品类模块 97.2 总分（UI 97.0/功能 96.5/数据 98.4）、瀑布流 95.5、券类型 96.3、玩法类型 93.4。目标：简单模块一次准确率 98% 交付 2h，复杂模块 95% 交付 4h
- **B 端 AIT 中后台**：简单页面（≤1 万行）~90 分、复杂页面（>1.5 万行）~80 分。目标：95% 交付 4h / 90% 交付 8h ^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

## 过程思考与判断（Architect 视角）

- **三角平衡**：生成准确度/效率/用户体验不可同时最大化——先保准确度（生成就能上线）再追效率，体验由标准化组件库兜底。"能生成"不是结束，"能上线"才是
- **资产 vs 补丁**：保留跨范式资产（标准化体系/领域知识库/Spec-Driven/端到端评测/上线卡口）；丢弃模型增强后消解的补丁（自建 RAG 检索链路/LangGraph 多轮决策树/过度设计智能路由矩阵）
- **可动/不可动边界**：影响生成质量的规则不可动，服务端实现可动——共识起点是让步边界清晰可见
- **真实 case**：早期 d2c Agent 简单模块跑 50 分钟，排查发现 1424 行 SKILL.md 无用加载，实际 LLM 生成仅 94 秒（90% 流程无用）——系统天花板不在模型能力而在架构设计
- **架构师定位**（阿伦特 Labor/Work/Action 三层）：AI 替代 Labor、参与 Work、无法触及 Action——架构师新定位是 Action 层"定义什么是对的 + 设计让对的事自动发生的机制"：判断/诚实/体感/发起/组织感知五种行为 ^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

## 深度分析

### 1. Architect 视角的 Harness 生成：组织级 AI-Native 的真实抓手

这篇最有信息量的不是某个具体技术组件，而是它的视角选择：以 Architect 而非工具用户的身份审视 Harness。当"定义什么是对的 + 设计让对的事自动发生的机制"成为架构师的新定位（对应阿伦特 Labor/Work/Action 三层中的 Action 层），Harness 就不再是工程脚手架，而是组织正确性的物化载体——13 个一级领域的治理体系、Owner 审批链、"命名即架构归属"，本质上是把组织内原有的架构治理权威重新编码成 AI 可执行的约束。这意味着 org 级 AI-Native 落地的最小闭环不是"选对模型 + 写好 Prompt"，而是组织先回答"谁有权定义正确"——这正是 Harness Engineering 范式下 [[concepts/harness-engineering-framework|Harness Engineering]] 与组织承载力相互约束的具体样本。标准化尺度不由技术完美性决定、由组织能承载的极限决定，这条结论值得所有 AI-Native 转型团队对表自查。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

### 2. 效果数据应该怎么读：97.2 分的边界条件

C 端商品类模块 97.2 分是全文最常被引用的数字，但拆开三个口径看更准确：其一，分数是"评测体系自评"的产物——UI/功能/数据三维都由同一套 Spec 体系定义断言依据，分数高同时说明"标准化的内卷化成功"，即评测断言与生成约束同源，存在自我确认风险；其二，模块类型分化明显——玩法类型模块 93.4（功能仅 90.5）显著低于商品类 97.2，说明标准化的边际收益集中在"高频、结构可枚举"的模块形态，复合交互型功能仍依赖三层 Spec 结构化澄清而非组件直出；其三，B 端复杂页面（>1.5 万行）~80 分与 95% 交付目标之间还有明显缺口，印证了"页面规模 × 契约复杂度"仍是当前 Harness 的真实瓶颈。横向对比可参考 [[entities/agent-evaluation-fine-grained-system-aliexpress-2026|AI Agent 应用精细化评测（AliExpress）]] 的分层评测视角，两者在"评测断言必须有独立来源"这一点上互补。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

### 3. 三大能力的依赖顺序：标准化才是第一性

文中将标准化/评测/自进化并列，但依赖关系是严格的单向链：评测的断言依据来自标准化定义的契约（没有 Data Spec 就没有数据完整性检查），自进化的修复对象是标准化产物（Skill/Spec/Rules），"消费保障"（每条知识必须有 Skill 读它）更直接把知识资产锚定在标准化结构上。因此三大能力的真实排序是 标准化 → 评测 → 自进化，跳过标准化直接建评测体系（很多团队的 AI 质量平台正是这个顺序）会陷入"断言无处安放"的困境。这也解释了为什么 4-Gate 自修复流程要强制"前置 Gate 未通过时后置 Gate 结果无效"——结构性问题（契约层面）永远优先于语义性问题（功能层面），与依赖链方向一致。对照 [[concepts/evaluation-harness-design|Evaluation Harness 设计]] 可以看到，断言来源问题在行业层面是共性难题，而非本文独有。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

### 4. Evolve 的克制：不写没人用的 knowledge

自进化部分最有价值的不是三阶段流程本身，而是两条反向约束："永不降级"（证据不足时暂缓）和"消费保障"（无消费者的知识拒绝合入）。这两条把系统设计从"知识越多越好"扭转为"知识少而准"——本质上是承认 LLM 自进化循环最大的风险不是修不动，而是错误知识被大规模再消费、产生级联污染。"修复方向优先级 skill > spec > hooks > rules > knowledge" 则隐含一个判断：执行层比知识层更容易验证对错，所以越靠近执行侧的修复越值得做。这与 [[concepts/agent-self-improvement-loops|Agent 自进化循环]] 中"证据门槛与消费者约束是自进化系统防污染的两大闸门"的结论互相印证。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

### 5. 开放张力与未回答的问题

- **范式淘汰的自反性**：文中把 LangGraph 多轮决策树、自建 RAG 归入"应丢弃的补丁"，但保留/丢弃的判定依据是"模型增强后是否自然消解"——这个判据本身依赖对未来模型能力的预测，与"资产 vs 补丁"二分法的确定性之间存在张力。判断力需要三个输入（对现有系统的理解 / 对新范式的认知 / 真实场景验证数据），第三个输入是回溯性的，前两个是前瞻性的，三者权重如何定并未给出答案。
- **50 分钟 vs 94 秒 case 的可复制性**：1424 行 SKILL.md 无用加载的问题暴露的是 Harness 自身的"上下文治理"欠账，但文中未给出 Harness 自我诊断的通用方法——这类"Harness 之上的 Harness"（对 Harness 做元评测）是否会成为下一轮建设重点，值得追问。参考 [[concepts/loop-engineering-methodology|Loop Engineering 方法论]]，循环反馈机制目前主要作用于代码生成结果，尚未作用于 Harness 自身的执行效率。
- **"生成就能上线"与人工兜底的边界移动**：verify 阶段始终由人做最终决策，这意味着当前准确率数据（97.2/95.5 等）全部停留在"人机协同后"的口径上。随着准确率提升，人的角色会从"兜底判断"退向"抽检 + 异常受理"，这个边界如何迁移、迁移到什么位置算"生产可用"，文中三角平衡（准确度/效率/体验）没有给出量化标准，是 org 级推广时必须先回答的问题。^[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03.md]

## 相关链接

- → [[raw/articles/aliexpress-harness-ai-native-architect-2026-08-03|原文存档]]
- 同源姊妹篇：[[entities/agent-evaluation-fine-grained-system-aliexpress-2026|AI Agent 应用精细化评测（AliExpress）]]、[[entities/global-product-center-qa-agent-aliexpress-2026|全球化商品中心智能答疑 Agent（AliExpress）]]
- 相关主题：[[entities/end-to-end-codingagent-design-taobao-subsidy-2026|端到端 CodingAgent 设计（大淘宝）]]、[[entities/gaode-autosdk-ai-native-pipeline-2026|高德 AutoSDK 全链路 AI Native（架构篇）]]、[[entities/skill-system-design-taobao-technology-2026|AI Agent Skill 系统设计（淘宝技术）]]
