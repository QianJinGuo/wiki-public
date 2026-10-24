---
title: "RoboSkill：Explore-Execute-Evolve，让具身 Agent 把摸索出的方法存成可复用 Skill"
created: 2026-10-24
updated: 2026-10-25
type: entity
tags: [embodied-agent, skill, robotic, vla, self-evolution, fudan, arxiv]
sources: [raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026]
confidence: 0.76
provenance_state: extracted
---

# RoboSkill：Explore-Execute-Evolve，让具身 Agent 把摸索出的方法存成可复用 Skill

复旦大学可信具身智能研究院（吴祖煊、姜育刚课题组，与上海新智具身合作）提出 RoboSkill（arXiv:2609.37810，代码 github.com/SII-dannyXSC/RoboSkill），处理的是具身 Agent 从"能完成"到"高效完成"之间的那道坎：通用 Agent 已经能现场观察、推理、写代码操作机器人，但每次任务都从零摸索，时间成本高得不可接受。

## 背景：VLA/WAM 的数据墙 vs 通用 Agent 的时间墙

- **VLA / 世界动作模型**通常先用机器人数据预训练再适配具体任务，部署到新场景往往还要补一批目标场景数据。采集成本极高，且操作员习惯、场地物品有限，采得更多不一定见得更广——换物体、换摆法就又失效。
- **通用多模态 Agent**（如 GPT-6-Astra、Opus 5）绕过了数据墙：不预采示范、不做针对性训练，靠现场观察 + 写代码就能操作机器人。代价是慢—"观察—推理—执行—检查结果"反复循环，动作之间的推理常常持续几分钟。

RoboSkill 的定位正是把这两条路接起来：**用触觉反馈压缩探索成本，用技能积累复用探索成果，再通过执行不断更新技能**，即 Explore → Execute → Evolve。

## 基线：Agent 已经很强，但很慢

| 设置 | 结果 |
|---|---|
| LIBERO-10（四小时预算、多轮尝试、不用已有技能） | GPT-6-Astra 与 Opus 5 成功率 100%；Fable 5.1 97.5%；GPT-5.6-Sol 82.5% |
| 真实机器人（6 简单 + 6 高难任务） | GPT-6-Astra 成功率 91.7% / 80% |
| 仿真平均耗时 | GPT-5.6-Sol 94.5 min、Opus 5 80.9 min |
| 真机平均耗时 | GPT-6-Astra 简单 20.3 min、高难 27 min |

成功率不再是瓶颈，**执行时间才是**。这也构成一个方法论前提：评估具身 Agent 时若只看成功率，会掩盖"每次都要重新摸索"的真实代价。

## 机制一：触觉反馈让 Agent 更快确认物理状态

"夹爪有没有碰到物体""机械臂向下移动时是否已经接触桌面"这类判断仅靠图像很难确认，Agent 只能反复小幅调整再确认。压力、力与力矩等接触信息提供了直接依据。

- 仅视觉的 GPT-6-Astra 最终也能完成全部仿真任务，但加入触觉后平均耗时 **25.35 → 15.93 min**；
- 真机"取出试管"任务成功试验平均耗时 **33.63 → 20 min**。

要点不是触觉提升了成功率，而是**触觉把"状态确认"从视觉推断变成直接读数**，缩短了每一轮 observe-verify 循环。

## 机制二：Skill = 文字策略 + 实际执行过的代码

任务结束后 Agent 回顾执行过程，把有效方法整理为 Skill，并根据后续执行结果继续更新。关键设计分歧在于**技能里留什么**：

- 只留文字（如"先测量桌面高度"）给了策略，但如何移动、读反馈、判断接触仍需重新写程序；
- RoboSkill 把**文字指导 + 实际跑通过的代码**一起存：文字说明适用条件与操作策略，代码保留具体实现，供新场景修改调用。

