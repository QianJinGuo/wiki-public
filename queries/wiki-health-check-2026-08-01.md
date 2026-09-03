---
title: "Wiki 健康体检报告 2026-08-01"
created: 2026-08-01
updated: 2026-08-07
type: query
tags: [meta, maintenance, health-check]
---

# Wiki 健康体检报告 2026-08-01

## 总体状态

| 指标 | 体检前 (07-31) | 体检后 (08-01) | 变化 |
|------|---------------|---------------|------|
| 总页数 | 7,459 | 7,778 | +319 |
| 错误数 | 0 | 0 | = |
| 警告数 | 289 | 277 | -12 |
| 孤儿页面 | 634 (8.5%) | 330 (4.2%) | -304 |
| 实体页面 | 3,509 | 3,827 | +318 |
| 概念页面 | 45 | 224 | +179 |
| MOC 页面 | - | 45 | - |
| 平均出链 | - | 3.8 | - |
| 平均入链 | - | 3.7 | - |

## 360° 优化完成项

### P0 级（关键）
- **P0.1 自动引用补全** (前序会话): 8,961 条引用添加到 1,070 个实体页面
- **P0.2 实体 sources 修复**: 75 个模糊匹配，49 个标记 inferred，1 个 frontmatter 修复

### P1 级（重要）
- **P1.3 孤儿收割机**: 317 个骨架实体页面从孤儿 raw articles 创建
- **P1.4 source_url 修复**: 518 个 raw articles 补充 source_url，595 个 frontmatter 分隔符修复
- **P1.5 占位页面填充**: 17 个 🚧 占位页面全部填充实质内容

### P2 级（改进）
- **P2.6 Lint 增强**: wiki-lint.mjs 添加 --fix-index-header 自动修复功能
- **P2.7 MOC 审计**: 199 个概念全部关联 MOC，添加反向链接，0 孤儿概念
- **P2.8 log.md 轮转**: 333 行，低于 500 阈值，无需操作

## 前序会话完成项
- 1,142 个 type 不匹配修复
- 27 个文件重命名（大写/空格），150+ 引用更新
- 364 个实体添加 sources
- 91 个孤儿概念映射到 13 个 MOC 页面
- 78 个 skills 同步

## 遗留项

### 非阻塞警告 (277 条)
- **EXCESS INFERRED (276条)**: 历史页面未标注引用，按 AGENTS.md 约定随新摄入有机解决
- **NO FRONTMATTER raw article (1条)**: 个别 raw article 无 frontmatter

### 待审查
- **20 个空 MOC**: 有 MOC 文件但未链接任何概念（部分为 entity-focused MOC，属正常）
- **330 个孤儿页面**: 多为新增骨架实体，等待内容填充和交叉引用

## 工具改进
- `scripts/wiki-lint.mjs`: 新增 `--fix-index-header` 和 `--fix-index-entries` 标志
- `scripts/auto-cite.py`: 自动引用补全工具 (v7)
- 前序会话: `scripts/wiki-contradiction-scan.py` 矛盾检测

## Git 记录
- `9c96e8665` - Phase 1: type 修复 + 文件重命名 + skills 同步
- `b5b8db903` - Phase 1.5: auto-cite 8,961 citations
- `a93c4815f` - Phase 2: P0.2-P2.8 完整优化

## 历史体检报告
- [[queries/wiki-health-check-2026-07-31|前一次体检报告]]
- [[queries/wiki-health-check-2026-08-07|2026-08-07 全身体检报告]]
