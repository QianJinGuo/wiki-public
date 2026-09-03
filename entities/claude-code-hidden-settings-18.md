---

type: entity
id: 17356436
title: Claude Code 18个隐藏设置
slug: claude-code-hidden-settings-18
tags: [claude-code, settings, optimization, token-cost, agent]
source: ChallengeHub
source_url:
author: Mnimiy (@Mnilax) via ChallengeHub
published: 2026-05-24
rating: 7
confidence: 0.7
types: [claude-code-config, agent-engineering]
review_value: 7
review_confidence: 7
created: 2026-05-25
updated: 2026-08-29
sources:
  - raw/articles/claude-code-hidden-settings-18
---

# Claude Code 18个隐藏设置

原文：Mnimiy (@Mnilax)，2026-05-23。18个设置分为三组：Claude.ai(8个)、Claude Code(7个)、API/Console(3个)。 ^[raw/articles/claude-code-hidden-settings-18.md]

## Claude Code 核心配置（settings.json）

### 1. enabledPlugins：禁用而非卸载
```json ^[raw/articles/claude-code-hidden-settings-18.md]
{ ^[raw/articles/claude-code-hidden-settings-18.md]
  "enabledPlugins": { ^[raw/articles/claude-code-hidden-settings-18.md]
    "formatter@acme-tools": true, ^[raw/articles/claude-code-hidden-settings-18.md]
    "old-experiment@personal": false ^[raw/articles/claude-code-hidden-settings-18.md]
  } ^[raw/articles/claude-code-hidden-settings-18.md]
} ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
每个活跃插件加载 3K-8K token。false 保持安装但不加载，需要时用 `/plugin enable name@marketplace` 重新打开。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 2. permissions.deny：已知 bug，需 OS 层备份
```json ^[raw/articles/claude-code-hidden-settings-18.md]
{ ^[raw/articles/claude-code-hidden-settings-18.md]
  "permissions": { ^[raw/articles/claude-code-hidden-settings-18.md]
    "deny": [ ^[raw/articles/claude-code-hidden-settings-18.md]
      "Read(.env)", "Read(.env.*)", "Read(**/*secret*)", ^[raw/articles/claude-code-hidden-settings-18.md]
      "Bash(rm -rf:*)", "Bash(sudo:*)" ^[raw/articles/claude-code-hidden-settings-18.md]
    ]^[raw/articles/claude-code-hidden-settings-18.md]
  } ^[raw/articles/claude-code-hidden-settings-18.md]
} ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
**已知 bug**：deny 规则有时不生效（GitHub issue #11544）。必须配合 `chmod 600 .env` OS 层防护。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 3. hooks.SessionStart：按分支加载 context
```json ^[raw/articles/claude-code-hidden-settings-18.md]
{ ^[raw/articles/claude-code-hidden-settings-18.md]
  "hooks": { ^[raw/articles/claude-code-hidden-settings-18.md]
    "SessionStart": [{ ^[raw/articles/claude-code-hidden-settings-18.md]
      "matcher": "startup", ^[raw/articles/claude-code-hidden-settings-18.md]
      "hooks": [{ ^[raw/articles/claude-code-hidden-settings-18.md]
        "type": "command", ^[raw/articles/claude-code-hidden-settings-18.md]
        "command": "cat .claude/context-$(git branch --show-current).md 2>/dev/null || true" ^[raw/articles/claude-code-hidden-settings-18.md]
      }]^[raw/articles/claude-code-hidden-settings-18.md]
    }]^[raw/articles/claude-code-hidden-settings-18.md]
  } ^[raw/articles/claude-code-hidden-settings-18.md]
} ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
main 分支加载 context-main.md，feat/auth 加载 context-feat-auth.md，防止 CLAUDE.md 滚雪球到 5K token。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 4. disableAllHooks：紧急开关
```json ^[raw/articles/claude-code-hidden-settings-18.md]
{ "disableAllHooks": false } ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
80% 的怪现象（莫名跑命令、会话挂起）由某个 hook 乱触发引起。出问题时切到 true 定位。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 5. model per-project 覆盖
```json ^[raw/articles/claude-code-hidden-settings-18.md]
// .claude/settings.json 在 docs 项目 ^[raw/articles/claude-code-hidden-settings-18.md]
{ "model": "claude-haiku-4-5-20251001" } ^[raw/articles/claude-code-hidden-settings-18.md]
// infra 项目 ^[raw/articles/claude-code-hidden-settings-18.md]
{ "model": "claude-sonnet-4-6" } ^[raw/articles/claude-code-hidden-settings-18.md]
// core-engine 项目 ^[raw/articles/claude-code-hidden-settings-18.md]
{ "model": "claude-opus-4-7" } ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
打开项目自动用对的模型，避免用 Opus 价格做 Haiku 1/20 成本就能搞定的事。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 6. mcpServers enabled 标志
```json ^[raw/articles/claude-code-hidden-settings-18.md]
{ ^[raw/articles/claude-code-hidden-settings-18.md]
  "mcpServers": { ^[raw/articles/claude-code-hidden-settings-18.md]
    "github":   { "command": "...", "enabled": true }, ^[raw/articles/claude-code-hidden-settings-18.md]
    "postgres": { "command": "...", "enabled": false }, ^[raw/articles/claude-code-hidden-settings-18.md]
    "slack":    { "command": "...", "enabled": false } ^[raw/articles/claude-code-hidden-settings-18.md]
  } ^[raw/articles/claude-code-hidden-settings-18.md]
} ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
每个 server 吃 800-6000 token schema。12 个连接里可能只有 3 个在用，剩下 9 个每次会话消耗 25K-40K token。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 7. cleanupPeriodDays：180 天
```json ^[raw/articles/claude-code-hidden-settings-18.md]
{ "cleanupPeriodDays": 180 } ^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
默认值 30 天，Dreaming 和过往对话搜索都依赖这些 transcript。改成 180 天有 6 倍信号可用，磁盘约 200MB。 ^[raw/articles/claude-code-hidden-settings-18.md]

## API/Console 高杠杆设置

### 8. cache_control 断点位置（最大单一成本杠杆）
```python ^[raw/articles/claude-code-hidden-settings-18.md]
# 错误：断点放在用户消息后
messages = [ ^[raw/articles/claude-code-hidden-settings-18.md]
    {"role": "system", "content": SYSTEM_PROMPT}, ^[raw/articles/claude-code-hidden-settings-18.md]
    {"role": "user", "content": user_question, ^[raw/articles/claude-code-hidden-settings-18.md]
     "cache_control": {"type": "ephemeral"}} ^[raw/articles/claude-code-hidden-settings-18.md]
]^[raw/articles/claude-code-hidden-settings-18.md]

