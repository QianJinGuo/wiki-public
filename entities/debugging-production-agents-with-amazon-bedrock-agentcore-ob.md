---
title: "Debugging production agents with Amazon Bedrock AgentCore Observability"
created: 2026-08-01
updated: 2026-11-24
type: entity
tags: ['raw', 'article']
sources: [raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob]
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Debugging production agents with Amazon Bedrock AgentCore Observability

> -> [[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md|原文存档]]

## 摘要

AWS 机器学习博客介绍如何用 Amazon Bedrock AgentCore Observability 调试生产环境 Agent。生产 Agent 常常"静默失败"——返回看似合理但错误的答案、陷入无限推理循环、选错工具，且不触发任何报错，标准日志和指标无法捕捉决策过程。文章将生产问题分为质量（幻觉、事实错误、在多 Agent 系统中传播）、可靠性（工具调用失败、上下文丢失）、效率（高延迟、token 浪费）三类，并给出三层可观测性工具：CloudWatch 仪表盘（系统级）、OpenTelemetry 分布式 traces（执行级，遵循 OTEL 协议，可导出到 Datadog/Grafana/Elastic）、关键指标（性能 p50/p95/p99 延迟、资源、按类型细分的错误率）^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md]

文章用两个实战场景演示结构化排查流程：无限循环案例中，仪表盘显示消耗 266.9K token 但错误率 0%，单个 trace 含 177 个 span、平均延迟 85,590ms（正常 1–5 秒），根因是系统提示词要求"never give up"却没有终止条件，修复方式是显式终止条件、每会话 token 上限（建议 5,000–10,000）和 10–15 步推理硬限制。工具调用失败案例覆盖五类错误（401/403/400/404/500），用 Logs Insights 查询按工具和状态码聚合定位，并给出 Gateway service role 权限修复等方案。核心结论：结构化可观测性把数小时的猜测变成几分钟的定向排查；系列 Part 2 讲性能优化与内存管理 ^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md]

## 关键要点

- 三层观测工具：GenAI Observability 仪表盘（会话量、延迟、token 用量、错误率）、OpenTelemetry traces（bedrock-agentcore CloudWatch 命名空间）、Logs Insights 查询（每个查询约 2–3 分钟）。
- 无限循环的识别信号：高 token 消耗 + 低错误率；案例中 86 次重复调用 calculate_percentage（输入几乎相同）却永远得不到 25.00% 的精确值。
- 无限循环三大根因：提示词缺少终止条件、框架缺少循环检测（应 3 次相同动作后强制终止）、工具选择错误（描述需含明确用法示例）。
- 工具调用失败五类错误对应不同修复：401 凭证（用 Secrets Manager 自动轮换）、403 IAM 授权（补 Gateway service role 策略）、400 校验（对齐 schema）、404 资源、500 工具自身故障；任一工具错误率超 5% 应立即排查。
- 从被动调试到主动监控：把诊断查询转成 metric filter + CloudWatch 告警与仪表盘，并用 AgentCore Evaluators 对会话做持续自动化质量评分。

## 深度分析

### 三类生产问题：按「错误是否可见」划分排查空间

文章对生产 Agent 故障的三分法——质量、可靠性、效率^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:34-50]——的深层价值在于按「错误信号的可见性」切分问题空间。质量类失败最隐蔽：执行链路全程绿灯，产出却是错的——幻觉、引用不存在的策略、编造数据填充空白；更危险的是在多 Agent 系统中，上游错误输出会作为下游输入继续传播，形成错误放大链^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:36-38]。可靠性类失败介于两者之间：工作流中断但至少有迹可循，典型形态是三类工具调用错误（401 凭证、403 角色权限、400 入参非法——错误码本身就锁定了根因方向），以及上下文丢失（Agent 把追问当新对话处理，指向会话管理或记忆配置缺陷）^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:44-46]，后者与 [[concepts/agent-memory-architecture]] 讨论的会话状态保持问题直接对应。效率类失败则完全正确却持续烧钱。这个分类法的实际用途是逆向的：先看手上有什么信号（无错误但结果错 / 显式错误码 / 成本曲线异常），再决定去哪个数据层找证据；而第一类恰恰是传统告警覆盖不到的盲区，参见 [[entities/detecting-silent-agent-failures-with-amazon-bedrock-agentcor]]。

### 三层可观测性工具的分工逻辑

调试工具链按「态势 → 执行 → 基线」三层组织^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:52-68]：CloudWatch 仪表盘回答「系统是否异常」（会话量、延迟、token、错误率，可按 agent/session/时间过滤并触发告警）；OpenTelemetry traces 回答「为什么异常」——AgentCore 在 bedrock-agentcore 命名空间产出分布式 trace、span 级结构化日志与指标，走 OTEL 协议，且同一份遥测无需额外埋点即可导出到 Datadog/Grafana/Elastic；指标层负责把前两层沉淀为可持续对比的基线。文章将关键指标分为三组^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:76-93]：性能组要求 p50/p95/p99 分位延迟，且把记忆检索耗时与工具响应耗时拆开测以便定位瓶颈段；资源组监控会话时长（过短可能是用户受挫，过长可能是循环）、并发数与输入/输出分开统计的 token；可靠性组要求错误率按认证/授权/校验/超时四类细分，任何一类突增都指向特定排查区域。这一分层与 [[concepts/llm-observability-4-layer-model]] 的分层观测模型及 [[entities/agent-observability-5-layer-architecture]] 的架构设计相互印证。

