---

title: "万级实时推理的商品领域Agent实践思考和总结"
created: 2026-05-25
updated: 2026-09-19
type: entity
tags: [agent, ai-agent, e-commerce, taobao, function-calling, real-time-inference]
source: [[raw/articles/taobao-product-domain-agent-architecture]]
confidence: 0.75
review_value: 5
sources:
  - raw/articles/taobao-product-domain-agent-architecture
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 万级实时推理的商品领域Agent实践思考和总结

## 深度分析

本文来自淘天集团商品中心技术团队，详述商品域如何构建"事件驱动的Function-Centric Agent架构"，实现万级实时推理，覆盖亿级商品。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

商品理解业务早期沿用 T+1 离线批处理的生产思路，在商品卖点、规格等核心链路已取得不错的前台效果；但一旦追问"AI 化主线要储备什么基建、能否从离线推理走向实时推理、到 SKU 粒度是否撑得住、成本如何收敛"，批处理范式就触及天花板。业界几条主流落地路径各有边界：Prompt Engineering 难以承载复杂逻辑与多步骤决策；RAG 本质仍是"问答式"单轮交互、缺乏主动规划；Fine-tuning 成本高、迭代慢、无法动态调用工具；MaaS 让业务逻辑与模型强耦合；Function Calling 自身不包含状态管理、目标分解与长期记忆。文章的判断是，Agent 以"操作系统级"容器的形态整合上述能力——内嵌 Function Calling 完成系统集成、天然融合 RAG 机制、可结合微调模型但保持松耦合、具备目标驱动的动态推理与回溯。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

### 两层结构：workflow 编排层 × 统一能力供给层

架构被抽象为两层：上层是面向业务场景的 workflow 编排层，回答"要做什么"；下层是统一的能力供给层，回答"能做什么"，沉淀模型调用、检索、工具、记忆等原子能力。两层之间只通过抽象的 AIFunction 接口交互，因此上层不必知道下层用的是大模型、向量检索还是传统服务，下层也不感知具体业务场景，能力复用与场景演进因此解耦。框架选型上，团队 2025 年选择轻度"耦合"的 spring-ai-alibaba 构建商品域 Agent 应用，看重的是集团成熟的 Java 生态底座——引入一整套新技术栈会带来额外复杂度；后续演进方向是朝多应用、类微服务的形态走，并引入 deepeval 做评测、deepresearch 做事实性验证等开源组件。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

### 轻量 aiagentsdk：注解体系与链式调用规范

能力供给层靠一套轻量 aiagentsdk 声明与复用，注解包括 @AIWorkflow、@AIAction、@AIFunction、@AIParameter、@AIResult、@AIResultField。其中 AIFunction 的字段规范是整张骨架：name 是 function 唯一标识；description 是一句话能力描述且必需；parameters 通过 @AiParam 反射自动推导；returns 说明返回值类型与含义；expose 决定是否对外暴露；tags 给能力打上 llm、rag、tool、memory 等标签；sideEffect 标明是否有副作用；timeoutMs 给出推荐执行超时时间。换句话说，"是什么、要什么参数、返回什么、能否外露、有无副作用、最多跑多久"都被声明式地固化下来，上层的自动编排、权限治理与安全约束才有了依据。访问规范则按 {Registry}.{DomainRegistry}.{FunctionClass}.{FunctionName} 逐级收敛，落地形态就是 `registry.item().query().invoke(params)`：人读得懂，程序也容易做治理。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

### 商品领域知识库的三层结构

第一层显性事实知识是对"显卡的 GPU 品牌""SPU 下不同 SKU 差异"这类内容的客观描述，服务于运营决策、prompt 增强与数据清洗；第二层关联情景知识刻画商品-商品关系与商品-场景连接（典型如主配件场景），团队在 10 个类目近 10000 条案例中总结出 53 条规则；第三层隐性经验知识来自用户使用经验、专家评测与品牌文化，最能建立信任，用于商品卖点与参数说明。存储上采用两层异构：MySQL 作主持久化、保障强一致性，TisPlus 承担批量向量化处理与大规模 KV 存储、支撑语义检索。事实走强一致、经验走语义召回，是这套知识库能同时兼顾准确性与泛化性的原因。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

### AIWorkflow：在离线业务流程统一

