---
title: "Hermes Agent 与 OpenClaw/Claude Code 的核心差异和适用场景？"
created: 2026-05-21
updated: 2026-05-21
type: query
tags: [hermes-agent, openclaw, claude-code, comparison, architecture, self-evolution]
sources:
  - raw/articles/gateway-architecture-openclaw-claude-hermes-comparison
  - raw/articles/hermes-9-module-architecture-winty
  - raw/articles/agent-tools-research
related:
  - concepts/hermes-agent
  - concepts/hermes-agent-skill
  - entities/hermes-9-module-architecture
  - entities/gateway-architecture-openclaw-claude-hermes-comparison
confidence: high
---

# Hermes Agent 与 OpenClaw/Claude Code 的核心差异和适用场景？

> 本页综合 [[concepts/hermes-agent|Hermes Agent 核心概念]]、 [[concepts/hermes-agent-skill|Skill 系统]]、  及 ，结构化回答三框架的核心差异与选型场景。

---

## 一、核心定位差异

三框架的本质分歧可以归结为一个根本问题：**Agent 是工具还是员工？**

| 框架 | 定位 | 进程模型 | 主动性 |
|------|------|---------|--------|
| **Claude Code** | IDE 编程 Copilot（工具） | 每次调用新建进程，无状态 | 被动响应 |
| **OpenClaw** | 工具框架 + 数字员工雏形 | 常驻进程 | 主动心跳+Cron |
| **Hermes** | 持久化个人超级助手（员工） | 常驻进程 | 主动 + 自进化 |

---

## 二、五维深度对比

### 2.1 进程与记忆模型

| 维度 | Hermes | OpenClaw | Claude Code |
|------|--------|----------|-------------|
| **进程生命周期** | 常驻进程 | 常驻进程 | 每次调用新建 |
| **对话状态** | 写入记忆系统（向量检索+层级衰减） | 写入文件系统 | 无状态 |
| **上下文积累** | 跨会话持续增长（复利效应） | 部分积累 | 每次重置 |
| **故障恢复** | 三层持久化（state/session/memory） | 文件系统快照 | 无 |

**关键洞察**：Claude Code 的每次调用都是"新人"，无法积累上下文；Hermes 的上下文是**复利**——运行时间越长，Agent 越懂你。^[raw/articles/agent-tools-research.md]

### 2.2 自进化能力

| 维度 | Hermes | OpenClaw | Claude Code |
|------|--------|----------|-------------|
| **进化路径** | Skill + RL 双路径 | Skill only | 无 |
| **Skill 生成** | 自动轨迹蒸馏 | 半自动 | 需人工 TS 编写 |
| **RL 微调** | 支持 | 不支持 | 不支持 |
| **冷启动** | 0 Skill → 50+（需数月积累） | 少量预置 | 无 Skill 系统 |

**核心差异**：Hermes 的 Skill 生成是**全自动的**，Claude Code 需要 TypeScript 编写+测试+发布的完整工程流程，OpenClaw 是半自动。

### 2.3 Gateway 架构哲学

| 维度 | Hermes | OpenClaw | Claude Code |
|------|--------|----------|-------------|
| **Gateway 定位** | 轻量桥接层（薄适配器） | 神经中枢（重控制面） | 无原生 Gateway |
| **多平台接入** | 18 适配器 | 50+ 渠道 | MCP Channels |
| **默认安全** | fail-closed | 开放→13.5万暴露实例 | 本地位安全 |
| **主动性** | 无内置 Cron | Heartbeat+Cron | 无 |

**安全警示**：OpenClaw 默认开放策略导致 13.5 万暴露实例，部署时必须配置为 fail-closed 并绑定防火墙。^[raw/articles/gateway-architecture-openclaw-claude-hermes-comparison.md]

### 2.4 Session 与上下文隔离

| 维度 | Hermes | OpenClaw | Claude Code |
|------|--------|----------|-------------|
| **隔离模型** | 四层 session key（agent:main:channel:peer） | session key 模型 | 终端生命周期 |
| **多租户** | 单用户 | 无 | 无 |
| **向量检索** | 内置 | 无 | 无 |

**技术细节**：OpenClaw/Hermes 采用 `agent:main:channel:peer` 四段式结构化 session key，在不引入向量数据库的前提下实现天然上下文隔离。查询成本 O(1)，远低于向量 embedding 计算。

### 2.5 工具/Skill 系统

| 维度 | Hermes | OpenClaw | Claude Code |
|------|--------|----------|-------------|
| **Skill 本质** | 经验模式（Markdown） | Skill 文件（Markdown） | 工具定义（TS） |
| **Skill 生成** | 自动轨迹蒸馏 | 半自动 | 手工 TS |
| **版本策略** | 持续迭代（v0.1→v1.x） | 静态 | 需独立版本管理 |
| **Skill 复用** | 上下文自适应 | 参数化调用 | 固定参数 |

