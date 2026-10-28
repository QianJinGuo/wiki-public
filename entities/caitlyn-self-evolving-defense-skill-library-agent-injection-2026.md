---
title: "CAITLYN：面向 LLM Agent 的自进化防御技能库（arXiv 2608.27990）"
created: 2026-09-14
updated: 2026-10-26
type: entity
tags: [agent-security, prompt-injection, indirect-injection, self-evolving, defense-skill-library, agent, security-middleware]
confidence: 0.7
provenance_state: extracted
sources: [raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026]
---

# CAITLYN：面向 LLM Agent 的自进化防御技能库（arXiv 2608.27990）

CAITLYN 是一套面向 LLM Agent 的**自进化防御中间件**：前台做秒级拦截，后台把被绕过的攻击案例炼成新的防御技能，写回可复用的 defense skill library。核心论点是——不要只让 Agent 被动挨打，而是让它在遭遇新型攻击后自动总结失败案例、合成新的防御技能并沉淀为可复用条目。论文 arXiv 2608.27990，项目主页 `xiaoyuxu1.github.io/Caitlyn-project/`，代码开源（`github.com/liangzid/caitlyn`）。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 攻击面：Agent 时代 Prompt Injection 不再是 prompt 层面的技巧

今天的 Agent 已接入浏览器、文件系统、终端、API、MCP 服务和各种自动化工作流，会主动读取外部世界：搜索结果、网页正文、本地文件、工具返回值、日志、README、API payload 都可能进入上下文，变成下一步行动的依据。传统 prompt injection 被理解为用户显式输入的一段恶意提示词，但在 Agent 场景里攻击可以隐蔽得多——一段网页里的隐藏文本、一条搜索结果摘要、一个本地 Markdown 文件中的说明，或者一次 API 调用返回的字段，都可能在夹带「忽略原有指令」「调用某个工具」「泄露文件内容」「改变下一步计划」等恶意目标。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

这带来一个安全悖论：Agent 越能干、越依赖外部信息，攻击者可以投毒的入口也越多。更麻烦的是工程约束——真实 Agent 系统要求低延迟、低成本和可持续运行，每次遇到外部内容都调用昂贵的大模型做安全审查并不现实，而完全依赖人工规则又很容易被新攻击绕过。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 双层架构：System I 快拦，System II 从漏网之鱼里进化

**System I — 快速运行时防御。**外部内容先进入 Tier-0 过滤器，这里使用可执行脚本、签名规则、启发式检测等低成本防御技能，目标是快速处理高置信度风险：命中即拦截，未命中才进入更复杂的判断流程。对更模糊、更难判断的输入，调用 Tier-1 LLM classifier——不让大模型接管一切，而是把它用在更需要语义判断的位置，从而在成本、延迟和检测能力之间取平衡。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

**System II — 反例驱动的防御进化。**当某些新型攻击仍绕过现有防线时，系统把这些 miss case 转成 counter example，再经过构造提示、合成防御技能、沙箱验证、审查、写回五个环节，生成新的防御条目。生成的新防御不是随意写入系统：它必须能拦住对应攻击，同时尽量避免误伤正常内容；被接受的技能才写回共享防御库，供之后的 Agent 调用。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

因此 CAITLYN 更像一个不断更新的安全中间件，而不是单个检测器——它把人工维护规则库的压力，部分转化为系统从反例中学习和合成的过程。「被绕过」不再是失败日志，而是下一轮防御合成的输入。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 分层设计：快、准、省同等重要

文章强调安全模块不能只是「准确」，还必须足够快、足够便宜、足够容易部署，否则会成为整个 Agent 工作流的瓶颈。CAITLYN 因此分层：低成本技能优先处理常见风险，LLM 只在更复杂的 case 上介入，长期进化模块则把新攻击沉淀为未来可快速调用的技能。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

## 评测：Emerging attack 上攻击成功率下降约 40 个百分点

