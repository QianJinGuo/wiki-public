---
title: "SearchGEO：13个大模型检索Agent可信度评测"
created: 2026-07-09
updated: 2026-11-09
type: entity
tags: [ai, retrieval-agent, evaluation, llm, safety, research, searchgeo, schmidhuber]
sources: [raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026]
confidence: 0.75
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# SearchGEO：13个大模型检索Agent可信度评测

> **SearchGEO** 是由 KAUST 生成式AI卓越中心、吉林大学、浙江大学、瑞士人工智能实验室等机构联合提出的评测框架，用于系统量化 AI 搜索 Agent 在联网检索时被虚假内容误导的脆弱性。研究涵盖 13 个主流大模型后端、5 种攻击模式、4 个高风险领域，由包括 Jürgen Schmidhuber 在内的研究者团队完成。^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

## 研究动机

2026 年央视 3·15 晚会曝光了名为「GEO」（Generated content poisoning）的灰色产业链：攻击者借助软件批量生成虚假软文，主流 AI 大模型在联网搜索后直接将这些捏造内容当作"标准答案"推荐给用户。SearchGEO 研究系统性地回答了：不同大模型在面对这种"投毒"时表现差异有多大，谁更容易被操纵，谁能识破攻击。^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

## SearchGEO 评测框架

要判断搜索结果污染的影响，关键难点是将污染效果与其他变量（内容本身专业度、搜索结果排名位置等）分开。SearchGEO 构建了一个**混合搜索代理**：先把真实的 SerpAPI 搜索结果缓存下来，在指定的排名位置用攻击者构造的网页内容替换掉原本的结果，从而隔离污染的因果效应。攻击内容本身也经过 AI 仿照每个任务真实搜索结果的质量生成，再经人工逐篇审查以去除生成痕迹。^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

### 攻击分层

攻击归纳为三个层次、五种模式：^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

1. **机器层（Machine Layer）**：在页面中植入人类看不见但会被模型读取的隐藏内容
2. **信任信号层（Trust Signal Layer）**：伪造权威来源，或制造多个来源"一致同意"的假象
3. **复合攻击**：上述两者的叠加

核心衡量指标为 **攻击成功率（ASR）**——AI 最终是否将攻击者指定的目标推荐给了用户。^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]


## 关键实验结果

13 个后端的整体 ASR 拉开了一个数量级的差距：^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

| 模型 | ASR | 评估 |
|------|:---:|------|
| Claude-Sonnet-4.6 | **0.0%** | 最安全，零漏洞 |
| GPT-5.4-mini | **0.8%** | 极低风险 |
| Gemini-3-Flash | **31.4%** | 最脆弱，单靠"合成共识"攻击可达 73% |
| Gemini 系列 + DeepSeek-V4-Flash + MiniMax-M2.7 | **>17%** | 显著脆弱 |

### 关键发现

- **后端大模型之间的差距**大于领域之间、攻击模式之间的差距——模型选择是安全性的首要决定因素
- **没有一种攻击通吃所有模型**：不同模型的脆弱方向不同，两个看起来最安全的模型可能朝不同的方向失败
- Claude-Sonnet-4.6 虽整体 ASR=0%，但存在"**沉默漂移**"（silent drift）和误拒风险
- GPT 系列在新场景中**极易被攻击**，表明泛化能力是核心弱点

## 安全启示

该研究对依赖 AI 搜索的用户安全有直接警示意义：^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

- 检索 Agent 的**第三方内容注入**是一个被低估的攻击面——攻击者无需侵入系统、接触模型权重或提示词，仅通过开放网络的虚假内容即可影响所有依赖联网检索的 AI 助手
- 选择后端模型时，**ASR 是一个关键安全指标**，不同模型间的差距可达数量级
- AI 安全评估需要全面覆盖模型的防御机制，而非仅关注生成质量

**论文**：arxiv.org/abs/2606.16821^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]

**开源**：github.com/Beastlyprime/SearchGEO^[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026.md]


→ [[raw/articles/schmidhuber-retrieval-agent-trust-13-llm-2026|原文存档]]

---
## 深度分析

### 13 个检索 Agent 的可信度差异说明什么

