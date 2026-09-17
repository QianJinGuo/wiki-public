---
title: "企业 Agent 平台三支柱：统一 Harness + Skill Control Plane + 虚拟文件系统（腾讯云开发者/随机比特）"
created: 2026-09-17
updated: 2026-09-17
type: entity
tags: [agent-harness, skill-control-plane, virtual-filesystem, stripe-kai, deep-agents, progressive-disclosure, session-scope, vfs-sandbox-boundary, enterprise-agent]
sources: [raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026]
confidence: 0.9
provenance_state: extracted
---

# 企业 Agent 平台三支柱：统一 Harness + Skill Control Plane + 虚拟文件系统

腾讯云开发者（随机比特，2026-08-14 原创长文）：企业 Agent 从演示走向生产的分水岭不是模型而是运行时——三个更基础的问题（一次任务可以访问什么/分散在各团队的业务经验如何进入 Agent/跨越几百步的工作状态保存在哪里）没有统一答案，接入再多模型和 MCP 工具也只会得到能力强但不可治理的聊天机器人。解法是三支柱架构：**统一 harness 承载执行和安全，skill 分发领域能力，虚拟文件系统管理长任务的上下文、证据和产物**。^[raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026.md]

## 两种不可持续的落地方式（Stripe Kai 反例佐证）

①「一个场景一个 Agent」（各自复制 prompt 连各自工具，初期快但重试/权限/审计约定逐渐分叉）；②「直接把 coding agent 发给所有人」（默认面向工程工作区，知识工作者需要业务对象+受控数据+可分享报告，把强执行能力原样暴露=安全风险转嫁）。**Stripe Kai 案例全库零覆盖**：NoCode Agent Builder 曾产生超 4,000 个工作流 Agent → 相似提示词重复/质量不一/难统一维护 → Kai 转向共享运行平台+领域团队贡献 skills（产品与运行时边界重构，非检索优化）。^[raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026.md]

## 四层架构与 Harness 最小职责

入口层（surface-agnostic API，Web/IM/数据平台/工单/浏览器扩展共享）→ 控制面（skill/Agent 配置/默认工具集/评测集/版本质量信号，领域所有权的快速变化资产）→ 企业 harness（统一身份/会话范围/权限/审计/基础设施适配）→ 通用 runtime（模型调用/middleware/流式事件/checkpoint/恢复）。原则：**通用 Agent 问题只解决一次，企业特有问题留在企业层，领域知识交给最了解它的人维护**（Kai 用 Deep Agents/LangGraph 处理通用运行时+叠加 Stripe 安全与内部服务）。^[raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026.md]

## Skill Control Plane（全库零覆盖维度）

