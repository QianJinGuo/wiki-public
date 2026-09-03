---
title: "腾讯 AI Coding 深水区 — 事实vs判断尺子与提示词→框架→runtime 下沉方法论"
created: 2026-08-27
updated: 2026-08-27
type: entity
tags: [tencent, ai-coding, harness, runtime, evaluation, prompt-engineering, agent-teams, sovereignty, fact-vs-judgment, production]
sources: [raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026]
---

# 腾讯 AI Coding 深水区 — 事实vs判断尺子与提示词→框架→runtime 下沉方法论

## 核心命题：事实 vs 判断尺子

"手上这件事，是事实，还是判断？是事实就交给机器，别让人去核对，也别听 AI 声明；是判断就留给人，别指望机制替他拍板。"这是贯穿全文的尺子，也是 AI Coding 深水区里"编码让位、人退到哪里"的回答——人退到判断、把取事实的活交给机器。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

背景实证：纯 AI 交付的内部平台（对照组）——三个月、七个服务、三十多万行代码、人一行没写（16万行生产+11万行测试+8万多行前端），设计文档全由 AI 落笔；但真实业务是百万行 Lua、自研框架、模型几乎没见过的深水区。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

## 核心洞察

### 人的工作量没减少，只是从落笔挪到拍板
读的东西从"读代码"变"读方案/结论/证据"，拍板从"这段怎么写"变"这条路走不走/这个失败算不算问题"。核心从来不是 spec 载体，而是产出它过程中被迫想清楚的事（brainstorming）——动手前的 spec 用完可扔，事后沉淀的设计说明得留着。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

### AI 犯错是规模问题不是概率问题
错误绝对数量由累积量决定，不随单次准确率降低。模型越强产出越多越快，人这端上限不动，缺口更大——"模型越强 harness 就该越薄"是想当然。有一类错 AI 特别容易犯：**失败不可见**（AI 交付功能的能力明显强于交付"让失败可见"的纪律）。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

## 方法论一：提示词的尽头是基础设施
提示词边际收益会耗尽，解法在 agent 底下的框架层——提示词只能"叮嘱"，框架可以"取消这个错误的存在"。
- 声明式 mock + 自动还原（像自动 GC）消掉一整类错误，把 AI 注意力还给主任务。
- 超时报错时框架自动返回准确原因+链路分析包（反向探测分类+下一步动作+代码执行路径）——**报错通道本身就是提示词注入通道，"最好的提示词是恰到好处的反馈"**。
- **两种"厚"别混**：写提示词里越厚模型越笨，写进框架越厚越省心主任务越强。打补丁档（"严禁 mock ss"）尤其不要写——有限规则覆盖无限问题。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]
- 下沉判断标准：**判据是确定的事实才能做硬门禁，判据是代理指标最多做提醒**（路径前缀=硬拦截；"读过编码规范"=代理指标降警告）。信号要能从外部取到，不能靠 AI 自述（防假测试：抄内部逻辑重算再断言，断言全过覆盖率零，只有从外面看"被测函数行一次没执行"戳得穿）。门禁误报代价高（逼人和 AI 为过检查写更差代码）。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

## 方法论二：编排的尽头是 runtime（runtime 主权）
workflow-engine（状态机固定流程）换来稳定但子 agent 反复探索/不透明/主 agent 闲置。agent teams 实验（产物驱动状态、七角色、审查点对点）踩坑：模型不认机制、中断活锁（不催正在工作的 agent 只认 DONE/BLOCKED=PROTOCOL-A）、成员生命周期靠不住（一切以磁盘产物为准）、机制反直觉。完整流程太脆没推广，但熬出 agent-teams skill（纪律全是"不信声明，信事实"）。收窄的 /module-test（spec 自带答案+双成员点对点）跑稳。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

三起黑盒事故证明 agent loop 每环节（上下文压缩/工具参数拼接/限额/遥测）必须可审计可干预：①Claude Code 遥测投毒；②CLI 写长中文 UTF-8 损坏（按字节切断多字节字符）；③压缩 bug 11 小时空转（压缩摘要误把"生成摘要"当用户请求+阈值卡 100K，203 轮压缩=203 次从头再来）。→ **自建 agent runtime**。选型的真问题不是"选哪个产品"而是**主权的边界划在哪**（插件化能划到哪一层）；审计底线是上下文注入/遥测能按来源回看（append-only 会话日志）。决策：pi agent 先动（最小 harness + 上下文治理压伤口）、DeepSeek Harness 跟进（"一切皆插件"，元框架只管装卸，但"可预期"要自己验）。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

## 方法论三：分数不可信，问题在题集（评测）
- **撞墙**：107 天 17 万行评测平台只留下两个分数（69.9/51.9）后被删。四问题：判据不能执行（主指标交给 Cohen's Kappa 0.10-0.21 的 LLM 裁判）、判分器自己有 bug（同一日志 0 分 vs 98 分差 98 分）、把流水线维持住是份全职工作、卷子追不上考生。教训：**值钱的是想清楚，不是承载它的载体（文档如此，平台也如此）**；判据能不能执行、结论能不能变成一次提交才重要。立了纪律：一行平台代码都不写——只有题集/配置/脚本/跑数记录。
- **V1 卡死**：挖空回填测评序——评分器量错东西（按文件粒度）+ 单次跑数追噪声。**错误的评测信号比没有更危险**。
- **V2**：业界调研（50+ 论文）后用业界尺子量自己方案——唯一自研的挖空回填塌掉（人造缺陷普遍比真实缺陷好发现）。**判据必须是执行结果，不能是合成清单**。主指标落**变异击杀率**（五道闸门：有产出→lint→能跑→连跑三次稳定→杀死多少变异体），因为覆盖率能靠无断言测试刷出来（GPT-4o 覆盖率 35.2% 变异分仅 18.8%），杀变异体必须有真断言。变异体算子取自代码评审 checklist（真实 bug 分类学）。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

## 方法论四/五：人退到决策点 + 把判据变成流程
机器提供事实的难点不在"人做判断"这一半，而在**机器给出的那个事实本身是不是真的——提供事实的机器自己也要被验证**。结尾哲学：模型每季度变强，提示词/编排/评测数字都会随之作废（**它们是流**）；但把问题想清楚的过程、运维纪律、runtime 主权判断、框架层能力、验收杠杆、评测题集、治理判据依然值钱（**它们是常**）——"做 AI 工程，就是在湍急的流里，找到并守住那些常的东西"。^[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026.md]

## 相关实体
- → [[entities/tencent-token-optimization-agent-architecture|腾讯 Token 优化 Agent 架构]] — 同团队前作（workflow-engine 编排）
- → [[concepts/harness-engineering-framework|Harness Engineering 框架]] — 边界约束与下沉
- → [[entities/qunar-ai-coding-large-core-system-refactor-2026|去哪儿 AI Coding 大型重构]] — 同类 AI 驱动工程化实践
- → [[raw/articles/rethinking-harness-evolution-evaluation-mozhi-space-2026|Harness 自进化实证批判]] — 评测与自进化边界（互为参照）

→ [[raw/articles/tencent-ai-coding-deep-water-fact-vs-judgment-2026|原文存档]]
