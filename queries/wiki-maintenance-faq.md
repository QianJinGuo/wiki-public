---
title: Wiki 日常维护的最佳实践与常见问题解决方案？
created: 2026-04-30
updated: 2026-05-21
type: query
tags: [meta, wiki-maintenance, skill-system]
sources: []
confidence: high
---
# Wiki 日常维护的最佳实践与常见问题解决方案？

## 核心研究问题

Wiki 知识库的日常维护涉及哪些关键实践？如何系统性地解决常见问题？

## 一、Skill 自动创建与管理机制

**背景**：Hermes Agent 在完成 10+ 工具调用的多步骤工作流后，会自动将操作模式提取为 Skill 文件存入 `[本地运行时路径已隐藏]`。这是一种经验沉淀机制，而非用户主动创建或 LLM 自主决定的行为。

**Skill 评估标准**：
- **保留条件**：模式成熟（步骤有明确顺序和判断准则）、操作复杂（5+ 工具调用）、容易出错（有踩坑记录有价值）、复用频繁（每 1-2 个月会用一次以上）
- **删除条件**：临时修复（只解决特定 bug）、环境特定（路径/版本号写死）、过于简单（2-3 步能完成）

## 二、健康检查清单

### 每次批量操作后
- index.md Total pages 匹配实际文件数
- 所有 entity/concept/raw 文件都被索引，无缺失无重复
- 新入库页面至少有 1 个原文存档回链
- 新页面至少有 2 个 entity/concept 出链
- frontmatter 必填字段齐全（created/updated/type/tags/sources）
- tags 全部小写
- log.md 不超 500 行

### 每月巡检
- SCHEMA.md 是否需要更新（新字段/新tag/新约定）
- review metadata 使用是否一致
- 交叉引用密度是否保持（每个 entity 页至少 2 出链）
- 实体页面是否超过 200 行（需要拆分）
- confidence: low 页面是否需要验证/升级

## 三、跨 Skill 引用规范

**最佳实践**：每个 Skill 自包含其引用文件，不交叉引用其他 Skill 目录。跨 Skill 引用应视为 code smell——除非有明确的理由共享，否则优先自包含。

**悬挂引用修复**：当 Skill 的 SKILL.md 中出现对其他 Skill 文件目录的路径引用，但目标文件不存在或不属于该 Skill 管理范围时，将内容抽成独立文件放入本 Skill 的 `references/` 下。

## 四、frontmatter 健壮性规范

- 标题中有冒号必须用引号包裹
- tags 禁止 `#` 前缀和大小写混用
- sources 数组中禁止包含 `.md` 扩展名

## 五、patch 工具污染问题

在 index.md/log.md 的 `- ` 前缀列表区域执行 replace 时，patch 的 diff 算法会错误地将 `- ` 变成 `|- `。修复方法：
- patch 后用 `sed -i '' 'Ns/^|-/-/' file` 修复受影响行
- 或在 execute_code 中用字符串替换

## 六、维护节奏

1. 批量操作后检查清单
2. 每月全量巡检
3. log.md 超过 500 行时归档到 `log-2026-XX.md`

## 相关概念
-  — Skill 自动创建机制
-  — 知识库进化工具
-  — Skill 格式与编写规范
