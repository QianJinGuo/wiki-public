---
title: "Agent Skills 无依赖设计：Skill 之间不传数据的哲学与实践"
type: entity
created: 2026-07-10
updated: 2026-09-25
tags: [agent, skill, specification, architecture, design-philosophy, orchestrator]
rating: v9c8
sources:
  - raw/articles/agent-skills-spec-no-dependency-design
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Agent Skills 无依赖设计：Skill 之间不传数据的哲学与实践

Agent Skills 官方规范的一个反常识设计：**skill 之间在规范层面彻底断开，没有任何字段让 skill 声明对其他 skill 的依赖**。这不是规范写漏了，而是刻意的设计取舍——skill 层保持极简、无状态、可独立分发，真正的依赖关系和数据流转由 agent 的上下文窗口和运行时决策承载。^[raw/articles/agent-skills-spec-no-dependency-design.md]

## 核心设计原则

### 6 个自描述字段，零个指向外部

Skill 的 SKILL.md frontmatter 只有 6 个字段：`name`、`description`、`license`、`compatibility`、`metadata`、`allowed-tools`。全部是自我描述，没有任何一个字段指向别的 skill。这与传统包管理器（Node 的 dependencies、Python 的 pyproject.toml、Java 的 Maven）形成根本性对比。^[raw/articles/agent-skills-spec-no-dependency-design.md]

### Progressive Disclosure 三阶段

| 阶段 | 内容 | 加载时机 |
|------|------|----------|
| Discovery | name + description | 会话启动时，只读 frontmatter |
| Activation | 完整 SKILL.md 正文 | agent 判断 skill 与任务相关时 |
| Execution | references/scripts/assets | 指令引用时按需加载 |

每个阶段都只涉及单个 skill 的内部状态，没有跨 skill 的数据流动。^[raw/articles/agent-skills-spec-no-dependency-design.md]

### available_skills 是平铺清单，不是 DAG

agent 启动时看到的 skill 清单是 `<available_skills>` 平铺列表，每个 skill 是独立节点，没有任何连接关系——不是图，不是树，不是 pipeline。^[raw/articles/agent-skills-spec-no-dependency-design.md]

## 与四种体系的反差对比

| 维度 | Java import | 微服务调用 | MCP 工具 | Agent Skill |
|------|------------|-----------|---------|------------|
| 依赖声明 | 编译期硬依赖 | 服务注册/发现 | 接口 schema | **无** |
| 数据传递 | 方法参数/返回值 | RPC/HTTP 报文 | 结构化输入输出 | **agent 上下文** |
| 组合方 | 开发者写代码 | 网关/编排服务 | client 程序 | **LLM 运行时决策** |
| 契约强度 | 最强（编译器强制） | 中（接口约定） | 中（schema 约束） | **最弱（自然语言描述）** |

skill 的契约就是 `description` 那段自然语言，靠 LLM 理解语义来匹配。弱契约的好处是极度灵活，坏处是确定性差。^[raw/articles/agent-skills-spec-no-dependency-design.md]

## 编排责任推给了 agent 层

skill 不传数据、不声明依赖，复杂任务的协作靠 agent（LLM）层完成：
1. **语义匹配**：扫 available_skills，根据每个 skill 的 description 判断相关性
2. **按需激活**：加载相关 skill 进上下文窗口
3. **隐式编排**：自定执行顺序，上一个 skill 的产出传给下一个 skill——通过 agent 上下文，而非 skill 间接口

这种设计可称为"**无依赖的依赖**"：skill 层面无耦合，但任务执行存在事实上的先后和数据流转，载体是 agent 的上下文窗口。^[raw/articles/agent-skills-spec-no-dependency-design.md]

## 实证案例：12 个 skill 互相不认识

作者的项目（webchat-writer）有 12 个 skill 覆盖全链路，但 frontmatter 里没有一个提到另一个 skill。编排逻辑放在 orchestrator agent 文件里，用 `task.allow` 白名单列出可调度的子 agent，用自然语言写清楚流程。skill 保持彻底的无状态、无依赖、可独立分发。^[raw/articles/agent-skills-spec-no-dependency-design.md]

