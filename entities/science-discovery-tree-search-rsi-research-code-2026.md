---
title: "九问ScienceDiscovery：树搜索驱动科研代码 RSI，加速科学发现"
created: 2026-09-05
updated: 2026-09-25
type: entity
tags: [ai4science, rsi, code-generation, tree-search, agent, open-source, sandbox]
sources: [raw/articles/science-discovery-tree-search-rsi-research-code-2026]
confidence: 0.7
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# 九问ScienceDiscovery：树搜索驱动科研代码 RSI，加速科学发现

> 来源：量子位（允中）。openJiuwen 社区的 ScienceDiscovery 用树搜索对「科研代码→产物」做递归自我改进（RSI），不训练模型、不调参数，每周只变产物。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 核心机制：科研场景的产物 RSI

把科研中「几十到上百次试错」交给程序自己完成：被演进的是**科研代码**本身，模型不训练、搜索规则不改，每一轮变的只有**产物**。搜索展开成一棵树——每一版产物是树上一个节点，可被再次选中/改写/长出分支，也可暂搁几十版后继续；高分支获得更多改写机会。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

每轮迭代四步：**选择**父版本 → 交模型**改写** → 进沙箱**评分**并挂成新节点 → 把这次访问**记账**到全部祖先。运行失败的版本同样入树（标记失败）。沙箱隔离运行失败/死循环/超时，因此不必对模型生成代码预设限制；有了分数，程序才知道哪版更好、下一轮从哪接着改。树持续生长、无需人工逐轮介入时，产物即在自行迭代。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 案例一：半无穷振荡积分

取 38 道积分（题面/答案 AI 全程看不到，唯一反馈是精度分）。基线 scipy.integrate.quad 得 -3.40（19 道只有 3 道进 3% 容差，最差差约 24 亿倍）；搜索到第 119 版得分 -0.0007（19 道全算准，平均相对误差 0.07%）。最终 247 行程序先判被积函数发散/振荡快慢再分情况选算法——一套通用规则，在未打分的 19 道上同样有效。全程 2 小时、236 个版本、无人干预。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 案例二：₂F₁ 双精度求值

高斯超几何函数 ₂F₁ 双精度求值是特殊函数体系枢纽，公认无单一算法覆盖全参数域。基线 scipy.special.hyp2f1 在测试参数分布上约 1/3 点正确有效数字不足 10 位。用 glm-5.2 跑 48 次扩展、598 秒，产出 199 行程序：1000 个未见点平均正确位数 9.836→11.771，可算 10 位以上点数 659→965。搜索找到经典恒等式把 z 换成 1/z 避开 z<-1 标准算法不收敛区，并自行确定切换条件。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 案例三：AlgoTune 代码加速

AlgoTune 收录 154 个 numpy/scipy/networkx/cvxpy 真实数值任务，得分是相对参考实现的加速比。基准自身结论：现有模型「倾向表层优化，未能发现算法层面创新」。ScienceDiscovery 同配置跑两个种子平均加速 2.279×（耗时降为四成多），提示词未点名任何加速技术。对照：官方榜最高 claude-opus-4.6 的 1.837、需先 RL 训模型的 MetaEvolve 为 2.045——此处用现成模型、没做任何训练。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 案例四：从观测数据反推方程

LLM-SRBench 的 LSR-Transform 子集给纯数字表（4000 行采样点、一列目标值），树上一节点是一段完整 Python 程序返回解析表达式。111 道题里 41.4% 写出正确方程（含玻尔能级反解主量子数、普朗克分布反解温度、相对论多普勒），每题平均仅 16.5 次模型调用，deepseek-v4-flash 费用不足 3 元。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 定位与边界

把「分数做可信」作为关键推进方向——通过更严验证让分数与真实目标不脱钩；每一环前推一步，能交给搜索自己走完的任务就多一类。人负责定义目标和判断标准，剩下的几十上百次改写由搜索自己完成。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md]

## 深度分析

### 树搜索驱动的科研代码 RSI：不训练模型，只演进产物

ScienceDiscovery 把 RSI 的载体从模型权重移到科研代码产物上：模型不训练、搜索规则不改，每轮变的只有产物。相比需先 RL 训模型的 MetaEvolve、靠海量遗传变异的 LaSR，它用现成模型 + 树搜索，把科研中「几十到上百次试错」的主要工作量交给程序自己完成。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:24,86]

AlgoTune 案例最有说服力：基准自身承认现有模型「倾向表层优化，未能发现算法层面创新」，而 ScienceDiscovery 提示词不点名任何技术，平均加速 2.279×，超过官方榜最高的 claude-opus-4.6（1.837×）和需 RL 训练的 MetaEvolve（2.045×）。瓶颈不在模型能力，而在「试错的组织方式」。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:64-68]

