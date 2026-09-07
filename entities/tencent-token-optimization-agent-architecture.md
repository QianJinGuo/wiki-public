---
title: "腾讯 Token 优化实战 — 省 Token 和用好 AI 是同一件事"
created: 2026-07-08
updated: 2026-09-07
type: entity
tags: [token-optimization, context-management, agent-architecture, sub-agent, prompt-engineering, cost-efficiency, harness-design, tencent, progressive-disclosure, deferred-tools]
provenance_state: extracted
confidence: 0.8
sources:
  - raw/articles/tencent-token-optimization-agent-architecture
  - raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 腾讯 Token 优化实战 — 省 Token 和用好 AI 是同一件事

腾讯工程师 Kario Chen 提出的 AI Token 优化体系，核心命题：**在合适的时间，把合适的上下文，装载进合适的模型**。^[raw/articles/tencent-token-optimization-agent-architecture.md]

## 核心框架

### Context Rot 分析

Transformer 架构的固有限制：n 个 token 之间是 n² 的两两关联，上下文越长注意力被摊得越薄。无效信息冲淡有效上下文，导致模型"越用越笨"。^[raw/articles/tencent-token-optimization-agent-architecture.md]

Context Rot（上下文腐烂）是一种渐进式退化现象：^[raw/articles/tencent-token-optimization-agent-architecture.md]

- **FRESH 阶段（0-20 轮）**：上下文新鲜，模型召回率高，适合精密推理
- **DRIFT 阶段（20-60 轮）**：上下文开始漂移，需要主动压缩或开启新会话
- **ROT 阶段（60+ 轮）**：上下文深度腐烂，必须重开会话^[raw/articles/tencent-token-optimization-agent-architecture.md]

### 四步工程化动作

| 动作 | 核心方法 | Token 节省效果 |
|------|---------|---------------|
| **最小上下文** | JIT 检索 + 分层卸载 + Prompt Caching | 输入成本降低 ≤90% |
| **清晰理想态** | 约束金字塔：示例 > 规则 > 需求 | 减少返工轮次 |
| **Harness 分层** | 会话三段生命周期：FRESH→DRIFT→ROT | 主动压缩避免上下文腐烂 |
| **动态选型** | 大脑+手脚子智能体架构 | 总成本降至 20-30% |

#### 1. 最小上下文：JIT 检索 + 分层卸载

AI 上下文当存储系统来管理，采用三级缓存架构：^[raw/articles/tencent-token-optimization-agent-architecture.md]


| 层级 | 类比 | 策略 |
|------|------|------|
| L1 缓存 | 当前会话 | 最活跃的上下文 |
| L2 内存 | Prompt Caching | 系统提示词 / 工具定义前缀 |
| L3 磁盘 | 文件系统 | 历史摘要存档 |

- **JIT（Just-in-Time）**：不把整个代码库塞进上下文，只维护轻量索引（文件路径 / 查询语句 / 链接），用到时 grep/glob/read 现场搜索。^[raw/articles/tencent-token-optimization-agent-architecture.md]
- **Prompt Caching**：固定系统指令和工具定义放前缀，重复部分按零头计费。可降低 90% 输入成本（DeepSeek 尤其明显）。^[raw/articles/tencent-token-optimization-agent-architecture.md]

#### 2. 清晰理想态：约束金字塔

出发前回答三个问题：
1. 你希望 AI 用什么语言/风格输出？（示例）
2. 有什么限制条件？（规则 / Spec）
3. 核心目标是什么？（需求）

递进金字塔：**示例（One-Shot）> 规则（Spec）> 光提需求**。约束越精确 → AI 越不用猜 → 一次到位 → 返工越少 → Token 越省。^[raw/articles/tencent-token-optimization-agent-architecture.md]

> 护栏是信任的基础。没有护栏，你只敢让 AI 做简单的事；有了护栏，你才敢把复杂的活交给它。^[raw/articles/tencent-token-optimization-agent-architecture.md]

#### 3. Harness 架构分层：会话三段生命周期

| 阶段 | 轮次 | 状态 | 策略 |
|------|------|------|------|
| **FRESH** | 0-20 | 最佳 | 放心用，不用省 |
| **DRIFT** | 20-60 | 开始漂移 | 主动摘要压缩，新会话继续 |
| **ROT** | 60+ | 腐烂 | 没救了，必须重开 |

这种生命周期划分启发了 [[entities/loop-engineering-feedback-control-system|Loop Engineering]] 中的反馈控制设计——当检测到 Context Rot 信号时触发会话切换。^[raw/articles/tencent-token-optimization-agent-architecture.md]

#### 4. 动态选型：大脑+手脚（子智能体架构）

