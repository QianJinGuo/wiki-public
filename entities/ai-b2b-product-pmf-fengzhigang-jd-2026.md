---
title: "B 端产品经理的 AI 工作方式重组：从 34 到 15 人日（京东零售 冯志纲）"
created: 2026-09-11
updated: 2026-09-11
type: entity
tags: [b2b-product, ai-workflow, context-engineering, harness-engineering, prd, skill, jd-retail, prompt-engineering]
sources: [raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026]
confidence: 0.9
provenance_state: extracted
---

# B 端产品经理的 AI 工作方式重组：从 34 到 15 人日（京东零售 冯志纲）

京东技术（京东零售 冯志纲）第一方实践：B 端产品经理把 AI 从"问答工具"重组为稳定工作流——一批原预估 34 人日的需求（收银系统五大模块：POS 开票/部分退款/手动折扣/菜品本地化/排序优化）以 15 人日完成并通过评审、质量不打折。效率来源不是省略流程，而是工作方式重组：AI 做结构化与遗漏检查，人做判断与复核。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## 核心工作流（五步循环）

把"从空白页工作"变成循环式工作：**输入背景 → AI 反问 → 补齐上下文 → 生成初稿 → 人工复核 → 继续迭代**。分工铁律：AI 是劳动者（整理/追问/初稿/资料收集/异常识别），人是决策者（业务目标/流程边界/风险取舍/评审结论/最终责任）。在收银、支付、对账、定价等场景，AI 可参与分析解释建议，但核心交易逻辑必须保持确定、可验证、可追溯。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## 能力演进三阶段：Prompt → Context → Harness

| 阶段 | 关键问题 | 典型做法 |
|---|---|---|
| Prompt Engineering | 怎么把问题问清楚 | 角色设定/步骤拆分/输出格式约束——仍是起点但很快见瓶颈 |
| Context Engineering | 怎么把上下文给足 | 文档/截图/代码/历史规则/业务背景；**需求文档场景大部分时间不是"让 AI 写"而是 Context Engineering** |
| Harness Engineering | 怎么让 AI 进入工作现场 | 工具/文件系统/权限/执行环境/循环验证——同一个模型在不同 Harness 里能力可差一个数量级 |

判断 AI 工具价值看四点：能否进入工作现场、读取上下文、调用工具、完成闭环验证——不应只看模型本身。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## 需求文档四步法（先澄清再生成）

1. **建立需求文档 Skill**（结构/规则/口径/注意事项沉淀，复用免从零解释）
2. **交代已知关键节点**（目标/核心流程/关键规则/主要对象/影响范围/不确定点）
3. **让 AI 反向提问**——先不生成，让 AI 追问导入格式/失败策略/权限控制/异常提示/历史数据影响；"如果让 AI 先生成，你会进入修改初稿模式、思路跟着 AI 走；让 AI 先反问，你是主动想清楚再交代"
4. **调用 Skill 生成文档**——初稿约 80% 结构可用，人补异常文案/待评审项边界/跨系统联动

案例实证（收银大需求）：AI 反向提问提前逼出核心规则——支付完成时海博订单未生成的开票二维码归属、混合支付能否开票、手动优惠与营销活动优先级（手动>营销，取消后营销自动恢复）、部分退款后已打印二维码走冲红逻辑；"指定金额退款不做，需求直接缩水一半"在澄清阶段即拍板。开发反馈"退款和开票关系这次写清楚了，之前总是评审完了还要问"。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## 交互设计与 AI 边界

Figma Make/Figma MCP 让 AI 生成页面结构/流程草图/交互方案——价值是"先可视化、先有可讨论版本、先摊开流程关系"，再由人判断页面层级/用户路径/异常状态完整性/业务规则表达/开发可控性/可测试性。AI 不能代替产品判断。推荐分工模式：代码原型跑通逻辑 → Figma 精调视觉（AI 点餐案例：触发→识别→确认/纠错→更换菜品→价格调整→结算全节点交互状态）。设计过程会逼出需求里没写的逻辑（如"更换菜品后要不要上报训练数据"）。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## Skill：把个人经验变成可复用资产

Skill 本质是把一类常做任务沉淀为可复用结构化指令——不是简单 Prompt，而是含任务目标/工作流程/输入要求/输出格式/质量标准/边界规则的小型方法论。**做过两次以上的工作都值得沉淀成 Skill**（写 PRD/周报/接口文档/竞品分析/交互评审/会议纪要/影响范围分析）。两条原则：①不要什么都从零开始——先复用成熟方法论（Apple HIG/awesome-design/高质量 Skill 仓库）再本地化改造；②本文七个案例的 Skill 全部是跑真实需求跑出来的，不是设计出来的（channel-product-integration Skill 含对接模式判断/信息收集清单/字段映射表模板/PRD 模板）。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## 值得记录的两个细节案例

**AI 反问纠正概念错误（接口文档可读化）**：对接京东海博餐饮 SaaS 时，AI 初稿写了"售罄推送"，作者发现海博没有售罄/沽清概念只有上下架；AI 随即反问"海博库存模块确实没有沽清概念吗？这会影响 Webhook 事件设计"——售罄（今天卖完明天可能恢复）与下架（从菜单移除）是不同业务逻辑与接口设计。三版迭代定稿。AI 初稿的价值不是一次写对，而是让问题更早暴露。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

**墨水屏硬件从零跑通（Token Display）**：4.2 寸三色墨水屏（ZKC42V/ESL_BWR，零售电子价签走 Zkong 私有协议无开源驱动）→ 找到网页蓝牙工具绕开私有协议 → 把网页源码发给 AI，从 JS 提取完整 GATT 结构/命令字节表/数据包格式/分块传输逻辑（"不需要抓包，读前端 JS 就把协议搞清楚了"）→ Codex 写 Python 脚本（拉取 Claude Pro 额度→黑白+红双通道图片→BLE 推送→cron 每 5 分钟刷新）→ 解决 CoreBluetooth 缓存 TimeoutError/广播窗口/图标消失等真实硬件问题，最终 epd_daemon.py + epd_manager.py + cron 三件套。^[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026.md]

## 与其他实体的关系

- [[entities/jd-haibo-ai-native-harness-dual-loop-knowledge|京东海博 AI-Native]]：同属京东零售第一方 AI 工程实践——那篇讲 Harness 双 Loop/知识库/技能自迭代的研发侧框架，本文讲 B 端产品经理侧的工作方法重组，互为上下游视角
- [[entities/京东健康opc团队的产品全流程skill探索|京东健康 OPC 全流程 Skill]]：团队级 Skill 探索；本文贡献了个人级量化数据（34→15）与更完整的案例链
- Prompt→Context→Harness 三段演进与 [[entities/harness-engineering|Harness Engineering]]、[[entities/agent-harness-12-components-7-decisions|Agent Harness 12 Components]] 的工程框架一致，本文是其产品经理视角的落地实例

→ [[raw/articles/ai-b2b-product-pmf-fengzhigang-jd-2026|原文存档]]