实测（技能从"仅文字"变为"文字＋代码"）：四种 Agent 平均耗时均下降，GPT-5.6-Sol 从 **67.1 → 39.9 min**，一次成功率从 **15% → 67.5%**。技能积累本身的收益：仿真平均耗时 **94.5 → 35.9 min**。

这一点与 LLM Agent 侧的 Skill 工程是同一命题的不同载体——留"可执行起点"比留"自然语言结论"复用率高得多（对照 [[concepts/skill-engineering-principles|Skill 工程原则]]、[[concepts/skill-framework-writing-patterns|Skill 框架写作模式]]）。

## 机制三：跨任务、跨模型、可持续更新

实验进一步表明技能可以**跨任务节省时间、跨模型改善表现，并通过后续执行继续更新**。把单次任务的效率问题与长期技能积累连起来，是作者称为"具身 RSI（recursive self-improvement）"的路径：探索不再是一次性开销，而是能力的增长起点。

## 边界与怀疑点

- 数据来自团队自建评测（LIBERO-10 + 真机 12 任务）+ 自家架构，技能"可迁移"的范围限于同类操作与相近本体；
- "文字＋代码"技能的成功率跃升（15% → 67.5%）幅度极大，需注意基线本身较低、任务集有限；
- 技能库变大后的检索、冲突与陈旧化（技能失效检测）未在本文展开，而这恰是 Skill 系统在生产环境的主要失败模式。

## 深度分析

**一、承重的是「可执行证据」，不是「经验」。** 文字策略只能被判断"像不像"，代码可以被执行、被观测、被否证：跑一次，接触信号有没有出现、夹爪停在哪里，都是硬事实。这解释了 15% → 67.5% 的跃升为何可能：未必是 Agent 变聪明了，而是"从文字重新推导动作序列"这段最易出错的环节被整个删掉^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:80-84]。对 LLM Agent 同理：Skill 的价值不在"记录做过什么"，而在"留下一个已通过检验、可被 diff 的起点"。检验标准很直接：删掉正文，只留代码和条件，还跑得通吗。

**二、Explore-Execute-Evolve 是信用分配问题，原文却只给了信号、没给分配器。** 对"进化"的全部交代是"根据后续执行结果继续更新"^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:74]。一次失败，该归咎于适用条件判错、阈值过时，还是任务本就不属于这个技能族？没有分层归因，"进化"就退化成"失败即整条重写"——技能库最贵的操作。失败技能如何处置也缺位：删除？降权？还是标注"仅在本体 X 上有效"？只追加不裁剪的库，检索噪声单调上升，最终吃掉复用的收益（对照 [[entities/regression-tax-skills-hurt-llm-agents|技能的回归税]]）。

**三、触觉是延迟工具，不是感知升级。** 原文的对照极干净：仅视觉的 GPT-6-Astra 最终也能完成全部仿真任务——成功率没被触觉改变，被改变的是 25.35 → 15.93 分钟；真机取试管是 33.63 → 20 分钟^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:68]。触觉不动模型能力，动的是"每一轮 observe-verify 要多久"这个乘数，即样本预算。推论可检验：瓶颈不在状态确认的任务（长程规划、多物体排序）收益应缩小，"碰到了没有"这类二值判断密集的任务收益应放大。代价则被略过——力／力矩传感是机器人硬件里标准化最低、标定最麻烦的部分。映射到软件 Agent，类比不是"多装传感器"，而是"用确定性探针替代自我询问"（参见 [[entities/vla不够了触觉将改写具身智能新格局|触觉改写具身智能]]）。

