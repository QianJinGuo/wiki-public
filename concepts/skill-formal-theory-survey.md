---
title: Skill 形式化理论：表示、执行、评估与进化
created: 2026-05-07
updated: 2026-08-01
type: concept
tags: [skill, formal-theory, dag, blueprint-skill, agent-gpa, skill-mining, harness-engineering]
related:
  - [[raw/articles/skill-formal-theory-survey-10papers|Skill形式化理论综述]]
  - [[entities/skill-design-patterns|Skill 设计模式]]
  - [[entities/hermes-skill-system|Hermes Skill System]]
sources:
  - raw/articles/skill-formal-theory-survey-10papers
confidence: high
---
# Skill 形式化理论：表示、执行、评估与进化
- Type: concept
- Tags: #skill #formal-specification #dag #blueprint-skill #knowledge-activation #context-loading #agent-gpa #skill-mining #harness-engineering
- Score: Value=8 × Confidence=8 = 64
- Source: [[raw/articles/skill-formal-theory-survey-10papers|Skill形式化理论综述]]
## 核心命题
> "**模型能力正在趋同，而组织对有效行动路径的编码化程度，将成为智能体时代真正的性能分水岭。**"
## Skill六元组形式化定义
```
Skill = (ID, I, O, P, Pre, Eff)
```
| 元素 | 含义 | 作用 |
|------|------|------|
| **ID** | 唯一标识符+元数据 | 名称/版本/适用范围/所需权限 |
| **I** | 输入模式 | 技能所需的参数及其类型约束 |
| **O** | 输出模式 | 技能完成后的状态变化与返回值 |
| **P** | 步骤计划（DAG） | 偏序的步骤集合 |
| **Pre** | 前置条件 | 必须满足才能激活的逻辑断言 |
| **Eff** | 效果描述 | 对系统的预期影响，含补偿动作 |

> [!参见]
> - [[entities/skill-design-patterns|Skill 设计模式]] — 14种官方技能编写模式
> - [[entities/hermes-skill-system|Hermes Skill System]] — Hermes Agent 的技能系统实现
## 三种技能类型
| 类型 | 执行引擎 | 确定性 | 典型表示 | 适用场景 |
|------|----------|--------|----------|----------|
| **命令式技能** | LLM逐步执行 | 中等 | Markdown/YAML清单 | 灵活但依赖LLM指令遵循 |
| **蓝图式技能** | 蓝图引擎+LLM填充 | 高（CV→0.02） | TypeScript/Python代码 | 控制流必须确定的场景 |
| **知识激活技能** | 知识图谱遍历+子图匹配 | 高 | 知识图谱节点+规则 | 上下文感知的动态组合 |
> 蓝图式技能的核心：控制流从LLM剥离 → 变异系数可降至0.02以下
## 步骤计划的DAG模型
```
P = (V, E, v_0, V_f)
```
- V：节点（动作类型/参数模板/成功失败条件/重试次数/超时）
- E：依赖边（偏序约束）
- v_0：唯一入口
- V_f：终止节点集合
**执行调度**：按拓扑顺序 + 有限并行（入度为0且互不冲突的节点可并发）
## 执行机制
### 隐式激活匹配算法
```
1. 硬性过滤：Pre必须在上下文C中满足 → 否则score=0
2. 语义匹配：任务描述 vs 技能描述 → 余弦相似度
3. 上下文关联度加分：文件路径/实体名重叠 → 加权
4. 选择top skill：若均<阈值τ → 回退自由工具调用
```
> [!参见]
> - [[entities/hermes-agent|Hermes Agent]] — 实现了完整技能激活与上下文管理机制
> - [[entities/autobrowse-browserbase-persistent-skill-files|Autobrowse]] — Browserbase 将浏览器 Agent 探索成果持久化为 SKILL.md 的实践
### 渐进式上下文加载
**核心问题**：上下文窗口Token预算有限，技能文档可能超载。
**策略**：仅当前节点+直接后继节点的详细指令进入活跃窗口；其余以压缩摘要驻留外部记忆，激活时才展开。
- 活跃上下文压缩至**150–300行**的"甜点区"
- **17%步骤依从性提升**（vs 一次性加载全部文档）
### 双向验证机制
| 方向 | 机制 |
|------|------|
| 正向（计划→技能） | 计划解析为子目标 → 检索匹配技能 → 结构冲突则标记"需人工审查" |
| 反向（技能→计划） | 技能Pre/Eff投射回计划 → 检测效果冲突 |
## 多技能编排
### 依赖关系来源
1. **显式声明**：`depends_on`字段
2. **效果-前置条件分析**：S_j的断言满足S_i的条件 → 添加依赖边
### 失败回滚：补偿事务模式
每个副作用节点v定义补偿动作v_comp。执行链在v失败时 → 逆序执行所有v_comp → 状态恢复初始点。理想情况：补偿动作幂等。
## 评估框架：Agent GPA
### 五维指标
| 指标 | 公式 | 衡量 |
|------|------|------|
| **任务成功率** | 成功次数/总执行 | 最终目标是否达成 |
| **步骤依从性** | 偏离步骤/总步骤 | 是否遵循DAG执行 |
| **执行一致性** | 执行时间/均值 | 时间稳定性 |
| **Token效率** | 自由模式/技能模式 | Token节省倍数 |
| **知识新鲜度** | 有效断言/总断言 | 与代码库一致性 |

