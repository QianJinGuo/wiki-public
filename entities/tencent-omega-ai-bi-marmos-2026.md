---
title: "腾讯 Omega：AI BI 生成式 Dashboard 产品（QueryRegistry + DTBridge）"
type: entity
created: "2026-08-03"
updated: 2026-10-20
tags: [wechat, ai-bi, bi, dashboard, harness, query-registry, tencent, data-product]
rating: v8c9
confidence: 0.85
provenance_state: extracted
sources:
  - raw/articles/tencent-omega-ai-bi-marmos-2026-08-03
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 腾讯 Omega：AI BI 生成式 Dashboard 产品（QueryRegistry + DTBridge）

**来源**: 腾讯技术工程（jiaxingjin）

**发布日期**: 2026-08-03

**原文链接**: https://mp.weixin.qq.com/s/EF1nVNMhCwYivMFey7uI-g

**产品**: 马尔摩斯/marmos（marmos.qq.com，内测中）

## 摘要

腾讯 DataTalk 团队在推翻 2024 年 ChatBI（AI 生成 SQL + 画图，但"接不住后面的分析工作"）后重新设计的 AI BI 产品。核心立场：**聊天框不是 BI 的下一代**——AI 只完成"问题到 SQL"只是缩短一次查询，Omega 要接管完整分析工作：理解数据源字段 → 规划指标（总销售额/环比/趋势/品类结构/地区差异）→ 查询数据 → 选图表 → 组织页面 → 生成完整 Dashboard，且支持局部修改、联动筛选、持续刷新、分享协作。核心开发仅 3-4 人，2026-01 中开工，前两个月跑通主链路，后续数月解决"离开演示环境仍然像个产品"。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## 数据契约架构：QueryRegistry + DTBridge

**核心问题**：很多 AI 看板把查询结果直接写进页面（`const data = [...]`），生成后数据凝固——换时间、加筛选、刷新都得重新生成。页面看着是活的，数据其实已经死了。

**解法（职责拆分）**：AI 负责表达"我要什么"，系统负责"它怎么稳定拿到"。

- **QueryRegistry**（结构化数据契约）：每张图的查询、参数、结果绑定、筛选器联动关系全部声明清楚。字段包括 queryId（稳定引用）、datasetId（路由到已登记数据集）、params（参数类型/日期精度/成对关系/级联来源/选项查询）、fieldBindings（SQL 输出列→维度/指标角色映射，避免"读取销售额、SQL 返回 total_gmv"类静默失败）
- **DTBridge**（运行时）：扫描 SQL 模板占位符构建"参数名 → queryId[]"依赖表，筛选器更新时只刷新受影响图表；valueType + aliasOf 统一处理时间格式差异（事实字段 2026-07-22 00:00:00 vs 分区字段 20260722）；防抖合并、取消旧请求、级联筛选沿依赖图工作；参数注入转义（日期/秒毫秒/字符串字面量/去分号注释）但不宣称"绝对防注入" ^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## 指标证据链（先找证据，再谈"理解指标"）

**原则**：离用户当前工作现场越近的证据优先级越高。层级：**已有查询口径 → 用户指定的数据 → 语义模型和字段元数据 → 被明确引用的业务知识 → 检索候选**。

- 修改已有图表时，复用该图已执行的真实 SQL 中的业务定义（SUM(pay_amount-refund_amount) 等），不重新发明
- 新分析时，用户明确选择的数据集优先；生成前必须调用 Schema 工具拿真实字段（物理字段/展示名/类型/说明/分区/引擎/路由），外层 **Schema Guard** 拦截"指定了数据却没读 Schema 就生成"
- 未指定数据时在个人/团队空间/可访问目录检索（名称/描述/物理表名/关键词/归属/更新时间，合并去重）
- 证据不足时暴露候选或继续确认——"LLM 可以降低使用数据的门槛，不能顺手替企业补完从未做过的数据治理"

**语义模型路径**：逻辑字段分类 DIM/FACT/TIME_DIM → 生成 MODEL('<modelCode>') 的 SemanticQL → 语义查询服务转物理 SQL（底层 JOIN/指标公式/引擎适配不由模型临场猜测）。转换失败带错误+上一版 SemanticQL+可用字段列表反馈给 Agent，限定次数内从真实字段重新选择。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## 三层安全边界

1. **身份授权**：查询身份由服务端从登录态解析（不接受页面声明 userId）；普通访问校验文件+数据集权限，分享访问先校验分享关系；数据集 Schema/查询执行/文件打开分别做权限判断
2. **查询层**：可完整解析的 SQL 方言用 AST 检查（单条只读 SELECT、拒绝多语句/危险函数、提取真实表引用含子查询 CTE、统一行数上限）；ClickHouse 等用轻量只读检查 + readonly 模式 + 超时 + 结果上限
3. **页面运行环境**：数据库凭证不进 HTML，页面经受控查询代理请求数据；iframe sandbox + CSP；不把 iframe 当防泄漏系统（敏感数据仍需上游行列权限/字段脱敏/网络出口策略）

