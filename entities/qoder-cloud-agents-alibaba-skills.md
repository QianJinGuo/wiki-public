---
title: "用云新范式：Qoder Cloud Agents × Alibaba Cloud Skills"
created: 2026-07-05
updated: 2026-10-28
type: entity
tags: [qoder, cloud-agents, alibaba-cloud, skills, cloud-engineering, agent-platform, cloud-native, devops, infrastructure-as-code, ai-ops]
sources: [raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills, raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27]
confidence: 0.85
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 用云新范式：Qoder Cloud Agents × Alibaba Cloud Skills

Qoder 推出了 Cloud Agents 与 Alibaba Cloud Skills 的深度整合，开创了用云的新范式——不再通过 GUI 控制台管理云资源，而是通过 Agent 直接调用云 Skills 完成资源创建、配置管理和故障排查。这标志着云计算进入"第四代界面"——从控制台到 CLI 到 IaC 到 Agent，使用者可以不再是人类。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

该方案将阿里云 300+ 个云产品的操作抽象为可组合的 Skills，Agent 通过自然语言理解用户意图后自动编排执行。系统内置了多层安全检查、操作审计和回滚能力，确保即使 Agent 出错也不会造成不可逆的破坏。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

实际测试显示，通过 Cloud Agents 完成一个典型的多服务部署任务（ECS + RDS + SLB + OSS）的时间从人工操作的 45 分钟降至 Agent 的 3 分钟，同时减少了 70% 的手动配置错误。这一范式将深刻改变云计算的消费方式。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

## 核心摘要

- **来源**：阿里云开发者公众号，2026 年 7 月 3 日
- **核心主张**：Agent 是"云的最后一个界面"——之后界面本身就不再需要了
- **关键技术**：Alibaba Cloud Skills（将云能力封装为可组合、带语义、带副作用的标准化 Skill）+ Qoder Cloud Agents（有机器身份的常驻 Agent 承载体）
- **典型场景**：一句话部署、睡后运维、Skill 即服务、数据自主生长
- **与 Agent 平台竞争格局 的关系**：Qoder 定位为云原生 Agent 平台，聚焦基础设施操控

## 深度分析

### 一、云计算界面的四次代际跃迁

Alibaba Cloud Skills 团队提出了一个清晰的云界面进化框架，值得深入拆解：^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]


| 代际 | 界面形式 | 代表技术 | 核心变化 | 认知负荷 |
|------|---------|---------|---------|---------|
| 第一代 (2006) | Web 控制台 | AWS Console | 不用买服务器、租机房 | 高（手动点选） |
| 第二代 (2010) | CLI / SDK | AWS CLI, Aliyun CLI | 操作可编程、可批量 | 中（脚本编写） |
| 第三代 (2014) | IaC / 声明式 | Terraform, Pulumi | "要什么"而非"怎么做" | 低（模板配置） |
| 第四代 (2026) | Agent | Qoder Cloud Agents | "为什么"而非"要什么" | 极低（自然语言） |

每一代都没有淘汰上一代——CLI 内部调用 SDK，Terraform 包装 CLI，Agent 内部执行 CLI 和 Terraform。这类似于 TCP/IP 协议栈的层次封装：每一层为上一层提供抽象，同时隐藏内部实现细节。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

第四代界面的关键洞察是：**当云能力的"货架已经搭好"（300+ 产品、20K+ OpenAPI 可调用），真正的瓶颈不再是能力本身，而是人能否高效使用这些能力**。Agent 并非替代人做决策，而是将人从"操作界面"中解放出来，让人聚焦于"表达意图"和"确认结果"。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

### 二、Alibaba Cloud Skills：把 API 升级为 Agent 可理解的能力单元

Skills 是 Qoder 架构的核心创新点。它不是简单的 API 封装，而是一种**语义增强的能力抽象层**。每个 Skill 包含：^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

