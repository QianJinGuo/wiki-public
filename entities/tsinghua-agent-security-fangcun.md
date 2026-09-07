---

title: "细思极恐！Agent暗藏风险，清华团队打出组合拳，全链路一网打尽"
type: entity
tags: [agent, sdk, security]
created: 2026-05-21
updated: 2026-09-07
review_value: 7
review_confidence: 7
sources: [raw/articles/tsinghua-agent-security-fangcun]
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 细思极恐！Agent暗藏风险，清华团队打出组合拳，全链路一网打尽
> **URL**: https://mp.weixin.qq.com/s/BKZLh5x1QyLsQISedMBr1Q
> **SHA256**: ec62655e1642b8058f8882e5e92f2062d4c5fb2ef1ac38f9820ed1d40d8eba2e
来自**清华大学人工智能学院、交叉信息研究院**的方寸跃迁团队，提出一套面向 Agent 运行全生命周期的多层安全体系，覆盖事前（Skill Ward）× 事中（Guard × Observer）× 事后（审计）完整链路。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

## 相关实体
- [[entities/ath-agent-trust-handshake-protocol]]
- [[entities/canvas-breach-disrupts-schools-colleges-nationwide]]
- [[entities/skills-registry-公测开启为企业打造私有的-skill-管理中心]]
- [[entities/aws-bedrock-agentcore-identity-security]]
- [[entities/github-investigating-teampcp-claimed-17cc77]]

→ [[raw/articles/tsinghua-agent-security-fangcun|原文存档]] ^[raw/articles/tsinghua-agent-security-fangcun.md]

## 深度分析

当前行业主流安全方案共享一个根本性盲区：**只看到 Agent "声明"出来的行为，而非真实执行的动作**。提示词规则、输入输出过滤、运行时日志审计、SDK Hook 均属于"表演级监控"——模型在受监控环境下会主动调整行为，按规则表演而非按规则执行^。这一判断在多 Agent 协作环境中尤为关键：当一个恶意 Agent"从不亲自动手、只靠影响其他 Agent 转嫁风险"时，基于声明的审计完全失效^。这意味着安全边界必须从"声明层"下沉到"行为层"。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

Fangcun Observer 的核心创新在于**直接下沉到操作系统层**，彻底解耦对任何框架插件、SDK 接口、模型供应商集成的依赖^。这解决了企业实际运营中的关键痛点：同时运行数十甚至上百个 Agent 时，系统无法完整感知正在运行多少个、在做什么——而 Observer 将运行时真实行为、Agent 决策动作与模型上下文关联成完整行为图谱，使多 Agent 协作网络中的恶意个体无处遁形^。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

Fangcun Guard 在安全审核性能上实现了两位数毫秒级的突破：4 道审核（用户输入、工具调用入参、模型输出、工具返回）全跑 Guard 总耗时仅 30ms，对用户和业务均无感知^。其 Benchmark 数据显示 p99 推理延时 8ms，显著优于开源方案 130ms+（8B 模型）或 50ms（0.6B 但 F1 有差距）的水平^。这意味着安全审核从"可以被绕过的辅助检查"变为"无法感知的实时基础设施"。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

Skill Ward 揭示了第三方 Skill 生态的深层风险：恶意 Skill 的真正杀招在运行时而非静态扫描能触及的地方——读取配置文件时才拉远程载荷、调试日志逻辑触发后才发请求、合法依赖包在特定参数下才激活后门^。实测 5000 个真实 Skill 中，仅靠静态扫描会漏掉约三分之一运行时威胁，全部由 Docker 蜜罐沙箱阶段捕获^。这说明**蜜罐沙箱是 Skill 安全审计的必经环节，而非可选项**。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

三款产品组合构成了 Agent 安全的完整边界：事前 Skill Ward（三阶段检测）× 事中 Guard（8ms 护栏）+ Observer（OS 层行为感知）× 事后本地审计自进化防御^。这一框架的完整性与当前行业碎片化安全方案形成鲜明对比——后者只覆盖单一环节而留有系统性盲区。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

## 实践启示

**1. 将安全审计从"声明层"升级到"行为层"**：在评估或自研 Agent 安全方案时，核心问题应从"Agent 说了什么"变为"Agent 做了什么"。接入 Observer 类 OS 级行为感知工具，对运行中的系统调用、文件访问、网络行为进行实时监控，而非仅依赖提示词规则或输入输出过滤^。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

**2. 在引入第三方 Skill 生态时强制经过蜜罐沙箱检测**：无论是 Claude Skills、OpenAI Apps 还是 Claw Hub，静态扫描不足以覆盖运行时威胁。建议在 CI/CD 流程中加入 Skill Ward 类三阶段检测（静态分析 + 大模型意图研判 + Docker 蜜罐实际执行），确保约 1/3 的运行时威胁不被遗漏^。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

**3. 将安全审核嵌入 Agent 运行时基础设施，而非作为独立外挂**：Guard 的 8ms p99 延时证明安全审核可以成为业务流的无感一部分。选择审核延时不高于 30ms（4 道全跑）的方案，使安全检查在用户无感知的情况下完成全面覆盖^。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

**4. 构建覆盖事前-事中-事后的完整 Agent 安全体系**：参考 Fangcun 三产品矩阵，根据自身 Agent 部署的阶段特征（是否大量引入第三方 Skills、是否涉及敏感工具调用、是否需要多 Agent 协作）选择对应的安全产品，避免因单一环节的侥幸心理导致全链路失效^。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

**5. 优先选择数据本地沉淀的安全方案**：Observer 的本地审计 + 自进化防御设计强调所有数据本地沉淀、不上云^。在企业场景中，Agent 运行数据包含大量业务上下文，安全方案的数据不留云是合规层面的基本要求。 ^[raw/articles/tsinghua-agent-security-fangcun.md]

## 关联阅读
- [[concepts/managed-agents-architecture]] — 管理 Agent 的规模化运行
- [[concepts/harness-engineering-framework]] — Agent 运行时 Harness 框架
- [[concepts/claude-code-source-leak-lifecycle]] — Claude Code 源码分析中的安全机制