**身份生命周期**：持久化时剥离运行时脚本和查看者信息；真正打开/分享时再注入唯一 QueryRegistry + DTBridge SDK + 系统参数——"数据契约可以保存，运行时身份不能被永久烘焙进快照"。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## Harness 层（不押注模型"自觉"）

- **工具渐进选择**：运行时按 Canvas/文件类型/用户选择/计划阶段自动预激活高置信度工具，其余由模型 select_tools 渐进选择，避免几十份工具 Schema 同时进上下文；业务工具框架无关契约，换 Agent 框架只调适配层
- **错误分类修复**：结构完整性检查（截断裁掉不完整片段补齐闭合结构）、查询声明存在性、图表-真实查询绑定；字段别名/时间格式/筛选器选项用规则处理（"规则能修的，不让模型重写"）；语义问题交给 Agent 带错误上下文做有目标修复——反对"让模型检查修复所有问题"（既贵又不可测试，可能修好一处改坏另一处）
- **写入前重验权限**：不能因 Agent 拿到文件 ID 就默认有权覆盖 ^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## 运行时四契约（慢 SQL 事故教训）

**事故**：高基数 COUNT(DISTINCT) 查询正常执行需 110-130 秒，被 120 秒空闲看门狗当僵尸杀掉；前端连接断了但停止信号没传到最底层，SQL 继续跑 30 分钟后才超时。

**四契约：有界、可取消、可观测、可自纠**。

- 叶子工具 deadline 由工具自己声明、统一适配层执行；取消信号传到真正执行查询的地方
- 每轮运行必须返回且只返回一个类型化 finish{reason, detail}；异常终止时前端保留已生成内容
- 持续流式子 Agent 豁免 deadline-race（delegate_agent_*/backend_agent 工具），改用整轮生命周期上限+活性检测+流式心跳
- 重复调用护栏：同一工具连续 32 次相同请求先提醒换路、到阈值熔断
- "Agent 能不能上线，取决于超时/截断/重复/取消时有没有一个体面的收场" ^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## 人机协作与产品化

- **A-MD 协议**：外层文件协议，AMDFile 含 SQL/Python/图表/指标/筛选器/说明文字 Block + 各视图 Layout；Canvas 是 dashboardLayout.canvas 下的一种 Dashboard 视图载荷（存 HTML + QueryRegistry）——A-MD 管文件/版本/可编辑分析过程，Canvas 负责最终页面动态查询
- **协作模式**：小改动直接告诉 AI 只改目标区域；微调布局 GUI 拖拽；大变化 Plan Mode 先列修改内容确认后执行；"AI 原生不等于所有交互都必须经过聊天框"
- **审美基础设施**：23 种视觉风格 + 18 种可视化引擎；"薄配置，厚引擎"——内置风格源配置约 120 字（几个颜色/字体/关键词/调性），后端展开为约 800 字设计规格（色板/文字关系/边框/布局/动效约束）
- **监控与推送**：图卡监控规则（如"转化率<5% 且订单量>1000"告警，可跨卡组合）、按周/月定时推送（绕过旧缓存最新数据截图、失败有记录地降级）；版本历史跟随产物、目标空间重新授权
- **成本对比**：内部固定样例——Claude 4.6 约 14.2 元 vs 混元 Hy3 约 0.47 元（差约 30 倍），驱动"高性价比模型稳定完成复杂工作"的 Harness 设计 ^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md]

## 深度分析

### 契约先行：正确性由中间表示决定

生成式 BI 的失败极少报错，多为静默错位：SQL 返回了数据而图表是空的，筛选器动了只有一半图卡跟着变，换了日期数字还停在昨天。加长 prompt 治不好这类问题——症结不在表达，而在模型产出的界面与真实执行的查询之间缺一份双方都得遵守的声明。QueryRegistry 把声明外化：queryId 固化引用、datasetId 决定路由、params 记录类型与级联、fieldBindings 把输出列映射到维度/指标角色，"回调读销售额、SQL 返回 total_gmv"于是从偶发玄学变成可断言的检查。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:46-48] 结论是：**正确性不由模型能力决定，而由可验证的中间表示决定**。prompt 抬高单次生成的上限，契约抬高失败的下限，而每天被打开的看板用的是下限。

### 慢 SQL 事故：超时是运行时语义

高基数 COUNT(DISTINCT) 正常需 110–130 秒，被 120 秒空闲看门狗当僵尸杀掉；前端连接断了、停止信号没抵达底层，那条无人需要的 SQL 又跑了 30 分钟。三处故障指向同一缺口：系统散落着 timeout，却无人回答"每步最多多久、取消能否真正释放资源、结束时前端能否知道原因、局部失败能否换路"。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:178-188] 四契约（有界/可取消/可观测/可自纠）把可靠性从运维参数改成接口义务：deadline 由叶子工具自己声明、适配层统一执行，每轮只返回一个 finish{reason, detail}，流式子 Agent 豁免 deadline-race 改用生命周期上限加心跳，重复请求先提醒换路、到阈值熔断。它承认故障必然发生，于是把目标从"避免异常"下移到"异常时仍然体面"——演示环境永远不测这条路径，而它正是 demo 与产品的分水岭。参见 [[concepts/harness-engineering]]。

