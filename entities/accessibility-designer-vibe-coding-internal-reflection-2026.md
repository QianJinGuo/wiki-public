---
title: "无障碍设计师 vibe coding：当所有同事都在用 AI 写代码时"
description: "Eric Bailey（GitHub accessibility designer）2026-06-15 发表：当团队 100% 转向 LLM-first 工作流、且个人 token 使用被追踪排名时，无障碍设计师也被结构性强制 vibe coding。文章探讨：(1) LLM 让不能写 JS 但能写 detailed specs 的设计师能直接做出产品增强；(2) \"vibe coding\" 的真实定义远不止简单提示词；(3) 质量提升（aria-label、信息层次、互动组件）vs 实际可访问性的张力。"
source: "[[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding]]"
created: 2026-06-18
updated: 2026-09-07
type: entity
tags: [vibe-coding, accessibility, llm, ai-coding, ux, aria, github, design, agentic-coding]
review_value: 7
review_confidence: 7
review_recommendation: worth-reading
review_stars: 4
sources:
  - raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding
confidence: high
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 无障碍设计师 vibe coding：当所有同事都在用 AI 写代码时

> 原文存档：[[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding|原文存档]] ^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

## 概述

GitHub accessibility designer **Eric Bailey** 2026-06-15 发表的内部反思：当团队 100% 转向 LLM-first 工作流（GitHub 自家 app 也是 LLM-driven），且个人 token 使用被追踪排名时，**无障碍设计师也被结构性强制 vibe coding**。作者坦诚：作为详细 spec 写得好但 JS 写不好的人，借助 LLM 现在能直接做出产品级无障碍增强。^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

文章核心张力：**Emotionally（"有人拿枪指着我的头"+ 三万亿柴油机碳排放）vs Professionally（必须用 LLM 才能贡献业务）**。这反映 LLM 在 2026 年企业内部已从"个人选择"变为"结构性要求"。^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]


## "Vibe coding" 的真实定义（作者版本）

作者澄清：**vibe coding ≠ 简单英语提示词**。完整定义包括：^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

- 简单的英语语言请求
- 更技术性的 plans
- 纠正性 instruction files（instruction 文档）
- 脚本
- **skills**
- 其他适用的技术

**作者 NOT 在做**：直接写代码。^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

## 实际产出（GitHub App 无障碍改进）

通过 vibe coding 实现的真实产品增强：^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]


| 改进 | 类别 |
|------|------|
| 互动列表（interactive lists） | 组件级 UX |
| Treeviews + F6 导航 | 键盘可达性 |
| Typeahead 节点选择 | 效率 |
| aria-label 构造逻辑 | 屏幕阅读器体验 |

**关键洞察**：作者强调他做的不是"技术合规但实际不可用"的产品，而是**真正提升无障碍体验的设计师级判断**。无障碍设计师的判断力 = 把"应用状态/配置无关的最显著信息放在最前面"的规则化能力。^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

## 关键贡献

1. **首次从企业内部身份视角（无障碍设计师）描述 vibe coding**：与"软件工程师 vibe coding"叙事不同，无障碍设计师代表**非编码专业人员的 LLM 工作流适配**，是 LLM 时代"工作身份重构"的早期信号。^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

2. **Token 使用被追踪/排名 = 新型绩效管理**：作者提到 token 使用被追踪排名，意味着企业内部 LLM 化已不仅是技术变革，**也是 HR/绩效管理变革**。这是 2026 年新兴的"AI 时代员工评估"问题。

3. **"vibe coding = LLM 时代的 T 型技能扩散"**：作者总结 — vibe coding 让他这个"详细 spec 写得好 + JS 写不好"的人能产出完整产品增强。LLM 实质上让**单一技能的专家**（spec 写作者、设计判断者）**跨过编码门槛**，直接产出。

## 一句话定位

**当企业 100% 转向 LLM-first 工作流，非编码专业人员的 vibe coding 不再是"个人选择"而是"结构性要求"** —— GitHub 无障碍设计师的第一手内部反思 ^[raw/articles/the-case-for-an-accessibility-designer-vibe-coding-when-all-his-coworkers-are-also-vibe-coding.md]

## 相关实体

- [[moc/coding-agent-practice|MOC]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]
- 相关: Agent 架构

