---

title: "拆解OpenClaw架构（七）：安全漏洞，阿喀琉斯之踵"
type: entity
created: 2026-07-04
updated: 2026-09-23
tags: [wechat, ai]
rating: v9c8
sources:
  - raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 拆解OpenClaw架构（七）：安全漏洞，阿喀琉斯之踵

**来源**: 科技充电站

**发布日期**: 2026-03-04^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


**原文链接**: https://mp.weixin.qq.com/s/ReiDY6EWY195s4f2xTWvzA ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

---

AI 时代，有两种行为：

一种，活在别人的评测里，把模型的强当自己的强，痴人说梦；^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


另一种，活在真实的实战里，用最顶级的 AI，武装自己。^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


前者在噪音里坐享"技术平权"，后者在 疼痛中完成"自我进化"。^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


朋友们好，我是行小招。

这是 OpenClaw 深度技术解析系列的第七篇。前六篇拆了消息流水线、人格系统、Agent Runner、记忆系统、工具链和 Skills 生态，今天的话题我犹豫了很久要不要写，因为它不是功能拆解，而是一个正在发生的危机。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

上篇结尾我预告了：13.5 万暴露实例，26% 的 Skills 含漏洞，Meta 研究员的邮件被 Agent 全部删除。这些数字不是危言耸听，每一个背后都有公开的 CVE 编号、安全研究报告和真实的受害者。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

OpenClaw 的安全问题，不是某个模块写得不好，而是 整个产品定位和实际使用方式之间出现了断裂^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


先说核心矛盾。

回顾第一篇讲的 Gateway 架构：一个长驻的 Node.js 进程，默认绑定  127.0.0.1:18789  ，单实例运行，所有消息路由、Agent 调度、工具执行都经过它。这个设计的前提假设是什么？ ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

单用户、本地运行、操作者可信

你在自己的 Mac 上跑，你就是唯一的用户，你信任自己不会给自己发恶意指令。在这个前提下，Gateway 不需要复杂的认证体系，不需要多租户隔离，不需要担心跨用户的权限越界，整个架构简洁而优雅。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

但现实呢？

OpenClaw 爆火之后，用户开始把它部署到云服务器上，开始通过公网 IP 暴露 Gateway 端口，开始在团队中共享同一个实例。有些人改了默认绑定地址从  127.0.0.1  到  0.0.0.0  （监听所有网络接口），有些人甚至没改，只是不小心暴露了端口。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

一个为"自己人"设计的系统，突然站在了互联网上。^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


这就是所有安全故事的起点。

## 五个 CVE 的故事

让我用具体的 CVE 来说明这个断裂有多深。^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


CVE-2026-25253，CVSS 8.8， 这是最严重的一个。Control UI 接受一个  gatewayUrl  查询参数来指定要连接的 Gateway 地址，问题在于 没有任何验证， 攻击者构造一个链接，把 gatewayUrl 指向自己的恶意 WebSocket 服务器发给受害者，受害者点击后浏览器自动连接恶意服务器并把认证 token 发过去。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

拿到 token 就拿到了一切：Agent 控制权、命令执行权、文件系统访问权。一键 RCE。^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]


有意思的是，即使 Gateway 绑定的是 loopback 地址，这个攻击照样有效，因为桥接发生在受害者的浏览器里，不需要直接访问 Gateway 端口。v2026.1.29 已修复。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

CVE-2026-25157，CVSS 7.8， SSH 命令注入。如果项目路径被恶意构造（比如包含 shell 特殊字符的目录名），SSH 连接时会触发命令注入，这个漏洞利用的是 OpenClaw 对"项目路径是用户可控输入"这个事实的疏忽。 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

CVE-2026-24763，CVSS 8.8， Docker 沙箱逃逸。你以为在 Docker 里就安全了吗？攻击者通过 PATH 环境变量 ^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md]

→ [[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵|原文存档]]

## 深度分析

**攻击面三层结构：Gateway、Skills、上下文**。OpenClaw 的暴露面可以拆成三层：外部攻击面是暴露的 Gateway（13.5 万实例、15,200+ 存在已知 RCE 漏洞、549 个与 Kimsuky/APT28 基础设施关联），内部攻击面是 Skills 生态（31,000 个 Skills 中 26% 含漏洞、ClawHavoc 审计中 2,857 个里有 341 个恶意），最隐蔽的第三层则是上下文本身——prompt 注入和 context compaction 都在这一层起作用^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:82-122]。三层攻击面的共同点是：攻击载荷都可以是纯文本，一段自然语言就是协议^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:172-176]。

**五个 CVE 的共同根源是架构假设过期，而非编码疏漏**。CVE-2026-25253（gatewayUrl 无验证导致一键 RCE）、CVE-2026-25157（SSH 命令注入）、CVE-2026-24763（Docker 沙箱逃逸）、CVE-2026-25593（config.apply 未认证 RCE）、CVE-2026-26322（Gateway SSRF）——没有一个是忘记转义字符串或缓冲区溢出，每一个都源于同一条未受挑战的假设："使用者是可信的"^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:58-76]。Gateway 默认绑定 127.0.0.1、不做多租户隔离、不加认证体系，在"单用户、本地运行、操作者可信"的前提下这些是简洁优雅的设计；但产品爆火后部署形态跳出了前提（公网暴露、团队共享实例），假设失效而架构没跟着变，这就是断裂的本质^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:40-52]。