不是一个大模型从头到位，而是**主虾（大脑）+ 子虾（手脚）**：^[raw/articles/tencent-token-optimization-agent-architecture.md]


- **主虾（强模型）**：理解全局、拆分任务到原子粒度、调度执行、汇总决策。上下文永不积压。^[raw/articles/tencent-token-optimization-agent-architecture.md]
- **子虾（便宜模型）**：接收原子任务、一次完成、只交结果、可并行执行。^[raw/articles/tencent-token-optimization-agent-architecture.md]

核心设计原则：拆分到**原子任务粒度**——每个完全独立，不需要跨轮上下文，可用中等模型一把搞定（如"补充单测""校验 JSON""提取关键信息"）。^[raw/articles/tencent-token-optimization-agent-architecture.md]

**效果**：总成本降到全程最强模型的 20-30%，子虾可并行执行，主虾永不疲劳。恰似软件工程的高内聚、低耦合——把模型当团队编制，大脑和手脚职责明确，信息流清晰。^[raw/articles/tencent-token-optimization-agent-architecture.md]

## 深度分析

### Context Rot 的本质是注意力经济学

Token 优化表面上是个"省钱"问题，其本质是**注意力资源的稀缺性经济学**。Transformer 的自注意力机制中，每个 token 与其他所有 token 建立关联，有效信息密度随上下文增长呈指数级下降。^[raw/articles/tencent-token-optimization-agent-architecture.md] Context Rot 不是 bug，而是架构约束的必然结果——就像人类的短期记忆一样，信息过载时最先丢失的是边缘细节。腾讯的四步法本质上是在给 AI 的"工作记忆"设计缓存替换策略：L1（上下文窗口）、L2（Prompt Caching）、L3（文件系统），对应计算机体系架构中经典的 cache hierarchy。^[raw/articles/tencent-token-optimization-agent-architecture.md]

### 约束金字塔：从"喂数据"到"喂约束"的 Agent 工程范式

"清晰理想态"中的约束金字塔（示例 > 规则 > 需求）揭示了一个更深层的 Agent 工程原理：**Agent 不需要更多数据，它需要更好的约束**。^[raw/articles/tencent-token-optimization-agent-architecture.md] 传统软件工程中，约束以类型系统、接口契约、测试用例的形式存在；AI Agent 时代，约束以自然语言示例、结构化 Spec、分层需求的形式存在。这两种范式本质相同——都在定义"什么可以做、什么不可以做"的边界。当约束足够精确时，Agent 的搜索空间被有效裁剪，返工轮次减少，Token 消耗自然下降。这与 [[entities/spec-driven-development-harness|Spec-Driven Development]] 的核心思想一致。^[raw/articles/tencent-token-optimization-agent-architecture.md]

### 大脑+手脚架构：Harness 工程中的分层智力分配

主虾-子虾架构是 Harness Engineering 中"分层智力分配"的典型实践。^[raw/articles/tencent-token-optimization-agent-architecture.md] 强模型充当"架构师"角色——理解全局、分解任务、集成结果；弱模型充当"工程师"角色——执行原子任务、输出确定结果。这种分层的价值不仅在于成本节省（20-30%），更在于：

1. **隔离复杂度**：主虾不需要了解每个子任务的内部细节，只关心任务分割和结果集成
2. **并行度提升**：子虾互不依赖，可以全并行执行，突破单模型推理的串行瓶颈
3. **容错性增强**：单个子虾失败不影响主线，主虾可以重新调度

这与 [[entities/lilian-weng-harness-engineering-self-improvement|Harness Engineering]] 的模式三（子智能体与后台任务）一脉相承。^[raw/articles/tencent-token-optimization-agent-architecture.md]

### Prompt Caching 的成本结构革命

2026 年多家模型厂商（Anthropic、DeepSeek、OpenAI）推出 Prompt Caching 定价——系统提示词和工具定义前缀按缓存命中率折扣计费。^[raw/articles/tencent-token-optimization-agent-architecture.md] 这一变化对 Agent 架构设计产生了深远影响：

- **前缀固定化**：将系统提示词、工具 Schema、技能定义放在前缀中，利用缓存降低 90% 输入成本
- **会话持久化**：长会话的前缀会被持续缓存，设计 Agent 时应确保前缀在会话期间不变
- **模型选择策略**：不同模型的 Prompt Caching 机制不同（DeepSeek 缓存粒度更细），选型时应纳入成本考量

## 实践启示

1. **上下文分级管理**：将 Agent 上下文视为"缓存系统"而非"无限窗口"。建立 JIT 检索 + 分层卸载机制，主动管理上下文健康度。任何超过 60 轮的会话都应当触发「存档→重启」策略，而不是继续堆叠上下文。^[raw/articles/tencent-token-optimization-agent-architecture.md]

