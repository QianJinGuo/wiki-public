---
title: "Claude 形式化验证：1.8 亿行 Rocq 证明覆盖 RISC-V 机器码"
created: 2026-10-26
updated: 2026-10-26
type: entity
tags: [formal-verification, agent, claude, anthropic, rocq, riscv, security, code]
sources: [raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026]
confidence: 0.8
provenance_state: extracted
---

# Claude 形式化验证：1.8 亿行 Rocq 证明覆盖 RISC-V 机器码

Anthropic 前沿红队与 Theorem Labs 用 Claude 对真实生产级开源代码库发起形式化验证总攻：**123 个开源库、239 个测试入口**，最终产出约 10 亿行 Rocq 证明，其中 **1.8 亿行通过机器核验**。这不是"AI 帮忙写证明"的演示，而是一次以机器码为对象的、可被独立检查器复核的规模化工程。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

## 从"测试"到"证明"：漏洞存在于输入空间之外

过去十年开源世界的主力防线是模糊测试（OSS-Fuzz，2016 年起累计上万亿次执行），但它的输入通常被限制在 1MB 以内，覆盖不到罕见路径。研究团队把每个测试入口改写为一条数学命题：**在约定条件下，小于 64MiB 的任意输入都不会触发断言失败或 ASan 可捕捉的内存错误**，然后让 Claude 去证明它。证明卡住时有两种解释——还没找到证法，或者程序里真有 Bug；后者由 Claude 自己打补丁后继续证明。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

结论具有可对照性：某个解压器的计数器下溢只有在约 20MB 且排布罕见的输入下才触发，测试永远撞不到；团队在 Claude 用 Rust 写的 Python 解释器里故意埋了 7 个 Bug，**随机测试放跑 5 个，证明一个都没放过**。这给出了一条可复用的判据：验证强度不看运行次数，而看是否覆盖了"任何输入"这个量词。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

## 证明工程的成本结构：规模膨胀 1600 倍，检查器是真实瓶颈

工程量的量级来自一个具体数字：JSON 解析库 cJSON 的单个解析入口只有 **391 行 C 代码，Claude 为它写了 63 万行 Rocq 证明**。膨胀原因不在业务逻辑，而在"证明要覆盖整个进程"——C 程序启动、内存分配器、printf、内存检测工具自身的运行时要一并证明，和 cJSON 本身相关的部分只占约十分之一。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

最后把关的是 Rocq 证明检查器：按逻辑规则逐步验算，一份证明要花 30 分钟到数小时 CPU 时间。**检查器确认的是"证明符合规范"，人类仍要审查"规范是否对应真实需求"**——这是形式化验证无法外包给模型的残留责任。约一半以上的证明由 Claude 自己写的工具自动生成，说明这一层的杠杆在工具链而非单次推理。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

## 证明对象是机器码，不只是源码

这批证明做在**编译之后真正在 CPU 上运行的 RISC-V 机器码**上，因为源码和编译产物未必一致。为此 Theorem Labs 先把 RISC-V 指令与 Linux 进程行为写成数学模型，当前范围限定在单线程、静态链接、固定内存预算。深度案例包括：cpp-httplib 为排除递归撑爆栈，Claude 追进 libc++ 红黑树并在机器码上证明树高始终对数级；zlib 先把压缩格式写成形式化规范，再证明 Linux 内核自带实现完全符合。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

越界到硬件是同一路径的自然延伸：**x86 指令集的数学模型由 Claude 完全自主写出**，在模型里逐条指令单步推演能把 Ubuntu 的用户态完整启动起来；写模型的过程中找出 x86 官方参考文档数百处错误，并发现少量藏在 CPU 硬件本身的缺陷。从软件证到芯片，等于让 AI 重新核对整台计算机的地基。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

## 经济性拐点：验证速度每两个月翻一番

传统门槛不是技术而是成本——传奇微内核 seL4 **写代码只用 2 人年，做完形式化验证花了 20 人年**，所以这种级别的保证长期只出现在空客飞控这类人命关天的代码上。现在两条曲线同时下移：Opus 5.5 完成 36 个小型验证任务中一半所需的 token 比 Opus 5 少约 40%；Theorem 对 5 月至 9 月发布的 7 款前沿模型测得的验证速度约**每两个月翻一番**。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

第三项杠杆是证明复用：Agent 共享工作区后同一组任务少花 17% token；libcups、freerdp、libzmq 这类互不相关的项目因共用运行库等底层代码，可以复用大段证明。这一点对工程组织的含义比单点速度更直接——**验证资产像缓存一样在项目间摊薄，规模越大单位成本越低**。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

## 边界与判断

当前成果的适用面被明确限定：单线程、静态链接、固定内存预算，规范正确性仍需人类审查，未通过检查器的证明一行都不算数。因此合理定位不是"Bug 从此消失"，而是**验证从不可负担变为可负担**，供给侧的瓶颈从写证明的人和钱，转向写正确规范的人与检查器算力。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md]

## 深度分析

### 规模的两级衰减：生成是搜索，核验是判决

约 10 亿行 Rocq 证明最终只有 1.8 亿行通过检查，不到五分之一。把它读成"模型写错了 80%"是误读：死路引理、被放弃的分支、为探索写下却未被复用的辅助定义都留在账上，只有过了内核检查器的那部分才沉淀为资产。真正可用的工程指标是这条转化率——它决定换一单位可信代码要烧多少 token；而"超过一半的证明由 Claude 自己写的工具自动生成"说明，压缩成本的杠杆在工具链而非提示词。检查器不认的证明一行都不算数，这条硬门槛正是"看起来对的数学"与"数学"的分界线，也是它与模糊测试最根本的分野：后者的产出是概率性证据，前者的产出是可独立复核的判决。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:89-93]

