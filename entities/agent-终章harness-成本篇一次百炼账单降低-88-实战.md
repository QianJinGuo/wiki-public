---
title: "Agent 终章（Harness 成本篇）：一次百炼账单降低 88% 实战"
created: 2026-08-21
updated: 2026-10-23
type: entity
tags: [token-optimization, harness-cost, context-management, prompt-caching, kv-cache, sub-agent, deferred-tools, mcp, model-routing, qwen]
provenance_state: extracted
confidence: 0.85
sources:
  - raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent 终章（Harness 成本篇）：一次百炼账单降低 88% 实战

阿里技术景钊（曹建新指导、陈威 idea）基于百炼 qwen3.7-max 生产实践写成的 Agent Token 成本优化全景实战，最终把一次 20 轮会话的输入费用从 ¥9.84 降到 ¥1.16（降幅 88.2%）。核心命题：**优化 token 不是外挂技巧，就是 Harness 实践——模型只负责生成，Harness 决定哪些内容要发、哪些过程要隔离、什么时候该停、用什么模型**。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

## 核心洞察：几个字撬动几十 M Token

一次 prompt 发给 agent 往往只有几个字，但水面下是大头：工作守则（system prompt）、几十个工具 schema、每轮对话历史、工具查回结果。故障诊断/成本分析 agent 一个模糊需求会铺开多维度找线索，一来一回 30-40 轮，烧掉几十 M token。**三个元凶**：循环反复跑、历史每轮全量重发、工具结果不断往上下文堆。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

一次三轮对话示例中，固定前缀（工作守则 + skills + 常驻工具 + MCP schema）×6 = 120k token，占总量 209.4k 的 57%，而用户输入仅 0.3k。**固定前缀每轮全量重发是最大的成本来源**。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

## 建立度量标准（先有尺子再下刀）

三条链路交叉验证，防止"为了省成本牺牲质量"：

1. **自研采集**：每次 LLM API 返回的 token 字段写入本地。`cache_hit_rate = cached_tokens / prompt_tokens`，接口未返回 usage 时记为空而非 0。`cache_mode` 从响应推断：有 `cache_type: "ephemeral"` 为显式缓存，有 `cached_tokens` 无 `cache_type` 为隐式缓存。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
2. **自建 Langfuse**：一次调用上报为一条 Generation，一个 Agent 任务 = 一条 Trace，Subagent 独立 Trace 通过 `parent_task_id` 跳回父级。`usage_details` 按 OpenAI completion schema 拆分，异步队列不阻塞主 loop。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
3. **自建评测 Agent**：从工具轨迹标记重复调用/连续失败等异常，轻量模型从相关性、工具效率、重复操作、错误恢复、完整性五维评分。低分任务自动沉淀诊断包，形成持续改进闭环。6 月与 7 月总分基本持平（证明优化未降质）。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

## 六刀优化（一次 agent loop 请求生命周期）

### 第一刀·少发：缩小每轮请求输入体积