### 三十倍价差下的职责再分配

同一份"生成并修改一次看板"的固定任务，Claude 4.6 约 14.2 元、混元 Hy3 约 0.47 元。样本不是质量排名，却逼出真问题：全量走贵模型，单位经济性只够服务少数客户；走便宜模型，失败模式变成"看起来已经完成"——被截断的 HTML 仍有前三张图，模型凭记忆写出不存在的 SDK API，三张表的数据顺手编出第四张。这类错误没有红灯，账单要等用户点筛选器或下周数据变化才结算。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:158-166] Omega 不是挑模型，而是重分职责：表达与规划给模型，可判定的部分（结构完整性、别名对齐、时间格式、表存在性、写入前权限）给规则与运行时，只有语义分歧才回到模型、且带错误上下文做一次有目标修复^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:168-176]。本质是一份 token 预算表：花在会产生分歧的地方。参见 [[concepts/ai-cost-optimization-framework]]。

### 确定性在哪里结束

三处主动停手划出了边界：语义模型路径上，模型只写基于 MODEL() 的 SemanticQL，JOIN、指标公式与引擎适配交给语义查询服务，转换失败时回传错误、上一版 SemanticQL 与可用字段，限定次数内从真实字段重选^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:88-94]；安全上，查询身份由服务端解析、可解析方言走 AST 只读校验、凭证不进 HTML，并拒绝把 iframe 当防泄漏系统^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:124-132]。审美是反例却印证同一条：自由发挥得到"平均脸"（白底、蓝柱、四张 KPI 卡），加几句"高级、克制"只能不稳定地改善；把约 120 字风格源配置展开成约 800 字设计规格后，生成才稳定落进有人为判断的框架^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:152-156]。**生成负责组合与表达，确定性负责口径、边界与品味的下限。**

### 证据链：产品诚实即商业边界

证据优先级（已有查询口径 → 用户指定数据 → 语义模型与元数据 → 被引用业务知识 → 检索候选）表面是检索策略，实质是责任分配：越贴近用户现场既有约定的，越不需要模型重新解释，也就越不会被"合理化"地改掉；Schema Guard 拦截"指定了数据却没读 Schema"，等于承认 prompt 里的"请先读字段"不可靠^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:74-86]。而"证据不足就暴露候选或继续确认"的真实含义，是产品给自己划了上限：LLM 可以降低使用数据的门槛，不能替企业补完从未做过的数据治理^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:86]。短期看这是功能缺口，长期看它是唯一让人敢把结果发给老板的选择——错误口径进入决策的代价，远高于多问一句。

## 实践启示

1. **先定契约，再选模型**。把每张图的查询、参数、字段绑定与联动关系外化成可校验的结构（哪怕先是一份 JSON Schema），让"图表读不到列""筛选器漏挂某条查询"在生成期报错，而不是等用户点筛选器时才发现。
2. **把失败当作一等公民**。为叶子工具声明 deadline，取消信号必须传到真正执行查询的那一层，每轮只允许一种类型化终止原因；流式子任务单独用生命周期上限加心跳，不要复用空闲超时——空闲会杀死正在正常工作的慢查询。
3. **规则收敛确定性错误，模型只处理语义分歧**。别名、时间格式、表存在性、写入前权限写成检查器；反对"请检查并修复所有问题"式自修，它贵、不可测，还可能修好一处改坏另一处。省下的预算投给规划与布局。
4. **给证据排优先级，并允许系统说"不确定"**。明确"复用已有 SQL 口径 > 用户选定数据集 > 元数据 > 检索候选"的顺序；治理缺失处暴露候选而非编造口径。这是可信度的来源，也要提前和业务方对齐预期。
5. **用便宜模型做产品，用贵模型定基线**。固定样例同时跑两个档位，统计"看起来完成"的静默失败率，而不只测最好一次的效果；定价与 SLA 按便宜档位加 harness 的稳定性测算。
6. **把"第二天还能不能用"写进验收标准**。监控规则、定时推送、版本 diff、跨空间重新授权、失败有记录地降级，都不是增值功能，而是"数据产品"与"AI 画的一张图"的区别；演示能过的东西，加上续看与降级路径才算交付。^[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03.md:190-198]

## 相关链接

- → [[raw/articles/tencent-omega-ai-bi-marmos-2026-08-03|原文存档]]
- 相关主题：[[entities/amap-nl2sql-knowledge-engineering-production|高德 NL2SQL 知识工程：确定性路由 + 知识卡片]]、[[entities/gaode-ai-native-data-agent|高德 AI Native 数据 Agent（NL2SQL 生产实践）]]、[[entities/alibaba-data-rd-harness-engineering-nl2sql|阿里数据研发 Harness：NL2SQL × Multi-Agent × 知识工程]]、[[entities/volcengine-data-agent-intelligent-query-agent|火山引擎智能问数 Agent]]
- 腾讯同源：[[entities/tencent-harness-engineering-team-specification-2026|驾驭 AI Coding：腾讯 Harness Engineering 落地规范]]