| Skill 组件 | 说明 | 对 Agent 的意义 |
|-----------|------|----------------|
| 语义描述 | 用自然语言描述能力边界和适用场景 | Agent 理解"什么时候该调用这个 Skill" |
| 前置条件 | 声明调用前必须满足的条件 | Agent 自动执行依赖检查和准备 |
| 副作用声明 | 说明操作会改变什么状态 | Agent 评估风险并通知人类确认 |
| 最小权限模板 | 执行该 Skill 所需的最小 IAM 权限 | 安全合规，零信任执行 |
| 可复跑剧本 | 带幂等性的执行流程 | Agent 可以失败重试而不产生脏数据 |

这种设计思想与 **Agent Harness Tool Specification** 和 [[concepts/model-context-protocol-mcp|MCP Protocol]] 的理念一致——工具/能力的描述需要足够语义丰富，Agent 才能在不依赖人类专业知识的情况下正确编排使用。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]


Skills 的发布平台（skills.aliyun.com）以"货架"的形式组织，按场景分类（应急、数据、成本、研发等），使 Agent 能够根据当前任务自动发现和选择合适的能力。这种"能力目录"的设计模式是 2026 年 Agent 平台的核心趋势。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

### 三、四个场景的工程学解构

Qoder Cloud Agents 的四个场景生动展示了 Agent 用云的典型模式：^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]


**场景一：一句话部署**——"帮我部署这个项目，要高可用、HTTPS、自动扩缩容，预算每月 2000 元以内"^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

- 实际执行了：分析依赖 → 按预算选规格 → 建 VPC+安全组 → 配 SLB+证书 → 写部署脚本 → 配扩缩容 → 验证 → 输出报告
- 工具编排量：6 个云产品编排，跨多个控制台的配置操作
- 关键在于：Agent 自主完成了**规格选择**（成本约束下的最优决策），而不仅仅是执行固定脚本

**场景二：睡后运维**——凌晨 3 点的告警自主响应^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

- 执行链：告警接收 → 根因定位（慢查询）→ 安全止血（参数调整、临时扩容）→ 服务恢复验证 → 诊断报告 → 等待人类 approve
- 实测漏斗：634 issue → 190 有效缺陷 → 25 CR 自动提交 → 12 人工合入
- 漏斗设计哲学：Agent 的价值不在"生成了多少"，而在"挡住了多少"

**场景三：Skill 即服务**——将专家经验封装为组织级可复用能力^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

- 资深架构师的"数据库选型"经验 → 结构化 Skill → 团队可一句话调用
- 核心价值：个人经验从"锁在某人的脑子里"变为"组织级的可复用资产"

**场景四：数据自主生长**——报告自动生成 + 按需下钻^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

- Agent 自动绕过资源冻结问题完成报告 → 用户一句话增加分析维度 → Agent 自主扩展查询范围
- 关键特征：能力随业务需求**自然生长**，不需要开发介入修改代码

### 四、"人在环中"的设计精度

Qoder Cloud Agents 的设计中最值得关注的是其**人在环中的精度控制**。与"全自动运维"的激进愿景不同，Qoder 采取了务实的"只做诊断和止血，不删不缩"原则：^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

```
告警触发 → Agent 诊断定位 → Agent 安全止血（只扩不缩、只调不删）
→ 生成诊断报告 → 等待人类确认（approve/reject）→ 执行修复
```

这种"Agent 前滚 + 人类确认"的模式是 2026 年 Agent 落地的标准范式。其核心原则包括：^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

1. **破坏性操作必须人工确认**：删除资源、修改权限等高风险操作需人类 approve
2. **止血操作自动执行**：临时扩容、参数调整等可逆操作 Agent 可自主执行
3. **漏斗设计优先于能力设计**：Agent 的价值由"挡住了多少错误"而非"生成了多少输出"衡量

这与 **Harness Engineering 安全原则** 中的"渐进信任"理念一致——Agent 的能力范围随信任积累逐步扩展，而非一开始就赋予全权。^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]

### 五、对云计算产业格局的影响

Qoder Cloud Agents + Alibaba Cloud Skills 的组合将对云计算产业产生结构性影响：^[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills.md]


1. **云消费门槛从"需要专家"降至"能说人话"**：企业不再需要为每个云操作配备持有认证的架构师。业务人员可以直接通过 Agent 完成数据查询、资源创建等基本操作。

