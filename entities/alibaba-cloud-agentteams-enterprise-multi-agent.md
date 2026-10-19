---
title: "阿里云 AgentTeams 企业级多 Agent 平台"
created: "2026-07-14"
updated: 2026-10-20
type: "entity"
tags: [agent, multi-agent, enterprise, alibaba-cloud, agent-teams, sandbox, security]
confidence: 0.8
provenance_state: "extracted"
sources: [raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent, raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 阿里云 AgentTeams 企业级多 Agent 平台

> 阿里云云原生团队推出的企业级多 Agent 协作平台，核心命题不是「一次任务怎么并行跑快」，而是「一个 Agent 组织怎么长期运转」。与 Claude Managed Agents 的设计哲学形成对照。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]

## 四层落地架构

1. **入口层** — 原生客户端、钉钉/飞书/企业微信 IM 集成、HTTP 服务化（不逼员工换工具）
2. **Agent Identity** — 对接企业 IdP/SSO，Agent 工作负载签发身份，操作可归属到人
3. **Agent Team** — 按职能编队（研发/客服/数据分析等），TL Agent 调度，引擎热插拔
4. **统一 AI 资产管理** — 模型/Skill/MCP/Worker 模板集中管理，BYOC 自主可控

贯穿四层：观测、度量、治理中台（Token 消耗、Prompt 分析、效果审计）。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]

## 四道安全防线（零信任 Agent 管理底座）

这是 AgentTeams 与市面上大多数多 Agent 框架的最大差异所在 ^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]：

1. **AI 网关（零凭证持有）** — LLM 调用和密钥统一网关加密托管，Agent 零明文持有凭证；身份认证 + Skill/MCP 指令级拦截
2. **Sandbox 沙箱** — 实例/网络/存储三维物理隔离，爆炸半径锁死
3. **通信安全** — 端到端加密 + Room 机制审计可溯源
4. **Skill 市场** — 安全扫描审核上架 + per-consumer ACL 最小权限授权

## 三层协作架构

| 层级 | 职责 | 类比 |
|------|------|------|
| Manager Agent | 全局监管 + 任务拆解 | CEO |
| Team Leader Agent | 团队调度分配 | VP/TL |
| Worker Agent | 底层执行 | 一线员工 |

与 [[entities/agent-harness-6-runtime-patterns-sdb|Agent Harness 6 种运行时模式]] 中提到的分层委托模式一致，但 AgentTeams 强调 TL 层解决管理幅度问题。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]

与 Claude Managed Agents（CMA）的关键区别：CMA 只有两层（Lead + Teammates），Lead 不可转移；AgentTeams 的 Manager/TL 均为独立实例可灵活调整。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]


### 引擎热插拔（协议层解耦）

底层引擎可混编——QwenPaw、OpenClaw、Claude Code 可共存于同一 Team。类比 Kubernetes CRI：编排层与引擎层解耦，企业不会被单一技术栈锁死。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]

## Sandbox 沙箱运行时

基于 ACS Sandbox，单 Session 单 Sandbox，三维隔离 + 三种弹性伸缩方式 ^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]：

1. **Session 级扩并发** — 多人交互拉起多个独立 Sandbox
2. **Team 级多副本分流** — 多实例负载均衡
3. **Subagent 同进程编排** — 一个 Worker 内建多个 Subagent 零网络跳数

**深休眠机制**：无请求时快照保留现场不产生费用，有请求秒级拉起。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]


## Agent 持续进化（AgentLoop 双飞轮）

- **左飞轮**：AgentTeams 协作底座沉淀执行轨迹和协作数据
- **右飞轮**：AgentLoop 调优引擎做清洗、评估、SFT/RLHF 训练

核心链路：**发现**（高频失败 Task）→ **对齐**（企业专属 DPO/RLHF）→ **进化**（Prompt/模型/Skill 优化）^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]


> 真正的护城河不是你用了哪个模型，而是你积累了多少属于自己业务的 Agent 协作数据。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md]

## 设计哲学

- Agent 不是工具，是员工
- 用基础设施思路而非脚本思路对待 Agent
- 声明式管理（`agentteams apply`），类比 `kubectl apply`
- 容器确定性 vs Agent 概率性 → 额外加 Heartbeat/阻塞上报/HITL/全链路可观测

→ [[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent|原文存档]]

---
## 深度分析

### 真正的成本拐点在「第二个 Agent」

单 Agent 立项几乎总能成功，因为成本全是「把事做出来」；到第二个，模型接入、凭证、MCP、知识权限、观测这些底座部分改以乘法扩散，每份重复各自演化成互不相认的孤岛。所以「做第二个不必从头开始」是立项时点判断：底座该在第二个 Agent 立项时就有，而不是等规模。更关键的是，重复建设表面浪费工时，实质是治理漏洞——同一份数据权限被三份代码各自解释，出事时没人答得出谁在什么授权下读过什么。底座价值的另一半，正是把权限、凭证、审计收敛到唯一执行点；安全一旦分片就等于没有。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md:15]^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md:28-38]

### 三层架构管的是管理幅度

三层不该读成「并行更快」。两层结构里 Lead 兼规划与派工，团队过八到十人后派工细节吃满上下文，规划质量下降；插一层 TL 是把管理幅度写进拓扑，而非指望窗口变大。与 CMA 的差别也不只是灵活：CMA 的 Lead 是固定主会话，腐化即整队腐化且不能不停机换人；AgentTeams 的 Manager/TL 是可替换实例，协调者能按无状态组件重启。代价是每层多一次语义损耗、跨层描述逐级失真——所以三层只对可清晰切分的任务划算，需要长期共享隐式上下文的活反而该压进一个 Worker 的 Subagent 链。^[raw/articles/alibaba-cloud-agentteams-enterprise-multi-agent.md:40-51]

