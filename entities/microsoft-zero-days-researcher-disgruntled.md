---

title: "Mystery Microsoft bug leaker keeps the zero-days coming"
type: entity
tags: [microsoft, research, security]
created: 2026-05-21
updated: 2026-05-21
review_value: 6
review_confidence: 6
sources: [raw/articles/microsoft-zero-days-researcher-disgruntled]
---

# Mystery Microsoft bug leaker keeps the zero-days coming
Security pros warn YellowKey claim could make stolen laptops a much bigger problem ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]
](https:&#x2F;&#x2F;www.theregister.com&#x2F;author&#x2F;connor-jones) ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]
Cybersecurity reporter

## 相关实体
- [[entities/microsoft-zero-days-researcher-disgruntled-theregister]]
- [[entities/disgruntled-researcher-microsoft-zero-days]]
- [[entities/13-disgruntled-researcher-releases-two-more-microsoft-zero-days-5239758]]
- [[entities/down-fall-of-bug-bounties]]
- [[entities/how-harnesses-and-post-training-close-the-open-weight-bug-finding-gap-20260606]]

→ [[raw/articles/microsoft-zero-days-researcher-disgruntled|原文存档]] ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]

## 深度分析

**Nightmare-Eclipse 的持续曝光策略**——该研究员采用了一种高度系统化的漏洞披露节奏：选在微软每月 Patch Tuesday 之后发布新 zero-day，使微软安全团队无法在常规补丁周期内响应，被迫进入应急模式。这不是冲动型泄密，而是精心设计的施压行为。原文存档显示，研究员已在 2026 年陆续披露 BlueHammer（CVE-2026-32201）、RedSun、UnDefend 和本次的 YellowKey、GreenPlasma，共五个 zero-day，形成持续性威慑 。 ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]

**YellowKey 的实质性威胁边界**——尽管需要物理访问的限制常被用来降低风险评级，但安全专家 Rik Ferguson 的判断揭示了真实隐患：BitLocker 是被盗设备的最后防线，一旦被绕过，"丢失的笔记本电脑不再只是硬件问题，而是数据泄露通知触发点"。这意味着企业资产管理策略需要重新评估——设备失窃的处置流程将涉及监管合规而非单纯硬件更换 。 ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]

**GreenPlasma 与真实攻击链的衔接**——GreenPlasma 提供了 SYSTEM 权限提升，Nightmare-Eclipse 虽只发布部分 PoC 代码而非完整利用程序，但 Knapp 指出这类权限提升漏洞的实际价值在于攻击后利用阶段：攻击者获得初始立足点后，用此类漏洞获取高权限、横向移动、窃取凭据，最终达成数据外泄或勒索软件部署目标。Huntress 观测到 RedSun 和 UnDefend 的 PoC 在发布后迅速被实际攻击采纳，说明即便是不完整的利用代码也具备实战价值 。 ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]

**"前员工"动机的可信度与泄密边界**——Chaotic Eclipse 在博客中声称"有人违反了协议使我无家可归"，暗示与微软存在雇佣或 NDA 纠纷。若属实，这揭示了大厂安全研究员与雇主之间关于漏洞归属权的法律灰色地带——内部发现的漏洞究竟是公司资产还是个人研究成果？此类争议在安全社区并不罕见，但将其转化为公开 zero-day 披露则跨越了法律和道德边界 。 ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]

**防御方的现实困境**——YellowKey 目前无已知补丁，GreenPlasma 也无官方缓解措施，只能依赖 BitLocker PIN + BIOS 密码锁降低风险。这意味着企业需要在"强加密防护"与"可用性"之间做出取舍，而这种取舍在面对有组织的报复性披露时显得尤为脆弱。安全体系不能假设漏洞永不流出，需要在纵深防御层面提前规划 zero-day 应急响应流程 。 ^[raw/articles/microsoft-zero-days-researcher-disgruntled.md]

## 实践启示

- **重新定义设备失窃响应**：YellowKey 将"丢失笔记本电脑"从硬件问题升级为数据泄露事件，企业安全策略必须随之调整——所有 BitLocker 设备失窃均应触发数据泄露评估和监管通报流程 。

- **建立 zero-day 应急储备**：鉴于 arrive-Eclipse 暗示持有"dead man's switch"且过往威胁均已兑现，组织应提前准备应急响应手册，包括备用加密方案、特权访问监控和设备远程擦除能力 。

- **物理安全与数字安全的交集治理**：BitLocker 绕过攻击重新将"设备物理访问"纳入攻击向量，安全团队应将对高敏感设备的物理访问管控纳入整体风险评估，而非简单依赖加密作为最后防线 。

- **PoC 代码快速武器化的监控机制**：Huntress 已观测到 RedSun 在野利用，组织应部署漏洞利用检测签名和异常进程行为监控，即便 PoC 不完整也可能在短期内被改造使用 。

- **内部漏洞协议明确化**：若组织有安全研究员团队，应明确书面规定内部发现漏洞的披露流程、归属权划分和激励机制，避免将争议泄漏转化为外部 zero-day 披露的法律和声誉风险 。
