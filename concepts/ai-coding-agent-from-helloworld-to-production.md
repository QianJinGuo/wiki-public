---
title: "AI Coding Agent：从 Hello World 到 Production"
created: "2026-05-21"
updated: 2026-08-29
type: concept
tags: [coding, agent, production, engineering, journey, architecture, harness, context, memory, evaluation]
sources:
  - raw/articles/pi-openclaw-coding-harness
  - raw/articles/ai-agent-exploration-path-legacy-tech
  - raw/articles/karpathy-vibe-coding-to-agentic-engineering
  - raw/articles/github-token-efficiency-agentic-workflows
  - raw/articles/prompt-context-harness-three-evolutions
summary: "AI Coding Agent 的演进路线图，从 Hello World 单文件生成，到单 Agent 会话管理，到可靠的长程任务执行，再到 Production 级的多 Agent 协作与治理。阐述每个阶段的核心挑战、关键技术方案、以及阶段跃迁的前置条件。"
---

# AI Coding Agent：从 Hello World 到 Production

> 本文阐述 AI Coding Agent 的五阶段演进路径：Hello World → Working Agent → Reliable Agent → Autonomous Agent → Production Agent。每个阶段都有明确的准入标准、核心挑战、和必要的工程投资。核心洞察：**能力不会从概念里自动长出来，它靠一层层工程边界托住。**

## 概述：五阶段演进模型

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AI Coding Agent 演进                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Hello World    Working Agent    Reliable     Autonomous      Production │
│   ────────────   ────────────    ──────────    ────────────    ─────────── │
│   单文件生成      工具调用+        Harness+       Goal-Driven     多Agent+  │
│   LLM API        Session管理      Eval驱动        长程任务         治理体系   │
│                                                                      │
│   Prompt 1份      Prompt 3份       Prompt 5份       Prompt 8份      Prompt N份  │
│   工程 0层        工程 1层         工程 3层         工程 5层        工程 7层+  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**关键原则**：每往上一层，不确定性增加一个量级，成本也增加一个量级。**能在下层解决的，绝不上推。** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

## Stage 1: Hello World — 单文件生成

### 1.1 什么是 Hello World 阶段

仅使用 LLM API（GPT-4、Claude 3.5 等）进行单次代码生成。用户输入需求，AI 输出代码，手动复制粘贴到文件。

**典型场景**：
- "帮我写一个 Python 快速排序"
- "生成一个 React 组件"
- "写一个 Bash 脚本"

### 1.2 核心特征

| 维度 | 特征 |
|------|------|
| **上下文** | 零持久化，每次都是全新会话 |
| **工具** | 仅 LLM API，无文件读写 |
| **验证** | 人工复制、运行、观察 |
| **错误处理** | 无，错了重新生成 |
| **状态管理** | 无 |

### 1.3 核心问题

```
用户需求 → LLM API → 代码（字符串）→ 手动保存 → 手动运行 → 观察结果
                                           ↑
                                        全部丢失
```

**致命缺陷**：
1. **上下文无法累积**：每次都是新开始，无法记住项目背景
2. **工具能力为零**：无法读写文件、无法运行命令
3. **错误无法自动恢复**：需要人工介入
4. **验证完全靠人**：没有自动化测试

### 1.4 阶段准入标准

- ✅ 能调用 LLM API
- ✅ 能接收用户文本输入
- ✅ 能将生成的代码作为字符串输出

### 1.5 升级路径：加入文件读写

```python
# 第一步增强：让 Agent 能读写文件
class HelloWorldPlus:
    def __init__(self, api_key):
        self.llm = LLMClient(api_key)
        self.tools = ['read_file', 'write_file']
    
    def handle(self, user_request):
        # 读取项目上下文
        project_context = self.read_project_files()
        
        # 生成代码
        code = self.llm.generate(
            prompt=f"需求：{user_request}\n项目上下文：{project_context}"
        )
        
        # 写入文件
        self.write_file(code)
        return "已写入文件"
```

> **这个阶段的核心工程**：上下文组装、文件路径规范、代码块解析。

