---
title: "Anthropic Claude Community 插件仓库劫持事件：SHA 校验如何避免供应链攻击"
description: "多个 Claude Community 插件存在 repo-jacking 漏洞，直接代码安装路径被 SHA 校验阻断，但 Claude Code 的插件 UI 浏览功能仍会重定向用户到被劫持仓库，形成社交工程攻击面。"
created: 2026-07-01
updated: 2026-09-07
type: entity
tags: [security, supply-chain, claude, claude-code, plugin, repo-jacking]
sources: [raw/articles/repo-jacking-anthropics-claude-community-plugins]
confidence: 0.9
review_value: 9
review_confidence: 9
review_stars: 4
review_recommendation: strong
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Anthropic Claude Community 插件仓库劫持事件：SHA 校验如何避免供应链攻击

> 原文存档：[[raw/articles/repo-jacking-anthropics-claude-community-plugins|原文存档]]

## 摘要

安全研究员 John Stawinski IV 发现 Anthropic 官方维护的 Claude Code Community 插件市场存在多个 repo-jacking 漏洞。5 个插件指向的 GitHub 仓库所有者已改名或删除账户，导致这些命名空间可被攻击者认领。虽然 SHA 校验阻止了恶意代码的自动安装，但"查看插件主页"功能会将用户重定向到被劫持的仓库，形成社交工程攻击面。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:1-50]

## 核心概念

### 什么是 Repo-jacking

Repo-jacking（仓库劫持）是一种利用 GitHub 命名空间回收机制的供应链攻击：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


1. **初始状态**：GitHub 用户改名或删除账户时，旧的用户名/组织名会保留重定向^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:31-34]
2. **漏洞窗口**：任何人可以注册被废弃的用户名，创建同名仓库^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:31-34]
3. **攻击完成**：原重定向消失，指向旧路径的链接现在指向攻击者控制的仓库^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:31-34]

GitHub 对高星标、高流量的项目有保护机制，但小众项目往往不受保护。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:34-35]

### Claude Code Community 插件

Claude Code Community 插件是社区开发的扩展，用于增强 Claude Code 功能：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


- **功能范围**：自定义命令、AI Agent、MCP 服务器、自动化钩子^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:21-22]
- **安装方式**：单条命令安装，易于团队共享^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:21-22]
- **市场机制**：Anthropic 维护官方目录，任何人可提交插件通过审核后上架^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:22-24]
- **技术实现**：市场本质上是一个 GitHub 仓库，核心是一个 `marketplace.json` 文件，列出插件并指向托管代码的远程仓库^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:22-26]

关键风险点：市场条目指向他人的仓库，这种信任会随时间悄然腐烂——人们改名、删除账户，但市场条目不会察觉。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:26-30]

## 漏洞发现过程

### 受影响的插件

研究员通过扫描 `marketplace.json` 发现 5 个指向已不存在所有者的仓库的插件条目：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


| 插件 | 原仓库路径 | 状态 |
|------|------------|------|
| mailfnguides-del/Claude-Paste | 所有者改名/删除 | 命名空间可认领 |
| comment-io/claude-code-plugin | 所有者改名/删除 | 命名空间可认领 |
| CharlieGreenman/ghostlty-dynamic-themes | 所有者改名/删除 | 命名空间可认领 |
| Chipkorvyn/Strategy-consultant | 所有者改名/删除 | 命名空间可认领 |
| oduffy-delphi/deep-research-claude | 所有者改名/删除 | 命名空间可认领 |

^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:38-45]

每个都是 Anthropic 官方市场中的悬空引用，每个都是开放邀请——注册名称、重建仓库、从生态系统仍视为合法的位置提供任意内容。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:44-46]

### 概念验证：deep-research-claude

研究员选择 `deep-research-claude` 插件进行 PoC：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


1. **原始状态**：指向 `github.com/oduffy-delphi/deep-research-claude`，但 `oduffy-delphi` 账户已不存在^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]
2. **真实迁移**：实际项目已移至 `dbc-oduffy/deep-research-claude`，但市场未更新^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]
3. **攻击执行**：研究员注册 `oduffy-delphi` 组织，重建同名仓库，克隆合法代码作为种子^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]
4. **唯一改动**：在 README 顶部添加 "PoC — John Stawinski" 标记^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]

### 攻击路径演示

从用户视角，攻击看起来完全正常：

```bash
# 添加市场
claude plugin marketplace add anthropics/claude-plugins-community

# 安装插件
/plugin install deep-research@claude-community

# 点击"打开主页"
# 跳转到被劫持的仓库，显示 PoC 标记
```

^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:51-58]

无警告、无摩擦。市场是官方的，主页打开。信任全程流动。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:58-58]

**社交工程攻击面**：攻击者控制的页面在 Anthropic 生态系统隐性背书下呈现，用户正处于"设置此插件"的心理模式中。这是完美的钓鱼场景——攻击者可声称"这是正确的安装方式"，或提供毒化配置，或利用目标已相信自己位于合法位置的心理实施各种攻击。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:58-60]

## 安全机制分析

### SHA 校验：安装路径的守护者

**关键发现**：如果用户不打开主页，而是允许 Claude Code 自动安装插件，SHA 校验会拯救他们。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


