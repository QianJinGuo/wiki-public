---
title: "Game-the-LLM-Reviewer：专防 AI 审稿「错杀」的开源 Skill（PaperWeekly）"
created: 2026-09-24
updated: 2026-09-24
type: entity
tags: [skill, agent, llm-reviewer, peer-review, paper-writing, iclr, prompt-engineering]
sources: [raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了]
confidence: 0.8
provenance_state: extracted
---

# Game-the-LLM-Reviewer：专防 AI 审稿「错杀」的开源 Skill

## 摘要

Game-the-LLM-Reviewer 是一个开源 Agent Skill（<https://github.com/Michael-Jiahao-Zhang/game-the-llm-reviewer>），Claude Code、Codex 等 Coding Agent 可直接安装使用。它把 5 项 AI 审稿偏好研究提炼成一套针对 LLM 审稿偏好的改写策略，在论文完成常规润色后再跑一遍，目标是防止「已经做出来的工作，因为没写成 LLM 更偏好的样子，被 AI 审稿人错杀」。核心约束是意义保持（meaning-preserving）：四个数字一个不改、科学判断不变，只调整表达方式。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

## 核心要点

- **问题**：科学内容不变，仅调整贡献表达方式与结果呈现重点，AI 审稿人的评分就可能变化——审稿人是否把论文丢给大模型，作者无法控制。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]
- **实证基础**：5 项相关研究，其中一项以 120 篇匿名 ICLR 2026 投稿构建 4200 个论文版本、交 5 个大模型评审；实验结果与创新性的写法对评分影响最明显。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]
- **安全边界**：不调用目标审稿模型、不按评分迭代改稿、无须知道审稿人用哪个 LLM——只利用研究中相对稳定的规律，不是「稳定加分机器」。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]
- **护栏机制**：证据先行（改前核对 LaTeX 源文件中的表格与假设）+ S6 等价检查（改后确认科学判断未变，措辞变强即撤回）。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]
- **产出**：另存修改稿 + `changes.md`，逐处注明策略、理由与保持不变的科学含义；有 LaTeX 工具链时验证可编译。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

## 深度分析

### 为什么「写法」会决定 AI 审稿人的评分

LLM 审稿人本质上是按文本表征做评估的：同样的实验结果，「30%→34%」的原始表述和「均提升 4 个百分点」的直接陈述，在模型眼中是不同强度的证据呈现。示例论文中，两个不同基座 Agent 加入执行记忆后，问题解决率分别从 30%→34%、35%→39%，改写后四个数字一个没动，只是把差距明确写出来，并让「无需更新模型权重」这一已有贡献提前进入引言开头。摘要同样只需小幅调整叙述顺序、让贡献更早出现，不必整段重写。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

但研究也揭示了一个关键的非对称性：**同一种改写对不同审稿模型的效果并不一致，有时甚至降分**。这决定了整个项目的设计哲学——它不承诺加分，只承诺把已被多项研究交叉验证的稳定规律用起来，并明确「找不到能保持原意的改法，就保留原文」。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

### 五类表达调整与等价检查

改写策略按优先级分层：优先处理贡献与结果的表达，其余按需使用。一个有代表性的边界案例是自谦措辞的区分处理——「我们只测试了英语查询」可改为「我们用英语查询进行了测试。其他语言尚未测试。」前者是不必要的自我弱化，后者如实陈述了局限性；而真正涉及不确定性、研究成熟度和结论强度的表达则原样保留。这条界线把「修辞层面的自贬」与「科学层面的限定」分开了，是整个 Skill 最精细的判断点。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

等价检查（S6）是双保险的下半环：即使数字完全没动，只要措辞让结论变得更强，这处修改也会被撤回；摘要和正文中的相关说法一并检查，避免前后不一致。修改前还有证据核对环节——处理 LaTeX 项目时读取主文件引用的章节源文件，核对相关表格和假设，确认要改的表述有证据支持。改写被约束在「人类审稿人能做出的科学判断不变」的范围内。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

### 工程形态：研究笔记到 Agent Skill 的落地样本

这个项目的意义不止于审稿博弈本身，它是一个「把多项研究蒸馏成 Agent 可执行 Skill」的标准流程样本：梳理研究 → 提炼稳定规律 → 固化为带护栏（证据核对、等价检查、变更日志）的 Skill → 一句话接入任意 Coding Agent（`npx skills add Michael-Jiahao-Zhang/game-the-llm-reviewer --skill game-the-llm-reviewer`）。范围可指定（只处理摘要或某一节），仓库附带编程 Agent 引言示例供试跑。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

作者立场同样值得记录：明确反对把同行评审的决定交给 LLM，但论文投出后审稿人是否借助 AI 不受作者控制——这个 Skill 回应的正是这种不对称处境：你能决定自己怎么写，不能决定别人怎么审。^[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了.md]

### 与 LLM 评估偏差研究的连接

该项目隐含着一个更广的问题域：LLM 作为评审/评估者时的系统性偏差。这与 wiki 已有 coverage 形成呼应——安全审查中的 model anchoring bias、reward hacking 中模型对表面指标的过拟合，都属于「模型对表征形式敏感而非对实质内容敏感」的同一族现象。Game-the-LLM-Reviewer 的独特之处在于它站在被评估者一侧利用这种敏感性，且坚持不欺骗、只重述的伦理底线。

## 实践启示

1. **投稿前最后一道工序**：论文完成常规润色后跑一遍该 Skill，成本极低（一句话接入），防的是「表达不符合 LLM 偏好导致的错杀」这一纯修辞损失。
2. **数字永远不动**：所有改写围绕「把已有差距写明」展开，若一处修改需要改动数值或让结论变强，立即回退——这是可迁移到任何 AI 辅助写作场景的红线。
3. **区分自谦与诚实**：删掉的是不必要的自我弱化（「只是」「仅仅」式措辞），保留的是真实的局限性陈述（未测试的语言、未验证的假设）。
4. **改前核对证据、改后留痕**：`changes.md` 逐处注明策略与理由的模式，适用于所有需要可审计性的 Agent 写作/编辑任务。
5. **不要迭代套分**：不调用审稿模型、不按评分反复改稿，既避免过拟合某个特定审稿 LLM，也规避了 gaming 评审的伦理滑坡。
6. **Skill 蒸馏模板**：多项研究 → 稳定规律 → 护栏化 Skill → 一键接入，可作为「研究笔记工程化」的参考流程。

## 相关实体

- [[entities/10万skill-背后腾讯skillhub如何帮用户找到真正好用的那20|Skill 生态]]、[[concepts/claude-code-best-practices-prompt-engineering|Claude Code prompt engineering]]：同域背景——这是「Agent Skill 应用于科研写作流程」的实例。
- [[entities/claude-code-security-review-bias-brainoverflow-2026-06|Claude Code 安全审查偏差]]：同属 LLM 评估者偏差现象族（model anchoring bias）。
- [[entities/anthropic-reward-hacking-hacker-opus-hugging-face-2026-09|Anthropic Reward Hacking]]：模型过拟合表面指标的对偶视角。
- [[entities/agent-skill-writing-guide|Agent Skill 写作指南]]：本项目即「研究笔记 → Skill」蒸馏流程的实例。
- [[entities/claude-code-academic-literature-review-sci|Claude Code 学术文献综述]]：Agent 应用于科研工作流的相邻场景。

→ [[raw/articles/重生之我在iclr连中八篇专防ai审稿错杀的skill来了|原文存档]]
