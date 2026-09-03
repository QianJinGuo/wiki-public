---
title: "Claude Code Best Practices: Prompt Engineering"
created: 2026-05-21
updated: 2026-08-29
type: concept
tags: [claude-code, prompt-engineering, best-practices, agent, harness, context]
sources:
  - raw/articles/claude-code-prompt-source-analysis-fanone
  - raw/articles/claude-code-prompt-context-harness
  - raw/articles/openclaw-prompt-context-harness
summary: Claude Code 提示词工程最佳实践，涵盖 Prompt 编写策略、上下文组织、指令优化、角色定义、输出控制等核心技法，以及与 Harness 系统的协同设计。
---

# Claude Code Best Practices: Prompt Engineering

> Claude Code 的能力不仅取决于底层模型，更取决于如何向它传递指令和上下文。本页面系统总结 Claude Code 场景下的 Prompt Engineering 最佳实践。

## 一、核心设计哲学

### 1.1 语义优先于结构

Claude Code 的 Tool Prompt 设计哲学核心：**充分相信大模型的理解能力**。

- **不用代码补丁**：把规则以自然语言放在 Prompt 里
- **典型结构**：「这个工具是什么 / 什么时候该用 / 什么时候不该用 / 参数约束是什么」
- **本质**：用语义描述工具使用规范，而非用代码/JSON 去限制模型行为

> 大模型对语义的理解远比结构化编码更灵活。信任模型的理解能力，比试图用参数定义约束它更有效。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

### 1.2 分层解耦原则

Claude Code 的 Prompt 系统采用**职责分层设计**，每层解决不同粒度的问题：

| 层级 | 解决的问题 | 典型内容 |
|------|-----------|----------|
| Core System | 角色、边界、风格、风险原则 | 静态规则缓存 + 动态会话指导 |
| Tool | 调用契约、使用条件、参数约束 | 自然语言行为协议 |
| Skill | 知识分发、按需加载 | 渐进式展开的文档 |
| Agent | 协作结构、角色边界 | SOP 模板定义 |
| Context Management | 上下文容量管理 | 三级压缩机制 |
| Memory | 长期记忆与知识保留 | 四类分级存储 |

> 这种分层使得每层可以独立演进，不互相耦合。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

## 二、Prompt 编写最佳实践

### 2.1 清晰具体的指令

| 原则 | 错误示例 | 正确示例 |
|------|---------|---------|
| **模糊** | "处理这个文件" | "提取 `src/utils/auth.ts` 中的所有 JWT 验证逻辑" |
| **过长** | 500字描述一个简单任务 | 3-5个要点说明关键约束 |
| **歧义** | "尽快完成" | "在本次会话结束前完成，或在 30 分钟内完成" |
| **否定句** | "不要引入新的依赖" | "只修改现有依赖版本，不添加新包" |

### 2.2 角色定义要强

Claude Code 的 Agent System Prompt 模板建议七维度定义：

```markdown
## 工作职责：负责什么 / 核心价值
## 强制边界：不能做什么 / 失败条件
## 可获取信息：输入什么 / 依赖什么上下文
## 执行过程：先做什么 → 再做什么 → 何时停止 → 何时升级
## 错误处理：常见错误行为 + 纠正方法
## 工具使用：优先用什么 / 什么不能碰 / 什么信号要验证
## 输出规范：汇报格式 / 必须字段 / verdict/critical files/summary
```

> 原则：用有逻辑的自然语言，不用 JSON/key-value 编码语言。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

### 2.3 指定输出格式

```python
# 指定格式示例
prompt = """
提取以下代码中的 API 端点信息。

输出格式（严格遵循）：
{
    "endpoints": [
        {
            "method": "GET|POST|PUT|DELETE",
            "path": "/api/...",
            "description": "端点功能描述",
            "auth_required": true|false
        }
    ],
    "total_count": 数量
}

代码：
...
"""
```

## 三、上下文组织策略

### 3.1 CLAUDE.md 层次结构

Claude Code 自动读取的上下文文件应采用分层设计：

| 文件位置 | 作用域 | 内容类型 |
|----------|--------|----------|
| `~/.claude/CLAUDE.md` | 个人全局偏好 | IDE 设置、通用习惯 |
| `项目根目录/CLAUDE.md` | 团队共享规范 | 代码库结构、架构决策 |
| `CLAUDE.local.md` | 个人私有指令 | 本地开发偏好 |
| `.claude/rules/*.md` | 按文件类型分类规则 | TypeScript 规则、Python 规则 |

> 根文件应该只是指针和关键注意事项；其他一切都漂移到子目录文件中。^[raw/articles/how-claude-code-works-in-large-codebases-best-practices-and-where-to-start.md]

### 3.2 渐进式上下文加载

**问题**：所有信息塞进上下文 → Token 严重浪费
**Solution**：Skill 作为 Prompt 资产先注册，运行时不立刻展开，按需加载