### 四个案例的共同模式：机器可判定 + 低验证成本 + 通用规则

四个案例共享一个前提：结果能被机器判定、验证便宜（几秒到几十秒）。因此积分 2 小时跑完 236 版全程无人干预，₂F₁ 仅 48 次扩展 598 秒，符号回归每题平均 16.5 次调用（同类约 250 次）、成本不足 3 元。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:88,136-144]

另一共同点：搜索产出的是通用规则而非题题分支堆砌。积分案例的 247 行程序先判断发散与振荡快慢再选算法，在未打分的 19 道题上同样有效；₂F₁ 案例中搜索自行找回经典恒等式（z→1/z）并确定切换条件。产出是可泛化的算法知识，而非过拟合补丁集。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:46,56]

### 选择规则与树形状：深挖与铺开的自动分配

选择规则：全树节点统一比较打分，名次越靠前越易被选中（深挖）；节点每被选中一次权重衰减一次（铺开），几轮后预算自动转向别处。积分案例的制胜一跳来自早已被超越的第 65 版（−2.22）被重新选中——若只贪心改写当前最优，这条路径不会出现。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:96-104,108]

同一套搜索跑不同题长出不同形状的树：失败版本不丢弃，标记失败挂回树上，配合祖先记账统计「这条子树值不值得再投入」——树本身就是搜索过程的完整记忆。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:112-118]

### 沙箱执行与验证闭环：RSI 的安全阀与边界

沙箱是循环无人运转的安全阀：失败、死循环、超时都被隔离，因此不必对模型生成的代码预设限制，搜索空间得以放开；有了分数，程序才知道哪版更好、从哪接着改。底座四插槽中沙箱位于「怎么跑怎么打分」一格，换算法/换任务时其余三格不动。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:32,124-128]

验证闭环也是边界：分数与真实目标脱钩，搜索就会朝错误方向高效狂奔。要推向验证昂贵的领域（合成样品、风洞、细胞培养），需一头用仿真、代理模型、自动化实验室把验证做快做便宜，另一头用更严验证让分数不脱钩。参见 [[concepts/verifier-driven-development]] 与 [[concepts/verifier-paradox]]。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:142-150]

## 实践启示

1. **把「试错」而非「实现」交给程序**：主要工作量在第一版之后的几十到上百次改写。验证可机器判定的任务，都值得搭建「选择→改写→沙箱评分→记账」循环。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:20-22]
2. **先做沙箱，再放开生成**：隔离失败/死循环/超时后就不必对模型输出预设限制——安全阀先行，搜索空间才够大（[[concepts/agent-sandbox]]）。优先投入验证基础设施，而非更长的提示词约束。
3. **选择规则决定搜索上限**：权重衰减 + 全树统一比较同时提供深挖与铺开；不要只贪心改写当前最优版本。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:98-104]
4. **失败版本入树，不要丢弃**：失败节点 + 祖先记账是判断路径投入价值的数据基础；丢弃失败版本等于销毁搜索记忆。
5. **用保留题集检验产物是否为通用规则**：若产物只是题题分支，说明评分函数被过拟合，需收紧验证。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:46]
6. **从验证便宜的任务切入 RSI**：数值计算、代码加速、符号回归先跑通，正因单次评分只要几秒；扩展新领域时，第一笔投资应花在把验证压快压便宜，而非更强的改写模型。参见 [[concepts/agent-self-improvement-loops]]、[[concepts/ai-self-improvement-bootstrapping]]。^[raw/articles/science-discovery-tree-search-rsi-research-code-2026.md:144-148]

## 相关实体

- [[entities/jiuwenswarm-coordination-engineering|JiuwenSwarm 协同工程]] — 同源 openJiuwen 社区；该 entity 的 ScienceDiscovery 4th Source 侧重「AI 科研工作台 / BiomniBench-DA SOTA」，本文侧重「树搜索驱动产物 RSI」机制，角度互补
- [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|AI 递归自我改进（NanoGPT-prime-intellect）]]
- [[entities/aide2-recursive-self-improvement-weco-2026|AIDE2 递归自我改进]]
- [[entities/autoresearch-feedback-loop-self-improving-agents-introspection|AutoResearch 反馈回路自改进]]
- [[entities/ai4s-2026-h1-frontier-panorama-yinxi|AI4Science 2026 上半年前沿全景]]

→ [[raw/articles/science-discovery-tree-search-rsi-research-code-2026|原文存档]]