---
title: "DAYJOB: Finance——Surge 的长程金融 Agent 基准与 RL 环境"
created: 2026-09-26
updated: 2026-11-14
type: entity
tags: [agent-evaluation, benchmark, rl-environment, long-horizon, finance-agent, tool-use]
sources: [raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026]
confidence: 0.75
provenance_state: extracted
---

# DAYJOB: Finance——Surge 的长程金融 Agent 基准与 RL 环境

Surge 发布 DAYJOB: Finance 长程金融 agent 基准（公司金融/银行/信贷/投资/实物资产五大类，真实请求 + 复杂环境 + 端到端职业判断），配套 RL environments；9 模型评测提炼 agent 核心能力层级：tool use / planning / adaptability / groundedness / common sense，附完整评分示例。 ^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md]

## 深度分析

### 从排行榜分数到可审计标本：原始页面的三类工件

与多数只公布分数表的基准页不同，这份 raw 存档保留了三类一手工件：三段真实模型输出的完整转录（Grok 4.6 的扩张建议、GPT 5.6 Sol 的报告复核 memo、Qwen3.8 Max 的成本研究交付说明）、每个任务整页的二元评分细则（三份样本合计约 152 条，页面 ❌ 87 / ✅ 65），以及任务原始 prompt 与交付格式约束。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:21-31,49,191-197,241-245] 这使它成为研究"评测如何给分"的稀有标本：不仅能问模型得了多少分，还能逐条核对每个分数从哪来。三个任务横跨公司金融（多门店扩张决策）、投资（估值复核）与运营财务（流程成本研究），交付物从 .txt/.md 文件到带可复现构建脚本的 5 页 PDF 不等。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:15,31,189,197,245] 细则全部是可机器判定的命题（数值区间、方向性结论、文件是否产出），这一结构正是其能直接充当 RL 奖励的基础。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:17-19]

### 误差传播链：Grok 4.6 在符号翻转预测上的逐跳失守

Dreamy 扩张任务是链条最长的样本：模型要在五家门店的多年期预测、DCF 估值、债务契约测算与融资建议之间保持数值一致，页面标题即点破结局——Grok 4.6"在符号翻转的销售预测上为扩张开了绿灯"。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:21-25] 其转录显示它在事实性检查上大多做对：采用 5.25% 的现行浮动利率而非过期信函里的 3.25%，正确核算 $4.0M 流动性储备底线与仅 $0.84M 的可用余量。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:37-45] 但它在关键一步失守：Atlanta/Richmond/Greensboro 三店预测把 Returns & allowances 加进了净销售额（应减去），Columbia/Greenville 两店把该输入录成负数，五店净销售额被系统性抬高。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:53-71]

评分细则把这条传播链逐跳拆开计分：先查各店符号修正及金额（如 Atlanta $532K），再查修正后公司净销售额 $81.5M–$83.0M 与 EBITDA $4.4M–$6.4M（对照未修正的 $8.12M），再查全额债务融资的 pro-forma 杠杆应在 3.35x–4.95x、超过 3.00x 契约上限（Grok 报出的 2.66x 正是用未修正 EBITDA 算出的假象），最后查结论是否随之翻转为 no-go。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:53-101,153,159-163] 每个中间环节独立给分，错误发生在哪一跳一目了然——这正是长程任务评测中最难的逐步归因问题的可操作解法。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:187] 细则还内置"反幻觉"条款：不得发明契约、政策或财务阈值，不得把未提取的 revolver 额度当作豁免契约的可用借款，不得用 2021 年约 8% 的过期关税表覆盖 2024 年后实际约 20% 的成本。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:89,107-119,185]

### 单位守恒与克制精度：GPT 5.6 Sol 错失的 $25.7M

Caldervane 估值复核样本展示另一类失败：GPT 5.6 Sol 抓出了两笔按错误层级来源定价的债券（KSA 2046 高估约 $143K、Pemex 2027 低估约 $275K）并建议暂缓发布，却把彭博终端以南非分（而非兰特）报价的 Nerovexa 价格 261.200 按面值入账，头寸凭空高估约 $25.7M，组合公允价值应从约 $765.08m 修正到 $739.2m–739.7m。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:191,199,201-227] 细则同时设有一组克制性准则：不凭未核实的股数冻结 Dravena 拆股后的发布、核实 Avyra 的价格是否真在五工作日滚动窗口内、确认 Emirates NBD 使用券商报价并不违规——都要求模型"知道何时不作为"。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:229-235] 对比两份样本可提炼共同点：防御性、事实性准则大多通过（拒绝过期利率、识别宏观下行、不误伤无关头寸），失守的恰是嵌入最深的系统性错误——主动查错比事实核对更难，为 agent 能力分层提供了直接证据。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:121-129]

