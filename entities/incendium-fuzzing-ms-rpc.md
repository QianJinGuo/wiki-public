---

title: "Incendium Fuzzing Ms Rpc"
type: entity
sha256: 7ac849d451f11d30be87187c8e9a7d75935199937b1a1c100ffa4d75a047acc4
date: 2026-05-08
source: newsletter
tags: [security]
review_value: 6
sources: [raw/articles/incendium-fuzzing-ms-rpc]
review_confidence: 8
created: 2026-05-10
updated: 2026-11-17
score_validated: 2026-09-05
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

> -> [[raw/articles/incendium-fuzzing-ms-rpc.md|原文存档]]

# Incendium MS-RPC Fuzzing
Incendium: Microsoft RPC 接口深度模糊测试，漏洞发现方法论   ^[raw/articles/incendium-fuzzing-ms-rpc.md]

## 摘要
[](/) [Remco van der Meer](/) Ethical Hacker, Security researcher ^[raw/articles/incendium-fuzzing-ms-rpc.md]

## 深度分析
Incendium 项目展示了 MS-RPC 模糊测试的方法论演进：通过递归式结构填充处理嵌套结构、通过 Union 类型识别处理 discriminant + arm 的复杂类型、以及用 ETW 替代 Process Monitor 实现全自动化。该研究最终发现了可从普通用户权限 Escalate 到 `NT AUTHORITY\SYSTEM` 的 privilege escalation 路径。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
**递归模糊的核心设计**：三个函数 `New-FuzzedInstance` / `Get-FuzzFieldValue` / `New-NdrEmbeddedPointerValue` 形成 mutually recursive loop，通过 `$Depth` counter（上限 8）和 `$Visited` type set 双重 guard rail 防止无限递归，同时正确处理 NDR 两种数据布局——embedded（inline struct）和 pointer（deferred `NdrEmbeddedPointer<T>`）。这是对 MS-RPC 复杂类型系统的系统性覆盖。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
**Union type 处理的关键洞察**：NDR marshaler 严格依赖 discriminant 字段决定序列化哪个 arm。如果 discriminant 与 populated arm 不匹配，会抛出 `No matching union selector` 错误。研究者采用保守策略：在 struct 内发现 Union_N 字段时，将所有 preceding integer-like 字段都设置为选中的 arm index。缓存复用场景下，含 Union 的类型必须用 `New-FuzzedInstance` 重新生成而非从缓存读取，否则 discriminant 可能与新上下文不匹配导致 marshal 失败。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
**ETW 替换 ProcMon 的工程价值**：传统 ProcMon 需要独立的 GUI 工具，增加了模糊测试的复杂度。通过 `SyscallMonitor.cs` 直接调用 `StartTraceW` / `EnableTraceEx2` / `ProcessTrace` 的原生 ETW API，并使用 raw `EVENT_RECORD` 指针（在 managed `System.Diagnostics.Eventing` 中不可用），实现了完全自包含的终端内监控，包括高权限账户（S-1-5-18/19/20）syscall 实时告警。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
### 从 canary 告警到 SYSTEM 代码执行：一次完整的发现链
方法论改进直接转化为战果的完整链条是本研究最有说服力的部分。对 spoolsv.exe 的 RPC 接口 fuzzing 时，ETW 报出 `RpcAddPrintProvidor` 以 SYSTEM 身份在文件系统各处探测 canary 字符串——且所有路径都被自动追加 `.DLL` 后缀，说明该 procedure 正按 DLL 搜索顺序寻找可加载模块。它完全无文档（搜索引擎零结果），签名 `int RpcAddPrintProvidor(string, Struct_40)` 中的 `Struct_40` 正是此前 fuzzer 无法处理的复杂类型——没有递归结构填充能力，这个接口根本无法被有效 fuzz。随后用 `-StringInput` 把字符串输入指向恶意 DLL，SYSTEM 上下文成功执行了其中的代码（whoami 写入 `C:\Windows\Tasks`）。唯一门槛是调用需 administrator 权限，普通用户返回 error 5；但接口同时注册了 `ncacn_ip_tcp` 协议序列，具备网络可达性。微软立场是 admin→SYSTEM 不构成安全边界，但对已拿到 admin 的红队，"让 SYSTEM 通过 TCP 加载任意 DLL" 仍有横向移动价值。这也印证 fuzzer 的定位：不产出 exploit，而是把"值得深挖的 procedure"从数百个接口中筛出来——与 [[entities/weve-been-here-before-decompilers-fuzzers-and-now-ai]] 中 fuzzing 工具的定位一致。
### 崩溃检测：与 ETW 正交的传输层信号
崩溃捕获走完全不同的信号源：fuzz call 致使服务进程终止时其 ALPC 通道失效，下一次调用返回特定 NTSTATUS `0xC0000701`，错误处理器精确匹配该状态码判定崩溃。由此形成两条正交遥测线：ETW 观察"输入落在哪里、以什么身份"，ALPC 状态观察"目标进程是否存活"——二者互补才能完整刻画一次 fuzzing 的后果。
### 可复现性：结构化输出与 ReplayFile 重放
ETW 事件增量落盘到 `Syscalls.json`，高权限事件单独写 `HighPrivSyscalls.json`，崩溃记入 `Crashes.json`，每条记录带完整归因字段（RpcServer、RpcInterface、RpcProcedure、Endpoint、UserSid），中断也不丢结果。`Invoke-RpcFuzzer` 的 `-ReplayFile` 参数读取这些 JSON 后替换自动生成的 data file 与 procedure 白名单，只重放触发过崩溃或高权限操作的调用。典型工作流：admin 全量 fuzzing 找出有意思的调用，再降权重放验证可达性；ALPC 端点多为 user-bound，重放前须改写到目标用户的上下文。
### 高权限身份归因的内核级实现细节
内核 provider 与 P/Invoke 会话机制已如上述，这里的关键增量是身份归因：每个命中事件读触发进程 token 的 `TOKEN_USER`、经 `LookupAccountSidW` 把 SID 解析为 `DOMAIN\User`；一旦命中 S-1-5-18/19/20 三个高权限服务账户，事件进入独立告警队列，fuzzer 在每次 RPC 调用后 drain 并实时告警，把路径、endpoint、procedure、身份归因到单条输出。相关脉络参见 [[entities/offensive-security-blog]]。

