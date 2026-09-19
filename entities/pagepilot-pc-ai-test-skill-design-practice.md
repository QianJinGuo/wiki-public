---
title: "PagePilot — PC端AI测试Skill设计与实战"
created: 2026-07-17
updated: 2026-09-20
type: entity
tags: [ai-testing, browser-automation, cdp, skill-system, testing-framework, alipay, ant-group, component-knowledge-base, agentic-testing]
sources:
  - raw/articles/pagepilot-pc-ai-test-skill-design-practice
confidence: 0.8
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# PagePilot — PC端AI测试Skill设计与实战

支付宝商家中心团队自建的 PC 端 AI 测试 Skill 系统，基于 Browser Agent (CDP) + 组件化知识库 + DB 验证的端到端 AI 测试方案。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

## 核心设计理念

**不写测试代码，写测试知识。** 把踩过的坑、验证过的交互代码、有效的 DOM 选择器封装成"组件 md"，AI 执行时查阅对应组件直接复用。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

核心公式：AI Agent + 组件化知识库 + 浏览器自动化 + DB 验证 = 端到端 AI 测试。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

## 方案演进

从平台方案（自然语言驱动/智能测试流）到自建 Skill。平台方案在商家侧场景暴露出账号体系受限、流程控制粒度不足、文件上传缺失、验证手段单一、扩展受排期约束等短板，自建 Skill 逐一解决了这些问题。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

## 技术架构

### 四趟编译管线（Case 解析）

1. **风格检测** — 自动识别三种 Case 写法（表格/自然语言/清单）
2. **元数据抽取** — 提取账号密码、入口 URL、DB ARN 等环境参数
3. **组件匹配 + 置信度评分** — 匹配 UI 组件并给出置信度（95%+/80-89%/60-69%/<50% 四级门控）
4. **可执行 step 列表输出** — 注入 trace-collector，置信度门控决定执行策略^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

### 组件查找优先链

三层优先级：local-components/（最高，业务覆盖平台）→ components/（平台通用）→ unmatched（提示新建）。类比 CSS 层叠规则。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

### Phase 0→6 门控

7 个阶段串联，每阶段有前置通过条件（precheck→登录→URL→DB 查询→比对→报告→归档），条件不满足即卡住，不向下传播错误。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

### 失败自学习

Case 失败 → errorCode 匹配 known-failures → 命中标注模式 ID / 未命中自动补录（只增不删）。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

## 关键技术难点与解法

| 问题 | 解法 |
|------|------|
| React 受控组件 input.value 不生效 | prototype.value setter + dispatchEvent 触发合成事件 |
| CDP 无法操作原生文件选择 dialog | DOM.setFileInputFiles 直塞文件路径 |
| Cascader 异步加载跳级选择 | 逐级选 + 500ms 等待 |
| 同一搜索功能三种实现 | DOM 预扫描判断组件体系 |
| SPA 路由销毁 fetch 拦截器 | submit 前 0.5 秒内重注入 |
| 按钮无响应 | CDP 真实鼠标事件（mouse move → down → up） |
| CDP session 崩溃表单丢失 | window.__formState 表单快照 + 崩溃重登恢复 |

随机出错率从首轮 40% 降到 4 轮后 5% 以下。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

## 业务成果

- **蚂蚁答疑助手评测**：4 波 137 条用例，3 小时完成（人工 2 天），发现 64 个 Bug（含 MCP 工具泄露 P0 漏洞）
- **店铺开通**：7 条 Case 30 分钟跑完（人工半天-1天）
- **品牌管理（B站）**：12 Case 全量覆盖，通过率 100%
- **品牌管理（P站）**：跨 Skill 联合验证，发现前端 Bug
- **跨域扩展**：万知管理平台 100 条 Monkey 测试 100% PASS；安全越权校验 4/4 PASS

## 组件生态

42 个组件 md + 34 个脚本，分 5 类覆盖商家侧 90% 高频操作。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

→ [[raw/articles/pagepilot-pc-ai-test-skill-design-practice|原文存档]]

## 深度分析

### 知识即代码：为什么沉淀知识比沉淀脚本更耐久

传统自动化把价值存在「脚本」里，脚本一旦失效（DOM 改版、组件库升级、框架换实现）沉淀就归零。PagePilot 把价值挪到组件 md 这一层：一个条目里同时装着语义定位策略、交互顺序、踩坑记录与脚本入口，执行时按需查表复用。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md] 收益是知识半衰期被拉长——选择器最易腐坏，而「多图上传会重置 radio/cascader，所以要先上传、再选 radio/cascader、最后填 input」这类时序约束来自组件库的稳定性质，定位方式整体换掉也不失效，组件 md 只改定位段而无需整条重写。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

配合「有脚本调脚本、禁手动拼 eval」的纪律，系统把知识拆成了两层：可描述的规律写进 md 供推理复用，已验证的代码路径固化成 .sh 保证每次跑同一条路径，压住了生成式 [[entities/agent-browser|Browser Agent]] 每轮重猜操作序列的方差。增量落点是 md 而非执行器，这是典型的 [[concepts/knowledge-base-output-flywheel|知识库输出飞轮]]：执行越多、组件越全、下一次越稳。