2. **云厂商竞争焦点从"功能多"转向"Agent 好用"**：当 Agent 成为云的"主界面"，云产品功能的可发现性、Agent API 的清晰度、Skill 生态的丰富度将成为新的竞争维度。

3. **新的"中间市场"崛起**：Skill 开发将成为一种新的角色——"云 Agent 工程师"不再直接操作云，而是将云操作封装为 Skills 供 Agent 调用。这与 **Agent Harness 工程师** 的角色定义吻合。

4. **跨云 Agent 的长期可能性**：如果 Skills 抽象层足够标准化（类似于 Terraform Provider 的抽象），未来 Agent 可以跨多云编排资源，使多云策略从"每个云配一套工具"变为"一个 Agent 管所有云"。

## 实践启示

1. **重新评估云运维策略中的人类参与度**：将运维操作分为三类——全自动（可逆操作、标准变更）、半自动（需 approve 的高风险操作）、全手动（首次执行的复杂变更）。逐步将前两类迁移到 Agent 工作流。

2. **优先构建"能力目录"而非"单个 Agent"**：参考 Alibaba Cloud Skills 的货架模式，先梳理团队常用的云操作清单，将它们封装为标准化的 Skills（带语义、前置条件、副作用声明），再让 Agent 编排这些 Skills。

3. **从"一句话部署"开始试点**：选择一个典型部署场景（如 Web 服务上线），创建对应的 Skill 集合，验证 Agent 从意图理解到部署验证的完整流程。关键验收标准：Agent 能否在失败时自主恢复而非直接报错。

4. **建立 Agent 运维漏斗指标**：参考 634→190→25→12 的漏斗模型，跟踪 Agent 在"接收任务 → 执行正确 → 产出有效 → 人工通过"各阶段的转化率。漏斗的收紧程度反映了 Agent 的可靠性和人类的信任度。

5. **保留逃生通道**：在任何 Agent 化的云操作流程中，保留手动回滚能力。Agent 可以建立自己的沙箱环境和幂等操作机制，但人类必须有一条"一键还原"的路径。

→ [[raw/articles/用云新范式qoder-cloud-agents-alibaba-cloud-skills|原文存档]]

## 增补（2026-10-27）：长程任务的运行架构——会话状态与计算生命周期解耦

