---
title: "高价率运营 AI 工作台：约定驱动与 AI 编排的评测优化实践"
created: 2026-07-17
updated: 2026-07-27
type: entity
tags: [ai-testing, evaluation-framework, llm-judge, agent-evaluation, skill-system, convention-over-configuration, gold-standard, rubrics, test-automation, taobao, alibaba, production-practice]
sources:
  - raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization
confidence: 0.85
provenance_state: extracted
---

# 高价率运营 AI 工作台：约定驱动与 AI 编排的评测优化实践

淘宝（大淘宝技术/营销&交易技术）建设的高价率运营 AI 工作台，基于"约定驱动 + AI 编排"架构，将 LLM Agent Skill 评测体系作为一等公民，实现 Skill 可用性从主观判断到可量化、可复跑、可对比的工程闭环。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

## 核心架构

**约定驱动 + AI 编排**：通过标准化目录结构（skills/ / skill-data/ / pinchbench-suite/）固化规范，通过 Claude Code 作为编排器实现自然语言驱动的全流程自动化。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

三层体系：
- **业务层**：16 个业务 Skill
- **规范层**：14 个通用评测维度 + 各 Skill 专项 rubric + 216 条评测用例
- **执行层**：auto-evaluation（评测大脑）+ pinchbench-eval（执行引擎）双引擎

## 评测体系核心设计

### 评测集生成
基于真实业务数据（MCP 工具查询员工岗位、高价率目标、高价商品汇总），AI 自动生成 6 类评测用例（典型/边界/追问/格式/触发词/真实 Case）。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

### 评测指标体系
14 个通用评测维度（路由准确性、流程遵循、参数正确性、工具合规、名称编码区分、输出合规、意图理解、异常处理、性能效率、表达清晰度等），按场景分为 4 类。采用 **severe_violation** 机制：若任一严重维度得 0 分，整体评分钳制到严重维度的最小值。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

### LLM Judge 评分
**二元评分**（0.0 或 1.0，禁止中间分数），评估 prompt 采用 6 段式结构，通过 Write 工具保存评分 JSON。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

### 双引擎架构
- **auto-evaluation**：6 Phase 闭环（初始化→生成→执行→标注→蒸馏→上传），最大 10 次迭代，AI 不能自动应用修改
- **pinchbench-eval**：项目无关的执行引擎，6 种运行模式，可整包迁移到其他 Agent 项目^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

## 金标（reference_data）设计

4 个关键陷阱：^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

1. **信息泄漏**——金标混进被测 Agent 上下文 → 类型层面隔离 runner/judge 输入
2. **粒度太细**——expected_flow 列到内部函数 → 粗化为"语义大阶段"，只校验大阶段顺序
3. **答案污染**——LLM 自动生成金标时参考了 Agent 实际输出 → 人工过审 + 限制参考来源
4. **多轮覆盖不足**——只检查最后一轮 → 增加 followup_expectation 字段

## 业务成果

- 16 个业务 Skill，14 个通用评测维度，216 条评测用例
- 15/16 个 Skill 定制了专项 rubric_config.json
- 单次评测从"5 小时人肉"变成"40 分钟无人值守"
- 反馈周期从周缩短到小时

## 真实挑战（飞轮的三个难点）

### 难点一：入口——线上问题怎么被看见
当前评测集是"产研团队想到的"，不是"用户问出来的"。5% 的真实 Case 靠人工从钉钉群截图转录。理想链路：线上日志→异常检测→Case 抽取→自动入库，尚未实现。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

### 难点二：出口——修改建议如何改对地方
LLM 给修改建议约 80% 不合格。根因：LLM 区分不了四种"低分"原因（Skill 逻辑/脚本 bug/rubric 过严/评测集污染），默认偏好"改 SKILL.md"。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

### 难点三：记忆——没有修改痕迹的飞轮
四类决策 log 设计（annotation/rubric_change/skill_change/ai_suggestion），JSONL 格式按月分文件，ID 互引构成因果图。目前实例为零——模板齐全但尚未落地。^[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization.md]

## 方法论贡献

- **约定驱动（Convention over Configuration）**：把规范沉淀到目录结构，Coding Agent 强制执行
- **SKILL.md 单一事实源**：评测集、rubric、Skill 定义三者绑定在同一份文件上同步演进
- **severe_violation 机制**：通过钳制规则确保核心能力底线不被其他高分维度稀释
- **二元评分**：强制评测者做出明确判断，消除模糊中间分数
- **评估 prompt 6 段式结构**：frontmatter + 维度展开 + 占位符 + schema + 计算公式 + 接入说明

→ [[raw/articles/taobao-high-price-rate-ai-workbench-eval-optimization|原文存档]]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

