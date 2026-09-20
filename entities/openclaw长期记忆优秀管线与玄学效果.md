---

title: "OpenClaw长期记忆：优秀管线与玄学效果"
type: entity
created: 2026-07-04
updated: 2026-09-20
tags: [wechat, ai]
rating: v8c8
sources:
  - raw/articles/openclaw长期记忆优秀管线与玄学效果
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# OpenClaw长期记忆：优秀管线与玄学效果

**来源**: 阿里云开发者

**发布日期**: 2026-04-15^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]


**原文链接**: https://mp.weixin.qq.com/s/pLqKTe1J2FkiiquoT4dOJA ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

---

"Memory is limited — if you want to remember something, WRITE IT TO A FILE. ‘Mental notes’ don’t survive session restarts. Files do." ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

—— OpenClaw AGENTS.md 默认模板^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]


对于 AI Agent 来说，“记住”是最基础也是最难做好的能力之一。当前的大语言模型在单轮对话中表现出色，但一旦会话结束，所有上下文都从窗口中消失。如何让 Agent 在多轮、跨天的交互中稳定地记住用户的偏好、事实和决策，以及值得记录的事件？OpenClaw 给出了一套以 Markdown 文件为载体的多层记忆体系，其管线覆盖记录、演进、召回全流程——设计理念优秀， 但其全流程以 LLM 弱约束的方式进行决策，实际记忆效果往往不够稳定 。 ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

本文将从源码层面拆解这套记忆系统的全链路，分析其中的不确定性环节，并介绍 RDSClaw 记忆插件 如何补强这些环节，其在LoCoMo10评测中得到了13.90%的提升效果。 ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

一、OpenClaw 记忆系统全景

OpenClaw 的核心设计原则是： 一切持久状态都是磁盘上的 Markdown 文件 。 Agent 的身份、规则、记忆、工具配置——全部以明文  .md  文件的形式存放在工作区目录下，每次会话启动时按优先级注入系统提示词。 ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

完整的文件体系如下：

文件
用途
加载时机

AGENTS.md
工作区规则、安全边界、红线指令
每次会话（最高优先级）

SOUL.md
Agent 个性、价值观、沟通风格
每次会话

IDENTITY.md
Agent 身份元数据（名字、角色、头像）^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

每次会话

USER.md
用户档案（名字、昵称、时区、个人背景）
每次会话

TOOLS.md
环境配置（设备信息、SSH 主机、TTS 偏好）^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

每次会话

MEMORY.md
长期记忆 （已验证事实、决策、持久学习）^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

仅 DM 主会话

memory/YYYY-MM-DD.md^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

日记忆 （当天观察、临时笔记）
当天 + 昨天自动加载

DREAMS.md
梦境日记（Dreaming 系统输出，仅供人类审查）^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

不自动注入

可以看到，  AGENTS.md  、  SOUL.md  、  USER.md  等文件定义的是 Agent 的身份、规则和用户档案，它们在每次会话启动时被加载，用户和 Agent 都可以在对话中更新（比如  USER.md  的模板明确写着 "Update this as you go" ）。而  MEMORY.md  和  memory/YYYY-MM-DD.md  则是另一套机制——它们承载的是 Agent 在对话中积累的动态记忆，并且有一套专门的写入、演进和召回管线。 ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

下面逐层展开。

二、记忆写入：两条路径

OpenClaw 的记忆写入有两条主要路径，它们共同负责将对话中的信息写入  memory/YYYY-MM-DD.md  日记忆文件。 ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

2.1 Agent 主动写入（LLM 决策）

这是最常用的写入路径。在对话过程中，Agent 可以随时主动调用  write  工具将信息写入记忆文件： ^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

→ [[raw/articles/openclaw长期记忆优秀管线与玄学效果|原文存档]]

## 深度分析

### 一、磁盘即状态：把语义检索问题退化成目录约定

OpenClaw 最激进的决定不在检索算法上，而在存储介质上：身份、规则、记忆、工具配置全部落成明文 `.md` 文件，会话启动时按优先级注入系统提示词。这等于把「Agent 该记住什么」这个语义检索问题，退化成一个目录约定问题——每条信息的层级由路径声明，而非由向量相似度在运行时推断。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

收益是可审计性与近乎为零的基础设施成本：文档即记忆，用户能直接打开、编辑、删除，无需后台服务或向量库在线。代价同样极端——「该归到哪一层」被提前固化，而真实对话里的信息往往横跨多层：一条用户偏好既可能是 `USER.md` 的档案字段，也可能是某天日记忆里随口提起的一句话。系统只能靠加载时机管理这种模糊性——`MEMORY.md` 仅在 DM 主会话加载，日记忆只带当天和昨天；一旦写错层，后面几乎无法补救。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

### 二、写入时机是整条管线最脆弱的一环

写入有两条路径，但两条都不是强制的。主路径是 Agent 在对话中自主调用 `write`——是否写、写什么、用什么格式写，完全由 LLM 在单次推理里决定；默认模板只给出「capture what matters」「write significant events」这类方向性建议，而非结构化提取规则。同一句「用户叫什么、住哪、忌口什么」，不同模型、不同上下文长度、甚至同一模型的不同轮次，落盘结果都可能不同。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

第二条路径 Memory Flush 看似安全网，实质是 Compaction 前的最后一次救济：由 token 阈值（默认 4000）与文件大小阈值（默认 2MB）触发，触发时 `write` 被包装成仅追加模式，只允许写当天日记忆。它覆盖了「长对话压缩前不丢」，却没有覆盖「短对话也要记住」——一轮没触发压缩、Agent 又没主动写的对话，信息就此蒸发。把唯一的兜底挂在触发条件上，等于把记忆的确定性绑定在阈值上。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

