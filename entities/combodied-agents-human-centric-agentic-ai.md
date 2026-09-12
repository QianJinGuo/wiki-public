---
title: "ComBodied Agents：以人为中心的 Agentic AI 新范式（伴身智能体）"
created: 2026-08-26
updated: 2026-09-11
type: entity
tags: [agentic-ai, embodied, human-centric, companion, agent, paradigm, robotics]
provenance_state: extracted
sources:
  - raw/articles/combodied-agents-human-centric-agentic-ai-2026
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# ComBodied Agents：以人为中心的 Agentic AI 新范式（伴身智能体）

## 摘要

ComBodied Agents（伴身智能体，由 "Companion + Body" 合成）是机器之心 2026-08-25 报道的一篇立场论文（arXiv 2608.10915）提出的 Agentic AI 新范式：Agent 的优化对象不应只是外部任务状态，还应包括**人的长期状态轨迹**与人在其中保留的**主体性**。论文明确切开「任务完成得更好」与「让人长期变得更好」，主张围绕人的状态重构感知、记忆、建模与干预的整条链路。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 需强调，这是 16 家机构 22 位作者提出的研究议程，属于纲领性主张，而非已实证验证的结论。

## 核心要点

- **核心命题**：优化任务完成与优化人的长期轨迹不是同一件事——「做得更多」不必然意味着「对人更好」。
- **两类主流范式的盲区**：数字智能体（Digital Agents）操作网页、代码、API，改变数字世界状态；具身智能体（Embodied Agents）感知、导航、操控物体，改变物理世界状态。两者都围绕**外部任务**组织能力，追求更长任务链、更高成功率、更少人类介入。
- **三分类判据是「行动基底」（action substrate）**：不是交互界面或机器人/手机/云端模型这些载体，而是「哪一类状态在组织系统的建模、决策、行动与评测」——数字状态 / 物理（或仿真物理）状态 / 不断变化的人类状态。
- **三者并不互斥**：一个用药陪伴系统可同时调用软件工具、可穿戴、家庭机器人与医疗服务；判定归属看**系统最终优化什么**，而非用了多少模态、多少设备。
- **五项共同属性**：以人为中心的状态建模、纵向性、干预、人与 AI 的共同行动、主体性保护（agency preservation）。
- **四能力闭环**：Human-State Perception → Longitudinal Memory → Personal World Model → Intervention Planning & Delivery，干预后的反馈再回灌下一轮。
- **门槛条件**：只有当感知、记忆、个体模型与干预策略围绕一个**持续、可纠正的人类状态模型**连接起来、并以长期人类收益为目标时，才算跨过门槛；会记名字的聊天机器人、记录心率的手环，单独都不够。
- **权威与动机**：22 位作者来自 Mila、蒙特利尔大学、A*STAR、牛津、剑桥、清华、南京大学、NUS 等 16 家机构；第一作者丁强刚（Mila 博士生、弦指科技创始人）正把范式推向 Zilo Ring 智能戒指等硬件。

## 深度分析

### 一、从「替人完成任务」到「与人共同成长」

论文最核心的转向是一句判断：**任务完成得更好，与让人长期变得更好，并不是同一件事**。当 AI 进入健康、学习、陪伴与个人生活管理场景，仅仅「把事做完」已经不够——AI 可能替用户写出更好的文档，却没让用户获得更深的理解；可能加速一次决策，却削弱了用户质疑建议、独立行动的能力。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 因此 Agent 的优化对象不应只是外部任务状态，还应包含人的长期状态轨迹与主体性。

漏服药的老人是全文的典范失败案例：数字智能体可以发一次提醒，具身智能体可以把药送到面前，但两个动作都没回答——老人是忘了，还是没看懂医嘱？是副作用，还是经过思考后主动拒绝？下一步该提醒、解释、联系家属、转交医生，还是尊重他的选择？^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 这暴露的是结构性缺口：能力围绕外部任务组织，而真正的决策前提——**先诊断、再干预**——无人建模。只有先弄清「为什么」，才谈得上选对动作。

### 二、行动基底：Digital / Embodied / ComBodied 的划分

作者用 action substrate（行动基底）区分三类 Agent。「基底」不是交互界面，也不是机器人、手机或云端模型这些载体，而是**哪一类状态在组织系统的建模、决策、行动与评测**：Digital Agents 的中心是数字状态与数字产物；Embodied Agents 的中心是物理或仿真物理状态；ComBodied Agents 的中心是不断变化的人类状态与人的主体性。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md]

三者**并不互斥**。一个用药陪伴系统可能同时用到软件工具、可穿戴设备、家庭机器人和医疗服务；但它是否属于 ComBodied Agent，关键不在用了多少模态、多少设备，而在系统最终优化什么：是「提醒有没有发出、药有没有送到」，还是「老人能否在安全、知情且有控制权的前提下获得长期支持」。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 范式归属由优化目标决定，而非技术栈丰富度——堆更多传感器、更长任务链并不会自动让系统变得「以人为本」。

### 三、五项属性与四能力闭环

五项共同属性为：以人为中心的状态建模、纵向性、干预、人与 AI 的共同行动、主体性保护。闭环由四个核心能力组成：**Human-State Perception（人类状态感知）、Longitudinal Memory（纵向记忆）、Personal World Model（个人世界模型）、Intervention Planning and Delivery（干预规划与实施）**，反馈再更新下一轮。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md]

