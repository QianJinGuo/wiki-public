---
title: 庖丁解牛式 AI 知识体系学习路径（完整方案）
created: 2026-06-24
updated: 2026-07-22
type: concept
tags: [learning-path, moc, study-method, feynman, pomodoro, spaced-repetition, active-recall]
status: implemented
---

# 庖丁解牛式 AI 知识体系学习路径

> 把整个 wiki（concepts + entities + raw）按知识依赖深度分层，
> 配合费曼学习法、番茄钟、间隔重复、主动回忆等技巧，
> 让新手能「由浅入深、层层递进」地掌握 AI 知识体系。

---

## 一、设计理念

### 1.1 庖丁解牛

知识库 7000 页有内在依赖关系：不懂 Transformer → 看不懂 Prompt → 看不懂 Agent → 看不懂 Harness → 看不懂生产安全。本方案顺着这条依赖链编排。

### 1.2 学习法融入

| 学习法 | 应用 | 落地形式 |
|---|---|---|
| **费曼学习法** | 每章末尾「教给 12 岁孩子」 | 复习题费曼题 |
| **番茄钟** | 每章切 25min 块 | 章节分小节 |
| **间隔重复** | 第 N 章回顾 N-3、N-7 章 | 前置回顾环节 |
| **主动回忆** | 5 道题答案折叠 | 复习题 |
| **交错练习** | 每层末尾关卡混出多章 | 关卡题 |
| **细化难度** | 场景题为主非定义题 | 复习题第 3 题 |
| **双编码** | 关键概念配 Mermaid 图 | 章节内嵌图 |

### 1.3 覆盖范围

- **concepts/**（172 篇）：主干引用
- **entities/**（2400+ 篇）：按 tag 归类到层 MOC
- **raw/articles/**：按被引用关系索引
- **moc/**（38 篇）：复用已有主题

---

## 二、整体架构

```
[canonical wiki 路径已隐藏]
├── learning-path.md                  ← 总入口
├── layer-0-foundation.md             ← 第 0 层全库索引
├── layer-1-llm-principles.md          ← 第 1 层
├── layer-2-interaction.md            ← 第 2 层
├── layer-3-agent-engineering.md      ← 第 3 层
├── layer-4-ecosystem.md              ← 第 4 层
└── layer-5-production-security.md    ← 第 5 层

[canonical wiki 路径已隐藏]             ← 24 章正文
├── chap-01-ai-wave.md
├── ...
└── chap-24-governance.md
```

合计 31 个文件（1 总入口 + 6 层 MOC + 24 章正文）+ 本 blueprint = 32。

---

## 三、5 层 24 章

### 第 0 层：认知地基（1.5h）
1. AI 浪潮：为什么是现在（25min）
2. 软件的下一个范式（50min）
3. 知识管理的新形态（25min）

### 第 1 层：技术原理（4h）
4. Transformer 架构（50min）
5. Token 与上下文窗口（50min）
6. 训练三阶段（75min）
7. Scaling Law（25min）
8. 推理优化（50min）

### 第 2 层：交互实践（3.5h）
9. Prompt 工程基础（25min）
10. Prompt 模式（50min）
11. 上下文工程（50min）
12. RAG 检索增强（75min）

### 第 3 层：Agent 工程（3.5h）
13. 从对话到 Agent（50min）
14. Agent 记忆架构（75min）
15. 规划与推理（50min）
16. 工具使用与 MCP（50min）

### 第 4 层：生态工具（3.5h）
17. Harness 工程框架（75min）
18. 多 Agent 协作（50min）
19. 开源生态（50min）
20. 评测与基准（50min）

### 第 5 层：生产安全（3.5h）
21. 生产级 Agent（50min）
22. 可观测性（50min）
23. 安全威胁与防御（75min）
24. 治理与红线（50min）

**总用时：约 20 小时**

---

## 四、每章模板

```markdown
---
title: 第 N 章：XXX
estimated_minutes: 50
prerequisites: [chap-(N-1)]
---
# 第 N 章：XXX
## 🍅 番茄钟规划
## 📋 前置回顾（间隔重复）
## 🔍 预习
## 📖 正文（含 Mermaid 图）
## 🎯 重点回顾
## 🧠 费曼练习
## ✅ 复习题（5 道，答案折叠）
## 📚 拓展阅读（concepts/entities/raw）
## ⏭️ 下一章预告
```

---

## 五、层 MOC 模板

```markdown
---
title: 第 N 层全库索引
layer: N
---
# 第 N 层：XXX — 全库索引
## 本层导读
## 学习路径
## 本层 concepts（全量）
## 本层 entities（精选）
## 本层 raw
## 🚪 关卡（交错练习）
## 学完这层你应该能
```

---

## 六、entities 自动归类（待实现）

tag → 层映射表（Python 脚本 `scripts/build-learning-mocs.py`）：
- llm/transformer/attention → 第 1 层
- prompt/rag/context → 第 2 层
- agent/memory/mcp → 第 3 层
- harness/evaluation/framework → 第 4 层
- security/observability/production → 第 5 层

---

## 七、学习法落地

- **费曼**：每章末尾写一段解释给 12 岁孩子
- **番茄钟**：每章开头标注用时和切块
- **间隔重复**：每章前置回顾 N-3、N-7 章
- **主动回忆**：5 道题先答再展开答案
- **交错练习**：每层关卡混合多章
- **双编码**：Mermaid 图配关键概念

---

## 八、实施状态

✅ **全部 32 文件已创建**（本 blueprint + 总入口 + 6 层 MOC + 24 章正文）

按 5 层递进，每章含预习/正文/重点回顾/费曼练习/复习题/拓展阅读，配合 7 种学习法。

> 庖丁解牛，始于见全牛，终于游刃有余。24 章给见全牛的地图，真正的游刃有余要在实践中练。
## 相关链接

- [[moc/learning-path|学习路径 MoC]]
- [[concepts/ai-agent-exploration-path|AI Agent 探索路径]]