## Stage 2: Working Agent — 工具调用与会话管理

### 2.1 什么是 Working Agent

具备基础工具调用能力的 Agent：能读写文件、执行命令、管理会话状态。能在一次会话中完成"理解需求 → 生成代码 → 写入文件 → 运行测试"的闭环。

**典型场景**：
- "帮我在这个项目中添加一个用户认证模块"
- "修复 src/api/users.py 中的登录 bug"

### 2.2 核心特征

| 维度 | 特征 |
|------|------|
| **上下文** | Session 级别持久化（内存） |
| **工具** | 文件读写、执行命令、搜索 |
| **验证** | 人工触发测试，部分自动 |
| **错误处理** | 简单的重试机制 |
| **状态管理** | Session 内共享 |

### 2.3 核心架构

```python
class WorkingAgent:
    def __init__(self):
        self.session = Session()          # 会话状态
        self.tools = ToolRegistry()        # 工具注册
        self.llm = LLMClient()            # LLM 接口
        self.executor = Executor()        # 工具执行器
    
    def handle(self, user_request):
        # 1. 构建上下文
        context = self.build_context(user_request)
        
        # 2. LLM 生成计划
        plan = self.llm.plan(context)
        
        # 3. 执行工具调用循环
        while not plan.is_complete():
            tool_call = plan.next_tool()
            result = self.executor.execute(tool_call)
            plan.observe(result)
        
        # 4. 返回结果
        return plan.final_result()
```

### 2.4 核心问题

1. **工具调用可靠性**：模型可能生成无效的工具调用参数
2. **上下文膨胀**：会话越长，上下文越长，token 成本越高
3. **错误恢复困难**：一个步骤失败可能导致整个会话状态丢失
4. **Session 隔离**：重启后完全丢失之前的状态

### 2.5 关键技术：Context 是投影，不是有界容器 ^[raw/articles/pi-openclaw-coding-harness.md]

三类状态必须分开：

| 状态类型 | 用途 | 持久化 |
|----------|------|--------|
| **给模型看的** | 经过治理的投影，控制 token 消耗 | 不持久化 |
| **给 UI 看的** | 用户界面消息 | 可选持久化 |
| **给审计/恢复看的** | 完整 event log | **必须持久化** |

```python
# Transcript 是账本，working context 是视图
class TranscriptManager:
    def __init__(self):
        self.transcript = []  # 完整行动轨迹（JSONL，不删除）
        self.working_context = ""  # 当前模型可见材料
        self.summary = ""  # 两者之间的压缩视图
    
    def append(self, event):
        self.transcript.append(event)
        self.update_working_context()
        self.update_summary()
    
    def can_recover(self):
        """验证：摘要丢失时能否从 transcript 完整重建当前状态"""
        return True
```

### 2.6 阶段准入标准

- ✅ 能读写文件（read、write、edit）
- ✅ 能执行命令（bash、grep、find）
- ✅ 有 Session 管理（创建、切换、恢复）
- ✅ 有基础的错误处理（重试、超时）
- ✅ **Transcript + Working Context 双层保留**

### 2.7 升级路径：加入 Harness

> **弱模型需要 harness 帮它做事，强模型更需要 harness 确保它做事不越界、可复盘、能交付。** ^[raw/articles/pi-openclaw-coding-harness.md]

```python
class WorkingAgentWithHarness:
    def __init__(self):
        self.hooks = HooksSystem({
            'pre_tool_call': [self.validate_path, self.check_permission],
            'post_tool_call': [self.log_result, self.update_context],
            'on_error': [self.handle_error, self.checkpoint],
        })
    
    def validate_path(self, tool_call):
        # 路径安全检查
        if '..' in tool_call.path:
            raise SecurityError("Path traversal detected")
        if not tool_call.path.startswith(ALLOWED_ROOT):
            raise SecurityError("Path outside allowed root")
    
    def check_permission(self, tool_call):
        # 权限检查
        if tool_call.name == 'bash' and tool_call.command not in ALLOWED_COMMANDS:
            raise PermissionError("Command not allowed")
```

