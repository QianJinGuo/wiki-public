---
title: "Amazon Bedrock AgentCore 数据持久化文件系统：Session Storage、EFS、S3 Files"
created: 2026-07-07
updated: 2026-07-07
type: entity
tags: [agent, aws, bedrock, agentcore, harness, storage, data-persistence, filesystem]
confidence: 0.8
provenance_state: extracted
sources: [raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e]
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

## 与相关实体

- [[entities/aws-bedrock-agentcore|AWS Bedrock AgentCore]] — AgentCore 整体架构
- [[entities/break-the-context-window-barrier-with-amazon-bedrock-agentcore|突破上下文窗口壁垒]] — AgentCore 上下文管理
- [[entities/structured-memory-filtering-metadata-agentcore-memory|结构化记忆与元数据过滤]] — AgentCore 记忆系统
- [[entities/agent-memory-engineering-tax-aws-china-2026|Agent 记忆工程税]] — 记忆系统的工程挑战

→ [[raw/articles/amazon-bedrock-agentcore-数据持久化文件系统session-storage-和-amazon-e|原文存档]]
