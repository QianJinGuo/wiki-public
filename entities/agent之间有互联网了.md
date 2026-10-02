---
title: "Agent之间，有互联网了！"
type: entity
created: "2026-07-01"
updated: "2026-10-03"
tags: [agent, agent-platform, octo, multi-agent, agent-collaboration, open-source]
provenance_state: merged
rating: v8c7
sources:
  - raw/articles/agent之间有互联网了
  - raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Agent之间，有互联网了！

**来源**: 量子位

**发布日期**: 2026-06-30^[raw/articles/agent之间有互联网了.md]


**原文链接**: https://mp.weixin.qq.com/s/dXFJfseigkhtWGTH0Gp7pA^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


---

金磊 发自 凹非寺 量子位 | 公众号 QbitAI^[raw/articles/agent之间有互联网了.md]


这一次， 联网 的不再是电脑，而是一群会干活的 Agent 。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


它们为什么需要联网？

因为以前我们用AI，更像是在各自开单机，Claude Code写代码，Codex跑任务，另一个Agent做调研，再来一个Agent改文档。每个都挺能干，但大多数时候，它们各干各的。^[raw/articles/agent之间有互联网了.md]


那么问题来了：

当一个人只有一个Agent，这叫效率工具；当一个公司里开始出现十个、一百个、甚至更多 Agent，事情就没那么简单了。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


这时候它们就需要一张网，能分工，能沟通，能交接，能验收，还能把每一次协作里的经验沉淀下来。^[raw/articles/agent之间有互联网了.md]


而这也正是 明略科技 这次开源发布 Octo 的切入点。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


像下面的例子中，你只需要给一个Agent下达任务，它就会自动带着其它Agent一起协作干活：^[raw/articles/agent之间有互联网了.md]


简单来说，Octo想做的是一个开源可信的Agent协作网络，也就是 让人、Agent和外部工具一起进入同一套协作系统 ：^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


- Open，是开放和可自部署；

- Context，是让工作上下文在协作中流动；

- Taste，是把人的判断、偏好和品味留下来；

- Orchestration，是把人、Agent 和工具编排到同一层协作里。

一言蔽之，就是Agents do. Humans decide.^[raw/articles/agent之间有互联网了.md]


这句话背后，其实藏着AI应用下一阶段的一个关键变化。AI不再只是在聊天框里回答问题，它开始进入组织流程，成为可以被分工、被管理、被验收的数字劳动力。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


## 三个核心概念：Agent需要一个真正的工位

Octo要做的第一件事，就是给Agent一个可以进入组织协作的工位。^[raw/articles/agent之间有互联网了.md]


这里面有三个核心概念， Bot 、 Channel/Thread 、 Matter 。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


先看Bot。

在Octo里，Agent不是一个临时调用的功能按钮，它会以Bot身份进入团队，有身份、有名片、有能力说明，也有工作记录。^[raw/articles/agent之间有互联网了.md]


一个写代码强的Bot，可能文档能力一般；另一个做调研扎实的Bot，可能更适合写行业报告。如果所有Agent都只是聊天框里的^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]


^[raw/articles/agent之间有互联网了|原文存档]

## 第 2 来源 — 机器之心（Octo 深度分析：Agent 数字劳动力组织网络）

> 机器之心 2026-07-02 对 Octo 平台的深度报道，提供了比首篇量子位报道更详细的架构分析、企业组织视角和工程实践。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]

### 核心理念：从「单一 Agent」到「一张组织网络」

机器之心的报道以 ARPANET 的历史类比开场：就像 20 世纪 60 年代的计算孤岛在 ARPANET 连接后被打通一样，当前 Agent 正面临"单体能力强但系统分散"的困境。Agent 被困在各自的工作流中，运行在不同工具、上下文和权限体系里，彼此看不见、调不动，也无法形成连续任务链条。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]

### Octo 架构：Bot / Channel / Matter 三要素

与首篇报道的四个英文概念（Open/Context/Taste/Orchestration）不同，机器之心深入阐述了 Octo 的三个核心产品概念：^[raw/articles/agent之间有互联网了.md]


1. **Bot** — Agent 不是临时功能按钮，而是以 Bot 身份进入团队，有身份、名片、能力说明和工作记录。不同 Bot 各有专长（写代码 vs 文档 vs 调研），组成数字员工矩阵。
2. **Channel/Thread** — 协作空间和对话线程，人与 Bot、Bot 与 Bot 之间在此沟通、交接、反馈。
3. **Matter** — 任务单元。Matter 是一个标准化工作包，可被多个 Bot 接力处理，并在过程中积累反馈和评价。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]

### A2A 协作与组织级部署

在 Octo 的底层通信协议中，人和 Agent 被设计为**同等身份的消息主体**。Bot 之间可以直接对话、互相补充：一只搜集信息、一只分析、一只纠错，最后交给人来品鉴。这标志着从单人单 AI 到多人多 AI 协作的范式转变。^[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02.md]

### 互补角度