**安全与自主是同一枚硬币的两面**。Karpathy 称安全状况是 "a dumpster fire" 却同时盛赞底层 Agent 能力，这种撕裂感恰恰说明：系统的强大正来自广泛授权（读写文件、执行命令、记忆上下文、持久化状态），而风险也来自同样的授权^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:190-192]。一旦一个系统能读文本、执行命令、持久化状态，它与传统 C2 implant 的区别只在于意图——SOUL.md 被篡改后的 Agent 与精心编写的木马在功能上没有本质区别，且检测难度高一个数量级，因为所有行为都在"正常操作"范畴内^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:238]。这不是某个产品的 bug，而是 [[concepts/agent-security-architecture|Agent 安全架构]] 的普遍困境。

**上下文压缩让安全约束"随时间衰减"**。Summer Yue 邮件事件的根因是 context compaction 静默丢弃了"操作前确认"约束：压缩算法不知道哪些信息是关键安全约束，当操作从几封邮件扩展到整个邮箱时，约束随早期对话一起被修剪掉了^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:152-160]。这触及一个根本性架构问题：LLM 上下文窗口有限，而安全约束不能有"过期时间"——当安全机制依赖"模型记住用户说过什么"，你构建的就是一个安全性随时间衰减的系统，且目前没有好的解法^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:160-240]。相关背景见 [[concepts/context-management-agent-systems|上下文管理]]。

**沙箱与最小权限的教训**。Docker 沙箱被 PATH 环境变量操纵绕过（CVE-2026-24763），说明容器隔离不等于权限隔离；Aquaman 的设计（API 密钥永远不进 Agent 进程、经 Unix domain socket 从钥匙串注入）则示范了正确方向：Agent 拥有的权限应与它能接触到的凭证分离^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:66-218]。对比 NanoClaw 用 500 行代码 + Apple 容器实现真正的进程隔离，OpenClaw 的处境是对个人助手场景过度设计、对企业场景又安全不足^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:236]。沙箱设计的系统化讨论见 [[concepts/agent-sandbox|Agent Sandbox]]，Gateway 架构层面的对比见 [[entities/gateway-architecture-openclaw-claude-hermes-comparison|Gateway 架构对比]]。

## 实践启示

**自托管 Agent 平台加固清单（按优先级）**：

1. **先查网络暴露**：确认 Gateway 绑定 127.0.0.1 而非 0.0.0.0；如需远程管理，套一层 VPN/隧道而不是直接暴露端口——13.5 万暴露实例的根因就一条：默认配置绑定监听所有接口^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:104-106]
2. **关闭所有"先跑起来再说"的开放默认**：把 groupPolicy 从 open 改成 allowlist、开启 logging.redactSensitive（脱敏工具调用日志）、凭据文件设 chmod 600/700——这些正是 OpenClaw 官方 security audit 自动修复做的事^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:198-212]
3. **认证不留例外**：任何本地/远程接口（包括 WebSocket 方法）都要过认证校验，CVE-2026-25593 的教训是"只有本地进程能连上来"这个假设在共享主机上不成立
4. **Skill 审查先于安装**：用 Cisco skill-scanner（13 条 YARA 规则）或 Koi Security 的审计方法筛一遍，ClawHub 排名第一的 Skill 都可能是功能性恶意软件^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:114-118]
5. **凭证与 Agent 进程隔离**：参考 Aquaman 模式，密钥经钥匙串注入而非写在 Agent 可读的配置里^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:218]

**如何审查 Agent 工具权限**：

- **按"最坏情况"建模每个工具**：对每个 tool 问一句"如果这个工具被 prompt 注入劫持，最大损失是什么"——Shell 执行、文件写入（尤其是 SOUL.md 这类持久化人格文件）、网络请求（curl/SSRF 面）三类是高危核心
- **区分"误导性指令"与"直接执行"**：Markdown Skill 不执行代码，但它们操纵的 Agent 可以执行代码——审查重点不是 Skill 文件本身，而是它指示 Agent 做什么^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:120-124]
- **给安全约束找架构级载体**：不要依赖模型"记住"约束（会被压缩掉），把破坏性操作放进需要外部确认的审批流——审计工具是事后检查而非运行时防护，官方 50+ 项检查解决不了运行时拦截问题^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:198-228]
- **警惕"偏差正常化"**：prompt 注入风险每天发生但没出大事，不等于风险不存在——Simon Willison 的挑战者号类比值得贴在每个 Agent 项目 README 里^[raw/articles/拆解openclaw架构七安全漏洞阿喀琉斯之踵.md:180-188]

延伸实践参考：[[entities/openclaw-security-and-feature-enhancement-practices|OpenClaw 安全加固实践]]，更广的安全框架见 [[concepts/ai-safety|AI Safety]]。

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