## Stage 3: Reliable Agent — Harness + Eval 驱动

### 3.1 什么是 Reliable Agent

具备完整 Harness 工程能力的 Agent：多层次的质量保障、自动化测试、错误恢复、上下文压缩（Compaction）。能可靠完成中等复杂度（1-4 小时）的独立任务。

**典型场景**：
- "重构这个模块，应用 SOLID 原则"
- "为这个服务添加完整的单元测试和集成测试"
- "将这个 API 从 REST 迁移到 GraphQL"

### 3.2 核心特征

| 维度 | 特征 |
|------|------|
| **上下文** | 多级持久化（Transcript + Summary + Index） |
| **工具** | 丰富的工具集（MCP、Skills、Extensions） |
| **验证** | 自动化测试 + LLM 评估 |
| **错误处理** | 多层恢复机制（回滚、重试、降级） |
| **状态管理** | 可恢复的 Checkpoint |

### 3.3 质量保障机制

| 机制 | 描述 | 实现方式 |
|------|------|----------|
| **测试驱动** | 先写测试再实现 | 内置测试生成工具 |
| **Lint 检查** | 代码风格验证 | Pre-commit hooks |
| **类型检查** | 类型安全验证 | Type checker 集成 |
| **安全扫描** | 漏洞检测 | Security scanner |
| **Human-in-loop** | 关键节点人工确认 | Approval gates |

### 3.4 核心技术：Harness Engineering ^[raw/articles/pi-openclaw-coding-harness.md]

**五工程模式**：

1. **Context 是投影，不是有界容器**
   - Working context 是给模型看的经过治理的投影
   - Transcript 是不可篡改的账本
   - 两者必须同时保留

2. **Compaction 不能只是摘要**
   - 必须同时保留完整 Transcript 作为审计和回滚证据
   - 验证标准：摘要丢失时能从 Transcript 完整重建当前状态

3. **权限要进运行时管线**
   - `beforeToolCall`：参数/路径/权限/风险检查（在命令生成后、执行前拦截）
   - `afterToolCall`：审计/截断/错误标记
   - 比 prompt 里写"不要做危险操作"可靠得多

4. **Runtime kernel 小，control plane 厚**
   - Runtime kernel 负责语义（模型+loop+工具调用）
   - Control plane 负责产品世界（路由+会话管理+通道适配）

5. **失败路径和证据链一起设计**
   - read：截断 + 提示续读 offset
   - bash：完整输出路径 + timeout + abort
   - edit：oldText 不唯一或重叠 → 拒绝执行

### 3.5 核心技术：Evaluator Agent ^[raw/articles/karpathy-vibe-coding-to-agentic-engineering.md]

**可验证性决定自动化上限**。没有验证体系托底，Agentic Engineering 顶多算更高级的 Vibe Coding。

```python
class EvaluatorAgent:
    """独立于 Generator 的验证者，模拟人类工程团队的利益博弈"""
    
    def evaluate(self, task, code, test_results):
        # 1. 功能验证：代码是否满足需求
        functional = self.check_functional(task, code)
        
        # 2. 质量验证：是否符合代码规范
        quality = self.check_quality(code)
        
        # 3. 风险验证：是否有安全漏洞
        security = self.check_security(code)
        
        # 4. 综合评估
        if functional and quality and security:
            return EvaluationResult(pass=True)
        else:
            return EvaluationResult(pass=False, issues=[...])
```

### 3.6 阶段准入标准

- ✅ 完整的 Hooks 系统（pre/post 拦截）
- ✅ 自动化测试 + LLM 评估
- ✅ Checkpoint + 恢复机制
- ✅ Compaction（压缩）机制
- ✅ 安全边界控制
- ✅ **6 步稳定路线验证**：
  1. 只读 Agent（read、grep、find、ls）稳定
  2. 精确修改（edit + diff）稳定
  3. 命令执行（bash + timeout）稳定
  4. Event log 持久化
  5. Context builder + compaction
  6. Skills/extensions/MCP/memory

