---
title: "AI Assisted Cryptanalysis: Anthropic Claude Mythos 破解 HAWK 和 AES"
created: 2026-07-31
updated: 2026-09-14
type: entity
tags: [anthropic, cryptanalysis, claude, artificial-intelligence, cryptography, ai-security, post-quantum]
sources: [raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green]
confidence: 0.70
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# AI Assisted Cryptanalysis: Anthropic Claude Mythos 破解 HAWK 和 AES

> 2026年7月，Anthropic 的 Claude Mythos（未发布的先进模型）产出了两项密码分析成果：攻击签名方案 HAWK 和改进型 AES 降轮攻击。本文基于 Matthew Green（约翰霍普金斯大学密码学家）的深度分析。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## HAWK 签名方案攻击

Claude Mythos 对非标准后量子签名方案 HAWK 实现了密钥恢复攻击。HAWK 基于 module Lattice Isomorphism Problem (module-LIP)。关键发现：^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]


- HAWK 尚未标准化，但与正在标准化的 Falcon 相关——攻击不转移到 Falcon
- 攻击仍为指数时间，但将安全强度**减半**（可通过加倍密钥长度修复，但牺牲了 HAWK 的效率优势）
- 已产出可运行代码，数小时内可攻破弱化挑战实例
- **最显著的是**：攻击未发明新数学，而是更彻底地应用已有工具——Claude 自评："坦率说，这对整个领域有点尴尬——没有任何成分是稀奇的"^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## AES 降轮攻击

第二项成果是对降轮 AES（7 轮，完整 AES 为 10-14 轮）的改进攻击：^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]


- 7 轮 AES 攻击并非首创——这是对 2013 年工作的常数因子改进
- 需要 2^89 次操作和 2^105 个选择明文——**完全不实用**
- 是纸上分析，无可运行代码
- 技术上仍有趣，但属小幅增量改进^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## 研究过程：AI 驱动的密码分析

Anthropic 的流程出人意料："他们似乎只是让 Claude 去找结果，然后把它按在磨刀石上直到找到为止。" 没有庞大的领域专家团队，没有精细调优——模型能够：^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]


1. 理解现有密码分析结果
2. 综合为真实的新攻击
3. 扩展已有方法
4. 无需详细的人工干预^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## 验证瓶颈

Green 指出核心问题：模型善于产出**看起来真实但可能误导**的结果。人类注意力比以往更重要。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]


- HAWK 类"完整攻击"（可运行代码）→ 验证容易
- AES 类"速度改进"（纸上分析）→ 需要形式化证明或专家审查^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## 对密码学的影响

- **对称密码学**：混乱而鲁棒。AI 的大量智能时数不太可能奇迹般地攻破
- **公钥密码学**：更脆弱。公钥系统依赖的数学对象种类有限（ECDLP、RSA、格问题、编码理论），AI 的智力密度可能在这些领域产生真正进展
- 当前正处于从传统公钥算法向后量子算法过渡的历史时期——如果 AI 要在此时展示密码分析能力，时机正好^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## 对 AI 能力的判断

Green 给出了一个引人深思的类比：使用这些模型**"如同在深度突降的池塘里游泳"**——前一分钟触手可及，跨过某条线就只能靠自己了。但这条线在向外移动。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]


> "如果你认为这些模型是'美化的自动补全'或进展在放缓——请停止这种想法。模型非常智能且有能力，正在快速进步。如果说有天花板，我还没看到证据。"^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## 深度分析

### 「更彻底地应用已有工具」才是真正的转变

Green 对 HAWK 攻击最在意的不是攻击本身，而是它的构成方式：Claude 没有发明新的数学，只是把领域内已有的技术组合、推演得比人类研究员更彻底。这意味着 AI 在密码分析上的优势来源，可能不是"发现人类想不到的东西"，而是"把人类做不完的搜索与组合做到底"——在一个需要穷举大量技巧组合、反复核对细节的领域，耐心和吞吐量本身就是能力。这也解释了 Claude 为何用"对整个领域有点尴尬"来形容这个结果：它攻克的是一个本可被人类更早发现、却因人力与注意力有限而被跳过的角落。如果这个判断成立，那么衡量 AI 科研能力的标尺就要改一改——关键指标不是"是否原创了数学"，而是"能否把已有工具箱系统地遍历干净"。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