- **随模型进化删教学文本**：早期 system prompt 里的 ReACT 循环说明、tool_use 格式教学、few-shot 示例约 1.2K token。最新模型已在海量 agent trace 上训练内化这些范式，删除后工具调用准确率和任务完成率无下降。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **System prompt 每轮重建，不写进历史**：system 占 context 5-10%（200K window）。OpenAI 接口原生不支持 system 字段，全靠框架自己处理；Anthropic 原生支持 system 字段则无此问题。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **工具 schema 按需读取（Deferred Tools）**：常驻工具 + 多个 MCP server 完整工具占的 context 可能比 system prompt 还大。Anthropic/OpenAI 原生支持该语义，但**百炼不支持**（实测 Chat Completions 和 Responses API 均静默忽略 `defer_loading`/`tool_search`）。因此在 harness 层自实现：把工具分常驻池（Read/Edit/Bash/Grep/Glob/Write 每轮带完整 schema）和延迟池（MCP/WebSearch/DB 系列只留一行目录在 prompt，最多 50 行"工具名—一句话提示"）。激活路径 A=模型 ToolSearch，路径 B=Skill 激活自动扫描。工具召回不动前面任何内容 → 缓存照常命中。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **常驻工具标准化**：常驻工具按 Claude Code 工具定义保持出入参一致。因为 Claude Code 是最流行的 coding 工具，其工具定义早已刻入 LLM 训练，模仿它既省 token 又避免工具调用失败（自定义同名 Read 出入参不一致会导致 LLM 第一次就出错）。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **Skills 去重（同一份只激活一次）**：一份 skill 正文动辄几千 token，连续 5 轮激活就占 5 次。改为全文只注入一次固定在历史作锚点，后续重复激活只在上下文尾部补一条 ~150 字符短提醒（"指令已在上文，本轮仍生效"）。谁先激活谁持有全文，后来的拿短回执。按 5 次计算这部分少写近 80%。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **重复 Read 去重**：维护已读路径集合（文件系统 mtime），每批工具执行前检查：已读过/同批重复 → 不执行，返回"Skipped duplicate read"。一个文件一轮只付一份全文 + 两条 ~50 字符错误，而非三份全文。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **MCP 迁往 CLI + Skills（把 schema 从根上干掉）**：MCP 形态即使延迟加载，目录每工具仍留一行、reveal 时完整 schema 至少付一次费，且 MCP 工具结果是模型没法过滤的 JSON 块全量进历史。更根本：一个 MCP server = 在运行时教模型一套新接口格式，教学成本每次按 token 付。实测 MongoDB MCP 24 个工具描述近一万字符、76 处参数注解，每轮 2-3K token 固定开销。迁移到 CLI + Skills 后：tools 数组字节级不变、输出可 head/grep/jq 过滤、对缓存友好。**MCP 2026-07-28 规范无状态化（退役 initialize 握手、tools/list 确定性排序可声明 ttlMs 缓存）+ Anthropic Code execution with MCP（单任务 150k→2k，降 98.7%）全是 CLI 化套路**。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

### 第二刀·发得便宜：让已发的尽量命中缓存折扣

**KV Cache 原理**：prefill（算力最密集）后生成 token 只查已有 K/V。命中后 GPU 跳过 prefill，百炼计费：显式缓存 = 标准输入价 ×10%（创建时 ×125%），隐式 = ×20%（无创建费但命中不可预期）。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

百炼显式缓存四规则：内容至少 1,024 Token、单次最多 4 个标记、有效期 5 分钟且命中续期、Tools 定义参与 system 前缀缓存计算。**为什么不能全依赖隐式**：官方不保证命中率，同场景实测隐式加权 0.800 但会单轮骤降至 0.00 全量 miss；显式加权 0.898、无灾难脱靶。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

- **System prompt 按稳定性分层**：用 `__SYSTEM_PROMPT_DYNAMIC_BOUNDARY__` 切成静态区（Constant 跨会话不变 / SessionStable 会话内禁止变化 / MidConversation 跨 reveal 稳定）和动态区（每轮可变）。万一某层变，左侧前缀仍命中，只有右侧连同 messages 失效——控制爆炸半径。工具列表先固定排序再截断（A,B,C→B,A,C 即使工具没变字节也变了会失效）。时间放本轮 user 消息末尾（新内容不破坏缓存）。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **缓存断点标在请求尾部**：`cache_control` 标记放 messages 最后一条消息上，整段前缀（sys+tools+历史）进入缓存。从 messages 末尾往前找第一条有文字消息，累计字符 ≥4096（百炼下限 1024 Token）才标，在其最后一个 text 块加 `{"cache_control":{"type":"ephemeral"}}`。第 1 轮全量创建 ×125%，第 2 轮起旧前缀命中 ×10%、仅 Δ 创建。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

