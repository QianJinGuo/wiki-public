---
title: "AI 安全全景图"
created: 2026-07-02
updated: 2026-08-29
type: concept
tags: [security, ai, framework, agent-security, supply-chain]
provenance_state: inferred
confidence: 0.75
---

# AI 安全全景图

> 知识库中 security 标签覆盖 310+ 页面，是第四高频标签。本页提供 AI 安全领域的统一概念框架。

## 四大安全维度

### 1. Agent 安全
- **威胁模型**：提示注入、工具滥用、权限逃逸、上下文污染
- **防御体系**：沙箱隔离、权限分诊（P0-P3）、Stop Hook 门禁、HITL 爆炸半径控制
- **供应链安全**：repo-jacking、Skill 劫持、MCP 服务端伪造

### 2. 模型安全
- **对抗攻击**：越狱提示、数据投毒、模型窃取
- **对齐安全**：RLHF/DPO/GRPO 对齐策略、Red Teaming
- **透明度审计**：DiffusionGemma 案例、可解释性（mechanistic interpretability）

### 3. 数据安全
- **训练数据**：PII 泄露风险、版权合规、数据溯源
- **推理数据**：上下文中的敏感信息、会话隔离
- **知识库安全**：溯源链条断裂风险、推断内容标注

### 4. 基础设施安全
- **沙箱技术**：Firecracker microVM、ebpf syscall policy、临时环境
- **网络安全**：AWS Network Firewall AI 检测、MCP 通信加密
- **身份认证**：无长期 AK/SK、Agent 身份联邦、Okta NHI 管理

## 知识库中安全相关实体的分布

- Agent 安全：[[concepts/agent-security-threat-models]] / [[concepts/agent-security-full-lifecycle-system]]
- 模型安全："LLM 安全与红队测试" / "AI 安全治理"
- 供应链：[[entities/repo-jacking-anthropics-claude-community-plugins]]
- 沙箱：[[entities/mxc-execution-containers-internals-origin]]

> 智库使命：AI 安全维度不应沦为"技术附注"，而应是每次重大进展的必要评估维度。

## 所属 MOC

- [[moc/cybersecurity-privacy|Cybersecurity Privacy]]