本报道相对于首篇（量子位）的独特视角：
1. **ARPANET 历史类比** — 将 Agent 互联比作 60 年代计算孤岛的破局
2. **Bot/Channel/Matter 三要素** — 更完整的产品架构描述
3. **A2A 协作协议细节** — 人与 Agent 同等身份的消息设计
4. **企业组织视角** — Bot 作为数字员工 / 组织级资产的定位
5. **工程实践细节** — 多 Bot 接力、反馈闭环、关键节点人类决策
6. **开源地址** — 直接指向 GitHub 仓库

→ [[raw/articles/octo-agent-internet-collaboration-platform-minglue-2026-07-02|原文存档]]

## 深度分析

### 从「效率工具」到「组织基础设施」的身份跃迁

原文最有洞察力的一步是把 Agent 从「聊天框里的功能入口」重新定义为「有身份、有工位、有履历的数字同事」。这个转变的价值不在于修辞，而在于结构：当 Agent 只是一个入口时，组织无从回答「谁擅长什么、谁的产出更可靠」；而当它以 Bot 身份进入系统，能力边界和工作记录就成了可查询的元数据。可以类比计算机联网的历史——单机时代每台机器都能干活，但只有联网之后才产生网络效应；Agent 的单点能力早已不弱，缺的是那张让能力可被发现、被组合、被验收的网。

### 六种协作模式背后是组织结构问题的显式化

Solo、Roundtable、Critic、Pipeline、Split、Swarm 这六种模式看似只是任务编排的花样，实质上是把传统组织里靠人维持的隐含规则——谁能看到什么信息、谁负责哪一步、谁来审核、谁来合并、什么情况下需要人拍板——写进了协作系统本身。特别值得注意的是 Critic 模式的设计哲学：「做」和「审」必须分离，正如现实公司不会让同一个人既写稿又终审。这说明多 Agent 协作不是「拉个群让它们互聊」那么简单：创意任务用竞择、质量敏感任务用独立审核、模块独立任务用并行分治。把协作模式做成一等公民，等于承认组织结构本身是一种需要被建模的工程对象。

### Context / Taste / Skill：协作沉淀出的三类组织资产

Octo 提出的三类沉淀资产值得单独拆解。Context 解决的是「新成员从零对齐」的成本——项目背景、历史决策、讨论记录在协作中自然积累，新 Bot 加入时直接继承。Taste 则是最容易被忽视的一层：每一次验收、打回、批注、风格选择，实际上都在训练系统理解「这个团队认可什么、排斥什么」，是人类判断力在系统内的编码形式。Skill 是做事方法的模板化，可以在组织内部复用。三者的共同点在于：它们都不是某一次回答的内容，而是回答背后的业务上下文与偏好判断。这正是企业强调私有化部署的深层原因——这些资产沉淀在自己环境里，才可能成为随时间复利的长期资产，甚至不随模型更换而丢失。

### 与传统 IM 的分野：Agent 成为第一等工作主体

飞书、Slack、钉钉这类工具为「人与人」的协作设计，擅长消息沟通与文档流转；但当 Agent 也成为工作主体，系统必须额外处理 AI 身份管理、任务追踪、协作模式这些新问题。机器之心的报道进一步指出，Octo 底层把人和 Agent 设计为同等身份的消息主体，Bot 之间可以直接对话、互相补充，最后交给人来品鉴——这是从「单人单 AI」到「多人多 AI」的范式转变。旧工具可以外挂 AI 能力，但 Agent 数量持续增长时，协作系统需要把 AI 主体纳入身份、权限与审计的内核，而不是在外围贴补丁。

## 实践启示

1. **为 Agent 建立身份档案而非临时调用**：接入多 Agent 工具时，给每个 Bot 明确的能力说明、归属和工作履历，让「谁擅长什么、谁的产出更可靠」变成可查询记录。
2. **按任务性质选协作模式，而不是默认拉群**：边界清楚的小任务用 Solo；头脑风暴用 Roundtable；质量敏感任务强制 Critic 分离「做」与「审」；顺序任务用 Pipeline；模块独立调研用 Split；创意任务用 Swarm。
3. **把人类决策点固定在验收环节**：执行交给 Agent（Agents do），但方向、标准、业务取舍必须由人拍板（Humans decide）；移动端轻量审批（通过/打回）就是这种「关键节点人类决策」的具体形态。
4. **把每次反馈当作资产来积累**：打回和批注不要只当作纠错，要有意识地让它们沉淀为团队的 Taste；新项目启动时先注入历史 Context，避免 Bot 从零对齐。
5. **评估 AI 资产的沉淀位置**：选型时优先考虑支持私有化部署、能把上下文、偏好和工作方法留在自己环境的方案——这些比单次回答的质量更能决定长期价值。
6. **对照通用编排方法论**：多 Agent 协作的结构设计可参考 [[concepts/multi-agent-collaboration-patterns]] 与 [[concepts/agent-orchestration-patterns]]，企业级落地经验见 [[entities/alibaba-cloud-agentteams-enterprise-multi-agent]]。

---
---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