### 第三刀·少想：让模型不要思考太多

prompt 里加"少想多干"没用（prompt 只是建议）。改为**从请求参数控制**：

- **不需要推理的调用直接关 thinking**：标题生成、意图分类、查询富化、轮后建议、字段提取、简单压缩——输入不长输出很短答案空间窄，直接关闭。主 Agent Loop、需判断的 Subagent 才开启。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **thinking_budget 设上限**：给主 Agent 和推理型 Subagent 选 4,096，最终回答再预留 4,096。这是故障诊断样本上扫出的拐点：从不思考提高到 4K 工具选择和根因判断明显变好，继续放 8K/16K reasoning 越长任务完成率却不涨。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

### 第四刀·SubAgent：探索过程不进主上下文，只回传结论

主 Agent 自己查日志/监控/搜索时，有用结论可能只有一句但探索过程几百行日志、十几次搜索、几条走错的路全进上下文，上下文窗口动不动塞满然后 compact（故障诊断场景 compact 浪费几分钟）。引入 Subagent 只带结论回来。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

新 Subagent 拿到自己 system prompt + 主 Agent 写的任务说明，独立 Agent Loop 完成调查，完整结果落到文件；主 Agent 默认只收到不超过 600 字符摘要和结果路径，需复核证据才按路径读。**主上下文少重发 token ≈ 父级剩余轮数 ×（探索过程 - 回传摘要）**。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

**派发原则**（是否会产生大量主 Agent 后面用不到的过程）：查多个目录/日志/数据源、独立方向、只读探索、过程长只需结论 → 拆出去；一次工具调用能回答、下一步依赖上一步、主 Agent 最终要通读原文、多人同改一文件 → 留主 Agent。**并发 agent 不自动省 token**——三份上下文和工具调用仍都要付费，只有几个方向互不重叠且都能压成短结论才适合一起派。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

委派说明必须写成能独立执行的六要素：背景/目标/范围/已知事实/边界/返回。结果回传也限六要素：状态/结论/证据/未确认项/下一步/结果路径。完整日志留文件（冷存储），摘要才是进主上下文的热数据。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

### 第五刀·Hook：在循环边界拦截，避免无效轮次

模型输入不稳定，会泄露非标准 OpenAI Function tool 格式、空回答等。定义 hook 拦截（从 github 各家 issue 抓通用规则 + 业务实测产生）：

- **一条 SSE 内重复**：边接收边按句检查新增内容，同一正常句子完整出现第三次即停（来源于 DeepSeek flash 0731 把一句话重复 685 次）。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **多 Turn 说同样的话/重复调同工具**：给回答前 500 字符生成指纹与最近三 Turn 比较，相似度 >80% 视为无进展。前两次提醒换方法，连续第三次结束。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **一直调工具但无效**：同一组工具参数连续 8 Turn 提醒；连续 5 Turn 工具结果全报错/空要求换思路；Shell 命令累计失败 4 次改路径；用户连续 3 次没响应确认说明人已离开。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]
- **空回答和假动作不进历史**：只输出 reasoning 无正文/工具调用、连续"让我查询"却未执行，当无效回答不保存，hook 注入要求直接调工具，纠正三次仍空则结束。兜底：大段 prompt 复述/自我指令丢弃并注入生成用户可读总结。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

### 第六刀·模型分流：不是每个任务都需要最贵的模型

`ModelRegistry` 设四个角色指针（后端 gRPC 下发、客户端缓存磁盘），LLM 调用不直接指定模型名而通过角色指针间接引用：

- `default_chat_alias` → 旗舰模型（主循环/Subagent）
- `classify_alias` → Flash（分类/标题/富化/内存更新/质量评估首轮/context 压缩降级），成本通常只有旗舰 1/10-1/20
- `responses_alias` → 场景模型（Responses 工具）
- `file_extract_alias` → 长文本模型（文件提取）^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

