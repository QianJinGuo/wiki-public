---
title: "FDE（Field Deployment Engineer）非共识与落地指南 — 硅谷一线实践者圆桌"
created: 2026-07-08
updated: 2026-10-24
type: entity
tags: [fde, field-deployment-engineer, ai-deployment, enterprise-ai, ai-engineering, tencent-research, harness-engineering, distillation, forward-deployed-engineer, fdx, forward-deployed-executive, determinism-uncertainty, product-boundary]
sources: [raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08, raw/articles/fde-forward-deployed-executive-fdx-infoq-2026-08-04, raw/articles/fde-industry-report-tencent-research-2026-08-05, raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# FDE（Field Deployment Engineer）非共识与落地指南

> 2026 年 6-7 月，AWS（$1B）、微软（$2.5B/6000 人）、OpenAI（$2.95亿收购 Tomoro）、Google Cloud、Anthropic 五巨头同时重注 FDE。腾讯研究院邀请 Cresta AI 的钟钱杰（Jove）和 Ventus AI 的陆骁鹏（Vincent）一线对话，拆解 FDE 的本质、打法与本土化可能。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## 什么是 FDE — 三种不同形态

同一个岗位名背后是三种截然不同的运转逻辑：^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

| 形态 | 代表 | 核心模式 | FDE 角色 |
|------|------|---------|---------|
| **SaaS 型** | Cresta（500人独角兽） | FDE 是产品的一部分，不背商业指标 | 三分之二时间落地，三分之一改产品代码；上午见客户，下午提 PR，明天上线 |
| **Startup 型** | Ventus AI（10人，a16z 投） | 从售前跟到售后，背商业指标 | 一个人当十个人使：客户成功+开发+交付+汇报 |
| **模型公司型** | OpenAI/Anthropic | 本质是想多卖 token | 集团作战，5-6 人伺候一个客户，偏驻场 |

## FDE 与外包/实施/SA 的本质分野

**钟钱杰的切割**：任何你能够教会客户的，就不是 FDE。SAP 实施可以教会客户，AI Agent 的幻觉和延迟偏偏教不会。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

**陆骁鹏的诚实**：在 Ventus 早期，FDE 就是解决方案加实施，从售前跟到售后。分歧不在名字，在公司阶段。当没有产品可改时 FDE 堆人头卖时间，跟外包没区别。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## 从 Demo 到生产：复杂度被严重低估

一个语音 AI Agent 可能同时跑 20 个模型：ASR、打断判断、噪音隔离、RAG、Tool Call、多模型并发 Guard Rail。典型节奏：**1 周走通端到端 → 1 个月写几千几万个测试**。这些测试不是 unit test，而是用历史通话训练小模型去模拟急躁的客户、牙齿掉了吐字不清的患者。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

> "不是客户没有工程师，是 AI Agent 的复杂度超出了传统工程团队的经验边界。"

行业合规（PCI/HIPAA）审计动辄半年一年，模型选择、低延迟、幻觉控制、多语言支持、知识库持续更新…复杂度的每一层都是 FDE 存在的理由。

## 「蒸馏」— FDE 商业模式成立的前提

如果 FDE 做完一个项目下一个又从零开始，跟外包堆人头没有区别。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

| 层面 | 蒸馏内容 | 复用方式 |
|------|---------|---------|
| **行业知识** | 保险/酒店/航空的词汇、流程 | FDE 做 5-6 个项目后驾轻就熟 |
| **工具链** | SDK、模板、CLI、Markdown | 封装成 agentic 工具，客户说"我想做什么"，背后自动调用最佳实践 |
| **Skill** | 高 GPU/Token 消耗的 AI 操作 | 蒸馏成 skill，客户或伙伴可直接使用 |

AI Coding 使蒸馏飞轮成为可能：听得见炮火的人能大刀阔斧改仓库，改完后产品更强，下一个 FDE 站得更高。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## Palantir 的 FDE vs 今天的 FDE — 已是两个物种

| 维度 | Palantir 时代 | 今天 |
|------|-------------|------|
| **产品话语权** | FDE 对 Foundry 话语权弱，"自己看文档去" | AI Coding 让 FDE 能迅速改产品 |
| **知识沉淀** | 难沉淀、难复用 | 蒸馏成 skill/CLI/Markdown，agentic 分发 |
| **供需关系** | 靠商务关系卖人头，签几年合同慢慢交付 | 大客户推力指数级上升，产品没到百分百，FDE 填缺口 |
| **动机** | 项目交付 | 模型公司：多烧 token / SaaS 公司：让产品更强 |

Cresta 还配备了 **20-25 人的 FDPM（Forward Deployed Product Manager）团队**，作为迷你的 CEO 聚焦商务与人际，FDE 作为迷你的 CTO 聚焦技术落地，形成互补。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## 什么样的企业才能推得动 AI

> **95% VS 5%** — 能推动 AI 的企业只有一个共同点：**老板自己得 AI native**。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

AI native 的定义：在任何事情上永远不要假设自己比 AI 更懂。先问 AI"你怎么看"，再去 iterate。有这种思维的老板在医疗行业最多 5%。

**策略**：先搞定标杆（那 5%），向下推。大部分人决策路径是"别人都用了我不用就落后了"。今年（2026）大爆发——没人愿意做最后一批被淘汰的人。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## 中国能跑通 FDE 吗？

核心障碍：**高客单价是前提**。国内老板常觉得"两三个人捯饬捯饬也能做出来"。钟钱杰判断：最怕的幻觉是老板有幻觉。陆骁鹏认为环境会慢慢变好，但 FDE 解决的是"怎么把标准方案真正用进企业"的问题，付费习惯需要培养——像腾讯视频培养用户付费用了十年。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## 招人：7000 份简历 → 20+ offer（录取率 ~0.3%）

**核心特质**：迷你 CTO，实干型，能写代码、能跟客户掰、非常靠谱。最喜欢招创过业的人或 founding engineer。技术背景要求：必须做过 AI Agent **且做过测试**。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

AI native 评分（1-10）：
- **钟钱杰：9 分** — FDE 是公司用 AI 最激进的人，"不用 AI 会死得很惨"
- **陆骁鹏：1 分** — 每个人效率高 10 倍，整个企业可能只高 1-2 倍。可观测性太差，不知道哪些事必须人做

FDE 面试关键：看你用 Claude Code 时，哪些东西是**反驳 AI** 的。人一定要比 AI 凶。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]