## 设计得失

**得**：skill 极度可移植。一个 skill 就是一个目录加一个 SKILL.md，无外部耦合，与 npm 包、Docker 镜像的可移植逻辑类似。

**失**：agent 层编排没有标准化。每个 agent 产品（Claude Code、Cursor、Gemini CLI、OpenCode）编排策略各不相同。同一个 skill 在不同 agent 上可能被组合出完全不同的执行路径。^[raw/articles/agent-skills-spec-no-dependency-design.md]

## 给 Skill 设计者的建议

1. **独立可跑**：别设计成"必须跟另一个 skill 配合才有意义"的结构，让 orchestrator 去编排
2. **description 是接口契约**：写清楚触发场景、能力边界、什么时候不该用
3. **数据流转靠文件**：skill 把产出写到约定路径的文件，下一个 skill 去读——文件系统充当消息队列
4. **编排集中放**：谁先谁后、哪里暂停等用户，全在 orchestrator agent 文件里统一管理

## 深度分析

### 为什么"无依赖"优于 skill 间直接传数据

表面上看，让 skill 声明 `requires` / `inputs` / `outputs` 似乎更"工程化"，但这条路线会把 skill 拖回传统软件的耦合泥潭。规范选择彻底断开，有三个深层理由：

1. **上下文成本不可累加**。skill 是给 LLM 读的指令文本，每多加载一个 skill 就多消耗一份上下文窗口。如果 A 必须拉着 B 才能跑，单个任务的实际成本翻倍；best-practices 文档明确反对这种"必须同时加载多个 skill"的窄设计。
2. **指令一致性难以保证**。skill 间一旦有显式依赖，多个 skill 的指令在同一个上下文里共存，容易出现指令打架（conflicting instructions）。独立自足的 skill 把冲突面降到最低——同一时刻只有一个 skill 的指令在主导行为。
3. **可分发性与组合正交性**。无依赖的 skill 是"平级的、孤立的、可任意组合的零件"，可以像 npm 包、Docker 镜像一样独立分发。一旦引入 skill-to-skill 的数据通道，分发就变成了依赖树的安装问题，组合的自由度也随之坍缩成图的拓扑约束。

换句话说：传统依赖声明的价值在于让编译器/运行时校验组合关系，而 skill 的组合方是 LLM 运行时的语义决策——它本来就不查依赖表，依赖字段只是死重。

### 编排责任如何转移到 agent 层

skill 层不承载依赖后，编排没有消失，而是整体上移到了 agent（LLM）层，形成"无依赖的依赖"结构：

- **匹配方式变了**：不是查依赖表，而是语义匹配——agent 扫一遍 `<available_skills>` 平铺清单，靠 description 判断哪个 skill 与当前任务相关。接口契约从 schema 退化（或者说升维）成自然语言。
- **激活是按需的、一次性的**：agent 判断相关后加载 SKILL.md 全文进上下文，任务完成即丢弃。没有长驻的服务引用，没有连接池，只有上下文窗口的进出。
- **数据通道变了**：skill 之间传数据的载体是 agent 的上下文窗口本身——上一个 skill 的产出被 agent 读进上下文，再作为下一个 skill 的输入。加上文件系统这个"事实上的消息队列"（skill 把产出写到约定路径，下一个 skill 去读），构成两条互补的传递通道。

这个转移的代价是编排逻辑失去标准化：规范只管 skill 不管 agent，每个 agent 产品（Claude Code、Cursor、Gemini CLI、OpenCode）的编排策略各不相同，同一套 skill 在不同 agent 上可能走出完全不同的执行路径。

### 案例研究：12 个互相不认识的 skill 如何完成全链路任务

作者的实际项目（webchat-writer）用 12 个 skill 覆盖从调研、写作、事实校验、去 AI 味，到配图规划、生图、上传、组装的全链路。这个案例是"无依赖哲学"的完整实证：