### 3.7 升级路径：引入 Goal-Driven

**Task-Driven 解决执行问题，Goal-Driven 解决迭代问题。** ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

| 维度 | Task-Driven | Goal-Driven |
|------|-------------|-------------|
| 人的角色 | 项目经理 + 执行监督 | 目标设定者 / 审核者 |
| Agent 的角色 | 执行器 | 自主推进者 |
| 决策中心 | 在人脑子里 | 在目标 + 边界 + 系统状态里 |
| 主要成本 | 人持续编排 | 前期建模和约束设计 |
| 适用场景 | 简单、一次性任务 | 长期、复杂、持续推进任务 |

## Stage 4: Autonomous Agent — Goal-Driven 长程任务

### 4.1 什么是 Autonomous Agent

具备目标分解、进度追踪、自主决策能力的 Agent。能处理需要数小时甚至数天的复杂长程任务，人只需要在关键节点介入审核。

**典型场景**：
- "将这个单体应用重构为微服务架构，预计需要 2 周"
- "搭建完整的 CI/CD 流水线，包括测试、部署、回滚"

### 4.2 核心特征

| 维度 | 特征 |
|------|------|
| **上下文** | 持久化文件系统（Git Log + progress.md + STATE.yaml） |
| **工具** | 完整的工具生态（Skills、MCP、CLI） |
| **验证** | 多层 Eval + 人工审核 |
| **错误处理** | 自主恢复 + 人工升级 |
| **状态管理** | 共享状态 + 多 Agent 协调 |

### 4.3 Goal-Driven 的 5 个前提 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

1. **目标必须清晰** —— 不是模糊愿望，而是可推进、可判断的目标表达
2. **边界必须清晰** —— 哪些能做，哪些不能做，资源上限是什么
3. **状态必须可见** —— 当前做到哪一步，卡在哪，为什么卡
4. **过程必须留痕** —— 否则无法知道成功或失败的原因
5. **权限必须可控** —— 它到底能调用哪些工具，能写到哪里

### 4.4 核心技术：SDD（Spec-Driven Development）

每个需求处理完留下一组文档：

| 文档 | 内容 |
|------|------|
| `spec.md` | 目标、验收标准 |
| `plan.md` | 技术方案、涉及文件、实现步骤 |
| `tasks.md` | 任务清单，每个任务有描述和状态 |

> **留痕不是为了 debug，而是为了进化。**

### 4.5 核心技术：24h 打工人系统

**文件 + 轮询** 的调度架构：

```yaml
调度层四件事：
1. 接收任务：用户反馈进来，写入文件队列
2. 分发执行：轮询队列，调用 CLI 执行
3. 状态管理：记录每一步的输入输出，持久化到文件
4. 失败切换：某个 CLI 配额用完，自动换下一个
```

**智能并发策略**：

| 策略 | 做法 | 理由 |
|------|------|------|
| 组间并发 | 前端任务和后端任务同时跑 | 代码在不同目录，不会冲突 |
| 组内串行 | 同一个前端项目的任务排队执行 | 可能修改同一文件，避免冲突 |
| 失败隔离 | 单个任务失败不影响其他组 | 故障不扩散 |

### 4.6 Agent 自举 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

**Agent 自己修了自己的 bug 的前提**：

1. **清晰的设计文档** —— AI 知道每个模块该做什么、不该做什么
2. **SDD 流程** —— spec → plan → tasks 的标准路径
3. **constitution.md** —— 架构约束文件，定义代码组织规范、命名规则、模块边界

> 自举的前提是 constitution.md（架构约束文件）。不需要写得多长，但至少要覆盖三件事：目录结构约定、模块边界、命名规则。

### 4.7 阶段准入标准

- ✅ 清晰的目标表达机制（spec → plan → tasks）
- ✅ STATE.yaml 共享状态
- ✅ 失败自动切换（多 provider/CLI 备选）
- ✅ Constitution.md 架构约束
- ✅ 24h 无人值守能力