## FDX（Forward Deployed Executive）— FDE 的组织推动升级

FDE 解决技术差距，却往往距离决定预算、组织结构与企业文化的核心管理者太远。Techstars 创业导师 Rick Manelius 提出：FDE 是 AI 项目的"外置 CTO"，**FDX 是"外置 CEO"**——兼具技术、管理和组织推动能力的新角色。^[raw/articles/fde-forward-deployed-executive-fdx-infoq-2026-08-04.md]

### 为什么 FDE 不够了

大型企业软件采购周期 18-24 个月、每周期只引入一两款新工具，外部环境以季度/月度/周为单位变化，共同导致 **超过 90% 的企业 AI 项目未实现预期回报**。FDE 能看真实工作流、快速交付可运行成果，但企业要真正建立 AI 基础还需要最高管理层成为推动者——而高管因决策风险产生恐惧/不确定/怀疑，即使得到理想答案也未必采用，除非信任提供答案的人。^[raw/articles/fde-forward-deployed-executive-fdx-infoq-2026-08-04.md]

### FDX 能力模型

Manelius 将 FDX 类比十年前热门的"解决方案架构师"（技术 + 管理 + 创业三能力），AI 时代还需：快速适应不确定性的能力、AI-first 思维（先考虑 AI 能承担哪些工作）、亲自执行 + 协同教学 + 任务委派三合一。^[raw/articles/fde-forward-deployed-executive-fdx-infoq-2026-08-04.md]

### 市场信号

- **AWS**: 2026-06 投入 $1B 成立 Forward Deployed Engineering 团队，数千名工程师派驻客户
- **OpenAI**: FDE 建成独立大型组织，数十岗位覆盖全球，旧金山岗位 16.2-28 万美元年薪 + 股权，最高 50% 出差
- **Anthropic**: 2026-05 联合 Blackstone/Hellman & Friedman/Goldman Sachs 成立企业 AI 服务公司；与 UST 合作培训 2 万名员工
- **Palantir**: 2026 演示 AI FDE 自动化（写 AIP Logic 函数、创建评估、调试系统）——FDE 本身可能被 AI 部分自动化，但组织判断工作更稀缺
- **Provectus**: 2026-07 招聘 "Senior Product Owner/Forward Deployed Executive—AI Delivery" 首个 FDX 岗位
- **国内近似岗位**: 飞书"企业 AI 转型顾问"（最接近 FDX，从产品上线到效果回收全链路）、百度智能云智慧工业解决方案架构师、华为 AI 解决方案架构师（主体仍是技术架构师）