### 1600 倍膨胀：证明的计量单位是进程，不是函数

cJSON 一个解析入口 391 行 C，对应 63 万行 Rocq。倍数不来自业务逻辑复杂度，而来自量词的范围——要断言"任意输入不越界"，就必须把 C 运行时启动、内存分配器、printf、以及内存检测工具自身的运行时一并纳入证明，与 cJSON 语义相关的部分只占约十分之一。这意味着约九成成本是环境与 ABI 的固定开销，它对同一软件栈上的每个库都要付一遍，于是自然导出两条策略：从依赖图底部向上证（地基一次付清），以及把底层证明做成可被引用的库而非每轮重写。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:85-87]

### 以机器码为对象：封住编译器与链接器的信任缺口

源码级验证留下一条缝：被验证的是源码语义，真正在 CPU 上跑的字节却由编译器、链接器和优化过程决定。这批证明直接做在 RISC-V 机器码上，前提是先把指令集与 Linux 进程行为写成数学模型，再把模型当公理使用——信任没有消失，只是从编译器转移到了模型本身。x86 那段是模型可信度的最好证据：模型精确到能逐条单步推演把 Ubuntu 用户态启动起来；同时也是模型的边界证词，写模型时暴露的数百处官方文档错误与少量 CPU 硬件缺陷，后者任何软件证明都无法修复，只能变成假设或从范围内切掉。这与 Lean 侧的形式化浪潮构成同一条路线的两端，一端证数学，一端证执行。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:81-83] ^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:109-113] 参见 [[entities/formatheoria-cfsg-ai-formal-verification-lean-2026]]。

### 三重叠加的增长，与随之迁移的瓶颈

验证速度每两个月翻一番，约合每年 6 倍；同时单任务 token 降约 40%、共享工作区再省 17%、跨项目证明复用又摊薄底层开销。三项相乘解释了"123 个库只用几周"与 seL4 的对照——写代码 2 人年、验证 20 人年，过去只有飞控级代码付得起。但乘法不是免费的：检查器一份证明要 30 分钟到数小时 CPU，这条曲线不翻番，所以证明产能一旦跑赢核验算力，瓶颈就迁到检查器与写规范的人。这正是 [[concepts/verifier-paradox]] 在硬件层的重演；同理，1.34 亿美元覆盖 Nix 生态 95% 机器码是外推而非报价，它成立的前提正是"每两月翻番"持续到 2027 年底——应把它读作对一个曲线的押注，而不是预算数字。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:91-97] ^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:121-137]

### 覆盖不到的：输入空间的量词边界与规格正确性

命题形如"小于 64MiB 的任意输入不触发断言失败或 ASan 错误"，量词一旦写定，证明强度就被它封顶：64MiB 之外、多线程交错、动态链接、非固定内存预算全在射程之外，而真实攻击面往往正长在这些缝里。第二层风险更隐蔽——检查器只保证证明符合规范，规范是否符合真实需求仍要人类审查；形式化甚至会把规格错误固化得比代码 Bug 更牢，因为它带着数学的权威外观。所以准确定位是：把"漏掉罕见路径"这一类错误压到接近零，而不是把安全外包给模型。这与 [[entities/weve-been-here-before-decompilers-fuzzers-and-now-ai]] 记录的循环同形——每一代新工具都被误读为终局，实际只是把防线上移了一层。^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:56-58] ^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:83] ^[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026.md:91]

## 实践启示

1. **先证规范，再证代码**。把期望行为写成可审查的规格（zlib 的做法是把压缩格式标准形式化），规格评审的人力投入应与其后全部证明的价值成正比——这一环检查器永远代劳不了。
2. **从依赖图底部选目标**。九成成本是运行时与 ABI 的固定开销，先证 OpenSSL、libc 这类被上万包引用的底层库，单位成本才随规模下降（OpenSSL 一次约 22 万美元，摊到每个下游包约 14 美元）。
3. **把证明当缓存资产，而非一次性产出**。建可检索的引理库、让 agent 共享同一份证明索引，是当前最确定的降本手段：实测共享工作区省 17% token，同栈项目可大段复用底层证明，规模越大单位成本越低。
4. **用检查器做唯一裁判，未过检的证明一律不放行**。验收写成二进制门禁；卡住时区分"缺引理"与"真有 Bug"两种信号，并允许 agent 打补丁后接着证——这条 patch-then-continue 回路才是它强于测试的根本原因。
5. **把量词范围写进交付文档，当作威胁模型的一部分**。单线程、静态链接、64MiB 以内这些前提必须显式声明；范围之外的风险不能因为"已验证过"而被默认安全，同时要为检查器算力做容量规划（单份 30 分钟至数小时 CPU），把并行核验与增量重检计入工程预算。

## 关联

- [[entities/agent-formal-verification-ai-code]] — 从经济学角度论证"代码变便宜后形式化验证变得合理"的前置分析，本篇提供了规模化实证数据
- [[concepts/evaluation-harness-design]] — 验证/评测 harness 的设计逻辑同源：判据强度决定结论强度
- [[concepts/agent-harness-engineering-paradigm]] — 证明复用与共享工作区是长程 agent 工程的成本优化手段

→ [[raw/articles/claude-formal-verification-rocq-riscv-theorem-labs-2026|原文存档]]
