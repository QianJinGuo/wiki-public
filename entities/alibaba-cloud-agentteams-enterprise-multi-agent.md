---
title: "阿里云 AgentTeams 企业级多 Agent 平台"
created: "2026-07-14"
updated: 2026-10-15
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