### 公钥与对称密码学的不对称性

Green 的判断把密码学劈成了两半：对称密码混乱而鲁棒，多投入智能时数也不太可能以奇迹方式攻破；公钥密码则更脆弱，因为它的整个安全基础都压在少数几类数学困难问题上——ECDLP、RSA 分解、格问题、编码理论。可被依赖的数学对象种类越少，攻击者可以系统搜索的表面就越集中。因此 AI 时代的密码分析压力不会均匀分布，而是集中在公钥一侧，尤其是那些设计时间短、评估尚不充分的新方案。HAWK 正是这类目标：它甚至尚未标准化，却已经走在被评估为未来标准的路上。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

### 验证瓶颈：人类注意力反而更重要

AI 越能批量产出"看起来真实但可能误导"的结果，人类的验证带宽就越成为稀缺资源。Green 指出的两条验证路径成本天差地别：HAWK 这类完整攻击产出了可运行代码，数小时内即可在弱化实例上复现，验证便宜；AES 这类速度改进只是纸上分析，必须通过形式化证明或资深专家审查才能确认，验证昂贵且难以自动化。这导出一个反直觉的结论——模型能力提升不会减轻人类审查负担，反而把人类推到关键路径上：产出越快，待验证队列越长，一个被漏掉的错误结论就越可能被当作成果采纳（参见 [[concepts/verifier-paradox|Verifier 悖论]] 与 [[entities/formatheoria-cfsg-ai-formal-verification-lean-2026|AI 辅助形式化验证]] 所描述的同构问题）。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

### 后量子过渡窗口为何放大了这次的示范价值

Green 特意强调时机：当前正处在从传统公钥算法向后量子算法迁移的历史窗口，大量新方案正在被设计、评估并即将写进标准，它们既年轻、缺乏数十年的攻击史，又必须立刻承载真实世界的数据安全。HAWK 被攻破的技术后果其实有限（仍是指数时间，可通过加倍密钥长度修复，代价是牺牲效率优势），但它发生在标准尚未冻结的阶段，示范意义远大于实际威胁——它说明新兴后量子方案在被充分检验之前，就可能被自动化工具系统性地审视。同样的能力也可以反向使用：在方案设计阶段就主动搜索弱点，而不是等到部署多年之后再补救。^[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green.md]

## 实践启示

1. **把"尚未标准化但在评估流程里"的新方案视同已被自动化审视的对象**。采用任何后量子签名或密钥封装方案前，先确认它的安全余量能否扛住"已有技术更彻底组合"式的攻击，而不是只复述方案作者给出的参数建议。
2. **在公钥环节预留冗余与可替换性**。攻击面收敛在公钥侧，架构上就应让算法可轮换、支持混合方案（传统 + 后量子并行），使任一方案被削弱时不必重写系统。
3. **给不同形态的结果分级设验证标准**。可运行攻击代码可以自动化复现验证；纸上速度改进必须排给专家审查或形式化证明。不要用同一套置信度去对待这两类产出。
4. **评估 AI 科研能力时，别把"是否发明新数学"当作门槛**。本期的价值在于工具组合的系统性遍历；评估基准应当包含"穷举式组合已有技术"这类任务，否则会系统性低估模型的实际冲击。
5. **把人类审查带宽当作发布节奏的约束条件**。模型产出速度提升不等于交付速度提升，验证队列就是新的瓶颈；高影响结论要预留专家复核时间，而不是让吞吐量决定什么时候对外宣布。
6. **在设计阶段就用 AI 找弱点，而不是等部署以后**。迁移窗口内，把自动化密码分析纳入方案评审流程，比等到标准冻结、实现铺开之后再发现问题便宜得多。

## 相关实体

- [[entities/apple-corecrypto-formal-verification-blueprint]]
- [[entities/drinking-llms]]
- [[entities/anthropic-95pct-data-analysis-skill-stack-architecture]]

→ [[raw/articles/anthropic-claude-mythos-cryptanalysis-hawk-aes-matthew-green|原文存档]]