### 三、演进与合并：机械去重与统计评分的信息损耗

Dreaming（opt-in，默认禁用）用 Light → REM → Deep 三阶段把日记忆巩固进 `MEMORY.md`，是整条管线设计感最强的部分，也是信息损耗最集中的部分。Light Sleep 摄取候选并去重，但明确不调用 LLM，只用 Jaccard 相似度（阈值 0.9）做字面重叠判断——「用户喜欢苹果」与「用户爱吃苹果」语义同一、字面却可能够不到阈值，同一事实于是在系统里留下多个版本；REM Sleep 把阈值收紧到 0.88，仍未脱离词汇重叠框架。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

Deep Sleep 是最终关口，用六维加权分（频率 0.24、相关性 0.30、多样性 0.15、时效性 0.15、巩固度 0.10、概念丰富度 0.06）叠加阶段加权，再过三重门控：综合分 ≥ 0.80、合并信号计数 ≥ 3、`max(独立查询数, 召回天数) ≥ 3`。关键是没有 LLM 参与语义判断——评分只看被检索次数、出现天数、查询多样性。这带来结构性偏置：一条极重要但只被提过一次的事实（「我对花生过敏」）会输给反复出现却并不重要的信息；而「我下周二飞杭州」等信号积够、门控通过，飞机也已起飞。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

### 四、召回粒度与 token 预算的取舍

召回通道是 `memory_search`，在 `MEMORY.md` 与日记忆文件上检索，支持 builtin（SQLite + FTS，可选配 sqlite-vec 向量扩展）与 QMD 两种后端。最关键的设计是降级路径：embedding 不可用时自动退化为 FTS 全文索引加词法排名——务实，却意味着召回质量是个浮动值，没向量时可能漏掉语义相关但用词不同的记忆。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

召回侧还要与 token 预算直接打架：完整注入的只有那几份身份与规则档案，`MEMORY.md` 只进 DM 主会话，日记忆只带当天和昨天。粒度控制压住了常驻上下文成本，代价是把「记起」变成需主动发起的动作——Agent 是否意识到该检索、查询词取得准不准，直接决定最终效果。Active Memory 插件在主回复前用子 Agent 预取，本质是把该判断从主推理外移，换来的是一次额外调用。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

### 五、评测与可观测性：「玄学」的工程根源

量化结论是 LoCoMo10 总体准确率从 58.18% 升到 72.08%（+13.90%），但四类问题收益极不均匀：事实查询 +28.50%、推理性 +21.60%，时间相关仅 +10.06%、描述性仅 +9.81%。这个分布本身说明问题——提升最大的是「个人事实结构化提取、Evergreen 免衰减存储」直接对应的那一类，收益最小的两类更依赖底层模型自身的推理与生成能力。换言之，这 13.90 个百分点主要量的是「写入与晋升环节的确定化」，而非「系统整体变聪明了」。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

再深一层，「玄学」描述的其实是可观测性缺口。链路上有四个都可能静默失败的环节——LLM 自主判断是否写入、Flush 是否触发、晋升是否过门控、召回是否降级——却收敛到同一个用户可见症状：Agent 记不起来了。每一环都没有独立可观测量，只有最终行为这一个信号，因此无法从一次失败回答反推真正的失效点。缺少分层埋点的记忆系统天然无法做回归测试：改提示词、换模型、调阈值之后没人能说清效果变好还是变坏，只能凭手感归因——这正是「玄学效果」的工程来源。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

这条链路与 [[concepts/memory-derivation-observability|记忆派生可观测性]] 关注同类问题，Dreaming 六维评分属于 [[concepts/memory-consolidation-decay|记忆巩固与衰减]] 的一种具体实现，LoCoMo10 的覆盖边界可对照 [[entities/agent-memory-evaluation-landscape-taobao-survey|Agent 记忆评测全景]] 来读。^[raw/articles/openclaw长期记忆优秀管线与玄学效果.md]

## 实践启示

1. **先给管线分层埋点，再谈调优。** 把写入、晋升、召回拆成三个可独立观测的阶段，各自记录「本轮是否发生、命中或跳过了哪条、走了哪条分支」。没有这层观测，任何阈值与提示词的改动都只能靠手感验证。

2. **别让关键信息的存续依赖触发阈值。** Flush 只在接近压缩阈值时工作，短对话完全在覆盖之外。凡是绝对不能丢的事实（身份、健康约束、硬性偏好），应走一条与对话长度无关的确定性提取路径。

3. **区分事实型与事件型记忆，配不同衰减策略。** 用户画像类事实设为 Evergreen 免衰减，事件与第三方信息按策略淘汰——这既是事实查询类目提升最大的直接原因，也暴露了六维评分中 14 天半衰期对长期事实的伤害。

4. **去重与合并必须有语义环节。** 字面相似度能处理复制粘贴，处理不了同义改写；一旦同一事实保留多个版本，矛盾就会留到召回端去猜。语义判定应发生在写入或整合阶段，最好由一次显式的 CREATE/UPDATE/SKIP/DELETE 承担。

5. **保留确定性降级路径，但要让降级可见。** 退化到词法检索远好过整条链路失忆；但若「当前走哪条路径」不可见，降级就变成静默的质量退化——用户只觉得 Agent 变笨，运维看不出异常。

6. **验收长期记忆要按类别拆开看。** +13.90% 在四类问题上从 +9.81% 到 +28.50% 分布极不均匀，只报总分会让「改了写入管线」和「换了更强的模型」看起来效果一样。上线前应固化按类别拆分的回归集。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

