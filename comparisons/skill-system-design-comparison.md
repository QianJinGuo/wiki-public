---
title: Skills 系统设计三方对比
created: 2026-04-27
updated: 2026-04-27
type: comparison
tags: [skill-system, openclaw, claude-code, hermes-agent, agent-architecture, skill-design, progressive-disclosure, supply-chain-security]
sources: ['raw/articles/skill-system-design-three-way-comparison']
confidence: high
---
# Skills 系统设计三方对比
## 概述
三个主流 Agent 框架（OpenClaw、Claude Code、Hermes Agent）在"**Skills 的经验应该从哪里来**"这个根本问题上，选择了三条截然不同的路径：
| 框架 | 赌注 | 经验来源 |
|------|------|----------|
| **OpenClaw** | 社区集体智慧 | ClawHub 开放市场，31000+ Skills |
| **Claude Code** | 个人经验沉淀 | 使用者自己编写 SKILL.md |
| **Hermes Agent** | Agent 自我进化 | 执行过程自动生成 |
## OpenClaw：社区生态
### ClawHub — Agent 界的 npm
任何人发布，任何人安装。冷启动最佳，数万个 Skills 即装即用。
### 供应链安全代价
2026年1月事件：
- 341 个伪装恶意包植入键盘记录器和信息窃取木马
- 专门针对 OAuth Token、API Key、浏览器密码
- 恶意或有安全隐患的 Skill **一度超过总量的 20%**
**根因**：ClawHub 无强制代码审核，任何超过一周的 GitHub 账号即可发布，无代码签名、无沙箱验证、无安全扫描。
安装一个 ClawHub Skill = 在机器上执行陌生人代码，且权限与 OpenClaw 本身完全相同。
## Claude Code：渐进式披露
### 三层 Token 加载机制
Claude Code 的 Skills 全部由使用者自己编写，放入 `.claude/skills/` 目录。
核心创新：**渐进式披露（Progressive Disclosure）** 解决 20+ Skills 上下文膨胀问题：
| 层级 | 触发时机 | Token 消耗 | 内容 |
|------|----------|-----------|------|
| 第一层 | 启动时 | ~100 Token | Skill 名称 + 描述 |
| 第二层 | 按需加载 | <5,000 Token | 完整 SKILL.md |
| 第三层 | 执行时 | 0 Token | scripts/ 辅助脚本直接 bash 执行 |
**关键洞察**：scripts/ 里的代码不注入上下文，直接执行——一个 Skill 可附带几千行 Python 代码，对 Token 成本毫无影响。
### Compaction 保护
上下文压缩时：
- 每个 Skill 保留前 **5,000 Token**
- 所有 Skills 共享 **25,000 Token** 预算
- Skills 里的规则在整任务周期持续有效
## Hermes Agent：自动进化
### 自我生成机制
当 Agent 完成复杂任务（≥5 次工具调用），系统自动将执行过程提炼为 Skill 文档：
- 步骤、工具选择、遇到的问题、解决方法
- 写入 `[本地运行时路径已隐藏]`
- **使用中自动更新**——Agent 发现更好的方法，自动修改 Skill 文档
用户测试：2小时内 Agent 自动生成3份 Skill，类似任务执行速度提升 **40%**。
### 三大难题
| 问题 | Hermes 解法 | 局限性 |
|------|------------|--------|
| 什么任务值得生成？ | "5次以上工具调用"启发式规则 | 不精确，有些简单任务也值得沉淀 |
| 抽象层次在哪里？ | 每次执行后自动提炼 | 太具体泛化差，太抽象无操作指引 |
| 质量谁来把关？ | 无内置评估机制 | 质量完全依赖执行质量 |
### 跨 Agent 共享
- 共享目录 `[本地运行时路径已隐藏]`：同一机器所有 Agent 共享
- PLUR 社区插件：一个 Agent 的纠正自动传播给同项目其他 Agent
## 三种路径取舍
**选 OpenClaw**：任务类型多样，需要快速覆盖各种场景。代价：必须认真审查每一个安装的 Skill。
**选 Claude Code**：特定专业场景，对质量和安全要求高。渐进式披露让你可以写得非常详细而不担心 Token 成本。
**选 Hermes**：任务类型固定且重复，愿意让 Agent 慢慢学。时间越长积累越多，执行越准。
**最优解是组合**：社区 Skills 快速启动 + 自己写的 Skills 覆盖核心场景 + Hermes 自动沉淀新 Skills。
> Skills 的格式已经走向开放标准（agentskills.io）。
## 相关概念
- [[concepts/harness-engineering-framework|Harness Engineering 框架]] — Skills 是 Harness 的核心组件
- [[entities/claude-code-architecture|Claude Code 架构解析]] — 渐进式披露是其 Prompt/Context/Harness 设计的一部分
- [[entities/hermes-agent-deep-dive|Hermes Agent 深度解析]] — Self-Evolving 机制：Skill 动态生成
- [[concepts/openclaw-architecture|OpenClaw 架构解析]] — ClawHub 生态是 OpenClaw 的重要组成部分
- [[entities/ai-agent-tool-count-trap|AI Agent工具数量陷阱——5个边界清楚的工具胜过20个模糊工具]]
- [[entities/claude-code-openclaw-memory-comparison|Claude Code vs OpenClaw Agent 记忆系统对比]]
- [[entities/context-window-management|Agent 上下文窗口管理对比]]
- [[entities/imclaw通过微信飞书操控claude-code-coodex-gemini-clipi-agent蜂群|IMClaw：通过微信/飞书操控ClaudeCode/Codex/GeminiCLI/Pi Agent蜂群]]
- [[concepts/harness-engineering-7-layers-framework|Harness Engineering 七层框架]]
## 相关查询
- [[moc/ai-skill-design|AI Skill 设计 Topic Map]] — Skill 设计原则与最佳实践汇总
- [[queries/hermes-agent-vs-openclaw-claude-code-core-differences-and-use-cases|Hermes vs OpenClaw vs Claude Code 核心差异]] — 三框架对比的补充视角
## 参考文献
- AllenTang / 架构师带你玩转AI — 《AI Agent 架构设计（七）：Skills 系统设计》