### 案例

- **Ben 案例**: 非技术创业者原计划 $30/小时外包两周交付 API→HTML 表格，Manelius 用 Claude Code 屏幕共享 2-10 分钟跑通，公司数天内转型 AI-first
- **Tyler 案例**: 一周内部署两个 OpenClaw（一个担任 CTO），产品交付速度 3-5 倍、团队缩减一半

Manelius 将 FDX 视为十亿美元级市场机会——AI 时代稀缺的不是只懂模型或只懂管理的人，而是能同时理解技术、业务和组织并推动实际结果的人。^[raw/articles/fde-forward-deployed-executive-fdx-infoq-2026-08-04.md]

## 深度分析

### 一、「可教性」是 FDE 的唯一定义轴，不是岗位名

把 FDE 与售前、实施、SA 区分开的不是职责清单，而是一个单一判据：**这件事能不能教会客户**。能教会，说明复杂度有界、知识可显性化，那就属于咨询或实施；教不会，才需要有人长期驻在边界上。钟钱杰的原话是「任何你能够教会客户的，就不是 FDE」。这个判据的好处是它不依赖公司规模、行业或产品形态，因此能在三种形态（SaaS 型、Startup 型、模型公司型）下都成立。

FDE 之所以是「翻译层」，是因为它同时翻译三样东西，而这三样恰好都不可教：**技术复杂度**（一个语音 Agent 背后 20 多个模型并发，模型版本差异、prompt 排版差异、Guard Rail 阈值，全是隐性知识）、**组织激励**（中层员工与被蒸馏的能力是互斥关系，他会本能地释放烟雾弹）、**语境默会**（tribal knowledge 写在人和人的默契里，不在文档里）。售前卖的是「能做什么」，实施卖的是「把已知方案搬过去」，FDE 卖的是「已经能跑，并且我替你扛下不可教的那部分」。这也解释了为什么模型公司做 FDE 会退化：它的动机是烧 token，没有意愿让前线反过来把产品改好，于是 FDE 只剩驻场服务的壳。

### 二、蒸馏经济学：只在「高毛利 × 高同构 × 中等复杂度」的格子里成立

FDE 的成本结构可以粗略写成：第 n 个同类客户的成本 ≈ 固定理解成本 ÷ n + 定制成本 ×（1 − 复用率）^n。Bob McGrew 给出的验收度量非常锋利：**第一个客户 10 人月，第十个同类客户还是 10 人月，说明模式没跑通**；通常七到八个同类客户后本体层稳定，交付成本才显著下降。所以「蒸馏」不是文化口号，而是这门生意唯一的分母。

它成立的三个前置条件缺一不可：

| 条件 | 阈值含义 | 缺失后的退化形态 |
|------|---------|-----------------|
| **客单价门槛** | 存在明确的最低金额线，低于此不做 FDE | 变成人头外包，margin 比外包更差（外包先付工时，FDE 按年付） |
| **行业同构性** | 同类客户数量足够摊平本体建设，行业词汇与流程可共享 | 每个项目从零开始，复用率为零 |
| **复杂度甜蜜区** | 客户自建成本 > FDE 报价，但复杂度又没高到只能自研 | 客户直接反问「我为什么不自招两个人还要考核你」 |

第三条正是国内最常听到的质问，也是模式边界的精确刻画：FDE 卖的不是工时，是 outcome；一旦客户相信自己能以更低成本达到同样 outcome，价格锚就塌了。反过来，只要复用率在上升，FDE 就会从人力服务变成产品的一部分——这也是 [[concepts/application-solidification]] 在组织侧的镜像：能力先以项目形态存在，再被固化成产品组件。[[concepts/ai-cost-optimization-framework]]

### 三、0.3% 录取率反推出的能力集，是「存量淘 founder」而非「培养」

7000 份简历到 20 多个 offer，这个漏斗的真正含义不是标准高，而是**市场上不存在可培训的供给**。被筛出来的画像有四个维度，且都是可验证的：工程纵深（必须真做过 AI Agent 且做过它的测试，用过 LangChain 不算）、端到端 drive（不等 PM 发 ticket，遇阻自己破）、客户对抗力（能跟对方技术负责人正面掰）、产品 sense（听到痛点当晚就能跨 repo 改掉）。