### 4.8 升级路径：引入多 Agent 协作

## Stage 5: Production Agent — 多 Agent + 治理体系

### 5.1 什么是 Production Agent

企业级的多 Agent 协作系统，具备完整的治理、合规、可观测、成本控制能力。能支撑大规模 AI 编程实践，数十或数百个 Agent 并发运行。

**典型场景**：
- 团队级 AI 编程平台
- 企业级代码审查和重构系统
- 全自动的 CI/CD + AI 辅助开发

### 5.2 核心特征

| 维度 | 特征 |
|------|------|
| **上下文** | 多租户隔离 + 企业知识库 |
| **工具** | 完整的 MCP 生态 + 企业集成 |
| **验证** | 全链路可观测 + 量化评估 |
| **错误处理** | 分层治理 + SLA 保障 |
| **状态管理** | 企业级多租户体系 |

### 5.3 企业 Agent 规模化的四个结构性挑战 ^[raw/articles/agent-从能用到管好中间差了什么.md]

1. **抽象层级错位**
   - 传统 IAM 控制云产品 API 权限
   - 企业需要业务语义层的权限管理
   - 例如"市场部可以使用营销文案生成 Agent"

2. **隔离粒度粗糙**
   - 组级资源配额
   - 用户级权限叠加
   - 空间级数据完全隔离

3. **协作链路断裂**
   - 业务开发者依赖基础设施团队提需求、等排期
   - 平台提供开发者控制台自助服务
   - 通过资源审批流平衡敏捷性与规范性

4. **成本黑盒与审计缺失**
   - 缺乏实时配额监控
   - 缺乏业务上下文关联的日志
   - 精细化配额管理 + OpenTelemetry 可观测大盘

### 5.4 Token 效率优化 ^[raw/articles/github-token-efficiency-agentic-workflows.md]

**效率问题的本质**：不是简单的"用少一点"，而是结构性问题。

**三种主要优化模式**：

#### 模式 1：消除未使用的 MCP 工具注册

```python
# 问题：40 个工具的 MCP server，每次 turn 增加 10-15KB schema
# 解决：交叉引用工具清单和实际调用，识别和消除未使用的工具
class MCPPruner:
    def analyze(self, tool_calls, mcp_schemas):
        used_tools = {tc.name for tc in tool_calls}
        unused = set(mcp_schemas.keys()) - used_tools
        return unused  # 识别后可选择移除或动态加载
```

#### 模式 2：将 MCP 替换为 CLI

```python
# MCP 工具调用 = 完整 LLM API 往返
# CLI 调用 = 确定性 HTTP 请求，无 LLM 参与

# 替代方案：
# - gh pr diff：确定性请求
# - pre-agentic CLI：agent 启动前运行
# - in-agent CLI 代理：轻量透明 HTTP 代理
```

#### 模式 3：区分不同类型的 token 成本

```
ET = m × (1.0 × I + 0.1 × C + 4.0 × O)

参数说明：
- m：模型成本乘数（Haiku=0.25×, Sonnet=1.0×, Opus=5.0×）
- I：新处理输入 token（1.0×）
- C：缓存读取 token（0.1×）
- O：输出 token（4.0×）← 最贵
```

### 5.5 生产级 Agent 系统地基（五层） ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

| 层级 | 职责 |
|------|------|
| 1. 目标表达 | 到底想完成什么 |
| 2. 能力单元 | 有哪些 Skill/工具/工作流 |
| 3. 运行时状态 | 当前正在做什么 |
| 4. 治理边界 | 允许做什么，不允许做什么 |
| 5. 评估反馈 | 哪些行为值得固化，哪些必须修正 |

> 少任何一层，系统都可能看起来能跑，但跑不稳。

### 5.6 阶段准入标准

- ✅ 多租户隔离体系
- ✅ 全链路可观测（OpenTelemetry）
- ✅ 成本控制和配额管理
- ✅ 企业级安全治理
- ✅ 团队协作工作流
- ✅ 量化评估体系

## 演进路线总结

### 各阶段核心投入对比