Subagent 也按角色分流：`explore`/`data-query` 走 classifyAlias，`general-purpose`/`plan`/`verification`/`code-review` 走 defaultChatAlias。K8s Agent 更彻底：四角色全指向 Flash（查 Pod/事件/日志/Prometheus 指标高度模式化），单次诊断费降到约 1/15。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

## 88% 降幅怎么算的

假设 20 轮会话，每轮输入 `S + i×Δ`（S=固定前缀，Δ=每轮新增）。无缓存 20 轮共发 `20S + 210Δ`（820k）。只加缓存：实际仍发 820k 但大部分按折扣计费（等价全价 151k，¥1.81，降 81.6%）。裁剪 + 缓存：发送量降到 532k（等价 96.9k，¥1.16，降 88.2%）。**88% 是"少发"和"发得便宜"叠出来的，不是单一优化**。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

## 深度分析

### 六刀的成本结构：截断、折扣、重复兑现

六刀常被并列成一张清单，但它们的收益形态完全不同，混在一起看会误判下一刀该往哪切。

**第一类是"每轮一次的常数截断"**：删教学文本（1.2K/轮）、deferred tools、常驻工具 schema 精简、MCP 迁 CLI。单轮省下的字节很小，但因为固定前缀每轮全量重发，真实节省等于 `单轮节省 × 轮数`——30 轮会话里砍 1K 就是 30K。这类刀的天花板由前缀本身的绝对体积决定，砍完就没了。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:81]

**第二类是"单价折扣"**：缓存断点不动任何一个字节的发送量，只把已发内容的计费乘上 0.1。它是唯一能作用于"已经删不动的内容"的刀，而且收益随历史累积递增——第 1 轮全量创建 ×125%，之后旧前缀永远只付 ×10%，命中率随对话推进趋近 1.0。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:286-288]

**第三类是"只在重复发生时兑现"**：skills 全文去重、重复 Read 去重、hook 截停空转。收益 = 重复次数 × 单次体积，方差极大，完全取决于模型当轮的行为，既不可预算也不可承诺，只能靠 hook 兜底。这类优化在账单上表现为"避免了尖峰"，而不是"降低了均价"。

三者叠在一起时，会发生一个反直觉的边际反转：**缓存生效之后，继续裁前缀的边际收益被 ×10% 折扣压成十分之一，而每轮新增的 Δ 仍按 ×125% 的创建价付费**。文章自己的测算就露了这个底——只加缓存已经拿到 81.6%，再叠加全部裁剪只多 6.6 个百分点，因为被删掉的 288k 发送量里大部分落在 ×10% 的桶里，等价全价只少了 54k。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:620-628] 换句话说：**六刀的执行顺序（先前缀、再缓存）不等于成熟系统的边际价值顺序**。前缀砍到收益衰减后，刀口应该转向 Δ——工具返回结果的体积、重复读取、无效轮次——那里还按全价计费。

### 为什么"缓存 + thinking_budget"杠杆最高，而"换模型"风险最大

按省钱多少排序会失真，按**改动面**排序才看得清风险梯度。

缓存断点改的是传输与计费，不改任何一个 token 的内容——同一个模型、同一段 prompt、同一个答案分布，质量风险在数学上为零。代价是前置条件极硬：前缀字节必须跨轮完全一致，`tools` 数组顺序、时间戳位置、memory 索引长度、skill 激活顺序，任一漂移都让断点之后全部作废。它的失效方式是**静默的成本回归**，不报错、不掉质量，只是账单涨。这就是 [[entities/llm-prefix-caching-comprehensive-guide|前缀缓存]] 类工作的共同特征：收益确定，维护脆弱。

`thinking_budget` 改的是输出侧推理预算，属于"可测但会动质量"的中间地带。文章真正的价值不在 4,096 这个数字，而在于方法：它是在自家故障诊断样本上扫出来的拐点——0→4K 时工具选择与根因判断明显变好，8K/16K 时 reasoning 变长但完成率不涨。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:415] 这个数字**没有通用性**，抄配置等于跳过实验。