2. **约束优先于数据**：在向 Agent 提供更多信息之前，先问"约束是否足够精确"。写好 Spec、提供示例、明确规则，比塞进整份文档更有效。将 "示例 > 规则 > 需求" 的金字塔原则集成到 Agent 的 prompt 设计中。^[raw/articles/tencent-token-optimization-agent-architecture.md]

3. **分层智力分配**：不要用旗舰模型做所有事。识别 Agent 任务中的"大脑"部分（全局决策、任务分解）和"手脚"部分（原子执行、数据提取），分别分配强模型和弱模型，总成本可降低 70-80%。^[raw/articles/tencent-token-optimization-agent-architecture.md]

4. **Prompt Caching 优先的架构设计**：在设计 Agent 系统时，将系统提示词、工具定义、技能描述组织为固定前缀结构，利用上下文缓存降低长期运行成本。单次会话内避免频繁变更前缀内容。^[raw/articles/tencent-token-optimization-agent-architecture.md]

5. **Context Rot 监控作为运行时指标**：在 Agent 运行框架中埋入 Context Rot 检测——跟踪每轮输入 token 数与输出质量的关联。当质量开始下降（DRIFT），主动触发摘要压缩或会话切换，而不是被动等待用户察觉。^[raw/articles/tencent-token-optimization-agent-architecture.md]

## 第 2 来源 — Multi-Agent 工作流成本降 50%+（lemonye，腾讯技术工程，2026-08-21）

腾讯程序员 lemonye 用 AI Agent 驱动前后端全流程开发（tech-leader 调度，1 TL + 6 子 Agent），先以 AgentLens 拆成本看清"钱烧在哪"（系统提示词/工具返回/历史消息是大头），再围绕"让 AI 只看到当前需要的上下文 / 减少无关上下文 / 减少重复上下文"三原则落地 10 个方向，全流程预估降本 50%~65%。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

**六类 token 来源**：system prompt（固定）、工具 schema（固定）、MCP 返回（各轮）、Skill 正文（条件性常驻）、历史对话（append-only 滚雪球，真正的大头）、用户提示词。没有按 Wave 粒度拆分前完全不知道钱烧在哪。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

**让 AI 只看到当前需要的上下文**：

- **渐进式披露（L2/L3 分层）**：SKILL.md 条件性内容外移到资源层，正文只留骨架。自动化测试 Skill 从 198 行降到 128 行（-35%）；tech-leader 主调度 Skill 把各步骤详细内容拆到 references/，正文只保留核心职责/规模预判/分流规则/进度追踪/容错机制。骨架负责"决定下一步走哪"，"怎么走"按需 read_file。安装 20 个 Skill 初始加载仅 1000-2000 token。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **确定性操作由脚本执行**：AI 不拼 `mysql -u root -p` 这类命令，只提供参数给 dev-env.sh 脚本（数据库迁移/编译/服务启动/健康检查）。**能用 CLI 就不用 MCP**——每次 MCP 调用是完整 LLM 推理轮次（决策 + JSON Schema 常驻 + 结果进 context + 处理结果），不可重跑、无法并行。Playwright MCP 换 Playwright CLI：大模型做推理（生成 spec），脚本做执行（跑 CLI），两者拆开。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **MCP 数据获取子 Agent 化**：主 Agent 直接调 MCP，原始 payload（TAPD 需求描述、Figma 节点树 JSON 上万行）永久卡在最长生命周期 Agent（TL）的 context 里后续每轮重计费。给 TAPD/Figma 各建专属子 Agent 只返回结构化摘要。实测单轮 input token 从 1,030,000 降到 634,905（-38.4%）。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **长期记忆按需索引加载**：context-keeper 先 read_file INDEX.md（几十行标题+标签+摘要表格），按关键词匹配 + 类别权重算相关度，取 Top 3 命中条目才 read_file 正文。INDEX 不存在才回退全文 search_content。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

**减少无关上下文**：

- **单 Agent 拆分为多 Agent**：拆分本身不靠"减少历史堆叠"直接省钱（6 Agent 并行 = 6 份 system prompt 计费），真正收益是分散滚雪球效应 + 打开后续优化空间。**必须先做规模预判（S/M/L）**：小需求单 Agent，中大型才拆。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **Agent 专属配置（工具白名单 + 模型分层）**：给每个角色创建自定义 Agent，frontmatter tools 字段指定白名单（未列出工具对该 Agent 完全不可见）。主 Agent 派发提示词从 10-15 行精简到 2 行动态内容。按角色模型分层：规则性强、轮次多的测试/视觉角色换 GLM-5v（成本约 Sonnet 36%），测试/视觉 Agent 成本 -64%。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **代码图谱替代盲搜**：graphify 用 AST + 语义建文件索引和依赖关系，搜代码前先搜目录锁定范围再 read_file。实测总 token -22.7%（87.5万→67.7万），输入 token -22.8%，缓存命中率 -3.4pp（因探索轮次减少）。代码图谱的收益主要体现在减少探索轮次（少一轮工具调用 = 少一次 API 请求 = 少一整个 context window 计费）。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

