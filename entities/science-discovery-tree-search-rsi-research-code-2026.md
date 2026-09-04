---
title: "九问ScienceDiscovery：树搜索驱动科研代码 RSI，加速科学发现"
created: 2026-09-05
updated: 2026-09-05
type: entity
tags: [ai4science, rsi, code-generation, tree-search, agent, open-source, sandbox]
sources: [raw/articles/science-discovery-tree-search-rsi-research-code-2026]
confidence: 0.7
provenance_state: extracted
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

## 相关实体

- [[entities/jiuwenswarm-coordination-engineering|JiuwenSwarm 协同工程]] — 同源 openJiuwen 社区；该 entity 的 ScienceDiscovery 4th Source 侧重「AI 科研工作台 / BiomniBench-DA SOTA」，本文侧重「树搜索驱动产物 RSI」机制，角度互补
- [[entities/ai-recursive-self-improvement-nanogpt-prime-intellect|AI 递归自我改进（NanoGPT-prime-intellect）]]
- [[entities/aide2-recursive-self-improvement-weco-2026|AIDE2 递归自我改进]]
- [[entities/autoresearch-feedback-loop-self-improving-agents-introspection|AutoResearch 反馈回路自改进]]
- [[entities/ai4s-2026-h1-frontier-panorama-yinxi|AI4Science 2026 上半年前沿全景]]

→ [[raw/articles/science-discovery-tree-search-rsi-research-code-2026|原文存档]]