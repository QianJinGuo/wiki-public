---
title: "从 ReAct 到 Agent Team：医疗系统研发任务的信息流与责任边界（若飞/架构师）"
created: 2026-09-18
updated: 2026-09-18
type: entity
tags: [react, agent-team, multi-agent, fhir, interface-contract, consistency, healthcare, parallel-benefit, idempotency, ruofei]
sources: [raw/articles/ruofei-react-to-agent-team-medical-dev-2026]
confidence: 0.9
provenance_state: extracted
---

# 从 ReAct 到 Agent Team：医疗系统研发任务的信息流与责任边界（若飞）

架构师 JiaGouX（若飞，2026-09-17，c=6 档原创）：以一个**只读临床信息汇总服务**（早 08:00 交班页面：检验/生命体征看最近 24 小时、用药分别呈现有效医嘱与执行记录、过敏保留既往有效记录）贯穿讲清 Agent Team 研发协作的信息流与责任边界。核心论点：**单测全绿，业务含义却没对齐**——字段名称相同业务含义也未必相同，先钉住接口语义再谈并行。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## 事件时间 vs 结果可用时间（FHIR 语义契约，全库零覆盖）

集成测试样例：检验记录 07:20 采样 10:00 才发布报告、生命体征 07:50 测量、医嘱有效但无给药记录、过敏接口返回空列表——单独看每个接口都正常，组合起来页面该显示什么？检验适配器单测只查数值/单位/报告时间可能完全通过，但合进「截至 08:00 已知的信息」，10:00 发布的报告就不该出现；医嘱有效不能直接写成「已给药」；**过敏接口空列表只能说明"这次没有查到记录"，不能顺手变成"患者无过敏"**。HL7 FHIR R4 语义：`Observation.effective[x]`（临床对应时间）vs `issued`（结果何时可用）；`MedicationRequest`（开立请求）vs `MedicationAdministration`（实际给药事件）。**"最近 24 小时"筛选的是事件发生时间；"截至 08:00"约束的是结果在何时可知**——复现历史页面需保存当时的源记录版本或查询快照，只在请求里加 `as_of` 并不能让上游接口自动具备历史查询能力。接口契约至少固定：patient_id/encounter_id、window_start/cutoff_time/timezone、effective_time/available_time/fetched_at、source_system/source_record_id/source_version、data_status/provenance——**事件时间、结果可用时间和本次拉取时间分开保存**；data_status 区分有数据/查询无记录/上游不可用；上游不提供记录版本时保存响应快照与拉取信息留核对依据。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## ReAct 循环与"测试通过只证明现有断言通过"

ReAct（推理与行动交替）：目标→判断下一步→调用工具→读取真实返回→检查是否更接近目标→继续/停止/交给人。让研发 Agent 修正检验适配器：读字段映射与查询契约→跑测试样例→发现 10:00 发布结果混进 08:00 页面→改过滤逻辑→重跑测试；工具返回具体失败断言和原始输入，下一轮才有依据。**但测试通过只证明现有断言通过——如果测试本身漏了报告可用时间，Agent 也可能一路顺利地把错误实现交回来**（ReAct 提供利用反馈的循环，反馈是否充分取决于测试和工具怎么设计）。研发环境权限/允许修改的目录/网络访问/执行预算由运行时控制；Agent 在隔离分支修改，合并/迁移/发布走工程系统已有流程；提示词里的限制需要有工具权限配合。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## 先拆责任再拆 Agent + 交接报告

契约未定时让实现和验收各猜一份定义很容易返工——**任务依赖要先于角色命名**。研发工作×Agent 协助×交付物表：身份与查询契约（对照主索引/就诊映射/时间定义，列待确认问题）→字段映射/契约草案/边界样例；检验与生命体征适配→代码变更/契约测试报告；用药与过敏适配（区分医嘱与执行记录，保留缺失和确认状态）→状态映射/异常路径测试；集成与复核（同一批数据回放）→集成报告/可复现阻塞项。每个产物都要能回到代码、输入和验收依据；接口定义有疑问时 Agent 提交问题及证据交工程师和业务确认，**不能为了继续执行自己把待确认项改成默认规则**。交接报告带字段映射/修改提交/失败与通过测试/未验证上游行为——单独一句"检验模块已完成"留给集成的人猜的事情太多。Anthropic 多 Agent Research 做法：子 Agent 把报告/数据/代码写入外部制品系统，再把路径/摘要/状态交回协调者（上下文不整段复制，后续人能回原始证据）。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## 一致性三件事 + 工作状态字段 + 结果状态机

三个共同确认：①交付的是同一版需求（相同患者与就诊映射规则）②实现和测试依据相同接口契约/回放数据/时间条件③子任务状态能回到工程记录（局部完成≠整体验收）。交接保留工作状态：global_task_id/sub_task_id、base_commit/commit_sha、schema_version/fixture_version、input_snapshot/query_contract_ref、confirmed_facts/assumptions、capability/constraints、artifact_refs/acceptance、status/version/attempt、lease_until/result_ref——**fixture_version 标识测试输入版本与契约版本一起保存，复核者才知道报告验证的是哪份代码哪组样例**。结果状态机：候选/已验证/冲突/已否决至少能区分，"测试命令执行成功"和"交付满足需求"不共用一个字段；复核发现 10:00 报告进入 08:00 页面时提交**可重放的失败样例**并退回时间过滤任务。消息延迟/重复/乱序：任务记录带 attempt 和 version，通过带条件的状态更新拒绝过期结果（旧消息不能覆盖新状态）；传统 CI 已有此规则——测试报告绑定提交，代码变化后旧报告不能证明新代码可交付。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## 信息流决定协作方式 + 并行收益核算