模型分流的风险最高，因为它改的是能力本身，而且失效是静默的：分类错、摘要糊、质量评估自相矛盾，账单上全表现为"变便宜了"。文章只能靠 `dimension_consistent == false` 这种自检信号把可疑判例升级回旗舰。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:590] 更要看清它省的是**周边调用**的钱，不是主循环的钱；K8s Agent 四角色全指向 Flash 之所以成立，前提是"查 Pod / 事件 / 日志 / Prometheus 指标"高度模式化——这个前提复制到开放域就会崩。风险梯度因此是：改计费（零风险）→ 改推理预算（可测风险）→ 改模型（不可测风险），而收益排序恰好相反，这才是决策困难所在。

### 88% 的口径：哪些是结论，哪些是测算

把一个降幅当 KPI 之前，需要拆开它的四个隐含假设。

**基线选得最宽**。分母是"零优化的 820k"，任何已经做过部分优化的系统都复现不出 88%——已删过 1.2K 教学文本、已收过 MCP 的团队，起点就不是 820k。

**只算主 Agent 输入侧**。输出侧（thinking + 最终答案）整个没进模型，而国产模型的思考链恰恰是输出账单里最贵的部分。文章关掉大量辅助调用 thinking 的收益，在这个 88% 里完全不可见。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:628]

**100% 命中是最强假设**。测算假设 20 轮全在 5 分钟 TTL 内连续命中。真实会话里等人审批、等回复、离开键盘，TTL 一过就要重新走 ×125% 创建——这是测算与真实账单之间最大的系统性偏离源，也是很多团队"照着做了却没省到"的直接原因。

**质量没有进公式**。6/7 月总分持平只证明"没有明显下降"，不证明"没有变化"；五维评分是粗颗粒，边际质量损失（多绕一轮工具、根因判断略偏）会被平均掉。再叠加缓存命中率本身来自"两组非严格受控测"的小样本（隐式 0.800 / 显式 0.898），方向可信，绝对值不宜外推。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md:372-389]

所以 88% 应当读作**这个方向的天花板量级**，而不是"你的账单会降到 11.8%"。它同时说明了一件事：单靠裁剪只有 35%，单靠缓存就有 81.6%，两者是乘法不是加法——**上限由缓存决定，裁剪只决定你离上限多近**。参见 [[concepts/context-window-economics|上下文窗口经济学]]。

### 规模化之后的二阶成本：省下的钱会以别的方式还回去

单团队跑通六刀，和一百人团队长期维持六刀，是两件事。

**观测基建是真门槛，不是配套设施。** token 采集、Langfuse、评测 Agent、hook 规则库四套系统全是自建。deferred tools + CLI + subagent 隔离之后，一次请求的 prompt 是运行时动态拼装的——没有 trace，"这一轮为什么贵"根本无法回答。先做优化后补观测，等于把可解释性债滚到无法偿还。

**不变量靠代码守，不靠文档约定。** `tools` 只构建一次、工具与 skills 先排序再截断、时间戳放 user 末尾、先激活者持全文——每一条都是"改一个字符就全盘失效"的约束。新人加一个工具、给 system prompt 多塞一个字段，就能静默炸掉前缀。这类约束必须变成字节快照测试和 CI 断言，否则半年后必然回到原点。

**平台语义不可信，必须运行时探测。** 百炼对 `defer_loading` / `tool_search` 返回 HTTP 200 却静默忽略，工具照旧按全量 schema 调用。任何"文档说支持"的能力，都要在启动时用真实请求验证，否则优化代码写完了、账单纹丝不动。

**冷热分离需要判定规则，不是一句口号。** subagent 把证据推进文件、只回传 600 字符摘要，真正难的是"何时该按路径回读原文"——标准松了漏证据，标准紧了又把 200k 读回主上下文，隔离收益归零。