最锋利的一条面试判据是「你在用 Claude Code 时，哪些地方是在反驳 AI」。它把抽象的「判断力」变成了可观测代理：完全顺从 AI 的人产出上限等于工具，而能指出 AI 错在哪的人才有边际价值。这也解释了为什么偏爱创过业或 founding engineer 背景——不是履历偏好，而是在无 PM、无流程、无兄弟团队的环境里自组织能力的唯一可信证据。

对应到能力框架，FDE 几乎是把 [[concepts/agent-engineering-capability-map]] 里「场景定义 + 系统集成 + 评估验证」三段压进一个人身上，再用 [[concepts/agent-role-specialization]] 的反面来运转：早期公司必须一个人演完所有角色，晚期公司才用 FDPM/FDE 分工把「迷你 CEO / 迷你 CTO」拆开。因此 FDE 团队的扩张瓶颈从来不是需求，是供给——这也是为什么头部公司要靠拿奖、造品牌来扩大候选池。

### 四、Palantir 与 LLM 时代是两个物种：定制成本的三次坍缩

同名不同物种，差别不在方法论，而在三项成本的量级变化：

1. **改代码的成本**：Palantir 时代 FDE 对 Foundry 话语权弱，反馈被「你自己看文档去」挡回；AI Coding 之后，一个人可以同时改多个 repo，听得见炮声的人直接变成 PR。
2. **沉淀的成本**：过去经验只能停在个人脑子里，现在可蒸馏成 Markdown skill、CLI、模板，并由 agentic 工具分发——沉淀第一次具备了边际成本递减的属性。
3. **分发的成本**：客户说「我想做什么」，背后自动串起 skill 与最佳实践，客户自助而不必学几千个 API。沉淀物从内部资产变成增值包。

三者叠加，把「前线→产品」的反馈环从季度压到天，FDE 因此从成本中心变成产品迭代的**传感器**。但 Palantir 的 Echo/Delta 分化并未消失，只是价值重心上移：Delta（怎么做出来）被 AI 大幅压缩，Echo（该做什么、值不值得做）随 AI 变强反而更稀缺。这正好解释了 9 分与 1 分的分歧——钟钱杰在评执行层效率（团队确实是全公司用 AI 最激进的人），陆骁鹏在评组织层天花板（每人效率 ×10，企业整体可能只 ×1.5，因为缺少可观测层来判断哪些事必须人做）。

由此得到一个反直觉结论：**AI 越强，FDE 越贵而不是越便宜**。被压缩的是实现，被放大的是定义问题与承担后果——而后者无法被 token 化。

### 五、中国的结构性障碍与三种可行变体

障碍有三层，且互相加固：付费习惯（硅谷六到七位数美元年合同 vs 国内十万到百万人民币量级）、自建幻觉（「两三个人捯饬捯饬也能做出来」，最怕的幻觉是老板有幻觉）、组织阻力（中层抵触，无 C-level 支持几乎推不动）。指望客单价短期抬升是不现实的。

但中国有一个反向优势常被忽略：美国是「精密机器加装大脑」（ERP/CRM 已成型，AI 只需接入），中国的流程写在默契和人情里，本来就没有可加装的机器。这意味着中国客户需要的不是「优化既有流程」，而是「一边跑、一边修路」——这恰恰是 FDE 的原生形态，只是计价单位和交付形态必须换。[[concepts/enterprise-ai-adoption]]

三种已经跑出雏形的变体：

- **原厂打样 + 伙伴复制**：用少数标杆项目（工作营、双人 Echo+Delta 组合）把行业能力打成产品，再交给伙伴按行业复制，把 FDE 的人力密度换成渠道密度。
- **预制件自助**：把 FDE 经验预制成 Skill、连接器、行业模板与增值包，让有能力的客户自助，FDE 只处理预制件覆盖不到的边界——本质是把固定理解成本一次性资本化。
- **底座型分层**：按「确定性下沉、不确定性上浮」把规则/Workflow 沉进底座，只把真正不可穷举的余量交给 FDE，用架构降低每个项目的不确定性处理成本。

这三种变体的共同点是：都不试图让 FDE 变成可规模招聘的岗位，而是让**同一个 FDE 的可复用面变大**。这与「蒸馏」是同一件事的三种工程表达。[[concepts/agent-self-improvement-loops]]

## 实践启示

