---
title: 第 3 章：知识管理的新形态——LLM Wiki
created: 2026-06-24
updated: 2026-06-26
type: concept
tags: [learning-path, chapter-03, layer-0]
estimated_minutes: 25
prerequisites: [chap-01, chap-02]
---

# 第 3 章：知识管理的新形态

> 📍 [学习路径](../../moc/learning-path.md) · [第 0 层](../../moc/layer-0-foundation.md) · 上一章：[第 2 章](chap-02-software-paradigm.md) · 下一层：[第 1 层](../../moc/layer-1-llm-principles.md)

## 🍅 番茄钟规划

25min，1 番茄钟：预习 → 正文 → 重点回顾 → 复习题 → 关卡

## 📋 前置回顾

- 第 1 章：vibe coding 为什么崩盘？
- 第 2 章：Software 3.0 的「程序」是什么？

## 🔍 预习

你正在读的这个知识库，本身就是一种**新范式**——不是普通笔记，是 Karpathy 提出的 **LLM Wiki**。理解它，你就知道为什么用 wikilink 而不是文件夹分类，为什么有 concepts/entities/raw 三层。

## 📖 正文

### 1.1 痛点：RAG 让知识无法积累

[[concepts/llm-wiki-paradigm|LLM Wiki 知识范式]] 描述：

> 拿到一篇几十页的行业报告，丢给 ChatGPT 聊了二十分钟，观点梳理得清清楚楚。第二天想引用其中一个数据点，打开新的对话窗口，重新上传报告，重新提问——一切从头来过。

这就是 RAG 的本质问题：**每次都是临时检索、临时回答**。知识停留在「能被检索到」，但页面之间没建立关联、没交叉引用、不会随新信息进化。

### 1.2 LLM Wiki：让 AI 当全职编辑

Karpathy 的方案：**与其让 AI 每次临时翻书，不如让它像一位全职百科全书编辑**，持续阅读资料、提炼整合、交叉引用，维护一部专属知识库。

三层结构：
| 层 | 内容 | 维护者 |
|---|---|---|
| 底层：原始资料库 | 论文、报告、文章原封不动 | 人类 |
| 中层：Wiki 知识库 | AI 生成的结构化笔记 | AI |
| 顶层：规则配置 | 告诉 AI wiki 的组织格式 | 人类 |

### 1.3 你正在用的知识库就是这个

- `raw/articles/` = 底层原始资料
- `concepts/` + `entities/` = 中层 Wiki（AI 生成）
- `moc/` + `SCHEMA.md` = 顶层规则

### 1.4 wikilink 是底层，不是装饰

> 没有双链，wiki 就退化为文件夹。

[[concepts/karpathy-llm-wiki-v2|Karpathy LLM Wiki V2]] 强调：wikilink（`Page`）不是装饰，是网络底层。它让每个知识点都能被其他页面引用，形成**网络效应**。

### 1.5 网络效应非线性

传统文件夹知识库：加一个文件，价值 +1。LLM Wiki：加一个文件，它能被 N 个已有页面引用，价值 +N。随着节点增长，知识库的「智能密度」非线性提升。

## 🎯 重点回顾

1. **RAG 痛点**：每次临时检索，知识无法积累
2. **LLM Wiki 解法**：AI 当全职编辑，持续维护结构化笔记
3. **三层结构**：raw（人）→ wiki（AI）→ 规则（人）
4. **wikilink 是底层**，没有它就退化成文件夹
5. **网络效应非线性**：节点越多价值放大越快

## 🧠 费曼练习

> 向 12 岁孩子解释「为什么这个知识库用 链接 而不是文件夹分类」。

提示：文件夹像把书放进不同柜子，wikilink 像给书贴满便利贴互相指。

## ✅ 复习题

1. **[选择题]** RAG 的根本问题是？ A. 检索速度慢 B. 每次临时检索，知识无法积累 C. 不能用 LLM D. 需要向量数据库
2. **[问答题]** LLM Wiki 的三层结构是什么？每层谁维护？
3. **[场景题]** 同事说「我用 Notion 分类整理资料，效果一样」。他缺了什么？
4. **[费曼题]** 用 3 句话向新手解释「为什么 wikilink 比文件夹更适合知识管理」。
5. **[关联题]** 回顾第 2 章 Software 3.0。LLM Wiki 的「AI 维护中层」属于几点几？

??? answer "参考答案"
    1. **B**
    2. 底层原始资料（人类维护）→ 中层 Wiki 笔记（AI 维护）→ 顶层规则配置（人类维护）。
    3. Notion 是文件夹分类，缺 wikilink 的网络效应。资料之间没交叉引用，加一个文件价值只 +1。
    4. 文件夹是静态归档，wikilink 是动态网络——后者让每个知识点都能被多处引用，形成关联。
    5. 属于 3.0（prompt 驱动 AI 生成 + 维护 wiki）。

## 📚 拓展阅读

- [[concepts/llm-wiki-paradigm|LLM Wiki 知识范式]] — 本章主源
- [[concepts/karpathy-llm-wiki-v2|Karpathy LLM Wiki V2]]
- [[concepts/wikilink-knowledge-graph|Wikilink 知识图谱]]
- [[entities/karpathy-llm-wiki-v2-2026|LLM Wiki V2 发布]]
- [[raw/articles/karpathy-llm-wiki-v2-2026|Karpathy LLM Wiki V2 原文]]

## 🚪 第 0 层关卡

恭喜完成第 0 层！回答 [第 0 层 MOC](../../moc/layer-0-foundation.md) 的 3 道关卡题。

## ⏭️ 下一层预告

第 1 层讲 **LLM 技术原理**——Transformer、Token、训练三阶段、Scaling Law。
