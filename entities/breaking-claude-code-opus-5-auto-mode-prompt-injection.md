---
title: "Breaking Claude Code Opus 5 Auto Mode — Prompt Injection攻击"
created: 2026-08-30
updated: 2026-08-30
type: entity
tags: [claude-code, prompt-injection, security, agent-safety, anthropropic, opus]
sources: [raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection]
confidence: 0.8
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

## 实践启示

Auto Mode作为Anthropic的"默认安全模式"被发现存在设计缺陷——安全分类器的粒度不足以区分"创建恶意进程"和"终止恶意进程"。这暴露了agentic系统安全的一个根本矛盾：安全机制需要足够的权限来阻止恶意行为，但同样的权限也可能阻止合法的清理操作。^[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection.md]

→ [[raw/articles/breaking-claude-code-opus-5-auto-mode-prompt-injection|原文存档]]