市场条目中的仓库被固定到特定提交 SHA：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


```json
{
  "name": "deep-research-claude",
  "repository": "oduffy-delphi/deep-research-claude",
  "commit_sha": "abc123..."
}
```

^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:61-65]

当 Claude Code 获取插件代码时，它不仅信任 owner/repo，还信任期望内容的哈希值。重建的仓库可以拥有相同名称和相同文件，但无法复现固定的提交。完整性检查失败，恶意安装不会发生。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:64-66]

**核心安全原则**：将源固定到不可变引用（如提交 SHA）。这是防御插件生态系统劫持的正确设计。^[raw/articles/build-interactive-pdf-text-extraction-from-amazon-s3.md:66-67]

### SHA 更新的潜在风险

发现漏洞后，研究员进一步思考：插件维护者会更新代码，Anthropic 何时以及如何更新 SHA？^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]


调查发现：Anthropic 工程师提交大型 PR，由机器人追踪最新 SHA、验证插件，然后为市场提供新 SHA。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:69-73]

**潜在攻击向量**：
- 欺骗插件验证流程，使机器人摄入指向被劫持仓库恶意提交的新 SHA
- 研究员未测试此理论，但怀疑 SHA 并非"身穿闪亮盔甲的骑士"，更像是"暂时保护你的詹姆·兰尼斯特"^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:73-75]

## 披露与响应

### 负责任的披露

在公开披露前，研究员采取了保护措施：

1. **注册受影响前缀**：注册每个受影响的 GitHub 组织名称，防止真实攻击者利用^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:76-77]
2. **保留重定向**：除 `oduffy-delphi/deep-research-claude`（用于 PoC）外，其他特定仓库保持未创建状态，使 GitHub 重定向仍解析到合法项目的新位置^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:76-77]
3. **Bug Bounty 报告**：通过 Anthropic 漏洞赏金计划报告全部发现^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:78-78]

### Anthropic 的回应

| 时间 | 事件 |
|------|------|
| 2026-05-25 | 报告提交 |
| 2026-05-25 | 通过初步审核 |
| 2026-05-25 | 关闭为"信息性"，超出范围 |

^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:79-85]

**Anthropic 的理由**：
- 依赖劫持明确超出范围
- SHA 固定保护安装路径
- 用户跟随主页链接到已更改所有权的页面属于社交工程^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:85-85]

## 深度分析

### AI 工具供应链安全的新挑战

AI 工具加剧了供应链问题。过去是人类可能在机器上安装随机软件，现在是 Agent 可以替他们完成——而真正使用 Claude Code 时，谁不使用 `--dangerously-skip-permissions`？^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:17-18]

未来数年，我们将在 AI 工具领域重新学习大量供应链安全教训。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:18-19]

### 插件生态系统的结构性风险

插件市场继承了包注册表的最古老问题：

1. **信任委托**：市场委托给外部维护者，但委托关系缺乏持续验证^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:88-89]
2. **命名空间脆弱性**：GitHub 用户名/组织名的可转移性创造了固有的抢注风险^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:30-35]
3. **UI/UX 的信任暗示**："查看主页"等功能给用户心理暗示——这是官方验证的安全资源^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:58-60]
4. **更新机制的盲点**：SHA 更新流程可能成为攻击者注入恶意代码的通道^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:68-75]

### 对比：VSCode 扩展市场的教训

报告提到"看看所有那些恶意的'已验证' VSCode 插件"。这表明：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]

- 市场验证 ≠ 安全保证
- 官方列表 ≠ 可信代码^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:87-87]

组织应将 Claude 插件市场视为任何其他第三方代码——不可信。仅仅因为它在某个官方列表中，并不意味着它是安全的。^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:86-88]

## 实践启示

### 对组织的建议

1. **将社区插件视为不可信代码**
   - 即使经过 Anthropic 审核，也应遵循第三方代码的最小权限原则^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:86-88]
   - 建立内部插件白名单机制，而非直接依赖市场目录^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:86-88]

2. **审计现有插件**
   - 检查团队已安装的社区插件来源^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:86-88]
   - 验证仓库所有者的活跃度和可信度^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:86-88]

3. **限制 `--dangerously-skip-permissions`**
   - 该标志绕过权限检查，与供应链攻击结合时风险倍增^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:17-18]
   - 在团队 CI/CD 中强制禁用此标志^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:17-18]

### 对开发者的建议

1. **插件安全设计原则**
   - 固定依赖到不可变引用（SHA > 分支 > 标签）^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:66-67]
   - 避免 UI 功能将用户导向外部命名空间^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:58-60]
   - 实现仓库健康检查（所有者存在性、活跃度）^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:30-35]

2. **监控命名空间**
   - 如果组织维护开源项目，监控相关用户名的抢注尝试^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:76-77]
   - 考虑注册常见变体和拼写错误^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:76-77]

### 对安全研究员的建议

供应链和社交工程是入侵组织的最简单途径。AI 工具领域尚未充分探索，存在大量类似机会：^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md]

- MCP 服务器市场
- AI Agent 工具注册表
- LLM 插件生态系统^[raw/articles/repo-jacking-anthropics-claude-community-plugins.md:15-19]

→ [[raw/articles/repo-jacking-anthropics-claude-community-plugins|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