- **零引用检查**：12 个 skill 的 frontmatter 里没有任何一个提到另一个 skill 的名字。这不是巧合疏漏，而是设计纪律——每个 skill 都被当成可独立运行的完整能力单元来写。
- **编排单点收敛**：全部编排逻辑放在一个 orchestrator agent 文件（shuge-article-writer）里，用 `task.allow` 白名单声明可调度的子 agent，用自然语言写清整个流程。skill 侧零编排，agent 侧全编排，职责边界干净。
- **事实上的流水线**：从外部看，12 个 skill 构成一条写作流水线（写作 → 校验 → 去 AI 味 → 配图 → 组装）；从规范层面看，它们互相毫无连接。这条"看不见的流水线"完全靠 agent 的上下文窗口和文件系统在运行时动态编织。

这个案例说明：无依赖设计并不排斥复杂的协作结构，它只是把"组合关系"从静态声明挪到运行时决策——结构越复杂，编排层越要有清晰的单一收敛点。

### 设计权衡与失败模式

这套设计不是免费的午餐，权衡矩阵和对应的失败模式值得明确列出：

| 得 | 失 |
|----|----|
| 极致可移植：一个目录 + 一个 SKILL.md，零外部耦合 | 编排无标准：跨 agent 产品行为不可预期 |
| 组合自由：任意 skill 可任意搭配，无拓扑约束 | 确定性差：契约是自然语言，换模型换 agent 表现漂移 |
| 独立演化：skill 升级不影响其他 skill | 排障困难：没有依赖图可查，失败链条只能靠复现 |

典型失败模式：

1. **description 失配**：语义匹配是唯一的激活入口，description 写得含糊，agent 就会在该激活时不激活、不该激活时误激活——这是无依赖体系里最常见也最隐蔽的故障。
2. **隐式数据契约断裂**：skill A 产出的文件路径约定变了，下游 skill B 静默读不到数据。没有接口 schema 的体系里，文件路径约定就是脆弱接口，需要靠文档纪律而非工具校验来维护。
3. **上下文过载**：agent 一次性激活太多 skill，指令互相干扰，行为质量显著下降。这正是规范反对窄 skill、鼓励完整能力单元的原因。
4. **编排逻辑泄漏**：编排决策（顺序、条件分支）散落到 skill 正文里，导致 skill 失去独立性、在不同编排下行为不一致——这是对"编排集中放"原则的违背。

## 实践启示

给 skill 设计者的 6 条可操作启示：

1. **把每个 skill 当作可独立分发的完整产品来写**：自测标准是"只装这一个 skill，任务也能跑通"。如果删掉任何其他 skill 后它就失去意义，说明耦合已经渗透，应把共享部分抽出或上移到 orchestrator。
2. **把 description 当作唯一的 API 文档来打磨**：写清触发场景、能力边界、明确说"什么时候不该用"。在语义匹配体系里，description 的质量直接决定 skill 的命中率，比正文质量更影响实际使用率。
3. **用文件路径约定替代接口 schema，并像维护 API 一样维护它**：每个 skill 的输入输出落在约定路径，下游按路径读取。路径约定一旦发布就不要随意变更，变更需视为 breaking change 同步通知所有下游编排。
4. **编排逻辑单一收敛点**：执行顺序、条件分支、人工暂停点全部放进 orchestrator agent 文件，skill 正文里只描述"怎么做"，不描述"何时做"。定期检查 skill 文件里是否混入了流程控制语句。
5. **控制同时激活的 skill 数量**：为每个任务类型定义最小激活集，避免上下文过载和指令打架。宁可让 orchestrator 分多轮依次调用，也不要一次性加载一堆"可能相关"的 skill。
6. **接受弱契约，用复现测试补偿确定性**：既然换模型换 agent 行为会漂移，就在目标 agent 环境里对关键 skill 组合做回归验证，而不是假设"规范兼容 = 行为一致"。

## 与其他实体的关系

- → [[entities/agent-skill-spec-building-design-patterns|Agent Skill 规范、构建与设计模式]] — 更多关注如何构建 skill（Skill-Creator、设计模式等）
- → [[entities/hermes-agent-skills-source-code-analysis-shuge|Hermes Agent Skills 源码级拆解]] — 同一作者（术哥）的源码级分析
- → [[raw/articles/agent-skills-spec-no-dependency-design|原文存档]]
