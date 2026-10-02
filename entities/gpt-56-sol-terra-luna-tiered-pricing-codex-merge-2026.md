---
title: "GPT-5.6 Sol/Terra/Luna 分层定价，Codex 合并入 ChatGPT，ChatGPT Work 发布"
created: 2026-07-10
updated: 2026-10-03
type: entity
tags: [openai, gpt, gpt-5.6, bedrock, aws, coding-agent, codex, agent-pricing, benchmark]
sources: [raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价, raw/articles/刚刚gpt-56全面上线codex被合并生产力工具chatgpt-work来了, raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available]
confidence: 0.85
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# GPT-5.6 Sol/Terra/Luna 分层定价，Codex 合并入 ChatGPT，ChatGPT Work 发布

OpenAI 于 2026年7月10日正式发布 GPT-5.6 系列，同时将 Codex 整合进 ChatGPT 桌面应用，并推出新智能体工具 ChatGPT Work。三件事同日发生，核心变化不是新模型跑分，而是 OpenAI 将顶级 Agent 能力拆成了可按任务购买的多个价格档位。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md]

## 模型层级与定价

GPT-5.6 系列分三个能力层级：旗舰 Sol、均衡 Terra 和轻量 Luna。API 标价分别为每百万 token 5 / 2.5 / 1 美元（输入）和 30 / 15 / 6 美元（输出）——Sol 的标价约为 Claude Fable 5 的一半。Sol、Terra、Luna 三档模型加上 medium / high / max / ultra 多级推理深度，同一模型体系下出现多个独立的价格-能力组合。显式缓存断点和多 Agent 并行进一步丰富了定价维度——单次任务的总成本不再由模型名称决定，而是由所选档位组合决定。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md]

## Benchmark 表现

启用最高推理强度的 Sol 在 Agents' Last Exam（55 个领域）上取得 53.6%，在 Coding Agent Index v1.1 上取得 80 分，在 Terminal-Bench 2.1 Ultra 模式下达到 91.9%——三个数字均为各自评测的当前最高或并列最高分，且均以低于 Fable 5 的 API 标价实现。SWE-Bench Pro 上 Sol 得分为 64.6%，而 Fable 5 为 80%，差距显著。OpenAI 官方对 SWE-Bench Pro 结果提出异议，声称约 30% 的评测实例存在结构性缺陷，但这一主张目前无独立裁决。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md]

GPT-5.6 在任务路径明确、步骤可拆分的场景（命令行操作、终端测试、浏览器自动化）表现突出，在开放式的仓库级代码修改上出现缺口。Fable 5 则相反——SWE-Bench Pro 80 分一骑绝尘，但 Terminal-Bench 2.1 上被 Sol 甩开近 9 个百分点。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md]

## 架构变化

Codex 被整合进 ChatGPT 桌面应用，不再作为独立产品存在。ChatGPT Work 作为新的生产力工具推出，与 Cursor Composer 等 AI 编码工具形成竞争关系。顶级 Agent 能力的按任务定价模式可能影响整个 AI 工具定价生态。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md]

## Amazon Bedrock Availability — Mantle Inference Engine & Enterprise Features

GPT-5.6 Sol, Terra, and Luna are generally available on Amazon Bedrock, running on Mantle, the next-generation inference engine. Pricing matches OpenAI first-party rates, and usage counts toward existing AWS commitments. ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]

### Bedrock Inference Engine
- **Durable state capture**: Every request captures its full state continuously; if hardware fails or a node restarts mid-call, the request picks back up where it left off instead of starting over ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]
- **Isolated queue**: Each customer gets their own isolated queue with automated capacity management, ensuring predictable performance under heavy load ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]
- **In-Region inference**: Requests stay within the specified AWS Region for data residency ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]

### Prompt Caching
GPT-5.6 on Bedrock introduces prompt caching with explicit cache breakpoints. Reusable parts of a prompt (system instructions, tool definitions, reference files) are cached. Cached input is billed at a 90% discount and stays available for at least 30 minutes — long enough to cover the burst of calls a single agent run generates. ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]