# 正确：断点放在稳定系统提示词后
messages = [ ^[raw/articles/claude-code-hidden-settings-18.md]
    {"role": "system", "content": SYSTEM_PROMPT, ^[raw/articles/claude-code-hidden-settings-18.md]
     "cache_control": {"type": "ephemeral"}}, ^[raw/articles/claude-code-hidden-settings-18.md]
    {"role": "user", "content": user_question} ^[raw/articles/claude-code-hidden-settings-18.md]
]^[raw/articles/claude-code-hidden-settings-18.md]
``` ^[raw/articles/claude-code-hidden-settings-18.md]
缓存读取仅需 10% 基础价格。盈亏平衡点：TTL 窗口内被读 2 次以上就划算。用 1h TTL 做跨会话不变的系统提示词。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 9. inference_geo 数据驻留溢价
不设 inference_geo 参数可省 10%（Opus 4.7+ 在美国地区额外溢价）。仅在合规合同明确要求时才设。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 10. Workspace 级别速率限制
每个 workspace 设账户层级的 60%-70%，留 30% 给突发需求。per-feature 上限防止一个功能把另一个饿死。 ^[raw/articles/claude-code-hidden-settings-18.md]

## Claude.ai 关键设置

- Memory：打开 Scope per Project 解决漂移问题
- Extended Thinking：默认设"轻量"，省 18-25% token
- Custom Styles：每个工作流配一个，是输出契约而非语气调节器
- Projects Instructions：填项目指令字段（相当于给 Claude 的 CLAUDE.md），70% 的项目都是空的
- 联网搜索引用：切到"脚注"模式，避免复制粘贴角标乱掉

## 审计脚本

```bash ^[raw/articles/claude-code-hidden-settings-18.md]
#!/usr/bin/env bash ^[raw/articles/claude-code-hidden-settings-18.md]
# claude-audit.sh
echo "=== 已启用插件数 ===" ^[raw/articles/claude-code-hidden-settings-18.md]
jq '.enabledPlugins // {} | to_entries | map(select(.value==true)) | length' ~/.claude/settings.json ^[raw/articles/claude-code-hidden-settings-18.md]
echo "=== 已启用 MCP server 数 ===" ^[raw/articles/claude-code-hidden-settings-18.md]
jq '.mcpServers // {} | to_entries | map(select(.value.enabled==true)) | length' ~/.claude/settings.json ^[raw/articles/claude-code-hidden-settings-18.md]
echo "=== cleanupPeriodDays ===" ^[raw/articles/claude-code-hidden-settings-18.md]
jq '.cleanupPeriodDays // 30' ~/.claude/settings.json ^[raw/articles/claude-code-hidden-settings-18.md]
# 目标：插件 3-5 个，MCP 3 个，cleanupPeriodDays 180
``` ^[raw/articles/claude-code-hidden-settings-18.md]

## 未入选设置（已排除）

- Adaptive Reasoning 开关：默认已最优，手动覆盖无显著效果
- Skill 自动激活：渐进式披露已调教很好，保持开启
- Workspace max_tokens：会毁掉代码生成场景

## 深度分析

### 成本控制的层级思维

这18个设置揭示了一个核心模式：Claude Code的成本优化本质上是**信息路由与缓存策略的工程问题**。enabledPlugins、mcpServers、cleanupPeriodDays 都围绕同一个目标——减少每次会话的 token 消耗。^[raw/articles/claude-code-hidden-settings-18.md]

**插件与 MCP 的 token 陷阱**最容易被忽视。每个插件加载3K-8K token，每个 MCP server 的 schema 吃800-6000 token。这些是静默成本——不会触发任何警告，但累计效应显著。12个MCP连接里可能只有3个真正在用，剩下9个每次会话消耗25K-40K token，这是一个月120万token级别的浪费。 ^[raw/articles/claude-code-hidden-settings-18.md]

**缓存控制的断点选择**是单一最大杠杆。cache_control 放在系统提示词后而非用户消息后，缓存读取仅需10%基础价格。被引用2次以上就划算，用1h TTL做跨会话不变的系统提示词，这是工程层面的精准控制。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 模型选择的成本经济学

per-project model 覆盖的精髓在于**用正确规模的模型处理对应复杂度的问题**。Haiku处理文档编辑（成本1/20 Opus），Opus处理核心引擎逻辑。这是token分配的基本效率——用Opus价格做Haiku能搞定的事是纯粹的浪费。 ^[raw/articles/claude-code-hidden-settings-18.md]

### Hook 机制的失控风险

hooks.SessionStart 按分支加载 context 是聪明的设计，但它也是双刃剑。disableAllHooks 作为紧急开关的存在说明80%的怪现象源自某个hook乱触发。hook机制强大但脆弱，需要谨慎管理。 ^[raw/articles/claude-code-hidden-settings-18.md]

## 实践启示

### 立即可行动的清单

1. **审计插件与 MCP**（5分钟）：运行审计脚本，确认活跃插件≤5个，活跃MCP≤3个。多余的立即设为false而非删除。 ^[raw/articles/claude-code-hidden-settings-18.md]

2. **调整 cleanupPeriodDays**（1分钟）：从30天改为180天。磁盘成本约200MB，但Dreaming和过往对话搜索的信号价值6倍于此。 ^[raw/articles/claude-code-hidden-settings-18.md]

3. **cache_control 断点审查**（代码相关）：检查所有API调用，确保断点放在稳定系统提示词后。这是最大单一成本杠杆。 ^[raw/articles/claude-code-hidden-settings-18.md]

4. **per-project model 配置**（团队协作）：为每个项目配置对应的默认模型。docs用Haiku，infra用Sonnet，core用Opus。 ^[raw/articles/claude-code-hidden-settings-18.md]

5. **权限双重保护**：permissions.deny 配合 `chmod 600 .env` OS层防护，deny规则有bug不能作为唯一防线。 ^[raw/articles/claude-code-hidden-settings-18.md]

### 中期优化方向

- **分支context策略**：实现按分支加载不同context文件的机制，防止CLAUDE.md滚雪球到5K token
- **MCP按需启用**：只保留当前项目真正需要的MCP连接，其余保持disabled状态
- **inference_geo审查**：确认是否真的需要数据驻留，除非合规合同明确要求否则不设置

### 关键警示

> **cache_control 是 30-90% 账单降幅的杠杆，但前提是断点位置正确**——放在不变的系统提示词后，而非每次变化的用户消息后。

> **permissions.deny 有已知 bug**（GitHub issue #11544），不能作为安全防线的唯一依托，必须配合 OS 层权限控制。

> **hook 机制是双刃剑**：强大的 context 加载能力伴随着失控风险，出问题时先切 `disableAllHooks: true` 定位。^[raw/articles/claude-code-hidden-settings-18.md]

## 相关实体
- [[entities/claude-code-performance-benchmarking]]
- [[entities/打造可靠的-ai-编程环境claude-code-hooks-完整开发者指南-v2]]
- [[entities/claude-code-source-architecture]]
- [[entities/claude-code-openclaw-memory-vector-db-doubt]]
- [[entities/skill-system-design-three-way-comparison]]
