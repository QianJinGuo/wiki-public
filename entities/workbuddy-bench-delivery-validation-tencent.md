---
title: "WorkBuddy Bench：从「修 Bug」到「完成工作」的 Agent 交付验收基准"
created: 2026-08-05
updated: 2026-09-07
type: entity
tags: [workbuddy-bench, agent-evaluation, benchmark, tencent, delivery-validation, artifacts, acceptance]
sources: [raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05]
confidence: 0.9
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# WorkBuddy Bench：从「修 Bug」到「完成工作」的 Agent 交付验收基准

腾讯 WorkBuddy 团队的评测基准：**Prompt/Context/Harness/Loop/Graph 管运行，WorkBuddy Bench 补验收端**——Agent 的"完成"由工件、状态和证据证明，而非对话结束。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 核心问题：假完成

Agent 找到失败测试改几行代码目标用例绿了，但跨时区订单没覆盖、接口少兼容字段、发布说明引用旧配置——修了 Bug 却没把工作交到下一个人手里。月度分析写完工作簿没更新；架构方案讲得顺但迁移顺序/回滚条件没落下来。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 完成四层

| 层次 | 看到了什么 | 还缺什么 |
|------|-----------|---------|
| 回答 | 一段解释/方案/代码片段 | 没有进入真实工作区 |
| 动作 | 改了文件、调用了工具 | 不确定结果是否完整交付 |
| 交付 | 留下补丁/网页/报表/PoC | 还要核对状态与约束 |
| 完成 | 交付物可用、状态一致、证据可复核 | 可以进入交接/发布/下一步 |

## 任务形态设计

不直接复用公开 Issue：从历史 commit、PR、真实 CVE 或业务场景反向还原任务，改写为同事间短请求。Code 任务用开发/算法/产品/QA/运维五种角色提需求，省略目标文件/根因/参考 diff/字段结构/边界条件。"请求可以留白，工作区不能没有线索"——信息缺失时 Agent 只能猜，评测失去稳定依据。隐私：借任务分布而非生产会话。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 任务包封装（可复跑）

```
task/
├── instruction.md # 自然语言请求
├── task.toml      # 类别、难度、资源与超时
├── environment/   # Docker 与 Agent 可见的工作区
├── tests/         # 任务结束后执行的评测资产
└── gold.patch     # Code 任务可选的诊断参考
```

固定边界：起点、可见性、工具/网络/资源权限、结果位置、验收程序。防污染：重构请求关闭"搜题面找答案"路径 + 数据集版本更新管理暴露；隐藏测试仅在求解期间不可见，公开后全量开放。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 四赛道与验收边界

| 赛道 | 任务数 | 验收方式 |
|------|--------|---------|
| Code | 80（18 细分类目，5 角色） | 找到契约：gold patch 验证 + 接口/字段检查 |
| Web | 70（35 从零 + 35 分布） | 留下工件：规则检查 + LLM/VLM 判断 + Agent Judge 实操 |
| Office | 50（xlsx/csv/PDF/文档/JSON/MD/文件树） | 保持一致：确定性规则（权重 0.70-0.95）+ LLM Judge 读固定证据 |
| Security | 60（38 红队 + 22 蓝队，真实 CVE） | 形成证据：确定性程序评分 + 五层反作弊 |

质量门槛：未修改基线得分 ≤ 0.3（防"什么都不做也能过"）；gold patch 后必须 1.0（确认可行解存在）。实测：bug_fix/api_contract 平均 0.47，feature_pipeline 0.94，testing 0.88——遗漏必需字段/参数形状/输出格式导致接口检查失败。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 评测结果与 Harness 敏感性

八张榜（CodeBuddy Code + Claude Code × 4 赛道）榜首：Code 双榜 Claude Opus 4.8（74.43/77.90）；Web Claude Opus 4.8（68.14/69.86）；Office Opus 4.8 82.37 / GPT-5.5 86.05；Security GLM-5.2（76.32/80.86）。**无模型包办四类工作**；同一模型换 Harness 表现变化显著（GPT-5.5 Security 从 cbc 第六升 cc 第二；MiniMax-M3 第二落第五；HY-3 passback 开启 Code +1.92~3.82）。评测记录必须保留模型+Harness+数据集+工具权限+指令协议。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 五份小合同（团队自建评测指南）

任务合同（目标/约束/停止点）→ 现场合同（基线/版本/数据/可见范围）→ 动作合同（工具/审批）→ 交付合同（结果位置/格式）→ 验收合同（规则/证据）。^[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05.md]

## 与其他实体的关系

- [[entities/mirrorcode-long-horizon-benchmark-epoch-ai-metr|MirrorCode]]（长时程编码基准）测"能跑多久多远"，WorkBuddy Bench 测"交付物是否真的完成"——互补
- [[raw/articles/lhtb-long-horizon-terminal-bench-musk-retweet-yucheng-shi-2026|LHTB]]（长时程终端评测）同样聚焦长任务，WorkBuddy Bench 扩展 Code/Web/Office/Security 四类工作现场
- [[entities/arbiteros-governance-kernel-cuhk-2026|ArbiterOS]] 管执行前授权，WorkBuddy Bench 管交付后验收——一个管开工前，一个管交付后
- Agent 评估基准 概念体系的新实例

→ [[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05|原文存档（若飞/架构师解读）]]