### Security
- **Zero-Operator Access (ZOA)**: Enforced at the chip level — no AWS operators can access prompts or completions ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]
- Every model call runs under IAM policies, inside VPC, logged in CloudTrail
- Data perimeter policies prevent exfiltration across account and network boundaries
- Classifier-flagged traffic data retained for up to 30 days for automated abuse detection

### Regional Availability
- **Sol**: US East (N. Virginia), US East (Ohio)
- **Terra and Luna**: US East (N. Virginia), US East (Ohio), US West (Oregon) ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]

### ChatGPT Work & Codex
Alongside GPT-5.6 on Bedrock, OpenAI launched **ChatGPT Work** — an agent for larger, multi-step tasks in the ChatGPT desktop app. The updated desktop app brings Chat, Work, and Codex together. Users can configure the app to use GPT-5.6 through the Responses API on Amazon Bedrock. ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md]

→ [[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价|量子位报道]]
→ [[raw/articles/刚刚gpt-56全面上线codex被合并生产力工具chatgpt-work来了|机器之心报道]]
→ [[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available|AWS Bedrock GA announcement]]

---
## 深度分析

**1. 分层定价标志着 Agent 经济的计价单位从"模型"转向"任务组合"。** 三档能力层级（Sol/Terra/Luna）× 四级推理深度（medium/high/max/ultra）× 显式缓存断点 × 多 Agent 并行（默认 4、最高 16），共同构成一个多维成本矩阵——单次任务的总成本不再由模型名称决定，而是由所选档位组合决定。OpenAI 把"用多少智能花多少钱"从一个模糊的套餐概念变成了可逐任务计算的算术，实质上是把智能作为可分割商品出售的第一步，同时把选择复杂度和优化责任转移给了用户侧。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:17-41] 这与 [[concepts/context-window-economics|上下文窗口经济学]] 和 [[concepts/ai-cost-optimization-framework|AI 成本优化框架]] 描述的"成本 granularity 下沉到任务级"趋势一致。

**2. SWE-Bench Pro 之争暴露了评测与商业利益的纠缠。** Sol 在 Terminal-Bench 2.1 Ultra 拿到 91.9% 的同时，SWE-Bench Pro 只有 64.6%（Fable 5 为 80%）；OpenAI 随即声称约 30% 评测实例存在结构性缺陷——这是厂商自述，无独立裁决，不能当定论。把领先与落后的评测并排看，模式是互补的：GPT-5.6 强在任务路径明确、步骤可拆分的场景（命令行、终端测试、浏览器自动化），弱在开放式仓库级修改；Fable 5 恰好相反。结论不是"谁更强"，而是模型优势侧与任务结构绑定——任何单一 benchmark 都不该被当作"真实编码能力"的全貌。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:29-35] 对评测方法的系统性思考见 [[concepts/agent-evaluation-benchmark-frameworks]]。

**3. Codex 并入 ChatGPT 不是产品下架，而是入口迁移。** 同一个 Agent 引擎按用户角色提供不同交互层：开发者继续在 Codex 里用终端和 diff（内联编辑、PR 审查、多仓库支持全部保留），非开发者在 Work 模式里用文档、表格和定时任务。商业逻辑清晰：ChatGPT 有超过 10 亿周活用户，但"Codex = 写代码工具"的心智模型把他们挡在门外；而事实上已有超过 100 万人将 Codex 用于软件开发以外的工作，验证了引擎的通用性。若策略成功，Agent 工具的用户基数将从数百万开发者扩展到数亿知识工作者——真正的竞争对象可能不是其他 AI 公司，而是 Microsoft 365 的生产力席位。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:61-73] ^[raw/articles/刚刚gpt-56全面上线codex被合并生产力工具chatgpt-work来了.md:263-368] 合并的战略动机另见 [[entities/codexchatgpt为何合体codex未来何去何从openai核心leader回应一切]]。