> [!参见]
> - [[entities/harness-engineering|Harness Engineering]] — Agent 支架工程框架，支撑技能执行环境
### 三种评估方法
1. **A/B对比**：技能组 vs 无技能组的同构Agent → Snowflake Agent GPA五轴评分卡
2. **回归测试套件**：输入-期望输出-期望轨迹；检查确定性+轨迹拓扑同构+安全告警
3. **人工专家审查**：评估知识正确性（KF）
## 安全与治理
### OPA/Rego安全策略引擎
```
P_i = (action, resource, condition, effect)
```
执行节点动作前评估；拒绝则拦截+记录审计日志。
### 知识衰减监测三机制
1. **依赖图跟踪**：技能记录引用的文件路径和实体名
2. **变更触发器**：被引用实体结构变更 → CI标记"需复审"
3. **定期回归**：每月执行回归测试；连续两月成功率下降超阈值 → 强制废弃
### 内容签名防篡改
```
h = H(规范化的技能文件) → 私钥签名
加载前验证签名 → 失败则只读安全模式
```
## 自动挖掘管道
**目标**：从"纯人工编写"→"算法挖掘+人工审核"半自动化

> [!参见]
> - [[entities/hermes-skill-system|Hermes Skill System]] — 自进化技能系统实践
### 三阶段
```
IDE操作日志序列D
  ↓ PrefixSpan/CloSpan频繁子序列挖掘
候选技能骨架 + 过滤（至少含"验证"或"测试"步骤）
  ↓ 语义聚类+参数化泛化
参数化技能模板（如"添加CRUD端点"，参数化为实体名）
  ↓ 三重质量过滤
(执行验证 + 步骤依从性 + 专家确认)
最终技能 → 入注册表
```
**潜力**：覆盖中小型项目**60%+**常见工作流
## Skill 版本管理与迁移策略
随着业务需求变化和模型能力演进，技能库必须支持版本管理和平滑迁移。版本管理不仅是历史记录，更是保障系统稳定性的关键机制。

### 版本演化模型
```
Skill_v1 → Skill_v2 (breaking change) → Skill_v3 (additive)
```
| 变更类型 | 影响范围 | 迁移策略 |
|----------|----------|----------|
| **破坏性变更**（Pre/O/P不兼容） | 依赖v1的调用方全部失效 | 并行运行期 + 降级通知 |
| **向后兼容添加**（新增O字段） | 已有调用方不受影响 | 热更新，无需迁移 |
| **安全修复**（Pre条件强化） | 部分调用方可能失效 | 灰度发布 + 监控 |
| **知识更新**（Eff效果修正） | 依赖旧效果的调用方需审核 | 差异通知 + 回归测试 |

### 迁移执行框架
当技能发生破坏性变更时，系统需要支持**并行运行期（Parallel Run）**：
- 新旧技能在同一上下文中同时激活
- 旧技能的调用方继续正常工作
- 新技能的输出与旧技能对比验证
- 确认无误后，旧技能标记废弃并设置终止日期

> [!警告]
> 迁移窗口期内若新旧技能存在状态分歧（state divergence），系统应记录分歧日志并触发人工审查。**不允许状态分歧进入生产环境**。

### 技能废弃的"夕阳条款"
技能废弃不是瞬间发生的，需要设置合理的**废弃缓冲期**：
1. **废弃公告期**：标记为 `deprecated`，通知所有依赖方
2. **兼容运行期**：旧技能仍可调用，新调用强制警告
3. **强制迁移期**：旧技能拒绝新调用，仅存量运行
4. **完全终止**：从注册表移除