```markdown
## Skill Prompt 标准格式
name / description / allowedTools / model / hooks / paths
## Reference Documentation
...Reading Guide...（文档索引）
## Included Documentation
<doc path="go/claude-api/README.md">...</doc>
## When to Use
## Common Pitfalls
```

> Token 消耗从"全部"降到"按需"。语言检测 + Reading Guide 进一步减少不相关文档传输。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

### 3.3 上下文压缩三层级

| 层级 | 名称 | 成本 | 触发条件 |
|------|------|------|----------|
| Layer 1 | MicroCompact | 零 | 时间阈值 / KV Cache 边界 |
| Layer 2 | Session Memory Compact | 零 | ≥10,000 tokens 且 ≥5 条消息 |
| Layer 3 | Full LLM Compact | 高 | 前两层不足 |

> 压缩时保留：Edit/Write 操作完整保留，Bash/Read/Grep/Glob 可压缩 ^[raw/articles/claude-code-prompt-context-harness.md]

## 四、优先级与覆盖策略

### 4.1 五级优先级决策链

Claude Code 的 Prompt 覆盖采用明确的优先级策略：

| 优先级 | 类型 | 触发条件 | 覆盖行为 |
|--------|------|----------|----------|
| **P0** | Override | `overrideSystemPrompt` 设置 | 硬覆盖，其他所有 prompt 全部失效 |
| **P1** | Coordinator | coordinator mode 开启 | 主线程变为调度者 |
| **P2** | Agent | `mainThreadAgentDefinition` 设置 | proactive mode 下追加而非替换 |
| **P3** | Custom | `--system-prompt` 参数传入 | 用户自定义 |
| **P4** | Default | 标准 prompt | 最终兜底 |

> 多来源 prompt 共存时，明确优先级比简单追加更有价值。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

### 4.2 静态/动态分离

- **静态规则缓存**：不频繁变化的部分（角色定义、风险原则、工具总则）缓存减少 Token 消耗
- **动态规则按需**：会话特定的信息（memory、session guidance）每次会话更新

> 两者之间有明确的 boundary 划分，使得缓存失效范围清晰可控。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

## 五、工具使用技巧

### 5.1 Tool Prompt 最佳实践

每个工具的 Prompt 应包含：

1. **功能描述**：这个工具是什么
2. **使用时机**：什么时候该用
3. **禁用场景**：什么时候不该用
4. **参数约束**：参数的具体限制

```markdown
## BashTool 示例
用途：执行 shell 命令，用于文件操作、git 操作、运行脚本
使用场景：
- 需要执行 npm/yarn/pnpm 命令时
- git 提交、分支操作时
- 运行项目脚本时
禁止：
- 不要用 bash 执行需要超级用户权限的操作
- 不要用 bash 替代 TypeScript 编译检查
参数约束：
- 工作目录默认项目根目录
- 超时时间 5 分钟
```

### 5.2 BashTool 的 SOP 化

BashTool 的 Prompt 已经复杂到像高风险工具专用操作 SOP：

> 定义 git 提交/PR 详细流程、禁止事项、Skill 替代部分 git 流程。

> 当某个工具的行为复杂度超过简单描述时，应该用 SOP 而非简单 prompt 来定义它。^[raw/articles/claude-code-prompt-source-analysis-fanone.md]

## 六、Memory 系统协同

### 6.1 四类 Memory 分层

| 类型 | 内容 | 写法 |
|------|------|------|
| **user** | 角色/目标/知识水平/偏好 | 直接描述 |
| **feedback** | 对 Agent 工作方式的反馈 | 规则 → Why → How to apply |
| **project** | 事实/决策/动机/截止时间/事故背景 | 事实 → Why → How to apply |
| **reference** | 看板/dashboard/Slack/Linear 入口 | 直接描述 |

**不进 Memory**：代码结构、git 历史、修 bug recipe、CLAUDE.md 内容、临时任务状态

### 6.2 两步保存法

1. 每条 memory 写**单独文件** + frontmatter（name/description/type）
2. 入口加到 `MEMORY.md`（仅索引，不写正文）

> Memory 内容可能过时——提到文件/函数/flag 时必须先验证是否还存在

## 七、与 Harness 系统协同

### 7.1 Hook 生命周期集成

Claude Code 支持的 Hook 事件类型：

| 类型 | 事件 | 用途 |
|------|------|------|
| 工具 | `PreToolUse` / `PostToolUse` / `ToolError` | 参数校验、结果拦截 |
| 会话 | `SessionStart` / `SessionEnd` / `SessionPause` | 初始化/清理/状态管理 |
| 消息 | `PreSampling` / `PostSampling` / `UserPromptSubmit` |  Prompt 修改/响应拦截 |
| 文件 | `PreFileEdit` / `PostFileEdit` / `PreFileWrite` / `PostFileWrite` | 文件操作监控 |