1. **创始人：把 FDE 编进产品预算，而不是销售成本。** SaaS 型 FDE 不背单客户商业指标、不向客户按人头收费，是「他有动机改产品」的制度前提；一旦开始按工时计价，前线反馈就会自动退化成交付验收。^[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08.md]
2. **创始人：先算清两条线再决定要不要设 FDE。** 一条是最低客单价线（低于此不接），一条是行业同构线（同类客户能否到七到八个以摊平本体层）。两条线任一不成立，就该用伙伴或自助产品替代 FDE，而不是先招人再找场景。
3. **工程负责人：把「前线问题→上线」的中位天数当作唯一核心指标。** 这个数字决定 FDE 是传感器还是外包：以周计是产品的一部分，以季度计就是人力服务。配套要求是每次上线必须有测试集增量，而不是只加功能。
4. **工程负责人：沉淀物必须落成三种可分发形态并进版本库**——recipe/CLI（确定性操作）、Markdown skill（需要烧 token 的判断）、模板/连接器（系统对接）。留在个人脑子里的不叫沉淀；没有进入仓库、无法被 agentic 工具调用的也不叫沉淀。
5. **企业买家：先用「可教性测试」决定买什么。** 如果内部团队两周内能学会并稳定运行，就别买 FDE，买工具；学不会的部分才是 FDE 的定价区。合同里应当写 outcome 与验收口径，而不是人天。^[raw/articles/fde-forward-deployed-executive-fdx-infoq-2026-08-04.md]
6. **企业买家：把 sponsor 层级当作项目第一风险项。** 只跟业务线负责人谈，项目会在中层被稀释；需要 CXO 级 sponsor，并为「被蒸馏的岗位」准备明确的转岗与激励方案，否则关键信息会被系统性隐藏。
7. **从业者：把「我反驳了 AI 什么」当成可积累的资产。** 实现能力正在被平权，稀缺的是定义问题、算出价值、并为结果负责。FDE 是通往创业的最短路径，但前提是每次交付都能留下可复用的预制件，而不是又打一场一次性的仗。^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

## 与现有知识体系的关系

- 与 [[entities/ibm-forward-deployed-units-ai-deployment]] 互补——IBM 的 FDE 是公司战略视角，本文是一线实践视角
- FDE 的"蒸馏"机制是 [[entities/loop-engineering-feedback-control-system]] 在组织层面的体现：经验从项目→可复用资产→产品
- FDE "听到炮声改产品"的模式是 [[entities/harness-engineering-practical-17ge-versus-6-subagent|Harness Engineering]] 的生产实践——前线和产品之间的超短反馈环
- "模型是最容易被替代的一层" 呼应 [[entities/agent-vs-workflow-control-continuum-framework]] 中护城河不在模型而在工程层
- 与 阿里巴巴 Harness 工程自主迭代 互补——Harness 工程在个体层面，FDE 在组织层面

---

→ [[raw/articles/fde-field-deployment-engineer-tencent-roundtable-2026-07-08|原文存档]]

## 腾讯研究院行业报告（2026-08-05 Supplementary）

腾讯研究院 12 章报告《前线共创，双向赋能》，系统补充 FDE 模式的全景数据与腾讯云实践：^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

### 行业数据

- 超过 60% 的企业 AI 试点停在实验阶段；Indeed 平台 FDE 职位 643→5330 条（+700%）；OpenAI 和 Anthropic 同日宣布组建部署团队；YC 上百家创业公司开始招 FDE（三年前几乎为零）^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

### 双向沉淀机制与本体层

- 面向客户：业务经验和流程规则 → AI 可调用的知识库/工作流/应用能力
- 面向平台：行业场景/系统接口/测试方法/失败案例 → 可复用产品组件
- 共同载体是**本体层**（结构化业务知识图谱），让同一行业不同客户共享对业务对象的理解方式
- 经济账（Bob McGrew 度量）：第一个客户 10 人月，第十个同类客户仍 10 人月 = 模式没跑通；七到八个同类客户后本体稳定，交付成本显著下降 ^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

### Echo / Delta 能力框架

- **Echo** 负责"该做什么"（场景定义/需求翻译/价值判断），AI 越强越稀缺
- **Delta** 负责"怎么做出来"（原型搭建/系统对接/测试验证），被 AI 工具大幅压缩
- "自然语言构建应用"门槛越低，FDE 价值反而越上升——从搭建者转为场景发现者、需求翻译者和方法教练 ^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

### 中国市场特殊性

- 美国是"精密机器加装大脑"（ERP/CRM 完善）；中国"对话驱动"（流程写在默契和人情里），更适合 FDE 陪跑"一边跑、一边修路"
- 挑战是经济性：硅谷六到七位数美元年合同 vs 国内十万到百万人民币量级；解法 = AI/平台降交付成本 + Skill/连接器/行业模板/伙伴生态提复用率 ^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