### 从三道任务到五大能力：任务型设计让能力层级可操作化

页面把 9 模型的 RL 环境评测提炼为五大能力：tool use / planning / adaptability / groundedness / common sense。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:17-19] 三道任务恰好为每个抽象能力给出可判定的代理指标：从 AP 登记簿中剔除电子付款与重复登记、独立重算 2021–2025 真实支票量（2025 年仅 138 张），对应 groundedness 的数据取证；拒绝采用银行"初步节省估算"营销稿、坚持按服务协议的 $0.75 量档与 $250 月度最低消费定价，对应 common sense 的二手材料怀疑；逐年拆解单位成本、全文无自相矛盾且结论可复现，对应 planning 的过程纪律；交付指定格式文件对应 tool use；在正面材料（同业 8-K、CEO 私人关系）包围下维持独立判断，对应 adaptability。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:47,121-129,189,243-281,325-385]

Qwen3.8 Max 样本还揭示"过程纪律"与"事实 grounding"可以完全脱钩：它的成本拆解、量档与固定费用逐年复算自洽且可复现（相应准则全部通过），却建立在未剔除电子付款的毛登记量上，结论方向整体反转（推荐采纳而非否决）。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:249-263,361-385]

页面下半部分披露了 Surge 基准族版图：DAYJOB: Healthcare（临床/运营/支付方/药房/合规工作流）、HANDBOOK.md（百页公司手册的长上下文指令遵循）、ComplexConstraints（互相依赖、条件触发的约束）、GDP.pdf（真实 PDF 专家工作流）、Chartography（专业图表理解）与 EnterpriseBench CoreCraft（创业公司沙盘中的 agent 实测），它们共享"真实请求 + 复杂环境 + 端到端判断"的同一评测语法；"Will agents ever survive a 9 to 5?" 的设问即是整套基准的问题意识。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:387,391-409,425-447]

## 实践启示

1. 离线评测选型优先看可审计性：同时公开 prompt、完整转录与逐条细则的基准（如 DAYJOB）才能支撑回归定位与错误归因；只给总分的产品应降权。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:49-51,189]
2. 金融 agent 输出先过"单位与时效 sanity gate"再谈推理能力：货币单位、报价日期、来源层级是三类低成本高杠杆的机械检查，$25.7M 的错失说明前沿模型仍会栽在非推理环节。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:201-207]
3. 长任务评分按"传播链逐跳给分"：把符号修正→汇总重算→契约测试→结论翻转拆成独立准则，比只看最终对错更能定位能力缺口，也天然适合做 RL 奖励的稠密化。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:53-101]
4. DAYJOB 的二元区间细则与 [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] 的"可验证奖励"同构：每条数值区间准则都可程序化判定，五大行业环境即成套 RL environments。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:17-19]
5. 负向准则是评测设计中被低估的一半：把"不得发明契约/不得把 revolver 当豁免额度/不得轻信营销估算"写成显式扣分项，才能区分"看起来专业"与"真正可靠"。^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:107-119,185,385]
6. 横向比较长程基准用"环境复杂度 × 判断完整度"定位：DAYJOB 的差异化在职业判断的端到端完整性 ^[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026.md:15,427]；软件工程侧的对照见 [[entities/mirrorcode-long-horizon-benchmark-epoch-ai-metr|Mirrorcode]] 与 [[entities/towards-long-horizon-agents-survey-mozi-space|长程 agent 综述]]，评测工具链衔接见 [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|AgentEvalKit]]。

## 相关链接

- [[concepts/agent-evaluation-benchmark-frameworks|Agent 评估基准框架]]
- [[entities/agent-evalkit-aws-opensource-cli-agent-eval-toolkit|AgentEvalKit]]

→ [[raw/articles/dayjob-finance-surge-long-horizon-agent-benchmark-2026|原文存档]]