### 无限循环案例：从指纹到根因的完整排查链路

案例的入口信号是反直觉的：token 消耗 266.9K 但错误率 0%——「高消耗 + 零报错」正是循环的指纹，因为 Agent 没有崩溃，只是无法收敛^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:113-119]。排查链路分四步：先按 TokenUsage 降序找出异常会话；再在会话内按时间正序拉推理序列，trace 显示单请求 177 个 span、平均延迟 85,590ms（正常 1–5 秒），span 数与耗时的组合坐实循环；然后在日志中识别重复模式——「尝试计算 → 得到近似值 → 判定错误 → 重试」逐字循环^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:140-151]；最后在 trace waterfall 中定位根因：系统提示词写着「never give up」「keep trying until you get the exact answer」却没有给任何终止条件——提示词缺陷直接制造了 177-span 循环，修复三件套是显式终止条件（同一动作三次失败即停）、每会话 token 上限（5,000–10,000）、推理步数硬限制（10–15 步）^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:152-154]。变体二（循环检测缺失）形态不同：86 次 calculate_percentage 调用输入几乎相同，返回值在 24.954%–25.049% 震荡却永远达不到要求的精确 25.00%——「近似值永远不过关」型循环，修复在框架层跟踪调用序列并在三次相同动作后强制终止，同时配 token 均值告警提前捕捉^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:166-176]。变体三（工具选错）是 Agent 拿 web_search 去算数学题，修复靠工具描述写清正反两面的用法示例^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:180-205]——工具路由问题与 [[concepts/tool-use-patterns-ai-agents]]、[[concepts/tool-use-reasoning]] 的讨论相通。

### 工具调用失败：五类错误的差异化定位

与循环的静默相反，工具调用失败至少会喊疼——错误率飙升、会话中断、用户拿到明确报错，难点只在快速归因。文章把失败收敛为五类，每类对应不同修复路径^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:211-213]：401 认证（凭证过期/缺失/方式错误）、403 授权（Agent 挂载的 IAM 角色缺策略）、400 校验（入参与 schema 不符）、404 资源不存在（工具名错/资源已删）、500 工具自身故障（内部错误/超时/限流）。定位第一步不是逐条看日志而是聚合：按工具名 × 状态码统计计数，案例中 Exception 45 次、PermissionError 6 次、RuntimeError 6 次，主导类型即第一优先方向^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:221-230]。401/403 的修复重心在 AgentCore Gateway 的 service role——工具经 Gateway 访问下游资源，role 策略必须显式覆盖（如 lambda:InvokeFunction）；凭证类问题交给 Secrets Manager 自动轮换，并为任一工具设 5% 错误率告警线^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:232-258]，与 [[entities/amazon-bedrock-agentcore-gateway-mcp-extension]] 及 [[entities/bedrock-agentcore-secrets-manager-identity]] 的身份设计互为补充。400 的排查是逐字段比对「Agent 给的」与「工具要的」（类型、必填字段、值格式）；404/500 则要先查工具自身日志——问题可能根本不在 Agent 侧，Agent 端的正确姿势是带完整上下文记录错误、瞬态失败指数退避重试、不可用时优雅降级并明确告知用户^[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md:279-286]。

## 实践启示

1. 错误率和 token 消耗必须绑在一起看：错误率正常不代表健康，「高 token + 零报错」是无限循环的指纹。直接在 CloudWatch 配置「会话平均 token 异常增长」告警，不等用户投诉或人肉跑查询。
2. 上线前对系统提示词做「终止条件审计」：显式定义重试上限、放弃条件、升级人工的时机；同时加与提示词无关的最后防线——每会话 token 上限（5,000–10,000）和推理步数硬限制（10–15 步）。提示词可被绕过，硬限制不会。
3. 错误排查先聚合后细看：用「按工具名 × 状态码计数」查询找出主导错误类型再深挖，比逐条翻日志快一个量级；401/403/400/404/500 各指向不同修复层（凭证 / IAM 策略 / schema / 资源名 / 工具自身），不要混在一起处理。
4. 工具描述写「正例 + 负例」：不只说「用于什么」，还要写「不要用于什么」（如 web_search 明确禁止数学计算），这是防止工具选错成本最低的手段；schema 变更后先跑含边界用例的独立集成测试再上线。
5. 404/500 先查工具自身：把「工具侧日志 + Agent 侧 trace」作为标准两段式排查；Agent 端实现指数退避重试与优雅降级，避免单工具故障放大为整链失败。
6. 把诊断查询固化为监控资产：排查结束后的 Logs Insights 查询转成 metric filter + 告警 + 仪表盘 widget，配合 AgentCore Evaluators 做持续会话质量评分——从「事后排查」走向「事前预警」。

## 来源

- 原文: [[raw/articles/debugging-production-agents-with-amazon-bedrock-agentcore-ob.md|Debugging production agents with Amazon Bedrock AgentCore Observability]]
