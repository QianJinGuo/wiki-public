---
title: "Amazon Bedrock AgentCore 数据持久化文件系统：Session Storage、EFS、S3 Files"
created: 2026-07-07
updated: 2026-09-17
type: entity
tags: [agent, aws, bedrock, agentcore, harness, storage, data-persistence, filesystem]
confidence: 0.8
provenance_state: extracted
sources: [raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e]
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Amazon Bedrock AgentCore 数据持久化文件系统：Session Storage、EFS、S3 Files

Amazon Bedrock AgentCore 提供了**三种持久化文件系统方案**，覆盖不同的 Agent 持久化需求：Managed Session Storage、Amazon EFS、Amazon S3 Files。这三种方案在私有性、共享范围、访问方式上有各有侧重——从按用户私有、到多方共享、再到文件与对象两端访问。会话结束也不会丢失数据。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

## 三种方案对比

| 特性 | Managed Session Storage | Amazon EFS | Amazon S3 Files |
|------|------------------------|------------|-----------------|
| 访问范围 | 按用户私有 | 多方共享 | 文件与对象两端访问 |
| 数据持久性 | 会话保持 | 持久 | 持久 |
| 典型场景 | 用户会话数据 | 团队共享知识库 | 大规模文件存储 |
| 性能 | 低延迟 | 高吞吐 | 高可用 |

^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

Managed Session Storage 适合需要按用户隔离的对话上下文和会话级缓存；Amazon EFS 适合需要跨多个 Agent 或用户共享的持久化知识库；Amazon S3 Files 适合大规模文件存储和需要同时通过对象存储接口访问的场景。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

## 使用场景

AgentCore 的数据持久化文件系统在以下场景中尤为关键：

- **长周期 Agent 任务**：跨多轮对话维护 Agent 的内部状态和进度，不会因会话中断而丢失数据
- **多 Agent 协作**：多个子 Agent 通过共享 EFS 文件系统读写同一份中间数据
- **知识库持续积累**：Agent 在运行中产生的知识片段持久化到 Session Storage 或 S3，供后续轮次使用
- **审计与回放**：Agent 的完整执行历史持久化，支持事后审计和调试

^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

这些场景与 [[entities/amazon-bedrock-agentcore-runtime-deep-dive-and-scenario-analysis|AgentCore 运行时深度分析]] 中覆盖的 Agent 生命周期管理直接衔接。而 [[entities/when-ai-agents-learn-to-forget-amazon-bedrock-agentcore-memory-philosophy|AgentCore 记忆哲学]] 则从更底层的记忆系统角度讨论了数据持久性的设计理念。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

## 性能测试

文章包含了对三种方案的性能对比测试数据，从延迟、吞吐量、并发连接数等维度提供了基准参考。对于需要在高并发场景下选择合适持久化方案的工程团队具有实际指导意义。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

## 深度分析

### 选型矩阵：先问隔离边界，再问生命周期

原文的选型逻辑可以压缩成一条判断链：隔离粒度先筛掉候选，生命周期再决定要不要引入跨 microVM 的共享层，访问协议最后决定是否需要对象存储的旁路。Session Storage 以 `runtimeSessionId` 为单位隔离，代价是只在一个 session 内可见，Agent 之外的进程无法挂载；EFS 与 S3 Files 走 BYO 路径，隔离退化到文件系统或 Access Point 层面，换来跨 session、跨 Agent、乃至跨业务 Pod 的可见性。更硬的约束是 `filesystemConfigurations` 绑定在 Runtime 上、并不按 session 生效，因此「按用户动态切换 Access Point」在架构上走不通，C 端大并发只能交给 Session Storage。合理的判断顺序是：先问这份数据要不要被 Agent 之外的东西看见，再问它需要活多久，最后才在 EFS 与 S3 Files 之间比较协议与访问模式。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

### 文件系统作为外部记忆：把中间产物赶出上下文窗口

原文中 Agent 需要持久化的对象并不只有「结果」。Claude Agent SDK 的三类数据——技能文件、`CLAUDE_CONFIG_DIR` 下的对话历史、workspace 中的中间文件——全部落在文件系统上，而不是塞进 prompt：技能按 progressive disclosure 按需加载，历史以 transcript 文件支撑续聊，工作产物用普通文件读写。这条路线与 [[entities/break-the-context-window-barrier-with-amazon-bedrock-agentcore|突破上下文窗口壁垒]] 是同一种思路的两面：上下文窗口是稀缺资源，凡是能被路径寻址、按需载入的内容，都不该常驻 prompt。文件系统之所以能承担这个角色，关键在于它是 POSIX 原生接口——读写不经过大模型、也不消耗 token。因此持久化文件系统的价值不只是「不丢数据」，而是「把状态从昂贵的上下文里搬出去」。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

### 共享 EFS 的一致性、锁与并发写风险