### 两个闭环优化的是不同对象

AgentLoop 吃轨迹与失败模式，动 Prompt、模型、Skill、训练集，靠离线测评验收，答的是「能力有没有变强」；Business Loop 吃业务结果，动场景设计、圈选、流程，靠经营指标验收，答的是「问题有没有被解决」。混同的后果很实在：把调用量当价值，或把一次业务波动误判成模型退化去白跑微调。飞轮还有个隐藏前提——运行次数增加不会自动变好，中间必须有归因，判定失败是能力、数据还是场景设计之过；缺归因，轨迹只是日志堆积，左轮不转则右轮无粮。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md:31-33]

### 复用能力不等于共享数据

Skill 复用的是「方法」，数据授权管的是「事实」的可见范围，两者正交：能力铺到四千家门店，不意味着四千家门店的数据彼此可见。前提是 Skill（流程与代码）与 Workspace（数据与凭证）做成两个独立授权维度，而不是把 Skill 打包成连数据一起分发的单元；否则「统一资产管理」会滑向「统一权限」，过不了安全评审。

### 这套模型在哪里失效

低频流程养不起 SOP 型 Skill 的固定成本（版本、负责人、测评集），人工更便宜。判断无法显式化的场景，做出来只是「能跑但没人敢用」。流程已被 ERP/CRM 按钮化的组织，可摘的增量只剩「没有按钮的地方」——滔搏的周末经营正踩这个空档。治理缺位的组织也不该先上平台：没有 IdP/SSO、没有能签字的场景负责人、没有测评能力，四道防线与双闭环都会退化成配置文件。

## 实践启示

1. **首个场景四个条件**：高频、刚需、可衡量、有原流程耗时基线。缺耗时基线就证不了效率提升；先排除「演示惊艳但一年跑三次」的候选，那是 POC。
2. **SOP 型 Skill 当资产管**：绑版本号、负责人、测评标准，写清适用条件、工具、异常与人工确认点。无测评标准的 Skill 不进共享库，否则会以「已验证」之名扩散未验证经验。
3. **度量四层按序爬**：使用 → 质量（采纳率、人工兜底率）→ 效率（耗时、单次成本）→ 业务（方案被执行）。停在第一层，团队就会把调用量讲成项目成功。
4. **上线三方签字**：业务定验收标准并担场景 owner，IT 管平台、身份与稳定性，AI 管工程、测评与版本；三方同时通过才算上线，否则就是「Agent 上线了，责任却下线了」。
5. **先划边界再谈复用**：Skill 推广前先把 Workspace、凭证、数据授权做成独立维度，明确能读什么、以谁的身份读。顺序颠倒等于重做授权模型。
6. **先建归因再建飞轮**：保证失败能被稳定归因为能力、数据、场景设计三类，否则训练集只放大噪声，度量只产出一份没人行动的报表。

## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构


## 第 2 来源：滔搏体育客户实践——从单点探索到规模化运营（阿里云 AgentCore 2026-10-15，v=7/c=9/63 SUPP）

滔搏体育（4,360 实体门店/3,700+ 小程序店铺/20+ 品牌/9,000 万+ 用户）基于 AgentCore+AgentLoop 的企业级多智能体落地案例，为本文的平台机制补上**客户侧运营方法论**。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md]

- **多 Agent 分工与统一底座动机（实践侧零覆盖）**："最有经验的人不可能同时出现在每一家店铺"；岗位目标相关但知识/数据/权限不同；多 Agent 三坑=重复建设/协作权限边界/运营三问（效果衡量/错误定位/上线谁负责）；AgentCore="做第二个 Agent 不必从头开始，做第 10 个也能管理清楚"。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md]
- **三场景（零覆盖）**：练货（经营 Agent 识别偏差→**"不能把卖得不好直接翻译成员工学得不好"**需现场确认→商品 Agent 给资料→陪练 Agent 练习→评分回流安排补练；单月 5,700+ 商品/39 万+ 次）；线索挖掘（"AI 再生成一份报告，阅读任务增加经营任务没减少"→把报告后面的准备连接起来，交到店长手里的是"准备充分的行动"而非洞察）；周末经营（目标+约束而非预设指令，2h→20min；**"复用能力不等于共享所有数据"**——只用本店授权数据）。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md]
- **SOP 型 Skill 沉淀方法论（零覆盖）**：四步=选场景（高频刚需可衡量）→建设沉淀→共同运营→优化复制（低价值及时退出）；SOP 型 Skill=反复验证有效的处理方法，讲清适用条件/输入/工具/输出/异常/人工确认，**要能被调用、被验证、随反馈更新，需版本化+负责人+测评标准**；稳定 Skill 组合成专业 Agent——"留下来的不是一次答案，而是一套已验证的解法"。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md]
- **共同运营与双闭环（零覆盖）**：业务（验收标准+场景负责人）/IT（平台与稳定性）/AI（工程与测评）各有分工但"不能各自验收各自结束"——否则"Agent 上线了，责任却下线了"；度量四层=使用→质量（采纳率/正确率/人工兜底）→效率（耗时/成本）→业务（方案被执行/结果改善），"仪表盘告诉我们发生了什么，运营还要回答：然后改什么"；**AgentLoop 关心能力怎么变好，Business Loop 关心经营问题怎么解决**——关键不是运行次数多就自动变好，而是反馈有没有被接住、有没有转化为经过验证的改进。^[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15.md]

→ [[raw/articles/taobo-multi-agent-agentcore-agentloop-2026-10-15|原文存档]]
