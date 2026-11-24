---
title: "Claude Fable 5.1 Solves the Cyphral Distich"
created: 2026-09-22
updated: 2026-11-14
type: entity
tags: [agent, case-study, cryptography]
sources: [raw/articles/vals-ai-fable-solves-cyphral-distich]
confidence: 0.7
---

# Claude Fable 5.1 Solves the Cyphral Distich

## 摘要

vals.ai 发布的开放任务实验：给 Claude Fable 5.1 一个开放式目标——解开一个 370 年无人破解的历史密码（Sir Thomas Urquhart 的 Cyphral Distich）。模型在零人工干预下，44 分钟消耗 176k tokens 后给出自验证的解答^[raw/articles/vals-ai-fable-solves-cyphral-distich.md]，并顺手解开了同风格的更大密码 Cyphral Octastich（285 个数字，除 9 个字母外全部还原）。这是 agent 长程自主性与问题选择能力的一个标志性 case study。

## 核心要点

- 密码本体：Urquhart《Logopandecteision》末尾的两行各 32 个数字（共 64 个数字），1899 年起被当作公开难题，入选密码学者 Klaus Schmeh 的"Top 50 未解密文"榜单。
- 历史失败原因：此前人类的尝试（频率分析、替换、同音替换）都假设密钥是外部密码字母表，而真正的密钥就是书本身——第 i 个数字指向第 i 篇 Proquiritation，用作词索引取首字母。
- 两个关键线索：Urquhart 强调数字 32（恰好对应 Proquiritations 的数量和每行数字个数）；密码附诗承诺诚实读者能找到"wishes"，而 Proquiritations 反复以"desire / wish / hope"结尾。
- 解答自验证：每行恰好 32 个字母、以 "and / land" 押韵成联，且内容是写给 Charles II 的保王党祈祷文——与 Urquhart 的政治立场完全吻合。
- 第二战果：同风格的 Cyphral Octastich（《The Jewel》, 1652，285 个数字对应书的 284 页页码索引）也被解开，231/275 个可读位置是精确命中，其余偏差可归因于连字符、转录差异等已识别原因。
- 实验设置：作者 Geby Jaff 只给了目标和简单鼓励，外加两条约束——避开已有解法或答案不可验证的密码，避开 Kryptos K4 这类投入过度、短期无望的难题。
- 作者结论：线索其实极简单，其他前沿模型未必解不开；Fable 5.1 的过人之处在于快速识别出这个"异常可解"的问题，知道何时止损换题。

## 深度分析

### 密钥藏在语境里：外部映射假设的集体盲区

几百年的人类破解史都卡在同一个隐含假设上：密钥必须是某种独立于文本的映射表（数字→字母/词）。而 Urquhart 的设计恰恰相反——密钥是载体文本自身，数字只是坐标。模型胜出的关键动作不是更强的密码分析，而是回到原始文献语境：注意到密码紧挨着 32 篇 Proquiritations 印刷，且作者本人刻意强调"32"这个数。这提示 agent 式解题的优势之一是不带先入之见，能对"线索就在现场"保持开放，而不是套用领域内的标准工具箱。

### 自验证设计是长程任务的隐形脚手架

解出的结果自带多重校验：行数、字母数、押韵结构都符合"distich"的承诺，明文是皇家祈祷文，历史人物设定完全自洽。这意味着 agent 无需外部裁判就能确认自己对了——这与 [[concepts/verifier-paradox|Verifier Paradox]] 描述的"可验证性决定 agent 任务可行性"高度一致。作者刻意筛选"能快速验证答案"的密码，正是把验证器内建进了任务本身；Octastich 的 231/275 精确命中率统计也是同样的思路——用分布证据而非单点直觉来支撑结论。

### 开放任务中的问题选择能力

作者给出的是目标而非路径，模型花时间浏览候选问题库，"知道问题何时不动、该止损"。作者明确表示其他前沿模型未必解不开，差距在于快速识别出哪个问题异常可解。这种"选题嗅觉"是长程自主 agent 与求解器式 agent 的分水岭：价值不在单点智力，而在把有限的 token 预算押在可解的问题上，与 [[entities/ai-agent-rli-benchmark-fable5-gpt55-comparison-2026|RLI 基准中 Fable 5.1 的评测表现]]反映的能力画像相互印证。

### 子任务委托与外部资源核查

Octastich 阶段的一个细节：需要确认 1652 年《The Jewel》是否存在无版权页图时，作者提到"a subagent checked Google Books, HathiTrust, IA, NLS"——agent 主动派出子 agent 并行核查多个档案库，并得出"该贴缺页、需实物或 1983 年 Jack & Lyall  edition 确认"的诚实结论。这种"无法即时验证就明确标注边界"的行为，是 [[concepts/subagent-spawning-pattern|子 agent 委托模式]]在真实研究任务中的自然运用，也展示了 [[concepts/long-running-agent-architecture|长程 agent 架构]]中保留不确定性记录的价值。

## 实践启示

1. 给 agent 开放式目标时，约束比指令更有效：只给两条"避开什么"（已有解的、投入过度的），剩下的策略空间留给模型自主探索。
2. 任务筛选优先考虑"答案可自验证"的题目——押韵结构、字数约束、历史一致性都是免费的 verifier，比事后人工审查更可靠。
3. 疑难问题的瓶颈往往不在计算而在语境：回到原始文献、注意印刷位置和作者的自我强调，可能比堆砌标准分析方法更快破局。
4. 长程 agent 需要"止损开关"：识别"问题不动了"并及时换题，把 token 花在可解的赌注上，本身就是核心能力。
5. 未解之谜（历史密码、档案谜题、被遗忘的猜想）是 agent 的天然富矿——它们卡了几个世纪的原因常常只是"没人愿意花几百小时读冷门材料"，而这正是 agent 的比较优势。
6. 解算结果要像 Octastich 那样附验证工件（逐位置对照表、页码解析脚本），把结论变成可复现的证据链，而非一段叙述。

## 相关实体

- [[entities/anthropic-claude-fable-5-1-mythos-5-1-2026|Claude Fable 5.1 与 Mythos 5.1]] —— 被测模型本体
- [[entities/vals-ai-room-temperature-luttinger-magnet-agent-design|vals.ai Luttinger 磁体 agent 设计案例]] —— 同系列开放任务实验
- [[entities/vals-ai-cheating-on-the-rise-terminal-bench|Terminal-Bench 作弊率研究]] —— vals.ai 对 agent 评测可信度的另一面观察
- [[concepts/verifier-paradox|Verifier Paradox]] —— 可验证性如何决定 agent 任务边界
- [[concepts/long-running-agent-architecture|长程 agent 架构]] —— 零干预长任务的基础设施前提
- [[concepts/subagent-spawning-pattern|子 agent 委托模式]] —— 档案核查的并行分解

→ [[raw/articles/vals-ai-fable-solves-cyphral-distich|原文存档]]