**减少重复上下文**：

- **稳定前缀设计（状态外化）**：KV Cache 本质——请求前缀与上次完全一致则服务商直接复用，只收缓存读取低价（约正常价 10%）。把派发子 Agent 提示词里稳定指令和动态内容交错排列改法为动态内容后置。**主 Agent 隐藏前缀破坏者**：进度状态每阶段切换输出完整进度看板堆积历史——改法把进度外化到文件，每次唤醒先 read_file 进度文件而非回放历史，阶段切换改单行输出（✅ Wave 1 完成 → Wave 2 🔄 已启动）。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **避免重复加载 Skill**：前端 Agent 发现"调用知识沉淀 Skill 加载项目上下文"没必要——主 Agent 已加载过并写入技术方案文档，前端 Agent 直接读文档拿结论即可。信息在最上游收集一次、通过文档传递下游，而非每个 Agent 各自重复获取。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **rtk 压缩 CLI 输出**：拦截命令执行前重写为压缩版本的开源 CLI 代理（官方实测 60-90%，但幅度因命令而异：ps aux -98.9%，纯 git status 仅 -31%）。坑：官方 --agent 不支持 CodeBuddy（改用 PreToolUse Hook 自己实现）；rtk 输出字段 updatedInput vs CodeBuddy 要求 modifiedInput 不匹配会静默失效。评估方法论：不能用"同一需求跑两遍"对比（大模型执行路径不确定噪声大），用 rtk gain 统计或命令行对比（纯文本过滤 100% 可复现）。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]
- **工具调用并行化**：无依赖的多次工具调用串行时，每次都是一轮独立 LLM 推理，前面所有轮次历史重新打包计费。TAPD/Figma 摘要获取互不依赖，改同一轮消息内并行发起（Task 工具同时传两个 subagent_name）。测试用例同理：Playwright CLI 一次接收多个 spec 文件内置多 worker 并行。判断原则：两次调用无数据依赖就该并行，"沉默的串行"藏在顺序思维里需专门排查。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

**效果数据**：主 Agent 端到端 token 708,783（17 轮）→ 315,266（9 轮），-55.5%、轮次 -47%；子 Agent 单轮固定开销常见几十万 → 约 2 万；CLI 输出 rtk 压缩 -60%~90%；测试/视觉模型成本 -64%。全流程反推（以实测 Wave 消耗分布代入各分项降幅区间）中型需求 token 成本降 50%~65%。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

**核心经验**：① 省 token 不等于功能降级，只是调整"何时加载/怎么表达"；② 上游收集一次、通过文档传递（最贵冗余是每个 Agent 各自重新发现同一份信息）；③ 最省钱的调用是不调用——确定性操作用 CLI/数据预取解决；④ 能并行就不要串行。落地优先级：规模预判 + Agent 拆分（架构前提）→ 度量 + SKILL.md 重排 + 全局接入 rtk（一个下午见效）→ 条件内容移出 SKILL.md 等。^[raw/articles/靠这10个优化点我们把multi-agent工作流成本降了50以上.md]

> **互补角度**：与第 1 来源（Kario Chen 的 Context Rot / 最小上下文 / 动态选型理论框架）不同，本来源提供 lemonye 的多 Agent 团队落地工程细节——渐进式披露实操（SKILL.md 骨架化 + references 外移）、MCP 数据子 Agent 化（-38.4%）、代码图谱（-22.7%）、rtk CLI 压缩、工具并行化、模型分层路由（GLM-5v -64%），并给出 S/M/L 规模预判与"拆分先行"的架构判断。

## 相关实体

- [[entities/lilian-weng-harness-engineering-self-improvement|Harness Engineering]]
- [[entities/loop-engineering-feedback-control-system|Loop Engineering]]
- [[entities/mattpocock-skills-grill-me-grill-with-docs-caveman|Skills For Real Engineers]]
- [[entities/spec-driven-development-harness|Spec-Driven Development]]
- [[entities/agent-harness-context-management-working-set|上下文管理]]
- [[entities/agent-architecture-harness-new-backend|Agent 架构模式]]

→ [[raw/articles/tencent-token-optimization-agent-architecture|原文存档]]