## 实践启示
1. **模糊测试嵌套结构时实现递归遍历**：不要只填充顶层字段，对每个复杂类型字段递归调用 fuzzing 函数，同时用深度上限和类型访问记录防止无限递归和循环引用。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
2. **Union 类型必须配套处理 discriminant**：不只是随机选 arm，还要设置 preceding integer fields 为对应 discriminant 值；在缓存复用场景下，对含 Union 的类型要强制重新生成。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
3. **用 ETW 替代 GUI 监控工具**：在 fuzzing 场景中，ProcMon 这类 GUI 工具不适合集成到自动化 pipeline。ETW 提供纯 programmatic 的 kernel-level 追踪能力，适合 headless 环境。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
4. **关注 RPC 接口的 privilege escalation**：MS-RPC 接口往往以高权限服务运行，本研究展示了用户输入如何通过 RPC 调用链到达 `system` 级别操作——这是横向移动和权限提升的经典路径。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]
5. **Canary + ETW 实时告警组合**：在 fuzzing 时使用可识别前缀（`incendiumrocks_`）作为 canary，配合 ETW 实时监控 file/registry 操作，实时捕获哪些 RPC 程序/端点触发了哪些路径，是高效缩小攻击面的方法。 ^[raw/articles/incendium-fuzzing-ms-rpc.md]

## 相关资源
- [[entities/agent-memory-architecture|Agent Memory 架构]]
- [[entities/claude-managed-agents-developer-guide|Claude Managed Agents 开发者指南]]
- [[raw/articles/incendium-fuzzing-ms-rpc.md|原文存档]]

## 相关实体

- [[moc/security-landscape|MOC]]