> Hook 可返回结构化 JSON 数据实现阻断执行、动态篡改、反馈注入。^[raw/articles/claude-code-prompt-context-harness.md]

### 7.2 Prompt 与 Hook 协同模式

```javascript
// PreToolUse Hook 示例：参数校验
const preToolUseHook = async (tool, args, env) => {
    if (tool === 'Bash' && args.command.includes('rm -rf')) {
        return {
            action: 'block',
            reason: '危险命令需要二次确认'
        };
    }
    return { action: 'allow' };
};
```

## 八、常见错误与规避

### 8.1 常见 Prompt 错误

| 错误类型 | 描述 | 解决方案 |
|----------|------|----------|
| **过度指令** | 将所有规则塞进一个 system prompt | 按职责分层设计 |
| **缺乏示例** | 复杂任务不给 few-shot 示例 | 为模式复杂任务提供示例 |
| **过时上下文** | Memory 长期不更新 | 定期验证 Memory 时效性 |
| **优先级混乱** | 多来源 prompt 不指定优先级 | 使用 P0-P4 优先级树 |
| **忽视压缩** | 不考虑 Token 成本 | 渐进式加载 + 适时压缩 |

### 8.2 性能优化 Checklist

- [ ] 静态/动态规则分离，减少不必要的 Token 消耗
- [ ] Skill 按需加载，避免一次性加载全部文档
- [ ] CLAUDE.md 采用分层结构（根 → 子目录）
- [ ] Memory 按类型分离，避免记忆过载
- [ ] 定期审查 Prompt 是否与当前模型能力匹配
- [ ] 工具定义使用自然语言而非代码补丁

## 九、实践框架总结

### 9.1 Prompt 设计检查清单

```
□ 任务描述清晰具体（避免歧义）
□ 角色定义完整（七维度覆盖）
□ 输出格式明确指定
□ 上下文分层组织
□ 优先级策略明确
□ 工具使用规范完整
□ Memory 机制协同
□ Hook 集成考虑
□ Token 成本优化
```

### 9.2 与一般 Prompt Engineering 的区别

| 维度 | 一般 Prompt Engineering | Claude Code Prompt Engineering |
|------|------------------------|-------------------------------|
| **上下文源** | 用户手动提供 | CLAUDE.md + Memory + Skills 自动注入 |
| **工具调用** | 无原生工具支持 | 内置 Bash/Read/Edit/Write 等工具 |
| **记忆机制** | 对话历史 | 四类分级 Memory 系统 |
| **压缩机制** | 手动摘要 | 三级自动压缩 |
| **执行控制** | 无 | Hook 全生命周期支持 |
| **优先级** | 简单追加 | P0-P4 优先级决策树 |

## 十、关联页面

### 深入理解
- [[entities/claude-code-prompt-source-analysis|Claude Code Prompt 提示词体系源码解析]] — 六大模块详细拆解
- [[entities/claude-code-prompt-context-harness|深度解析 Claude Code 在 Prompt / Context / Harness 的设计与实践]] — 12 模块动态组装

### 架构关联
- [[concepts/prompt-engineering-fundamentals|Prompt Engineering Fundamentals]] — 通用 prompt 工程基础
- [[entities/openclaw-prompt-context-harness|深度解析 OpenClaw 在 Prompt / Context / Harness 的设计]] — 三维度对比参考

### 最佳实践
- [[entities/claude-code-best-practices|Claude Code 大型代码库最佳实践]] — Harness 配置与部署模式
- [[entities/from-prompt-to-harness-claude-official|从 Prompt 到 Harness：Claude 官方学习资料]] — 官方课程解读

## 关联实体

**上游依赖**:
- [[entities/claude-code-prompt-source-analysis]] — 提供基础理论/方法
- [[entities/claude-code-prompt-context-harness]] — 提供基础理论/方法

**下游应用**:
- [[entities/openclaw-prompt-context-harness]] — 具体应用场景
- [[entities/claude-code-best-practices]] — 具体应用场景

**平行协作**:
- [[entities/from-prompt-to-harness-claude-official]] — 替代/补充方案
- [[entities/claude-code-html-artifacts]] — 替代/补充方案
- [[entities/dingtalk-stream-cli-dual-engine-ai-assistant]] — 替代/补充方案


→ [[raw/articles/claude-code-prompt-source-analysis-fanone|原文存档：Claude Code Prompt 提示词体系源码解析]]
→ [[raw/articles/claude-code-prompt-context-harness|原文存档：深度解析 Claude Code Prompt/Context/Harness]]

## 新增关联实体
- [[entities/claude-code-html-artifacts]]
- [[entities/dingtalk-stream-cli-dual-engine-ai-assistant]]

## 所属 MOC

- [[moc/layer-2-interaction|Layer 2 Interaction]]
