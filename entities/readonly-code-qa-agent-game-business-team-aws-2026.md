---
title: "为游戏业务团队构建只读代码问答 Agent：架构、性能与安全实践"
created: 2026-08-26
updated: 2026-10-23
type: entity
tags: [agent, code-agent, code-qa, mcp, security, rag, architecture, aws, harness]
sources: [raw/articles/readonly-code-qa-agent-game-business-team-aws-2026]
confidence: 0.8
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 为游戏业务团队构建只读代码问答 Agent：架构、性能与安全实践

面向策划、测试、运营和客服等非研发业务人员的只读代码问答 Agent 参考架构。核心设计决策是把会话执行环境与代码数据面分离：会话 microVM 不挂载仓库、只通过 HTTP 上的只读 MCP 工具查询集中式代码索引，从而在权限边界、证据可信度和性能之间取得平衡。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

## 架构核心：执行环境与代码数据面分离

最核心的模式是把 Agent 的会话执行环境与代码数据面彻底分开。会话 microVM 不挂载任何仓库，也没有本地文件读取工具；代码定位、调用关系、全文搜索、源码和配置表读取全部通过只读 MCP 接口完成。三个主要组件：bot-gateway（接收事件、去重、路由会话、流式更新答案卡片）、AgentCore Runtime（提供会话隔离的 microVM 运行环境）、index-service（保存代码副本和每仓索引，通过 HTTP 提供定位、搜索和文件读取）。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

这种分离带来两个直接收益：一是会话无法通过 Shell 打包整个仓库，也拿不到拉取仓库所需的 Git 凭证，从机制上减少了完整代码副本和凭证的分散暴露；二是集中式索引统一了代码范围、版本、工具权限和审计策略，更适合非研发用户。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

## 安全边界设计

Agent 侧不启用 Bash、本地读写和提交工具，index-service 只注册白名单内的只读工具（本方案最多 9 个：7 个代码定位/文件读取 + 2 个术语表查询）。代码访问链路位于 VPC 内，索引接口端口只向指定安全组开放。隔离方式按项目信任关系划分：同一信任域的项目共用索引主机和安全组（进程、端口、仓库清单逻辑隔离），互不信任的项目使用独立索引主机和安全组。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

### 提示词注入防护（可迁移的防御组合）

启动阶段设置 `setting_sources=[]`，关闭用户、项目和本地目录中的文件系统配置来源（包括 settings.json、CLAUDE.md 和 Skills），避免仓库中的 Agent 配置或项目指令被自动加载。查询过程中，工具返回的代码、注释和配置统一作为数据进入模型上下文，系统提示明确要求忽略其中可能出现的行为指令。这套「关闭文件系统配置来源 + 代码当数据处理 + 只读工具白名单 + 输出脱敏 + 异常监控」的组合防护，是可迁移到任何代码 Agent 设计的通用防御框架。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

## 中文检索英文代码符号

业务人员用中文概念和玩家表述，代码符号多为英文。方案由索引主机离线扫描候选文本，把真实出现的中文业务词与英文符号建立关联（候选范围含代码、配置、README 和设计文档，代码/配置来源置信度高于文档）。术语表仅为 Agent 增加检索词，不参与最终事实判断，答案仍需回到源码或配置表核对。约 1.4 万文件的开源游戏框架首次生成 2,634 个词条和 4,881 条关联记录，按当时 Bedrock Opus 4.8 价格折算模型费用约 372 美元——全量生成只适合作为可选构建任务，不应放入在线问答流程。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

## 证据可信度与性能权衡

答案可信依赖「先索引缩小候选、再读真实文件」：CodeGraph 预先解析代码建立符号/调用关系图用于缩小范围，最终结论以本轮实际读取的源码或配置为准。检索为空时明确说明未找到；存在多种解释时先确认范围；证据不足时标注不确定性并转交研发。系统提示要求 Agent 只把本轮读取过的源码/配置作为事实依据，未调用工具且无证据引用的长答案会被标记为异常。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

性能上，CodeGraph 索引查询冷/热缓存均 1–5 毫秒，远优于 grep（冷 127.4s/热 7.68s）和 ripgrep（冷 69.1s/热 0.49s）——但生产环境不能假定页缓存，冷缓存下几十秒的偶发扫描会形成长尾。端到端问答体验（建卡、microVM 调度、多轮模型/工具往返、流式收尾）才是真实指标：四批测试索引路径 36–58 秒 vs 基线路径 128–265 秒，等待时间降到约 1/5 至 1/3。优化应优先减少首轮检索词偏差、无效工具往返和总轮次，而非压缩毫秒级索引查询。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

## 会话复用与成本

网关维护空闲环境池：新对话优先绑定空闲 runtimeSessionId，没有可用环境再创建新会话启动 microVM。即使复用运行环境，每次调用仍创建新的 SDK 会话，网关把历史问答显式加入本轮提示词，模型不自动继承其他对话上下文——防止跨用户/跨对话上下文污染。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

成本方面，843 次问答单次模型费用平均 0.167 美元、中位 0.146 美元，约一半来自输出 token、另一半来自提示词缓存读取。优化方向是复用提示词缓存与减少总轮次。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md]