**4. Bedrock GA + 一方同价意味着竞争焦点下移到推理运行时层。** OpenAI 模型以与第一方完全相同的价格登上 AWS Bedrock，且计入既有 AWS 承诺额——模型分数不再是唯一差异点，差异化转移到 Mantle 推理引擎的运行时保障：durable state capture（硬件故障不丢请求状态）、每客户隔离队列、In-Region 数据驻留、芯片级零操作员访问（ZOA）。对突发型 Agent 流量（单请求触发数百次模型调用），显式缓存断点 + 90% 缓存折扣 + 至少 30 分钟保留直接对冲了成本复利。^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md:16-40] 多云同价时代，"在哪儿跑"的选择依据从价格变成安全合规与可靠性 SLA。

**5. 安全与自主性的张力尚未量化。** OpenAI 将 GPT-5.6 安全体系描述为"最强"，披露了更严格的行为监控和欺骗检测；但更强自主性是否会被更保守的安全策略部分抵消——比如正常请求被拦截的概率——目前没有独立量化数据，仍需实际使用数据验证。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:81] 生态层面，Anthropic 在发布前夕重置 Fable 5 订阅配额并延长访问期限（见 [[entities/anthropic-claude-fable-5-1-mythos-5-1-2026]]），竞争烈度直接转化为用户议价能力。

## 实践启示

- **按任务类型选档，不做整体迁移。** 初始框架（未经本地验证）：摘要/改写/轻量检索选 Luna；多文件分析、常规编码选 Terra 作为日常默认起点；复杂研究与重要代码修改选 Sol high/max，用推理质量对冲返工风险。用公式粗算总拥有成本：任务总拥有成本 = (Token 成本 + 等待时间成本 + 返工成本) / (完成质量 × 成功率)——Luna 的 Token 单价只有 Sol 的五分之一，但上下文检索退化到 41.3%（MRCR 8-needle），返工成本可能吃掉省下的部分。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:43-57]

- **仓库级重构决策前先自测，不要信单一榜单。** 公开评测明显分化（Sol Terminal-Bench 91.9% vs SWE-Bench Pro 64.6%），大型仓库架构重构类任务唯一可靠的方法是用实际任务对 Sol 与 Fable 5 做小样本对比。更深入的对比数据见 [[entities/better-call-sol-the-workhorse-openai-gpt-56-sol-vs-fable-zvi-2026]]。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:50]

- **把显式缓存断点纳入 Harness 设计。** 将 system 指令、工具定义、参考文件标记为缓存断点，读取享 90% 折扣、保留至少 30 分钟，恰好覆盖单次 Agent run 的调用爆发。注意缓存写入按未缓存输入速率的 1.25 倍计费——读多写少的 Agent 工作流才划算，一次性请求别用。^[raw/articles/刚刚gpt-56全面上线codex被合并生产力工具chatgpt-work来了.md:46] ^[raw/articles/openai-gpt-56-sol-terra-and-luna-are-now-generally-available.md:34]

- **多 Agent 并行不是默认免费午餐。** 并行 Agent（默认 4、最高 16）确实把得分-延迟曲线向左上推移，但 ultra 配置本质是用更高 Token 消耗换更优结果——仅当并行收益大于推理成本溢价时才值得开启（如极高价值且天然可并行的任务）。设计并行编排时可参考 [[concepts/multi-agent-collaboration-patterns]]。^[raw/articles/刚刚gpt-56全面上线codex被合并生产力工具chatgpt-work来了.md:105-118] ^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:51]

- **把 benchmark 争议当作评测卫生的提醒。** 建立(或复用)自己的任务评测集做模型决策，不依赖单一公开榜单；对厂商质疑保持记录但等独立裁决——SWE-Bench Pro 的差距既不能忽视，也不该被当作 GPT-5.6 的定性标签。选型结论应绑定任务类型与自测数据，而非排行榜名次。^[raw/articles/gpt-56-正式上线codex-和-chatgpt-合并顶级-agent-能力开始按任务定价.md:77-85]

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

