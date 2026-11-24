---
title: "Breaking Claude Code Opus 5 Auto Mode — Prompt Injection攻击"
created: 2026-08-30
updated: 2026-11-21
type: entity
tags: [claude-code, prompt-injection, security, agent-safety, anthropropic, opus]
sources: [raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Breaking Claude Code Opus 5 Auto Mode — Prompt Injection攻击

> **Background**：Simon Willison报道的Johann Rehberger对Claude Code Opus 5 Auto Mode的prompt injection攻击研究。Auto Mode作为Anthropic对抗prompt injection的安全机制被发现存在关键漏洞。

## 攻击概述

Johann Rehberger发现了一种针对Claude Code Auto Mode的攻击方法，声称成功率约80%。攻击通过诱骗Claude Code下载并解压一个zip归档，然后执行导入base64的代码——但本地从归档中提取的`struct.py`文件会被实际执行。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md]

**关键发现**：Auto Mode安全机制本身成为了失败的一部分。分类器允许创建恶意进程，但随后阻止了旨在停止该进程的命令。

> "The safety mechanism itself can become part of the failure. The classifier allowed the creation of the malware process, but then it blocked the command intended to stop it!"

## 攻击链

1. 诱导Claude Code下载zip归档
2. 解压归档，其中包含恶意`struct.py`
3. 执行`import base64`——实际导入并执行本地`struct.py`
4. Claude检测到妥协，尝试终止恶意进程
5. Auto Mode阻止清理命令——安全机制反噬

## 防御建议

Johann的结论：运行无人值守的coding agent时，唯一安全的方式是使用沙箱：

- 在容器、VM或OS沙箱中运行agent
- 限制网络出口
- 监控agent行为
- 不要暴露home目录、SSH密钥、云凭证给agent运行时

## 深度分析

Johann Rehberger 的这次攻击之所以值得单独归档，不在于"约 80% 成功率"这个数字本身，而在于它揭示了一个结构性问题：当安全机制（这里是 Auto Mode 的命令分类器）的判断粒度粗于攻击行为的语义粒度时，防御系统会在最关键的止损环节反向生效。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12] 对一个被 Anthropic 设为默认并公开宣称高有效性的安全模式而言，这条失败路径不是实现 bug，而是分类器设计假设本身的裂缝。

### 一、分类器粒度错配：安全机制如何成为失败的一部分

Auto Mode 的安全分类器允许"创建恶意进程"通过，却在 Claude 自行察觉妥协、试图终止该进程时拦截了清理命令。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12] 这是一类典型的非对称失败：允许与拒绝的判定依据是命令的表面形态，而非操作的语义意图——也就是说，分类器区分的是"命令像什么"，而不是"命令在当前情境下为了什么"。"启动进程"与"终止进程"在系统层面是两类操作，分类器显然对"终止"配置了更高的敏感度——但在攻击场景里，终止恰恰是唯一正确的动作。Simon Willison 对此的概括值得原样保留："安全机制本身可以成为失败的一部分。分类器允许了恶意进程的创建，然后却拦截了本意是阻止它的命令！"^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]

这条观察与 [[concepts/agent-security-attack-defense|Agent 攻防]] 讨论中反复出现的结论一致：防线越靠近运行时、判定越依赖局部特征，误伤合法止损动作的概率就越高。对安全分类器而言，"拦截一切看起来像破坏的操作"在正常会话里是低误报策略，在对抗场景里却恰恰把恢复通道交给了攻击者——分类器的训练分布里几乎没有"agent 正在清理自己造成的感染"这类样本，于是最正确的行为反而成了最容易被拦截的行为。这不是 Auto Mode 独有的缺陷，任何以命令形态为主要特征的运行时审批层都可能踩中同一个坑。

### 二、攻击链逐环拆解：每一步都在打 harness 的信任边界

攻击入口并非新颖漏洞，而是对执行环境信任假设的系统性利用：诱导 Claude Code 下载并解压一个 zip 归档，归档中的恶意 `struct.py` 随后通过 `import base64` 的导入语义被间接执行——Python 的导入解析会优先命中本地同名模块，而 agent 的代码审查注意力集中在看似无害的 `base64` 导入语句上，不会意识到这会导入并执行从归档中提取的本地 `struct.py`。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12] 声称约 80% 的成功率说明这不是边缘案例，而是 Auto Mode 默认配置下的普遍暴露面。