旧架构有四个痛点：数据处理复杂且维护成本高（复杂 SQL、UDF、离散节点编排）；流程扩展性与灵活性不足；推理调度因共享资源池争抢而存在不确定性；在线与离线体系割裂、需要维护两套技术栈。新架构基于 Spring AI Agent 把业务流程封装为 Workflow，向上屏蔽触发源差异（定时调度 vs 实时事件），向下屏蔽计算资源差异（单机执行 vs 分布式集群）。核心设计是 Function（原子能力）、Action（业务动作）、Workflow（流程编排）三个标准化组件；入口只有两种——离线批量推理由调度任务触发，在线增量推理由实时事件驱动；存储写入统一为 MySQL（在线）+ ODPS（离线）。同一套 Workflow 逻辑靠触发源区分在线离线，代码复用率最大化。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

### 事务型领域事件：万级实时推理的关键

实时化的关键基础设施是"事务型商品领域事件"：精卫链路基于商品 ID + 事务 ID 对数据行变更做聚合与转发，把秒级别处理的事务量级降低一个数量级；下游 Java 应用消费消息、补全数据，最终落到异构 SKU 数据表，商品 Agent 由此搭起实时推理能力，做到万级实时推理。应用分区上，客户端层 item-agent-client 是 JDK 8 兼容的轻量级 SDK；服务层包含 agent-server（业务逻辑）、Agent 实例层（商品数据加工、问答、SKU 引擎）与评估客户端层；功能层 item-agent-functions 放原子能力（向量写入、文本解析、属性提取）；SDK 层 item-agent-sdk 提供统一调用契约；公共模块是事件引擎层 item-agent-event-engine 与管理后台层 item-agent-admin。部署上分为 SKU 引擎分组（由商品领域事件驱动，完成商品粒度到 SKU 粒度的转换）、LLM 分组（大模型实时推理核心职能并支撑评测体系）与服务分组（读写分离 + 单元化，写服务在张北中心，读服务单元化并配 Tair 缓存）。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

**落地效果**：覆盖亿级在线商品，商品信息的完整性、准确率与丰富性显著提升，搜索、详情等核心导购场景成交转化率正向提升，新需求开发周期缩短到 1 周/人。团队把这一阶段概括为"事件驱动的 Function-Centric Agent 架构"，验证了大模型能力以结构化、工程化方式嵌入商品数据生产链路的可行性；下一步是以 Harness、Skill 等新一代 Agent 框架持续演进，以商品理解"大脑"为核心构建自适应决策机制，夯实"数据 + 工具 + 知识 + 决策"四位一体的 Agent 基础设施。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

## 实践启示

这套实践的每一条经验都对应一个具体的工程取舍，从选型、抽象到事件链路与知识分层，可以逐条迁移到其他实时决策域。 ^[raw/articles/taobao-product-domain-agent-architecture.md]

1. **Java 生态 Agent 选型看耦合成本**：spring-ai-alibaba 是集团内落地的最优选择，依托成熟 Java 底座，与既有系统集成成本最低；引入全新语言栈的隐性代价往往在半年后才显现。
2. **Function-Centric 是把领域能力沉淀成资产**：用 AIFunction 声明式封装工具与领域知识（name/description/parameters/tags/sideEffect/timeoutMs），上层 workflow 才能在不改下层的前提下自由编排、按需替换能力实现。
3. **事务型事件驱动是实时推理的前提**：先做对该领域事件的聚合与转发（商品 ID + 事务 ID），把变更处理量级降下来，再谈实时推理；否则秒级事件量会直接打爆推理成本。
4. **知识库分层递进、分开存储**：显性事实 → 关联情景 → 隐性经验三层各司其职，MySQL 保强一致、向量存储保语义泛化；三层混在一起会同时损失准确性与召回率。
5. **在离线统一靠"屏蔽差异"而非复用代码片段**：以 Function/Action/Workflow 三组件标准化，把触发源差异与计算资源差异压到 Workflow 之下，同一套逻辑既能被调度触发也能被事件触发。
6. **基建要为框架与评测预留替换位**：deepeval 评测、deepresearch 事实性验证、Harness/Skill 新一代框架都被写进了演进路线，说明 Agent 基建的评测层与编排层应当可插拔，而不是一次成型。
7. **这套骨架可迁移**：任何具备高吞吐变更流、要求低延迟决策且知识密集的业务（价格、库存、风控、推荐）都能照搬"事件聚合 + 声明式能力注册 + 三层知识库 + 在离线同构"的组合。

## 相关实体
- [[entities/tmic-ai-xiaoxin-deepagent-architecture-evolution]]
- [[entities/verizon-connect-agentic-ai-100k-users]]
- [[entities/skillos-learning-skill-curation-for-self-evolving-agents]]
- [[entities/co-existence-paradigm-shift-agentic-ai-mollick-2026]]
- [[entities/huggingface-ai-agent-glossary-model-scaffolding-harness-tool-skill-subagent]]

→ [[raw/articles/taobao-product-domain-agent-architecture|原文存档]]