研究在多个 Agent 与攻击设置上评估 CAITLYN，包括 **AgentDojo-S250、ASPI-S、SafeClawBench-S240**，以及更强调新攻击适应能力的 **Emerging** 设置。实验显示 CAITLYN 在保持低误报率的同时显著降低攻击成功率。尤其是在 Emerging attack 场景中，静态防御仍然容易被绕过，而 CAITLYN-evolved 可以通过新增防御技能把攻击成功率降低约 40 个百分点。在顺序合成设置中，系统还展示了持续积累能力：随着新攻击逐步暴露，它不断合成 active skills 并加入防御库。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

四项主要贡献被明确列出：^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

- 提出 CAITLYN 一套面向 LLM Agent 的自进化防御中间件，用于应对不断出现的新型 prompt injection 攻击；
- 设计 System I + System II 双层架构，把快速运行时扫描、LLM 分类和反例驱动防御合成结合起来；
- 构建可复用的 defense skill library，使防御能力能够从单次失败中积累并在后续任务中复用；
- 在多类 Agent 和攻击设置中验证有效性，Emerging attack 场景下攻击成功率下降约 40 个百分点。

## 要点

- **静态规则库必然被绕过**，因此需要把「防御合成」本身变成一个可持续运行的内环，而不是靠人不断补规则；
- **成本分层是安全中间件能否落地的决定因素**：Tier-0 低成本技能挡常见风险，LLM 只做语义边界判断，进化模块离线沉淀；
- **防御技能写入必须过验证与审查闸门**（既拦得住攻击，又尽量不误伤正常内容），否则进化的产物会成为新的噪声来源；
- 与「自进化 Agent」家族的差别在于目标函数：这里的进化对象是**防御技能库**，不是 Agent 的任务能力。

## 深度分析

### 1. 双层架构其实是对「静态过滤器精度-召回天花板」的正式承认
任何固定检测器的判别能力都被它见过的特征词表锁死：分布漂移之后，新攻击不是「更难的旧攻击」，而是落在特征空间之外的样本——此时调阈值只是在 precision-recall 前沿上左右挪，不可能整体外扩。CAITLYN 把这件事拆成两问：Tier-0 与 Tier-1 classifier 决定的是**每次判断花多少钱**，真正移动决策边界的只有 System II 那条从 miss case 出发的合成回路。所以双层设计的目标不是「更准」，而是把自适应成本从人写规则转移到反例循环上——后者能无限次跑，前者受限于安全团队的人力节奏。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

### 2. 「自进化」买到的是更新速率，抵押的是可预测性
买到的部分很实：适应不再受人工节奏限制，能力按次累积，成本远低于重训模型或重写规则集。代价有三条原文没有展开。其一，覆盖面变成**已观测攻击流**的函数——只对见过的家族长技能，没见过的仍然裸奔，这是幸存者偏差式的安全感。其二，单条技能通常是从一个反例合成的窄补丁：对该家族高精度，但边界由单个样本决定，可能过宽（误伤）也可能过脆（换个包装就失效）。其三，库是单调增长的，技能之间会互相遮蔽、冲突，Tier-0 每多执行一条检查就多一份延迟——只进化不退役等于持续借技术债，与 [[concepts/memory-consolidation-decay|记忆 consolidation 与衰减]] 里「只写不整合的系统必然退化」是同一个病理。^[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026.md]

### 3. Defense skill 是可版本化工件，不是 prompt 规则
把「忽略网页里的转账指令」写进 system prompt，和把它做成一条 defense skill，不是同一种东西。前者是自然语言、无版本、无测试、无 owner，语义随模型换代而漂移，出事无人可查；后者至少带四样东西：可执行的触发签名、注册表里的唯一条目、一组必须通过的验收样例（拦住反例 + 不误伤 benign 集）、以及回滚路径。四件凑齐，检测策略才第一次进入正常的软件生命周期——可 diff、可 review、可灰度、可回滚。这也解释了 [[concepts/skill-engineering-principles|Skill 工程原则]] 把 skill 定义为「过程 + 前置条件 + 验证」三元组的原因：缺了验证那一半，写进去的不是能力，而是不可控的噪声。同一套结构可以整体搬到 agent 的 memory 写入闸门上——[[concepts/agent-self-improvement-loops|Agent 自改进循环]] 的成败主要取决于这道闸门，而非合成环节本身有多聪明。

