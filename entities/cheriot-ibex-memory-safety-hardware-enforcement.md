---

title: "CHERIoT-Ibex: Closing the door on memory safety vulnerabilities with hardware-enforced protection"
created: 2026-05-16
updated: 2026-09-07
type: entity
tags: [memory, open-source, architecture]
sources:
  - raw/articles/cheriot-ibex-memory-safety-hardware-enforcement
review_value: 5
review_confidence: 7

score_validated: 2026-09-05
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

## 深度分析
CHERIoT-Ibex 是微软于 2023 年开源的 CHERIoT 平台的核心实现，首次将 CHERI（Capability Hardware Enhanced RISC Instructions）能力模型落地为生产级开源硬件。 CHERI 架构通过**能力指针（Capability）** 取代传统 flat pointer，从硬件层面强制约束每个内存区域的访问权限——包括空间边界（spatial）和有效期（temporal），从根源上堵死 buffer overflow 和 use-after-free 两类最高发漏洞。   ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
CHERIoT 在 CHERI 基础上专为嵌入式 / IoT 场景做了轻量化适配，底层选用 LowRISC 的 32 位 RISC-V 核心 Ibex。CHERIoT-Ibex 通过 CHERI Alliance 认证，验证其提供**空间安全 + 时间安全 + 细粒度隔离**三重保障，且硅成本与低功耗微控制器相当——打破了"安全必付溢价"的传统假设。 ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
微软数据显示其每年 CVE 中约 70% 源于内存安全漏洞（CISA 报告亦指出软件产品内存安全问题的紧迫性），CHERIoT-Ibex 的定位正是从硬件层消除这类缺陷的根因。 ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]

## 实践启示
**适用场景**：对安全有强制要求的嵌入式微控制器、IoT 端点、Azure 底层基础设施固件。 ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
**集成路径**：微软已开源完整 ISA + 工具链 + RTOS + RTL，开发者可通过 [microsoft/cheriot-ibex](https://github.com/microsoft/cheriot-ibex) 获取并参与生态。 ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
**架构决策**：CHERIoT-Ibex 体现 silicons-to-systems 战略——安全不从软件层打补丁，而是下沉至硬件基础设施，从设计之初即嵌入纵深防御。 ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]

# "CHERIoT-Ibex: Closing the door on memory safety vulnerabilities with hardware-enforced protection"
URL Source: https://techcommunity.microsoft.com/blog/azureinfrastructureblog/cheriot-ibex-closing-the-door-on-memory-safety-vulnerabilities-with-hardware-enf/4517904 ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]

# CHERIoT-Ibex: Closing the door on memory safety vulnerabilities with hardware-enforced protection | Microsoft Community Hub
Open Side Menu ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Skip to content](https://techcommunity.microsoft.com/blog/azureinfrastructureblog/cheriot-ibex-closing-the-door-on-memory-safety-vulnerabilities-with-hardware-enf/4517904#main-content)[![Image 1: Brand Logo](https://techcommunity.microsoft.com/t5/s/gxcuf89792/m_assets/themes/customTheme1/favicon-1730836271365.png?time=1730836274203)](https://techcommunity.microsoft.com/) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Tech Community](https://techcommunity.microsoft.com/)[Community Hubs](https://techcommunity.microsoft.com/Directory) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Products](https://techcommunity.microsoft.com/) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Topics](https://techcommunity.microsoft.com/) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Blogs](https://techcommunity.microsoft.com/Blogs)[Events](https://techcommunity.microsoft.com/Events) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Skills Hub](https://techcommunity.microsoft.com/category/skills-hub) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Community](https://techcommunity.microsoft.com/) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Register](https://techcommunity.microsoft.com/t5/s/gxcuf89792/auth/oidcss/sso_login_redirect/provider/default?referer=https%3A%2F%2Ftechcommunity.microsoft.com%2Fblog%2Fazureinfrastructureblog%2Fcheriot-ibex-closing-the-door-on-memory-safety-vulnerabilities-with-hardware-enf%2F4517904)[Sign In](https://techcommunity.microsoft.com/t5/s/gxcuf89792/auth/oidcss/sso_login_redirect/provider/default?referer=https%3A%2F%2Ftechcommunity.microsoft.com%2Fblog%2Fazureinfrastructureblog%2Fcheriot-ibex-closing-the-door-on-memory-safety-vulnerabilities-with-hardware-enf%2F4517904) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
1.   [Microsoft Community Hub](https://techcommunity.microsoft.com/) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
3.   [Communities](https://techcommunity.microsoft.com/category/communities)[Products](https://techcommunity.microsoft.com/category/products-services) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
5.   [Azure](https://techcommunity.microsoft.com/category/azure) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
7.   [Azure Infrastructure Blog](https://techcommunity.microsoft.com/category/azure/blog/azureinfrastructureblog) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[Report](https://techcommunity.microsoft.com/blog/azureinfrastructureblog/cheriot-ibex-closing-the-door-on-memory-safety-vulnerabilities-with-hardware-enf/4517904#) ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]

## Azure Infrastructure Blog
## Blog Post
Azure Infrastructure Blog ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
3 MIN READ ^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]

# CHERIoT-Ibex: Closing the door on memory safety vulnerabilities with hardware-enforced protection
[![Image 2: kunyanliu's avatar](https://techcommunity.microsoft.com/t5/s/gxcuf89792/m_assets/avatars/default/avatar-9.svg?image-dimensions=50x50)](https://techcommunity.microsoft.com/users/kunyanliu/3487734)^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
[kunyanliu](https://techcommunity.microsoft.com/users/kunyanliu/3487734)^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
![Image 3: Icon for Microsoft rank](https://techcommunity.microsoft.com/t5/s/gxcuf89792/images/cmstNC05WEo0blc?image-dimensions=100x16&constrain-image=true)Microsoft^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
May 08, 2026^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
Memory safety vulner^[raw/articles/cheriot-ibex-memory-safety-hardware-enforcement.md]
## 相关实体
- [[entities/05-11-the-great-memory-panic-of-2026]]
- [[entities/memory-in-the-llm-era-iclr2026]]
- [[entities/openchronicle-memory-layer]]
- [[entities/hermes-9-module-architecture-winty]]
- [[entities/openclaw-prompt-context-harness]]
- [[moc/memory-context-systems|MOC]]