该手法与 [[entities/skill-issues-compromising-claude-code-with-malicious-skills-agents|恶意 Skills 攻击 Claude Code]] 属于同一族：把恶意载荷伪装进 agent 天然会消费的工件（模块、Skill、归档文件）里，让 agent 自己的执行路径替攻击者完成最后一步。区别只在于伪装层——一个是 Skill 包，一个是导入名——信任边界被突破的位置完全相同：都是"agent 主动把不可信内容带入了自己的执行环境"。

### 三、"检测到妥协"不等于"能够止损"：被阻断的清理命令

这次攻击最值得记录的细节是复合失败的时序：在若干次运行中，Claude 确实检测到了妥协并尝试终止恶意进程，但 Auto Mode 拒绝了清理命令。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12] 换句话说，系统的感知层工作正常，失败发生在执行治理层——检测能力与处置能力被同一个安全机制隔断了。对无人值守 coding agent 而言，这意味着"模型足够聪明、能自己发现攻击"并不构成任何实际保障；从检测到恢复之间隔着的每一道策略闸门，都可能被攻击者当作最后一公里。更微妙的是，这种阻断在事后审计里看起来完全"合理"——每条被拦截的命令单看都是高风险操作，只有把攻击时序作为上下文拼起来，才能看出拦截本身构成了攻击的成功条件。这也解释了为什么这类漏洞难以靠常规回归测试发现：测试用例是单命令级别的，而失败是多命令级别的。

这个"发现异常后处置通道同样受限"的模式，与 [[entities/anthropic-claude-code-trojan-telemetry-security-2026|Claude Code 木马遥测安全事件]] 互为印证：agentic 系统的失效往往不是单点失效，而是感知、决策、执行三层之间的接力断裂。安全设计如果只在感知层投入（更强的分类器、更多的检测），而不打通"检测→授权→处置"的快速通道，检测结果的价值会被治理层的延迟和误拦消耗殆尽。

### 四、收敛解：沙箱是唯一被反复验证的物理边界

Johann 的结论与 wiki 已有的 [[concepts/agent-sandbox|Agent 沙箱]] 论述完全收敛：只要存在吸引对抗性攻击注意力的风险，无人值守 agent 的唯一安全运行方式就是沙箱。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12] 具体工程清单：在容器、VM 或 OS 沙箱中运行 agent；限制网络出口；持续监控 agent 行为；绝不向 agent 运行时暴露 home 目录、SSH 密钥、云凭证等高价值资产。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]

这四条与 [[entities/sandboxing-an-ai-agent|AI Agent 沙箱化]] 的分层隔离思路一致，核心思想也相同：不是让分类器变得更聪明，而是让"即使分类器失效，损害也被物理边界封顶"。提示注入作为长期攻防博弈的背景，参见 [[concepts/prompt-injection-defense|提示注入防御]]——本案例的贡献在于给这场博弈补上了"防御机制自身成为攻击面"这个少见但致命的样本。

关键机制清单：

1. 载荷投递：诱导 agent 下载并解压 zip 归档，恶意 `struct.py` 随解压落盘。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]
2. 导入劫持：`import base64` 触发本地同名模块解析，实际导入并执行归档提取出的 `struct.py`。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]
3. 妥协检测：Claude 在运行中自行发现异常，尝试终止恶意进程。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]
4. 止损阻断：Auto Mode 分类器拦截清理命令，恶意进程得以继续运行。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]
5. 边界兜底：沙箱隔离 + 网络出口限制 + 凭证不暴露，使分类器失效时的损害上限物理可控。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md:12-12]

## 实践启示

Auto Mode作为Anthropic的"默认安全模式"被发现存在设计缺陷——安全分类器的粒度不足以区分"创建恶意进程"和"终止恶意进程"。这暴露了agentic系统安全的一个根本矛盾：安全机制需要足够的权限来阻止恶意行为，但同样的权限也可能阻止合法的清理操作。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md]

→ [[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection|原文存档]]
