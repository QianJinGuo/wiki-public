---
title: "zg（zvec-grep）：面向人与Agent的本地优先检索基础设施"
created: "2026-08-31"
updated: "2026-08-31"
type: entity
tags: [retrieval, search, agent-tools, rag, local-first, open-source, alibaba]
sources:
  - raw/articles/zg-zvec-grep-local-retrieval-infrastructure-alibaba-2026
confidence: 0.85
provenance_state: extracted
---

# zg（zvec-grep）：面向人与 Agent 的本地优先检索基础设施

阿里技术开源，基于 Zvec 向量检索 + BM25 + ripgrep，提供语义检索、BM25、混合检索和 rg 精确匹配，通过 CLI + MCP 双接口服务人与 Agent。^[raw/articles/zg-zvec-grep-local-retrieval-infrastructure-alibaba-2026.md]

## 核心问题

Agent 处理复杂任务时，检索输入从明确符号转变为自然语言描述，与代码/文档实际表述缺乏词汇对应。多轮搜索增加调用/时间/上下文消耗。^[raw/articles/zg-zvec-grep-local-retrieval-infrastructure-alibaba-2026.md]

## 四层检索能力

语义检索（向量）→ BM25（关键词）→ 混合检索（RRF 融合）→ rg 精确匹配。覆盖从模糊探索到精确验证的完整过程。^[raw/articles/zg-zvec-grep-local-retrieval-infrastructure-alibaba-2026.md]

## 多格式支持

- 代码：C/C++/Go/Java/JS/TS/Python/Rust/Vue/Svelte（结构化解析符号签名）
- 文档：Markdown（按标题章节提取）/RST/HTML/XML
- 数据：CSV/JSON/TOML/YAML

## 本地优先架构

- 内置 11 种端侧 Embedding 模型，默认 local/potion-code-16m-v2（16M，32MiB）
- Zvec 嵌入式索引，无需独立数据库
- 文件扫描/索引/检索 默认设备内完成，远程需显式授权
- Django 仓库（3,457 文件）M4 Pro 索引 <30 秒

## 量化评测

- **SWE-QA-Bench**（20 代码仓库问答）：工具调用 -50%+，Token -50%，得分 +1.50
- **BrowseComp-Plus**（80 深度研究问题）：准确率 98.67%→99.00%，Token -37.56%，工具调用 -43.52%，耗时 -38.58%^[raw/articles/zg-zvec-grep-local-retrieval-infrastructure-alibaba-2026.md]

## MCP 集成

`zg install` 自动发现 Codex/Claude Code/Cursor/OpenCode，完成 MCP 配置。同一份索引 CLI + Agent 复用。^[raw/articles/zg-zvec-grep-local-retrieval-infrastructure-alibaba-2026.md]