**成本纪律是持续的运营，不是一次性项目。** 每个新能力都是对前缀稳定性的攻击面，每个新 hook 都是对循环语义的修改。省下来的 88% 需要组织持续付费维护，这是任何成本案例在复盘时最容易被省略的一行。

## 实践启示

1. **先装表，再动刀，顺序不可颠倒。** 落地 `cache_hit_rate = cached_tokens / prompt_tokens`、`cache_mode`（有 `cache_type` 为显式 / 仅有 `cached_tokens` 为隐式）、每任务一条 Trace（subagent 用 `parent_task_id` 挂回父级），并把质量评测基线一起建起来。接口未返回 usage 时记 **null 而非 0**——记 0 会让命中率统计凭空塌陷。

2. **第一刀砍"每轮重发"的固定前缀，按 `体积 × 轮数` 排序。** 顺序大致是：删过期教学文本（模型已内化 ReACT/tool_use 范式）→ 常驻工具 schema 对齐 Claude Code 定义 → deferred tools（只留一行目录）→ MCP 迁 CLI + Skills。判断标准只有一个：这段内容是不是每轮都在全量重发。

3. **缓存要最后落地，但要最先验证字节稳定性。** 正确顺序是：冻结 `tools` 构建一次、固定排序 → system prompt 按 Constant / SessionStable / MidConversation / Dynamic 分层 → 时间戳挪到本轮 user 消息末尾 → 才在 messages 末尾挂 `cache_control`。顺序反了，断点挂上去也不命中，白干一轮。

4. **前缀砍到边际收益低于一成价时，把刀转向 Δ。** 缓存已经把前缀压到 10%，此时更值钱的是每轮新产生的 Δ：工具返回结果（用 `head`/`grep`/`jq` 在回传前过滤）、重复 Read（mtime 集合去重）、无效轮次（SSE 内复读、跨 Turn 指纹相似度 >80%、同组工具参数连续 8 Turn）。

5. **`thinking_budget` 当实验参数扫，不当配置抄。** 在自家任务分布上扫 0 / 2K / 4K / 8K / 16K，同时记录完成率与单次费用，取完成率不再上升的那个点；辅助调用（标题、分类、富化、字段提取、简单压缩）直接关 thinking。任何"别人用 4096"都是无效依据。

6. **模型分流从"高度模式化的辅助调用"起步，并保留升级回路。** 先做角色指针（`default_chat_alias` / `classify_alias` / `responses_alias` / `file_extract_alias`），让换模型不需要发版；只把输入短、输出短、答案空间窄的调用切到 Flash，旗舰留在主循环与需判断的 subagent。同时为质量评估类调用保留"自相矛盾即升级"的信号，否则质量回退永远不可见。新建能力时先问一句：**能不能是一个 CLI + 一份 skill？**

## 结语：成本优化就是 Harness 实践

成本优化不是外挂式 token 技巧，**就是 Harness 实践**——模型只负责生成，Harness 决定发什么、隔离什么、何时停、用什么模型。Agent Loop 成本没有单一解法：前缀少一点、缓存多命中、探索少带回来、无效轮次少跑一次，单看都不惊人，但会在每一轮每次任务重复，差距就是这样乘出来的。**让 agent 长期跑得起、跑得高效，才是 Harness 真正要解决的问题**。^[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战.md]

> 与 [[entities/tencent-token-optimization-agent-architecture|腾讯 Token 优化实战]] 互补：腾讯侧强调 Context Rot 三段生命周期与"最小上下文/清晰理想态/动态选型"框架，本实体提供阿里百炼侧的六刀可执行工程量（deferred tools、CLI 替代 MCP、缓存断点、thinking_budget、Subagent 隔离、模型分流）+ 88% 测算方法。

→ [[raw/articles/agent-终章harness-成本篇一次百炼账单降低-88-实战|原文存档]]