Qoder Cloud Agents（QCA）是 Serverless 模式的云端 Agent 托管与执行服务：业务应用通过 Agents API 提交目标与上下文，QCA 组织模型推理与工具执行、保存任务进展，再把过程事件与任务结果返回应用；1.0 于 9 月 16 日发布，**月调用量超千万**。它面向三类场景——Agent 助手（投研/个人助手）、高并发长程任务（客服质检/告警运营）、专业方法服务化（把专家方法做成 Skill 供团队调用与验收）。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**三种容量口径，入口 QPS 不等于平台压力。** 任务时长（一次任务多久完成）、在途并发（同时多少会话未结束）、事件速率（单位时间请求/事件数）必须一起看：任务长且持续进入时，即使瞬时新请求不多，未完成会话也会累积，其中有的在计算、有的在等工具结果或人工输入，推进过程中还会继续产生模型命令、工具结果与进度通知——**入口请求结束不等于任务结束**。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**解耦：会话承载状态，执行实例只做当前这段。** 关键不是让同一执行实例始终绑定同一任务，而是把任务生命周期与计算生命周期分开——会话保留上下文与进度，实例完成当前步骤；任务进入等待后保留进度并释放执行容量，新输入或外部结果到来再形成新的执行轮次，由可用实例接手。等待本身仍有成本（会话状态、保留文件、平台基础资源），但释放出来的容量可以服务其他任务。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**手脑分离的五个角色**：**Agent Runtime** 是协调者，按"当前有哪些结果、哪些依赖已满足"决定下一步调模型还是派工具；**模型执行池** 承担模型步骤，命令经消息交给可用实例、推理完成后返回结果（可能含一个或多个工具请求）；**工具执行端** 按需调用远程 API、MCP 服务或 Sandbox，**结果直接回 Runtime**；**共享持久状态** 保存历史、进度与结果；**RocketMQ 只负责异步交接**，归并与依赖检查都由 Runtime 做。工程难点因此集中在三项：状态可续（换实例要能重建当前步骤上下文）、有序交接（独立分支可并行、共享结果按依赖归并）、运行可控（配额不足可等、异常能定位到具体步骤）。作者也明确：**分布式设计改变的是资源占用与交接方式，不缩短单次推理或工具调用，反而增加状态恢复、依赖归并与运行策略的复杂度**。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**为什么用 LiteTopic 而不是普通 Topic。** Parent Topic 预先配置、统一管理一类工作，应用按业务规则设置 LiteTopic 通道：**通道内有序、通道间并行**，且不必为每个短期任务维护普通 Topic。这匹配长程 Agent 的负载特征——执行路径在运行中不断展开（新工具请求、派生分支、等待运行前无法预知的外部事件）：共享少量通道会让某个长步骤或容量等待拖累其他任务，为每个短期任务建普通 Topic 又抬高管理成本。通道边界由应用定义：多分支按分支建通道，Webhook 按接收端 Endpoint 建通道（同一接收端有序、不同接收端并行），通道数量也不需与执行实例数一一对应。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**共享正文 + 轻量消费索引。** 一条消息即使对应多个目标 LiteTopic，正文只写 CommitLog 一次，Multi-Dispatch 为各通道建消费索引而不复制正文；普通队列按 Topic+Queue 定位消息，LiteTopic 用 **RocksDB KV 索引**组织消费位置。于是形成"父 Topic 预配置 + 通道按需建索引 + 正文共享 + 过期回收由配置管理"的双层模型。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**Suspend：容量不足时只暂缓该通道的本次消费。** 消费端收到消息先查容量：配额不足返回 Suspend，暂缓相应通道本次消费并在后续投递时重试；容量满足则执行当前步骤、产出完整结果并确认消费。**Suspend 只影响该通道，其他通道仍可并行**，待处理消息留在消息系统里，实例继续服务能执行的任务；配额、优先级与重试时机仍由应用策略决定——消息系统提供交接与消费控制，运行策略要结合业务约束制定。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**接力流程与复杂场景。** 一轮完整执行是：发布模型命令 → 模型池执行 → 结果（含工具请求）经 MQ 回 Runtime → Runtime 归并并登记依赖、发布工具指令 → 工具端执行、结果直接回 Runtime → 依赖齐备后生成下一条模型命令。复杂场景沿用同一交接方式：多分支由模型池独立消费并行、结果按会话归并；协作消息进持久化 Mailbox，到任务允许接收新消息的边界再处理；**取消走独立控制通知，先更新取消状态再中止相关执行**；Webhook 按 Endpoint 建通道并约束超时与重试预算；挂起与恢复先存进度、等事件返回再派后继步骤。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**Sandbox 按需，且"文件保留 ≠ 现场恢复"。** 纯模型推理由模型池承接，远程 API 与 MCP 由工具端发起后由远端执行，**只有需要隔离代码或文件工具时才准备 Sandbox**；预热、复用、回收与暂停后的恢复程度取决于策略与后端能力。按需使用 Sandbox 的价值是减少"无须占用 Sandbox 的等待时间"，而不是保证执行现场无损恢复。任务事实由持久存储承载：数据库保存会话与任务状态，对象存储保存文件与产物；**取消等瞬时控制信号与过程预览只用于控制与观测，不能作为任务事实的主存**。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]

**适用边界（作者给出的判断清单）**：是否采用这类拆分，要综合评估任务时长、等待占比、状态恢复要求、顺序边界能否按业务单元划分，以及容量隔离收益能否覆盖分布式复杂度——这与"任务时长 / 在途并发 / 事件速率"三种口径是同一条思路的两端：**先量清等待占比，再决定是否为解耦付费**。^[raw/articles/qoder-cloud-agents-litetopic-hand-brain-separation-aliyun-2026-10-27.md]


---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