### 腾讯云实践（原厂打样、伙伴复制、客户自助）

- 教育：工作营陪跑模式，沉淀教育专属能力孵化 **LearnBuddy** 行业产品
- 传媒：Echo+Delta 双人组合（一人讲案例一人快速搭原型），印证"搭建门槛降维、业务理解升维"
- CodeBuddy/WorkBuddy 客户成功团队：决策层认知拉齐→工具驱动执行→商业结果验证全链路 ^[raw/articles/fde-industry-report-tencent-research-2026-08-05.md]

→ [[raw/articles/fde-industry-report-tencent-research-2026-08-05|原文存档（腾讯研究院报告，Supplementary）]]

## 确定性-不确定性架构：FDE 作为产品体系前的不确定性处理层（2026-09-04 Supplementary）

叶小钗从一场数字员工公司 FDE 负责人与产品负责人的争论出发，给出了 FDE 在 Agent 产品体系中的结构性定位，以及一套可落地的确定性与不确定性分工框架。^[raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04.md]

### FDE 与产品负责人的本质分野

产品负责人面对的是已经知道怎么稳定解决的问题；FDE 负责人面对的是客户确实有问题、但还不知道怎样能被稳定解决的部分。^[raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04.md] 例如客户说"要做经营分析 AI 智能化"不算需求只算想法——继续往下可能发现没有统一数据口径、没有经营分析 SOP、不同管理者关注点各异，企业要的甚至不是报告而是异常发现后的追问、归因和行动建议。FDE 先把模糊问题逐级拆清：客户目标 → 业务建模 → 现状梳理 → 问题诊断 → 优化方案 → 核心 AI 能力 → MVP，去现场把产品暂时无法理解的问题变成可被理解、验证、交付的结构。这就是"FDE 负责人敢说不用产品交付"的合理性来源——他首先负责客户目标能否达成。

### 确定性下沉，不确定性上浮

这是对 Agent 应用内部如何分配确定性与不确定性的架构原则：能写成规则的不要交给大模型，能固化成 Workflow 的不要让 Agent 每次重新规划，真正无法穷举、必须结合上下文动态判断的部分才交给 Agent。^[raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04.md] 现实业务从确定性高到不确定性高构成一条光谱：规则 → 程序 → Workflow → LLM 节点 → Agent → 业务专家。系统内部分工：规则/程序处理高确定性逻辑，Workflow 承接已理清的流程，Skill/LLM Node 承接局部判断与语义理解，Agent 负责目标驱动下的动态规划、跨流程组合和剩余不确定性。Workflow 与 Agent 是分层协作且动态演化的——业务理解加深后，原本交给 Agent 的判断会继续向下沉淀成 Skill、Workflow 甚至规则。

### 生产任务三分与可接受的不确定性边界

生产应用需要验收标准，但 AI 项目要面对的恰是不可穷举任务，往往只能做到风险可控。^[raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04.md] 传统软件"生产可用 = 结果基本可预测"，Agent 系统则是"生产可用 = 确定部分足够稳定 + 不确定部分有边界 + 高风险可发现和接管 + 结果可评估"，由此把 AI 生产任务分成三层：确定性任务（规则/程序/Workflow）、约束型任务（LLM Node 判断，有边界和评价机制）、探索型任务（Agent 动态规划多步探索）。三类任务无法用同一产品设计方法，硬塞进一套平台必然越来越复杂——这也是很多通用数字员工平台不好用的原因。

### 边交付边吃业务，业务长产品产品长底座

B 端 AI 产品采用"边交付边生长"模式：先基于理解最深的业务做出第一个成立的数字员工，放进真实客户发现产品边界，FDE 处理边界外问题，再把反复出现的共性认知吃回产品，一轮轮慢慢长。^[raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04.md] 底层 Agent 底座不用预先设计成万能平台，而是等越来越多数字员工和 FDE 项目出现相同工程问题后，再把权限/安全/企业集成/运行环境/状态管理/监控等公共能力沉到底座。整篇文章收敛为两句话：**确定性下沉、不确定性上浮**（Agent 应用内部设计），**业务长产品、产品长底座**（AI 公司产品体系生长路径）。这与上方"蒸馏"机制的思路同构——都是围绕"经验回流产品"的飞轮，但本文提供了更明确的可交付框架与"不确定性处理层"这一定位。

→ [[raw/articles/fde-product-boundary-uncertainty-ye-xiaochai-2026-09-04|原文存档（叶小钗，Supplementary）]]