- **渐进披露只解决一半**：三层加载（L1 name+description 发现/L2 SKILL.md 执行策略/L3 scripts 按需）避免全部塞 system prompt，但**目录选择问题仍在**——Kai 披露 system prompt 叠加到约 150 个技能时 frontier model 选择质量已下降 → 从纯 LLM 选择转向「检索/分类器预筛 → LLM 精选」两阶段：第一阶段追求召回率（几百→十几项），第二阶段 LLM 精确选择，**真正执行时 harness 才注册被选 skill 允许的工具**（无关工具不是「不建议调用」而是根本不出现在模型请求里，同时降上下文成本与攻击面）
- **联邦所有权**：平台定义规范与运行时但不垄断知识，领域团队维护判断标准但绕不开平台约束（Kai 分层：基础技能固定+职能默认按画像装载+个人自增）
- **registry 最小数据模型**：仅 SKILL.md 入 Git 不够——机器可读 registry 把 risk（write→审批策略）/allowed_tools/data_scopes（project:${session.project_id}）/surfaces/eval_suite/status（promoted）变一等字段；registry 编译出运行时工具白名单、HITL 规则、可用入口、评测任务；CI 检查命名/目录一致性/重复描述/失效引用/无 owner promoted skill/**写风险无审批策略**等结构问题
- **两层评测**：第一层 catalog routing（该触发是否触发/近邻混淆/组合任务多选/无匹配拒绝硬套）先于第二层执行结果（工具正确/证据充分/产物结构/越权）；**近邻负例最有价值**（查询构建日志 vs 下载构建制品），目录越大越应报 top-1 accuracy/top-k recall/误触发/漏触发/token 成本/混淆对

## 虚拟文件系统（VFS）与状态三分类（全库零覆盖维度）

- **会话文件系统布局**：`/sessions/<id>/`{scope.json 权限快照, evidence/ 只读原始证据（默认不可被 Agent 覆盖防「修正」证据）, working/ 中间草稿, artifacts/ 用户可消费产物, checkpoints/, manifest.json 来源+hash+owner+审批+交付状态} + `/skills/`（版本化）+ `/memories/`（跨会话稳定偏好）+ `/shared/`（显式发布才进入——临时推理状态不无意变成组织事实）
- **VFS 与 Sandbox 必须两个边界**：文件持久化和代码执行不绑同一宿主——Agent runtime 在受控服务中经 VFS 管状态，需要 Python/图表/PDF 时 sandbox 作为工具调用；Kai 实现=S3 支撑多租户 VFS，执行前 materialize 到 sandbox 结束后 sync-out，Agent 本身在 sandbox 外；**sandbox 保护宿主环境，不代表 sandbox 内数据天然安全**（输入文件/网络/凭证/运行时长/CPU 内存/输出大小/sync-out 路径单独约束）
- **上下文膨胀缓解**：工具返回大日志 → 完整写 evidence/build-123.log，模型只收路径+摘要+行数+hash，后续 grep 分段取回；**summarization 压缩已发生对话 / 文件系统保留可再验证事实与产物 / checkpoint 保存执行状态——三者不能互相替代**
- **manifest 是 artifact contract**：report.md 生成≠交付——区分实时来源/数据窗口/本地验证/推送/部署/业务验收（validation: local passed/remote_pipeline not_run/deployed false/accepted false），让 UI、后续 Agent、审计系统读结构化事实而非从「已经完成」的自然语言猜状态
- **状态三分类**：SessionContext（创建后模型不可改：user/tenant/project/environment/capability）≠ AgentState（可 checkpoint：messages/plan/selected_skills/pending_approvals）≠ ArtifactState（VFS 路径与索引）——拆分防 summarization 意外改权限、防 checkpoint 反复序列化大文档

## 安全：从用户权限收缩到会话能力

**effective capability = user authorization ∩ agent configuration ∩ selected skill policy ∩ session/task scope ∩ tool-side enforcement**——用户有权访问客户 A 和 B，不等于针对 A 的任务可读 B；会话创建绑定 customer_id=A，工具服务端检查请求目标仍在 scope 内（换参数写法不能扩权，sandbox 脚本也不能绕过）。**Prompt 不是安全边界**（Deep Agents 明确 "trust the LLM" 模型，边界必须在工具/sandbox 层）；框架声明式 permissions 只约束内建文件工具，不覆盖自定义工具和 MCP → allowed-tools、MCP 权限、sandbox policy、业务 API 授权必须组合使用。一次请求执行链：scope 在 session gateway 固化（非模型自行推断）→ 候选 skill 逐步收窄 → 工具结果先成证据文件 → 写操作调用前审批 → 响应引用 manifest 状态。^[raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026.md]

## 演进四阶段与常见坑

阶段一资产与风险基线（registry/lint/eval baseline，无基线无法证明新架构更安全）→ 阶段二统一 harness 试点（高频/读多写少/明确 artifact 流程，先验证最基本闭环：任务可恢复/skill 选得对/证据可追溯/写操作可阻断/产物可接管）→ 阶段三两阶段 skill 路由（catalog 几十项时先测纯 LLM 路由，预筛层有净收益才引入向量检索复杂度）→ 阶段四 Trace-to-Skill 闭环（生产 trace 发现缺口 → 候选 patch+eval → owner review → 隔离环境回归 → 合并发布，**生产 Agent 不直接改生产 skill**）。五个坑：统一 harness 做成新单体 Agent（统一的是执行安全治理不是把领域说明塞回超级 prompt）/allowed-tools 当完整授权/VFS 当无限期知识库（无发布流程和 TTL=数据沼泽）/过早引入多 Agent/只验证成功路径（权限拒绝/工具超时/发布中断/sandbox 资源耗尽/skill 冲突/证据过期才是关键测试）。^[raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026.md]

## 与既有实体的关系

- `[[entities/deepseek-code-harness|DeepSeek Code Harness]]`：DSH 是 coding 场景 harness 深拆（Turn/Step 冻结/工具管线），本文是企业知识工作场景的三支柱平台架构，互补
- `[[entities/harness-engineering|Harness Engineering]]`：主题父集，本文提供 enterprise control plane 维度（skill registry 数据模型/联邦所有权/effective capability 交集模型）
- `[[entities/stripe-agent-economic-infrastructure-5-products|Stripe Agent 经济基础设施]]`：同 Stripe 不同侧——经济基础设施 vs Knowledge AI Platform 运行时

→ [[raw/articles/enterprise-agent-harness-skill-vfs-tencent-cloud-2026|原文存档]]
