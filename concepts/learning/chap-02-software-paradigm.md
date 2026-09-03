---
title: 第 2 章：软件的下一个范式——Software 1.0/2.0/3.0
created: 2026-06-24
updated: 2026-06-26
type: concept
tags: [learning-path, chapter-02, layer-0]
estimated_minutes: 50
prerequisites: [chap-01]
---

# 第 2 章：软件的下一个范式

> 📍 [学习路径](../../moc/learning-path.md) · [第 0 层](../../moc/layer-0-foundation.md) · 上一章：[第 1 章](chap-01-ai-wave.md) · 下一章：[第 3 章](chap-03-knowledge-management.md)

## 🍅 番茄钟规划

50min，2 番茄钟：番茄1（三代定义）→ 番茄2（叠加效应+复习题）

## 📋 前置回顾

- 第 1 章：vibe coding 为什么崩盘？
- 第 1 章：决策层级最底层和最顶层分别是什么？

## 🔍 预习

第 1 章你看到了「vibe coding 会崩盘」。但崩盘不是因为 AI 弱——是因为**它处于一个新范式的不成熟阶段**。这一章给这个范式起名字：**Software 3.0**。

## 📖 正文

### 1.1 三代软件的定义

[[concepts/software-3-0-stack|Software 3.0 技术栈]]：

| 代 | 名称 | 「程序」是什么 | 谁来写 |
|---|---|---|---|
| 1.0 | 显式逻辑 | Python/Java 代码 | 人手写 |
| 2.0 | 隐式权重 | 神经网络权重 | 训练管线生成 |
| 3.0 | 自然语言 | Prompt + Context | 人写指令 |

### 1.2 Software 1.0：显式逻辑

Python、Java、C++。特征：**确定性**/**可调试**/**可读**。

### 1.3 Software 2.0：隐式权重

2020 年前后，AI 工程师不再写逻辑，而是写**训练管线**：准备数据集、设计网络、调超参，最后输出模型 checkpoint。特征：**隐式**（逻辑藏在几亿权重里）/ **统计性**（输出是概率分布）/ **数据驱动**。ResNet、BERT、GPT-3 都属于 2.0。

### 1.4 Software 3.0：自然语言

2023 年大模型爆发后，Karpathy 提出 3.0：**用自然语言 prompt 编程一个通用语言模型**。特征：**程序 = 上下文窗口**/**解释器 = LLM**/**非确定性**。

> 「上下文窗口即程序。编程的核心问题从『怎么写代码』变成了『哪一段文字应该复制给你的 Agent』。」—— Karpathy, 2026

### 1.5 叠加而非替代

关键认知：**三代是叠加的，不是替代关系**。一个现代 Agent 系统：用 **1.0**（Python）写 harness 和工具，调用 **2.0** 的模型权重，由 **3.0**（prompt + context）驱动行为。

### 1.6 为什么 Vibe Coding 会崩盘（回到第 1 章）

vibe coding 是 **Software 3.0 的早期形态**——只有 prompt，没有 harness（1.0），没有 verifier。崩盘不是因为 AI 弱，是因为**只有 3.0 这一层，缺了 1.0 的工程化和 2.0 的稳定性保障**。

## 🎯 重点回顾

1. **三代定义**：1.0 写代码 / 2.0 训练权重 / 3.0 写 Prompt
2. **3.0 的程序是上下文**，不是代码文件
3. **三代叠加**，不是替代——现代 Agent 三层共存
4. **vibe coding 崩盘**因为它只有 3.0，缺 1.0 的工程化

## 🧠 费曼练习

> 向 12 岁孩子解释「为什么写 Python 和写 Prompt 是不一样的事」。

提示：1.0 像给厨师精确食谱，3.0 像跟厨师聊天让他做菜。

## ✅ 复习题

1. **[选择题]** Software 3.0 的「程序」是什么？ A. 代码文件 B. 神经网络权重 C. 上下文窗口里的一段文字 D. 训练数据集
2. **[问答题]** 为什么说三代是「叠加」而非「替代」？举例说明。
3. **[场景题]** 同事说：「我直接用 ChatGPT 写代码，不用学 Python 了」。他理解错在哪？
4. **[费曼题]** 用 3 句话向新手解释「为什么写 prompt 和写代码是两种不同的工程」。
5. **[关联题]** 回顾第 1 章决策层级。目标层/代码层/CLI 层/Prompt 层/Agent 层，分别对应 Software 几点几？

??? answer "参考答案"
    1. **C**
    2. 现代 Agent：用 Python（1.0）写 harness → 调用 LLM 权重（2.0）→ 由 prompt 驱动（3.0）。三层各司其职，缺一不可。
    3. 他把 3.0 当成 1.0 的替代。但 ChatGPT 写出的代码仍是 1.0 代码，需要 1.0 的调试/测试/部署能力。
    4. 写代码是给确定指令，写 prompt 是给语义描述——前者追求精确，后者追求表达。
    5. 目标层≈跨代；代码层=1.0；CLI 层=1.0；Prompt 层=3.0；Agent 层=3.0+1.0+2.0 叠加。

## 📚 拓展阅读

- [[concepts/software-3-0-stack|Software 3.0 技术栈]] — 本章主源
- [[concepts/agentic-engineering-paradigm|agentic engineering 工程范式]]
- [[concepts/vibe-coding-paradigm|Vibe Coding 范式]]
- [[entities/karpathy-vibe-coding-agentic-engineering|Karpathy: 从 vibe coding 到 agentic engineering]]
- [[raw/articles/karpathy-vibe-coding-to-agentic-engineering|Karpathy 原文]]

## ⏭️ 下一章预告

第 3 章讲**知识管理的新形态**——LLM Wiki。