**四、迁移必然降级为「任务族迁移」，但代码载体让跨模型迁移成立——后者更硬。** 原文例子很直白：换场景高度要重测，但测量步骤已有方法可循^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:74]。可迁移的是过程形状（接触探测→力突变→抬升），不可迁移的是常量（高度、位姿、阈值）——"跨任务"实际是"同族任务、不同参数"，不是缺陷而是物理上限。真正值得注意的是"跨模型改善表现"^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:86] 的机制：上下文里的经验随模型更换而失效，落盘的代码不会。也正因如此才谈得上"具身 RSI"——模型是替换件，技能库才是资产（对照 [[concepts/agent-self-improvement-loops|Agent 自我改进循环]]、[[entities/nvidia-aspire-robot-skill-library-code-as-policy|代码即策略的机器人技能库]]）。

**五、技能库的边际收益可以是负的，原文默认了它单调为正。** 全部实验都是"从零到有技能"的增量，没有 50 条到 200 条的规模扫描，也没有陈旧技能检测。生产环境里 Skill 系统的主要失败模式恰在后者：语义相近但条件不同的旧技能、两条技能对同一状态给出冲突动作、环境改版后静默失效。具身上还多一层：一条"跑通过"的策略被当模板复用，偶然成立的细节固化成方法，下一台就是撞机动作；而 Agent 自己写、自己存、自己调，没有权限边界，等于把能力累积与错误累积绑在同一根曲线上（安全侧参照 [[entities/roboharm-benchmark-robocurve-robot-safety-2026|机器人安全基准]]）。

**什么样的实验能推翻它？** 三个：把技能库规模做成 0 / 10 / 50 / 200 的扫描，若耗时在某点后回升，"积累即收益"就不成立；加一组"条数相同但内容随机"的对照，剥离上下文变长本身的影响，否则收益可能只来自 prompt 更长；把评测从"固定预算下的成功率"换成"成功率对齐后的 time-to-solve（p50/p90）"——本文结论全在时间维度上，基线则是四小时预算、多轮尝试的极宽松设置^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:50]（[[concepts/agent-evaluation-benchmark-frameworks|Agent 评测基准框架]]）。

## 实践启示

1. **技能里存"能跑的起点"，不存"结论"。** 除说明文字外，附一段真实执行过的脚本及其前置条件；只写"应该先测高度"，等于把推导成本原样留给下一次^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:78-84]。
2. **把适用条件写成触发条件，而不是任务描述。** 文字承担"什么时候用它"，代码承担"怎么用它"——这个分工直接决定检索时该命中还是跳过；写成一堆步骤的技能既命中不准，也无法执行。
3. **用 time-to-solve 而不是成功率做验收。** 成功率饱和时（基线里 Astra 与 Opus 5 都是 100%），唯一还在变的量就是耗时；报 p50/p90 和"第几次成功"，才看得见重复摸索的代价^[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026.md:50-52]。
4. **给技能库配失效与回滚，别只配写入。** 每条技能至少记 last_verified、适用本体／版本、失败计数；超阈值降权而非直删，留作负样本，免得下个 Agent 重走同一条死路（[[concepts/catastrophic-forgetting|灾难性遗忘]] 的反面同样成立：该忘的没忘，比忘了更贵）。
5. **用确定性探针替代模型的自我确认。** 软件 Agent 的"触觉"是 exit code、断言、健康检查、schema 校验——把"我看看这样对不对"换成"读一个布尔值"，收益不在准确率而在循环次数（[[concepts/verifier-driven-development|验证器驱动开发]]）。
6. **允许 Agent 写可执行策略的系统，必须先画权限边界。** 一次侥幸成功会被固化成模板：对物理系统是撞机，对生产系统是把偶发行为变成默认。上线前至少要有沙箱执行、人工抽检、一键回滚。

## 关联

- [[concepts/embodied-intelligence-frontier|具身智能前沿]]
- [[concepts/world-models|世界模型]]
- [[concepts/skill-engineering-principles|Skill 工程原则]]
- [[concepts/skill-framework-writing-patterns|Skill 框架写作模式]]
- [[moc/ai-skill-design|AI Skill 设计主题地图]]

→ [[raw/articles/roboskill-explore-execute-evolve-skill-acquisition-embodied-agents-fudan-2026|原文存档]]