---

## 三、9 模块架构参考（Hermes 独家）

Hermes 的 9 模块系统是其与其他框架拉开差距的核心：

| 模块 | 职责 | 关键设计 |
|------|------|---------|
| **Agent Loop** | ReAct 循环心脏 | 每轮与其他 8 模块对话 |
| **Prompt Assembly** | 动态拼装 system prompt | 每次任务从 SOUL/Memory/User/Skill 动态组装 |
| **Memory Store** | 有界+声明式+冻结 | MEMORY.md + USER.md，防止膨胀 |
| **Skill Manager** | 生命周期管理 | 创建/激活/Patch/回滚/安全扫描 |
| **Tool/MCP** | 工具层+权限分级 | 无害/副作用/危险 三级权限 |
| **Nudge Engine** | 后台计数器触发 | 阈值触发 Review Agent |
| **Review Agent** | 独立 fork 复盘 | 读 Trajectory 决定写 Memory/Skill |
| **Session/Trajectory Store** | SQLite 执行档案 | 供 review+回放+评估 |
| **SOUL.md** | 人格层 | 风格/价值观/底线/护栏 |

**关键设计原则**：执行链（Agent Loop+Tool+Session）与学习链（Nudge+Review+Memory+Skill Manager）通过 fork 机制严格分离——学习不增加主链路延迟，也不污染执行环境。^[raw/articles/hermes-9-module-architecture-winty.md]

---

## 四、选型决策框架

### 4.1 场景匹配矩阵

| 场景 | 推荐框架 | 理由 |
|------|---------|------|
| **个人效率助手，长期运行** | Hermes | 记忆复利、Cron 调度、自进化 |
| **团队协作/多租户** | OpenClaw（需安全加固）或商业平台 | 多 Agent 路由、绑定控制 |
| **单次代码任务/IDE 集成** | Claude Code | 即开即用，无状态 |
| **需要极低延迟安全检查** | Hermes + Fangcun Guard | 8ms p99 延迟 |
| **多框架并行使用** | Hermes Observer 方案 | OS 级监控，框架无关 |
| **快速原型验证** | Claude Code | 无需部署，直接试用 |

### 4.2 迁移路径建议

**OpenClaw → Hermes**：官方支持无缝迁移，Skill 格式兼容，工具注册机制继承。迁移后记忆系统从朴素文件系统升级为向量检索+层级衰减。^[raw/articles/gateway-architecture-openclaw-claude-hermes-comparison.md]

**Claude Code → Hermes**：需要接受从"无状态工具"到"常驻助手"的心态转变。Skill 系统从 TypeScript 手工编写变为自动轨迹蒸馏。

### 4.3 决策树

```
任务类型？
├── 单次代码任务（Code Review/Bug Fix/单次搜索）
│   └── Claude Code — 即开即用，无需部署
└── 长期助手场景（知识管理/自动化/多轮对话）
    ├── 需要 Cron 调度？→ Hermes
    ├── 需要多 Agent 协作？→ OpenClaw（加固后）
    └── 需要自进化能力？→ Hermes（Skill+RL 双路径）
```

---

## 五、行动检查清单

### 评估阶段
- [ ] 明确核心需求：工具（Claude Code）vs 员工（Hermes/OpenClaw）
- [ ] 评估团队技术能力：TypeScript 编写 Skills（Claude Code）vs 接受自动轨迹蒸馏（Hermes）
- [ ] 评估安全要求：fail-closed 优先（Hermes）还是扩展性优先（OpenClaw）

### 部署阶段
- [ ] OpenClaw 部署：必须配置 fail-closed + 防火墙限制 18789 端口
- [ ] Hermes 部署：配置 Cron 任务、飞书/Discord 消息通道、记忆策略
- [ ] Claude Code 部署：配置 MCP Channels（如果需要消息平台接入）

### 运营阶段
- [ ] Hermes：前 2-4 周为暖场期，手工补充 Seed Skills 加速能力圈形成
- [ ] OpenClaw：定期扫描暴露面，审计 Gateway 配置
- [ ] Claude Code：无状态，无需记忆管理，但每次任务需重新提供上下文

---

## 六、关联参考

- [[concepts/hermes-agent|Hermes Agent 核心概念]] — 持久化、自进化、记忆系统完整解析
- [[concepts/hermes-agent-skill|Skill 系统]] — 六元组结构、自动轨迹蒸馏、冷启动问题
-  — 执行链/学习链解耦、Prompt Assembly 动态拼装
-  — Gateway 哲学、session key 模型、ACP 上下文聚合趋势
-  — 整体选型三步法
- [[concepts/harness-engineering-framework|Harness Engineering]] — Agent 运行时框架视角