### 三层组件查找链：CSS 层叠规则的组织级复用

local-components/ → components/ → unmatched 不是朴素的目录查找顺序，而是一套把 CSS 层叠（cascade）语义搬到组件知识库上的冲突解决协议：同名组件按就近原则由业务本地定义覆盖平台通用定义，平台只维护兜底版本，未命中则显式暴露为 unmatched 并提示新建。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md] 它买到三件事：覆盖成本从「改平台」降到「加一个同名文件」；平台与业务演进解耦，业务不再被平台排期绑架（qiankun fetch 隔离、非标组件等扩展恰卡在排期上）；未覆盖区域不被伪装成可用，知识库盲区保持可观测。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

### 7 阶段门控与四趟编译：把错误拦在最早的一层

Case 解析拆成风格检测 / 元数据抽取 / 置信度匹配 / 输出 step 四趟，每趟只承担一个职责；执行侧是 Phase 0 precheck（ODC MCP、yuque CLI、allowlist）经登录、URL 与 ID 解析、DB 查询（SQL 至少返回 1 行）、比对、出报告到语雀归档的 7 段串联，每段带前置通过条件，不满足即卡住。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md] 关键不在「有阶段」，而在失败的位置被固定：agentic 测试最贵的故障是脏状态向下传播——登录没成功还继续点、DB 查出空集还继续比对，报出的失败现场与真实原因隔五六步，trace 再全也难归因。precheck 把环境类失败挡在浏览器启动之前，DB 要求至少 1 行则把「上游没落库」与「前端没提交」切开，失败信息因此可定位、可分派。这种把可判定条件全部前置的取向，与 [[concepts/evaluation-harness-design|评测 Harness 设计]]中的隔离与可复现原则同源。

### 置信度分级授权：Agent 何时可以自己动手

组件匹配为每一步打出置信度，输出趟据此分档授权：90-100% 静默执行，70-89% 摘要确认，50-69% 拒绝并要求修正，<50% 提示无法解析。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md] 这把 Agent 自主性从二元开关变成连续授信额度：触发词命中越明确越放权，纯语义推断越收权，人工介入因此集中在真正模糊的少数步骤而非全流程盯守。同一机制也给自学习提供了度量尺——随机出错率从首轮约 40% 收敛到 4 轮之后的 5% 以下，且每轮翻新只动踩坑段，说明误差下降来自知识累积而非模型或 prompt 调优，是 [[concepts/agent-self-improvement-loops|Agent 自我改进闭环]]在测试域的具象形态。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md]

### 追加式失败注册表：组织记忆的复利与腐烂风险

known-failures 按 errorCode 匹配，命中则标注模式 ID，未命中自动补录且只增不删，构成组织级失败记忆：同类问题的不同表现形式收敛到同一模式 ID 下，跳出了「换个业务场景又从头开始」的循环，也把排查经验从个人记忆转成 [[concepts/agent-memory-architecture|Agent 记忆架构]]意义上的可继承资产。^[raw/articles/pagepilot-pc-ai-test-skill-design-practice.md] 代价同样明确：注册表会随业务版本漂移逐渐失真，已随组件重构消失的旧模式若不清理，匹配会把新问题错误归入历史模式，产出「解释得通但修不对」的诊断——追加负责不丢信息，审计才是防腐层。迁移边界也在这里显形：知识与人分离、失败按 errorCode 归类、以自建 Skill 补强多步骤强状态场景，这些可搬走；账号独占性（一个账号只能开通一个店铺）、资质图片上传（需用 DOM.setFileInputFiles 绕开原生 dialog）、URL 提业务单号串 DB 比对（依赖稳定单号）都带业务印记，换到无数据可达性的域这条链会断。

## 实践启示

1. **按腐蚀速度分层存放知识。** 选择器、坐标、脚本属于易腐层，交互时序与组件库性质属于稳定层，只有后者值得长期投入。
2. **差异化用命名空间承载，不改平台代码。** local-components/ 同名覆盖的机制把业务特例从「改平台」降级为「加文件」，是通用框架与业务差异共存的最低成本解。
3. **门控的价值在失败位置，不在阶段数量。** 宁可停住也不带脏状态下行；把「上游没落库」与「前端没提交」在结构上分开，失败报告才能找到责任人。
4. **自主权分档授予，不做二元开关。** 高置信度静默、中置信度确认、低置信度拒绝，人工审核资源自然收敛到真正模糊的少数步骤。
5. **失败记录只增不删，但必须配失真审计。** 追加保护组织记忆，定期按版本清理已消失模式并复核命中准确率，防止历史模式退化为误诊来源。
6. **先划清可迁移边界再谈复用。** 可搬走的是知识沉淀结构、置信度门控与失败注册表这套方法；账号体系约束、文件上传、DB 验证属于业务耦合层，跨域前要先确认替代验证链是否存在。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