中心化汇总（协调 Agent 分派/收回/安排集成：权限预算任务归属清楚，但成员发现接口问题要经协调者转交、上下文转述丢失）vs 成员直接通信（Claude Code Agent Teams 共享任务列表+成员直接通信；**共享任务列表不等于业务数据库或发布记录**）。Akihiro Nakamura 对 Codex Multi-Agents 与 Claude Code Agent Teams 的区别也归到信息流：父 Agent 汇总还是成员直接交流。两个 Agent 商量好新的空值规则却只在消息里记一笔，其他适配器和测试仍用旧定义——**直接通信反而扩大错位；讨论可以发生在成员之间，确认后的变更要进入版本化契约并标出哪些任务/测试需要重跑**。并行检查沿依赖图：输入是否独立/产物是否独立/合并是否可以延后——**Agent 数量本身不能说明并行收益**。量化参照（比较对象不同都不能直接当医疗研发指标）：Anthropic Opus 4 协调+Sonnet 4 执行相对单 Agent Opus 4 +90.2%，但多 Agent 系统约消耗 15× token；Google Research 180 配置：Finance-Agent 集中式 +80.9% vs PlanCraft 严格顺序 -39%~-70%。Cedric Chee K2.5 Agent Swarm：天然并行/下载量大/深度研究较适合，软件开发还需细化子 Agent 提示/并行协调/工作流。衡量比较同一批任务的交付时间/返工次数/有效缺陷发现数/总调用成本——代码产出量本身很难说明是否更划算。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## 集成测试查业务含义 + 失败分诊 + 六方案选型

集成测试检查清单（非只验接口 200）：身份（patient_id/encounter_id 与授权范围一致）/时间（10:00 报告不进 08:00 页面）/用药（有效医嘱与实际给药分开显示）/过敏（空返回不生成"无过敏"，既往有效记录不被 24h 窗口过滤）/证据（页面事实回指原始记录及版本）/缺失（上游不可用/无记录/待发布分别表达）/边界（不输出诊断不改医嘱不写回临床事实）/审计（记录调用者/工具结果/版本/人工确认）/判定（测试报告绑定代码契约样例版本满足合并条件）。**写实现的 Agent 可以自测但不能只靠一句"通过"放行**；加复核 Agent 也不能保证独立性——如果它照着实现逻辑生成预期结果，同一个误解可能被确认两遍。复核报告指出具体输入/断言/代码位置并退回任务+补回归测试——每次修复留下一个能重放的测试，比反复在提示词里写"请仔细检查"可靠得多。失败分诊：读文档/查测试结果可重试；创建合并请求/执行迁移/触发发布超时不能断定远端没执行——状态分"未执行/执行中/已完成/失败/状态未知"，状态未知先查询远端，确认可安全重放才重新发起；写操作用幂等键（数据库唯一约束或下游接口挡第二次副作用）；失败落在子任务层（局部重试以依赖没变为前提，修复改变共享契约则受影响的集成测试也要重跑）；调度器重启处理任务认领——租约过期不会让旧 Agent 自动停止，接收端检查执行代次拒绝旧执行者迟到写入。六方案选型表：单适配器修改反馈可验证→单 Agent ReAct；步骤固定依赖明确→固定工作流或状态图；需要额外质量检查→生成与评估分开；多适配器独立开发→协调者与执行者分工；成员持续追问→Agent Team；多人改同一制品依赖密集→单 Agent 或串行。**是否值得增加一个 Agent，要看它有没有带来可核对的增量**（找到原来遗漏的"报告尚未可用"/补出"医嘱有效但执行记录缺失"的测试/修复后通过同一组回归样例——只是多生成一份内容相近的报告就不如单 Agent）。^[raw/articles/ruofei-react-to-agent-team-medical-dev-2026.md]

## 与既有实体的关系

- `[[entities/agent-orchestration-multi-agent-systems|多 Agent 编排系统]]`：主题主实体，本文提供医疗 FHIR 实证维度（事件时间 vs 可用时间/空返回语义/幂等与租约代次）→ SUPP
- `[[raw/articles/ruofei-multi-agent-consistency-four-questions-2026|面试官：多 Agent 协作一致性四问]]`：本文自引的姊妹篇——四问框架（为什么拆/交什么/谁拍板/凭什么算完成）在医疗研发任务的完整落地
- `[[entities/claude-code-agent-teams-task-decomposition-ruofei|Claude Code Agent Teams 任务拆解]]`：同作者 Agent Teams 系列（本文引用其共享任务列表机制并给出边界）

→ [[raw/articles/ruofei-react-to-agent-team-medical-dev-2026|原文存档]]