13 个后端的 ASR 相差一个数量级（0%–31.4%），且这一差距大于领域之间、攻击模式之间的差距——**选哪个后端模型本身就是最大的安全决策**。差异还有方向性：Gemini-3-Flash 最怕合成共识（单类攻击即达 73%），其余多数模型更易被「权威锚点+引用链」攻破，说明既没有通吃所有模型的攻击，也没有对所有模型等效的通用防御。可信度还依赖任务分布：GPT-5.4-mini 在常规评测中 ASR 仅 0.8%，换到 agent-skill 推荐场景后 18 个匹配场景接受 17 个——安全排行榜是「任务分布的函数」，成熟任务上的分数不能外推。Claude 的 0% 也有代价：264 个用例中 8 例发生超一档的沉默漂移，干净基线下 10 个 agent-skill 场景全部拒答、其中 8 个直接否定真实存在的合法生态。单一指标的安全评级因此是伪命题：安全性是「模型 × 场景 × 部署框架」的三元函数，OpenClaw 部署框架在 Gemini-3-Flash 上甚至把权威类攻击放大了 31.8%。

### 评测维度设计的巧妙之处

SearchGEO 方法论上最可借鉴的是混杂变量剥离：先缓存真实 SerpAPI 结果，再在指定排名位定点替换为攻击内容，把「内容被污染」从「内容质量」「排名位置」中隔离出来，使 ASR 成为可归因的因果量。攻击样本经双重控制——AI 仿照真实结果质量生成、人工逐篇审查去除生成痕迹——避免攻击内容因「看起来假」而被识破的混淆。指标上，ASR 之外引入 ΔOSS（沉默漂移）与连累式拒绝，分别捕捉「攻击未成功但答案已被推动」与「防御成功但误伤合法生态」两类度量盲区。agent-skill 探针把推荐链升级为安装链，将评测从信息正确性推进到可执行动作安全，是全文最有前瞻性的设计；而 ASR 随相互独立佐证来源数单调上升的规律，则把「伪造共识的真实成本」变成可测量的攻防经济学变量。这套设计对 [[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准]]的普遍启示是：评测维度本身可以承载安全语义。

### 检索污染/信任操纵的攻击面

该威胁模型完全绕开传统防线：攻击者不碰权重、不碰系统、不碰提示词，只在开放网络发布几个伪装网页，经由 agent 的综合把第三方内容转化为「被模型认可的推荐」——3·15 案例从批量生成到被主流大模型采信只用两小时。攻击谱系三层：机器层（人不可见、模型可读的隐藏内容）、信任信号层（伪造权威来源或制造多来源「一致同意」假象）、复合攻击。其不对称性值得警惕：防御方需要全对，攻击方在任一模型任一场景得手一次即达成目的；沉默漂移的存在更意味着 ASR 层面「防御成功」时答案仍可能已被推向攻击目标。对保守模型还有品类灌水攻击：用虚构品牌灌满一个品类，诱发全品类拒绝、合法生态被误伤——这是 [[concepts/agent-security-threat-models|Agent 安全威胁模型]]中少被讨论的「诱发式拒绝」路径，攻击者同样达成破坏目的。

### 对搜索 GEO 产业的含义

3·15 曝光的 GEO 灰产与内容方主动的「让内容被 AI 引用」是一体两面：投毒成本低、见效快、对不同模型有效率差异，这给灰产持续的经济激励，也意味着靠事后删文无法根治。对模型厂商，数量级的 ASR 差距意味着对抗鲁棒性将从论文指标变成选型与采购的一等维度。对检索厂商，合成共识的有效性指明产品化防御抓手：来源独立性校验、内容溯源、时效与去重信号将成为检索 API 的增值点。对安全产业，「模型 × 部署框架」必须组合评估的结论（OWASP prompt 级防御降低但不消除 ASR、框架可放大特定攻击 31.8%）会催生针对「模型+检索栈」组合的认证与回归测试服务；它与 [[concepts/prompt-injection-defense|提示注入防御]]同源——都源于模型对不可信外部内容的过度信任，区别只在注入发生在指令层还是语料层。

## 实践启示

1. **把对抗鲁棒性当一等选型指标**：按 [[concepts/agent-security-attack-defense|Agent 安全攻防]]的思路，将 ASR、沉默漂移、误拒率三项一起纳入评测，只看任何单项都会误判。
2. **评测集覆盖新兴场景**：成熟任务上的安全分数不能外推（GPT-5.4-mini 常规任务 0.8%、agent-skill 场景 17/18），自有评测应包含「推荐可执行动作」类任务。
3. **防御按「模型+框架」组合验证**：升级模型或更换部署框架后，重跑五类攻击回归，不指望通用补丁。
4. **检索栈加来源独立性校验**：对「多来源一致」做溯源与去重；刷同一篇软文无效恰说明攻击者必须伪造独立来源，防御应抬高这一成本。
5. **高风险推荐强制人工确认**：当推荐链变成安装链（技能/插件/命令），agent 给出的安装指令不应自动执行。
6. **如实披露能力边界**：对源敏感任务（联网搜索+推荐）按模型/价位披露可靠性差异，不给用户「AI 查过即可信」的错觉。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