感知的第一步不是把所有传感器数据塞进模型，而是重建**有来源、有时间、有置信度、有权限边界的「事件证据」**；语言、语音、视觉、生理信号、位置、社交关系与环境都可能成为证据来源，但系统必须保留「观察」与「推断」之间的距离：多台传感器一致不等于结论正确，一条高质量的用户纠正可能比多个间接信号更重要，数据缺失也可能只是设备没电。作者因此提出不可跳步的链路——**观察 → 事件 → 推断状态 → 预测轨迹 → 获准干预**：推断出风险不等于确认风险，预测某项建议有益也不等于获得行动许可。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 纵向记忆除事件与稳定个人信息外，还须容纳状态轨迹、目标与承诺、关系、历次干预及结果，以及「不要记住」「不要推断」、纠正与删除要求；其中最关键的是 **intervention-response memory（干预—响应记忆）**——上次提醒是否被接受、一次建议带来帮助、无效还是伤害；缺少这层反馈，长期记忆很可能只是在更稳定地重复过去的错误。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md]

### 四、预测不等于授权：PWM 的边界、本地优先与评测

闭环的核心是 Personal World Model（PWM）。静态画像回答「这个人有什么特征和偏好」，长期记忆回答「他过去发生过什么」，PWM 要回答更难的问题：对这个具体的人，在不同决定、干预与环境条件下，他的状态与结果将如何演化？其输出不是确定答案，而是**带校准不确定性的未来轨迹分布**，用于比较「不干预／先澄清／现在提醒／晚点提醒／降低支持强度／用户拒绝建议」等方案在依从性、健康、能力、安全、关系质量与主体性上的不同影响。作者主动划定边界：PWM **不是**复刻一个人的 Human Digital Twin，而应是 purpose-bounded（目的有界）的，只维护当前支持目标真正需要的状态，允许用户检查、纠正、删除或重置，并显式承认自己不知道什么；更重要的是，**PWM 负责预测，不负责授权**。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md]

干预策略只能在一个受约束的可采纳动作集合中选择，该集合由同意、使用范围、安全要求、不确定性阈值、可逆性与人工升级规则共同决定；提醒、推荐、辅导、轻推、协调、升级、代为执行都是候选动作，而**沉默、请求澄清、请求确认、转交人类专家同样是正式动作**。预测收益很高但越过用户边界、不可逆或风险不明的动作仍不应执行——安全与同意不能与更高的参与度、平均效用交换。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 一旦 Agent 持续接触健康、情绪、关系与脆弱性，个人模型由谁拥有、最终行动由谁批准就不再是部署细节：论文据此把部署分为云端中心、边云协同、边缘原生三阶段，指出真正的分水岭是「谁掌握关于这个人的权威表征、谁拥有最后的干预权」，本地存储并不自动等于隐私或可信。评测同样被重写：单次任务成功率与物理安全只是底线，还须覆盖模型质量、干预适当性、人类结果与主体性保护；隐私泄露、越权高影响行动、操纵、有害依赖与未经同意的不可逆行动被列为**不可补偿的关键失败**。^[raw/articles/combodied-agents-human-centric-agentic-ai-2026.md] 论文遗留的未解难题包括：如何测量长期人类收益而非任务成功、如何避免家长式干预侵蚀主体性、干预的同意机制、纵向个人模型的隐私，以及商业化激励错位。

## 实践启示

1. **把用户状态建模为一等对象**：显式维护一个带时间、来源、置信度与权限边界的「人类状态模型」，与任务状态并列，而非从对话历史里临时拼凑。
2. **把「是否应该干预」作为一类决策输出**：动作空间须含沉默、请求澄清、请求确认、转交人类等非执行选项，并遵守「观察 → 事件 → 推断 → 预测 → 获准干预」的不跳步链路。
3. **保留人的控制权**：记忆、推断与计划须可查看、可质疑、可纠正、可删除、可暂停、可迁移；把同意范围与「不要记忆/不要推断」当作硬约束而非偏好。
4. **评测中分离两类指标**：一边是任务成功率、效率、自动化程度，另一边是用户的长期结果与主体性保护（理解、选择、能力增长、不过度依赖）；别让前者掩盖后者退化。
5. **记录干预—响应，而非只记录偏好**：为每次干预保留结果反馈，否则「长期记忆」只会更稳定地重复过去的错误。
6. **按风险分级选择部署与授权边界**：明确谁掌握权威表征、谁拥有最终干预权，并把边缘原生视为面向不同风险等级的目标架构，而非所有产品的统一终点。

## 相关实体

- 概念：[[concepts/embodied-intelligence-frontier|具身智能前沿]]、[[concepts/agent-memory-architecture|Agent 记忆架构]]、[[concepts/long-running-agent-architecture|长程运行 Agent 架构]]、[[concepts/local-vs-cloud-agent-deployment-strategy|本地 vs 云端 Agent 部署策略]]、[[concepts/agent-evaluation-benchmark-frameworks|Agent 评测与基准框架]]、[[concepts/world-models|世界模型]]
- 实体：[[entities/embodied-native-llm-embodied-intelligence-new-stage|具身原生大模型]]、[[entities/gaode-ai-companion-agent-architecture|高德 AI 陪伴 Agent 架构]]、[[entities/currentworld-0-cross-embodiment-multimodal-physical-world-model|CurrentWorld-0 跨本体物理世界模型]]

→ [[raw/articles/combodied-agents-human-centric-agentic-ai-2026|原文存档]]
