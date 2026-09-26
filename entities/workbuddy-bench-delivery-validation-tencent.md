---
title: "WorkBuddy Bench：从「修 Bug」到「完成工作」的 Agent 交付验收基准"
created: 2026-08-05
updated: 2026-09-26
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

## 深度分析

### 长时程 × dense reward：完成度本身可以度量

LHTB（Long-Horizon Terminal-Bench）用 46 个长程终端任务把"完成"变成了连续量：单任务平均执行 85.3 分钟、消耗约 9.9M tokens、约 231 个 episodes，评分不问"最终绿没绿"，而用 dense reward 同时度量"做没做完"和"推进了多远"。结果很冷酷：15 个前沿模型中，最强模型 pass@1 仅 15.2%（0.95 阈值）/ 10.9%（1.0 阈值），模型均值只有 4.3% / 1.7%。这与 WorkBuddy Bench 的"完成四层"互为印证——当前 Agent 普遍卡在"动作"或"交付"层，真正抵达"完成"层的比例极低。榜单头部也远未饱和：Grok 4.5 以 0.505 均值登顶，solved 也只有 13/46；两个月前（5 月论文）15 个模型最高才完成 7/46，tetsuo 在推下的长评写道 "That ceiling is what fifth place looks like"——头部分数两个月内几乎翻倍，说明这个方向仍在快速爬坡期。

### Harness 敏感性：两个基准共同的头条结论

LHTB 发布者 Yucheng Shi（腾讯 HY LLM Frontier / Harness Handbook 共同作者）的核心观察是：同一个模型换一套 harness，表现可能完全不同——工具怎么组织、上下文怎么管理、失败后如何恢复，都直接决定 Agent 能否完成长程任务。这与 WorkBuddy Bench 的实测完全同频：GPT-5.5 Security 从 cbc 第六升至 cc 第二，MiniMax-M3 从第二跌到第五，HY-3 开启 passback 后 Code 提升 1.92~3.82。Grok 家族的变化更极端：4.2 以 0.080 在 LHTB 垫底，4.5 直接以 0.505 登顶——作者归因于 Cursor & XAI 把模型训练、RL、真实环境、harness 和评测闭环全部接起来后"进步速度肉眼可见"。两份榜单给出同一条方法论底线：评测记录必须同时保留模型 + Harness + 数据集 + 工具权限 + 指令协议，缺一项跨榜比较就失效。

### 无算力研究者的切入点：验收端与 verifier

LHTB 团队给资源有限的研究者指的路是不与大厂正面拼算力，转向 harness、context management、tool design、verifier 和 reward 这些"Agent 真正工作的系统"。WorkBuddy Bench 的五份小合同（任务/现场/动作/交付/验收）恰好构成一套可复用的 verifier 设计模板：验收合同规定规则与证据形式，交付合同规定结果位置与格式，加上未修改基线 ≤ 0.3 与 gold patch 后必须 1.0 的双重质量门槛——这与 LHTB 的 dense reward 评分属于同一条工程化路径：让"完成"可度量、可复跑、可审计。配套开源的 Harness Handbook 把复杂 harness 转化为人类可读的"行为地图"，与 WorkBuddy Bench 的任务包封装（instruction.md + task.toml + environment + tests）一样，都把可复现性当作一等公民。

### 榜单结构：不存在包办一切交付的模型

两份榜单的共同结构信号是冠军分散：WorkBuddy Bench 四赛道榜首分属 Claude Opus 4.8（Code/Web）、GPT-5.5（Office）、GLM-5.2（Security）；LHTB 前五名中 Claude 系占三席（Sonnet 5 / Opus 4.8 / Fable 5）但分数咬得极紧（0.487~0.497），Grok 4.5 和 GPT-5.6-Sol 各有强项。对团队选型的实际含义是：按工作类型（长程终端/编码/Web/Office/Security）分别跑基准，再按 harness 匹配做最终决策，比追求单一"最强模型"更贴近真实交付场景。

## 实践启示

1. **把"完成"写成分档验收标准**：按 WorkBuddy Bench 四层（回答→动作→交付→完成）为团队内 Agent 任务定义明确的验收边界，拒绝以"对话结束"为完成信号；每个任务的 instruction 里显式写清结果位置、格式与停止点。
2. **评测任何 Agent 前固定四元组**：模型 + Harness + 数据集 + 工具权限 + 指令协议必须完整留档——LHTB 与 WorkBuddy Bench 都证明换 harness 可造成名次级波动，不带 harness 元数据的跑分无法比较。
3. **给长任务上 dense/进度型评分**：不要只做二元 pass/fail；借鉴 LHTB 的 dense reward，记录"推进了多远"（如完成的子步骤数、通过的测试比例），否则长时程任务的中间失败全部不可见。
4. **用双门槛防"躺过"与"不可解"**：新任务入库前跑未修改基线（应 ≤ 0.3）和 gold patch（应 = 1.0），两者任一不过即说明任务本身有缺陷，而不是 Agent 不行。
5. **小团队的杠杆在验收与 verifier 而非算力**：五份小合同 + 确定性规则检查 + 固定证据的 LLM Judge 是可以在没有大规模 GPU 的前提下复用的整套模板；verifier 质量决定基准的可信度。
6. **选型按赛道分别评测，再按 harness 匹配**：不存在四类工作全优的模型；先确定团队的主场景（Code/Web/Office/Security 或长程终端），在该赛道榜单上选模型，再投入 harness 侧调优——这往往比换模型收益更大。

## 与其他实体的关系

- [[entities/mirrorcode-long-horizon-benchmark-epoch-ai-metr|MirrorCode]]（长时程编码基准）测"能跑多久多远"，WorkBuddy Bench 测"交付物是否真的完成"——互补
- [[raw/articles/lhtb-long-horizon-terminal-bench-musk-retweet-yucheng-shi-2026|LHTB]]（长时程终端评测）同样聚焦长任务，WorkBuddy Bench 扩展 Code/Web/Office/Security 四类工作现场
- [[entities/arbiteros-governance-kernel-cuhk-2026|ArbiterOS]] 管执行前授权，WorkBuddy Bench 管交付后验收——一个管开工前，一个管交付后
- Agent 评估基准 概念体系的新实例

→ [[raw/articles/workbuddy-bench-delivery-validation-tencent-ruofei-2026-08-05|原文存档（若飞/架构师解读）]]