EFS 提供完整 POSIX 语义，包含 advisory file locking 与 close-to-open 一致性，这是它相对 S3 Files 的实质优势：多客户端并发读写同一份数据时行为稳定。但原文也明确提示，EFS 与 S3 Files 的共享层不做 session 隔离，多租户隔离必须自建。真实客户案例的做法是在 `/invocation` 入口按 payload 中的 `tenant_id` 执行 `mount --bind`，把 `tenants/<tenant>` 子目录绑到 `/workspace`，再用 `resolve_path()` 拦截 `../` 路径遍历与 tenant_id 注入，并规定同一 session 首次绑定后不得再切换租户。这套隔离的可靠性依赖几个前提：容器需以 root 运行才能执行 mount，须采用 `BedrockAgentCoreApp` 模式而非纯 Flask，且 session 之间本身已由 Firecracker microVM 提供内核级隔离。可见共享知识库不能只靠目录命名约定，真正的边界要靠挂载与校验代码来守。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

### S3 Files 的双协议：写入即同步，同步即约束

S3 Files 把 S3 桶以标准 NFS 挂载出来，通过文件接口写入的内容会自动同步回桶，桶里的对象同样能在挂载点上看到，形成文件与对象 API 的双向通路。这给 Agent 工作流带来两种模式的共存：Agent 在线读写脚本、配置模板与工具二进制，运维侧则可在 Agent 之外用 S3 API 完成批量管理、审计与回放。适配的负载形态是写少读多、以追加为主。但双协议并非免费——性能测试中 S3 Files 的 4 KB 随机写被标为 n/a：在对单个大文件持续高强度随机写并开启强制刷盘的极端压测下，双向同步会因刷新底层 NFS 句柄而中断。这条限制把使用边界划得很清楚：S3 Files 是共享与互操作的通道，而不是重随机写的落盘层。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

### 性能数据的工程解读：差异在小块随机读，而非平均延迟

原文的实测结论是三种文件系统都不慢：1 MiB 块下随机写吞吐都落在 4.6 至 6.2 GiB/s 区间，延迟 p50 均在 0.12ms 以内、p99 在 0.2ms 以内。差异集中在小块随机读——Session Storage 与 EFS 都在约 40 万 IOPS，S3 Files 约为其一半。把这些数字翻译成选型规则：随机读写密集、延迟敏感的工作负载（代码落盘、工具链、频繁改写的协作数据）交给 Session Storage 或 EFS；写少读多、以共享与互操作为目的的负载交给 S3 Files。原文作者自己给出的判断也是这个方向——选型的主要依据是隔离与共享，性能并非关键矛盾。对工程团队而言，真正的信号不是「谁快几个百分点」，而是哪一维度的短板会撞上你的负载形态。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

## 实践启示

以下几条是把上面的判断落成具体动作的做法。^[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e.md]

1. **按隔离边界选存储，而不是按性能排序**：先确定数据要被谁看见。私有、按用户或会话隔离、C 端大并发选 Managed Session Storage；跨 session、跨 Agent 共享且需要完整 POSIX 与高随机写选 EFS；需要文件与 S3 API 两端访问、写少读多选 S3 Files。
2. **把长中间产物落到文件系统，而不是灌进上下文**：技能、对话历史、生成代码与分析结果都按路径组织在持久化存储上，按需载入，避免用 token 搬运本可由文件路径寻址的内容。
3. **共享层不要只靠目录命名约定**：EFS/S3 Files 不做 session 隔离，多租户边界要靠入口处的 bind mount 加路径校验（拦截 `../` 遍历与 tenant_id 注入、禁止 session 中途切换租户）来强制，而不是靠目录命名。
4. **用执行历史同时满足调试与审计**：把会话 transcript 与工作产物持久化，既能在排障时回放 Agent 的完整执行路径，也让可追溯性成为默认属性而非事后补做的工作。
5. **按负载形态选延迟与吞吐方案**：重随机写交给 Session Storage 或 EFS，S3 Files 承担共享脚本、配置与工具；必要时把私有项目、共享数据集、公共工具在同一个 Runtime 上分别挂载到不同路径，混合并存。
6. **算清运维责任的转移**：托管态（Session Storage）免去 VPC、Access Point 与挂载配置的维护，自管态（EFS/S3 Files）则要承担 mount target 的 AZ 对齐、安全组放通与执行角色权限；选择前先确认团队愿意接哪一侧的责任。

## 与相关实体

- [[entities/aws-bedrock-agentcore|AWS Bedrock AgentCore]] — AgentCore 整体架构
- [[entities/break-the-context-window-barrier-with-amazon-bedrock-agentcore|突破上下文窗口壁垒]] — AgentCore 上下文管理
- [[entities/structured-memory-filtering-metadata-agentcore-memory|结构化记忆与元数据过滤]] — AgentCore 记忆系统
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆工程税]] — 记忆系统的工程挑战

→ [[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e|原文存档]]