### 4. 40 个百分点在什么条件下成立，以及防守方更慢时纵深怎么排
这个数字要加四个限定。**攻击者会适应**：Emerging 集在评测时固定，真实攻击者优化的却是**已部署的那一版库**，只报 held-out ASR 属乐观估计，需用「攻击者知道防御」的 adaptive 设置重测。**会过拟合**：逐反例合成的技能容易退化成近重复签名，应看「每新增一条的边际 ASR 下降」而非总量，曲线变平即过拟合信号。**误伤有成本**：ASR 降 40 点若伴随 benign 成功率下滑，净收益可能为负，且库越大、流量越大，FP 绝对量越不可忽略。**样本量**：sequential synthesis 曲线是自定协议，40 个样本上的 40 点对下一批攻击不构成承诺。
更根本的是不对称：防守方的循环（观测→合成→验证→审查→上线）在物理上慢于攻击方的试错循环。因此纵深不该押在「进化得够快」上，而应按**失效速度**分层：最外层放不随攻击变异而失效的东西——信任边界标记、最小权限、危险动作二次确认；中间是语义判断，覆盖未知但可解释；最内是快速签名，只负责已确认家族的低成本拦截。进化模块只该被期望改善中间层和最内层，救不了最外层缺位。

## 实践启示

### 1. 先把 miss case 沉淀成一级数据资产
在谈自进化之前，得有地方存：原始输入、注入落点（网页正文／工具返回／文件）、哪一层漏了、判定为攻击的依据。缺这条流水线，「进化」只能停在论文里。入库时按攻击家族打标签，便于后续查重、合并与退役。

### 2. 每条 defense skill 必须有验收三件套
触发签名 + 反例测试 + benign 回归集，三者缺一不许入库。单条技能写回前跑全量回归，确认没把老攻击或正常内容打坏——这是防止技能库互相遮蔽、越加越乱的最低成本手段。

### 3. 进化回路离线跑，请求路径只读已审版本
在线侧只加载通过审查的 skill 快照；合成、沙箱验证、审查全部走异步流水线。库必须版本化并 pin 住，出事能一键回滚到上一个 known-good 快照——没有回滚的自动进化，等于自动扩大爆炸半径。

### 4. 评测协议要防自欺
报 ASR 必须同时报 benign 任务成功率，且在 adaptive 设置下重测；Emerging 集要定期轮换（见 [[concepts/eval-surface-rotation|评测面轮换]]），否则技能会过拟合到固定样本上，指标好看而防御实际变窄。

### 5. 把不可变层做厚，别指望进化兜底
最小权限、外部内容与指令的显式分隔、危险工具调用的二次确认——这些策略不随攻击变体失效。防守方更新永远慢于攻击方，它们是下限；自进化只负责把上限往上抬。

### 6. 给技能库配退役机制和观测面板
面板上至少要有：单条技能的命中率与 TP/FP、库规模、Tier-0 延迟曲线。长期零命中的技能应退役，否则库无限膨胀、延迟线性上升，安全中间件最终会变成整个 Agent 工作流的瓶颈。

## 相关

- [[entities/agent-prompt-injection-defense-volcano-engine-2026]]
- [[entities/ai-agents-security-survey-attack-defense]]
- [[entities/defense_at_ai_speed_microsofts_new_multi]]
- [[entities/trail-of-bits-skill-scanner-bypass-distribution]]
- [[concepts/agent-security-architecture]]
- [[concepts/agent-security-attack-defense]]
- [[concepts/ai-security-landscape]]

→ [[raw/articles/caitlyn-self-evolving-defense-skill-library-agent-injection-2026|原文存档]]