| 阶段 | Prompt 工程 | 工程投入 | 可观测性 | 团队角色 |
|------|------------|----------|----------|----------|
| Hello World | 1份 | 0层 | 无 | 个人 |
| Working Agent | 3份 | 1层 | 基础 | 个人 |
| Reliable Agent | 5份 | 3层 | 完整 | 个人+AI |
| Autonomous Agent | 8份 | 5层 | 全链路 | AI 主导 |
| Production | N份 | 7层+ | 企业级 | 团队+平台 |

### 脚手架 > 模型 ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

投入回报对比（个人经验估算）：

| 投入方向 | 成本增加 | 效果增加 |
|----------|----------|----------|
| 模型升级 | +300% | +20% |
| **脚手架升级** | **+50%** | **+200%** |

> **优先投资脚手架，而不是追最新最贵的模型。**

### 落地路径（六步） ^[raw/articles/ai-agent-exploration-path-legacy-tech.md]

| 步骤 | 做什么 | 核心产出 |
|------|--------|----------|
| 第一步 | 写清楚 spec | 要做什么、不做什么、怎么算完成 |
| 第二步 | 执行过程留痕 | Prompt/状态/输出/错误全记录 |
| 第三步 | 补 observability 和 eval | 知道为什么成功、为什么失败 |
| 第四步 | 高频动作沉淀为 Skill | 模板 + 规则 + 代码 |
| 第五步 | 引入调度和并发 | 调度层 + 轮询 + 失败切换 |
| 第六步 | 最后才尝试 Goal-Driven | 目标表达 + 治理边界 + 共享状态 |

> **先让一次执行可复盘，再让它可重复，再让它可规模化，最后让它可有限自主。**

## 相关实体

- [[entities/claude-code-harness-deep-dive-founder-park]]
- [[entities/oneusefulthing-claude-code-what-comes-next]]
- [[entities/adopting-ai-coding-agents-six-lessons]]
- [[entities/claude-code-open-source-model-enterprise-practice]]
- [[entities/agent-production-harness-engineering]]

## 关联实体

- [[concepts/coding-agent-architecture]] — 编程 Agent 架构
- [[concepts/coding-harness-engineering]] — Coding Harness 工程本质
- [[concepts/agentic-workflow-patterns]] — Agent 工作流模式
- [[concepts/ai-agent-exploration-path]] — AI Agent 探索之路
- [[concepts/harness-engineering-framework]] — Harness Engineering 框架
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2]] — Harness Engineering 详解
- [[entities/karpathy-vibe-coding-to-agentic-engineering]] — Karpathy 从 Vibe Coding 到 Agentic Engineering

## 关联实体

**上游依赖**:
- [[entities/claude-code-harness-deep-dive-founder-park]] — 提供基础理论/方法
- [[entities/oneusefulthing-claude-code-what-comes-next]] — 提供基础理论/方法

**下游应用**:
- [[entities/adopting-ai-coding-agents-six-lessons]] — 具体应用场景
- [[entities/claude-code-open-source-model-enterprise-practice]] — 具体应用场景
- [[entities/agent-production-harness-engineering]] — 具体应用场景

**平行协作**:
- [[entities/harness-engineering-让-coding-agent-可靠完成长程任务-v2]] — 替代/补充方案
- [[entities/karpathy-vibe-coding-to-agentic-engineering]] — 替代/补充方案
- [[entities/perplexity-computer-knowledge-work-empirical-study]] — 替代/补充方案


→ [[raw/articles/pi-openclaw-coding-harness|原文存档：Pi/OpenClaw Coding Harness]]
→ [[raw/articles/ai-agent-exploration-path-legacy-tech|原文存档：AI Agent 探索之路]]
→ [[raw/articles/karpathy-vibe-coding-to-agentic-engineering|原文存档：Karpathy 访谈]]

## 新增关联实体
- [[entities/perplexity-computer-knowledge-work-empirical-study]]

## 所属 MOC

- [[moc/agent-engineering-guide|Agent Engineering Guide]]