## 深度分析

### 只读性来自拓扑，而非工具清单

与「把仓库拉到终端、再靠提示词约束只读」的桌面 Agent 相反，这套方案让 [[concepts/agent-sandbox|microVM]] 不挂载仓库、不持 Git 凭证、无本地读取工具，查询全走 [[concepts/model-context-protocol-mcp|只读 MCP]]，「不能改」于是成为拓扑推论而非模型自律，攻击面由「读写执行」收敛为「越权读」。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:24-24]^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:58-60]

代价是索引成为共享关键路径：副本收敛为 1，扩容不必重拉代码，但建图 24 秒的预处理必须留在部署阶段；Git 按清单 ref 同步、本地仓靠推快照，答案并非「实时主分支」。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:70-72]

### 权限边界的三层与它不覆盖的部分

工具层是 9 个只读白名单接口（7 个定位读取 + 2 个术语表），不启用 Bash 与提交；网络层在 VPC 内，端口只对指定安全组开放，Runtime 无公网 IP——原文明确这只控入站，不是完整出站隔离。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:118-124]

第三层按信任域而非按项目切分：同域共用索引主机加进程/端口/仓库清单逻辑隔离，异域才拆独立主机。逐项目部署会把集中副本重新扩散回 N 份，信任域恰对应「谁允许被同一进程读到别人的代码」。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:126-130]

白名单管「能做什么」，还有一手管「能读到什么指令」：启动时关闭文件系统配置来源，仓库里的 CLAUDE.md 与 Skills 不会被自动加载，代码只以数据身份进入上下文。这是 [[concepts/prompt-injection-defense|注入防护]] 里最便宜、最易被跳过的一层。

但白名单挡的是写入与执行，不是机密性：文件工具仍读得到仓库内的密钥，片段仍进入模型上下文与复核区。准确表述是写入面被消除、泄露面被收窄且可审计，而非「代码安全了」。

### 中文提问、英文符号：把不可靠的层降级为导航器

业务人员问「技能为什么触发不了」，代码里叫 CheckSkillCooldown 或历史代号，中文直搜通常只命中注释。方案离线扫描代码、配置、README 与设计文档，把真实出现的中文业务词与英文符号关联，代码/配置来源置信度高于文档。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:40-42]

位置比生成方式更关键：术语表只增加检索词、不参与事实判断，结论仍须回源码核对——等于预认映射会错，把后果限制在「找错地方」而非「说错话」。原文自陈校验不核对引用行、也不验证符号是否存在，可见它只能是离线按需启用的导航层。

### 三个权衡：长尾、证据与轮次

索引的论据是方差而非速度：grep 冷 127 秒、热 7.7 秒，ripgrep 冷 69 秒、热 0.5 秒，图索引冷热皆 1–5 毫秒。热缓存下 ripgrep 并不慢，但新实例与仓库更新会制造冷扫描，几十秒的偶发延迟在交互问答里就是长尾；图查询还能回答 grep 答不了的调用关系与影响面问题。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:94-96]

工具延迟却不是瓶颈：113 次记录里工具调用中位 7 次、合计 0.26 秒，模型链路约占 98%，端到端中位 49–56 秒且几乎不随仓库规模变化，冷启动把中位推到 64 秒。空间在首轮检索词偏差、无效往返与总轮次。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:106-110]

证据规则把「事实」收紧到本轮：只用本轮读过的源码/配置，检索为空就说未找到，多解先确认范围，不足则标注并转交研发，无工具调用且无引用的长答案标为异常。原文承认这只是发现可疑回答，尚无准确率指标。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:80-86]

[[concepts/ai-cost-optimization-framework|成本结构]]决定该复用什么：843 次问答均 0.167 美元、中位 0.146 美元，半在输出 token、半在缓存读取；复用空闲 runtimeSessionId 省的是启动与拉代码，每次仍新建 SDK 会话并显式注入历史——省基础设施，不省隔离。^[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026.md:66-66]

## 实践启示

1. 先定拓扑再定工具清单：把只读性做成「环境无可写副本、无拉取凭证」，而非「提示词要求不要改」，否则一次工具扩充就会静默取消保证。
2. 用信任域做隔离粒度：同域共主机加逻辑隔离、异域拆主机，避免逐项目部署把集中索引打散成 N 份副本。
3. 把可能失真的层降级为导航器：术语表与图索引只影响「去哪里找」，事实一律以本轮实读为准，并把这条写进系统提示。
4. 为三类情况设出口：检索为空、多解、证据不足各有既定动作，并监控「无工具调用且无引用」的长答案。
5. 按端到端与轮次优化：分档记 p50/p95 与冷启动增量，把首轮检索词偏差和无效往返当主要成本项。

## 相关实体

- → MCP 协议生态
- → [[concepts/rag-retrieval-augmented-generation|RAG 检索增强生成]]
- → [[concepts/agent-security-architecture|Agent 安全架构]]
- → Agent 架构
- → [[concepts/agent-orchestration-patterns|Agent 编排模式]]
- → Agent 可观测性

→ [[raw/articles/readonly-code-qa-agent-game-business-team-aws-2026|原文存档]]