废弃时间线由技能影响范围决定：内部工具 2-4 周，对外 API 3-6 个月。
## Skill 与 Model Context Protocol 的协同
Skill 形式化理论与 **Model Context Protocol (MCP)** 的结合，代表了 Agent 工具系统的下一个演进方向：技能的可移植性与标准化互操作。

### MCP 对 Skill 系统的价值
MCP 协议为 Skill 系统带来了三个核心能力：

**1. 工具发现的标准化**
传统 Skill 注册表中，技能的可见性局限于单一 Agent 框架。MCP 的 `ListTools`、`GetToolSchema` 等标准接口让技能具备**跨框架发现能力**。当一个 Skill 符合 MCP 协议时，任何 MCP-compliant 的 Agent 都可以查询并调用它。

**2. Schema 验证的协议层保证**
Skill 的 `I`（输入模式）和 `O`（输出模式）在 MCP 协议中有对应的 schema 定义（input_schema、output_schema）。这意味着 Skill 的类型约束不仅存在于 Skill 系统内部，还被 MCP 协议强制验证。

**3. 技能组合的协议支持**
MCP 的 `ToolCall` 流程支持调用链追踪。当一个 Skill 调用另一个 Skill 时，MCP 的 trace 机制可以记录完整的调用链路，这与 Skill DAG 的执行追踪需求高度匹配。

### Skill-MCP 映射关系
| Skill 六元组元素 | MCP 对应组件 | 说明 |
|-----------------|-------------|------|
| ID | `name` + `version` | MCP 工具的唯一标识 |
| I (输入模式) | `input_schema` | JSON Schema 类型约束 |
| O (输出模式) | 返回值结构 | MCP 未显式定义，但可通过 custom annotation 扩展 |
| P (DAG计划) | 多个 ToolCall 序列 | MCP 本身不包含 DAG，但支持多步调用链 |
| Pre (前置条件) | `annotations.readOnlyHint` 等 | 语义标注部分覆盖 |
| Eff (效果描述) | `annotations.destructiveHint` 等 | 语义标注部分覆盖 |

### 实践中的 Skill-MCP 集成模式
**模式一：MCP 包装原生 Skill**
现有 Skill 系统保持不变，通过 MCP Adapter 对外暴露为 MCP 工具。外部 Agent 通过 MCP 调用内部 Skill，无需修改 Skill 本身。

**模式二：MCP 原生技能即时注册**
MCP 服务器上的工具定义直接导入为 Skill，触发 Skill 注册流程的自动版本分配和依赖分析。

**模式三：Skill DAG → MCP 调用链**
当 Skill P（DAG）被激活时，DAG 中的每个节点映射为对应的 MCP 工具调用。MCP 的调用追踪覆盖 DAG 的执行路径。

> [!参见]
> - [[concepts/claude-code-tool-design-evolution]] — Claude Code 工具演化中的 MCP 协议协同
> - [[entities/hermes-skill-system|Hermes Skill System]] — MCP 协议在 Hermes 中的实现
## 关键结论
1. **技能编码化程度 = 智能体时代性能分水岭**（模型能力趋同）
2. **蓝图式技能** = 控制流从LLM剥离 → CV<0.02
3. **渐进式上下文加载** = 150-300行甜点区 + 17%依从性提升
4. **双向验证** = 计划与技能的图同构一致性检查
5. **技能挖掘** = PrefixSpan+CloSpan → 60%覆盖率
## 相关概念
- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — 支架工程分层上下文体系
- [[entities/skill-design-patterns|Skill 设计模式]] — Anthropic官方Skill编写14种模式
- [[entities/skills-anthropic-openai-comparison-frontend-design|Skills Anthropic vs OpenAI对比]] — 双体系对照

## 关联实体

**上游依赖**:
- [[entities/skill-design-patterns]] — 提供基础理论/方法
- [[entities/hermes-skill-system]] — 提供基础理论/方法
- [[entities/skill-design-patterns]] — 提供基础理论/方法

**下游应用**:
- [[entities/hermes-skill-system]] — 具体应用场景
- [[entities/hermes-agent]] — 具体应用场景
- [[entities/autobrowse-browserbase-persistent-skill-files]] — 具体应用场景

**平行协作**:
- [[entities/hermes-skill-system]] — 替代/补充方案
- [[entities/hermes-skill-system]] — 替代/补充方案
- [[entities/skill-design-patterns]] — 替代/补充方案

## 所属 MOC

- [[moc/ai-skill-design|Ai Skill Design]]
