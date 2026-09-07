---

title: "CMU Language Models Need Sleep (arxiv 2605.26099)：SSM-Attention 睡眠巩固机制"
created: 2026-06-10
updated: 2026-09-07
tags: [agent, architecture, evaluation, fine-tuning, game, knowledge-mgmt, llm, memory, mlops, rag, search]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# CMU Language Models Need Sleep (arxiv 2605.26099)：SSM-Attention 睡眠巩固机制

→ [[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu|原文存档]] ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]

## 深度分析

CMU Language Models Need Sleep (arxiv 2605.26099)：SSM-Attention 睡眠巩固机制 ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
### 核心观点
1. # CMU Language Models Need Sleep (arxiv 2605. ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
2. 26099)：SSM-Attention 睡眠巩固机制 ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
> 来源：机器之心编辑部 · CMU + 马里兰大学
> 论文地址：https://arxiv.
3. 26099 ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
很长一段时间，「长上下文」一直是各大模型厂商军备竞赛的焦点，从 128K 到 1M，再到更长的上下文窗口，业界已然形成一个固有认知，只要窗口足够大，模型就能记住更多内容，也就能处理更长、更复杂的任务。 ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
4. 但问题也随之而来：上下文越长，KV Cache 越臃肿，不仅导致显存瞬间被「吃光」，推理速度愈发缓慢，成本也迅速上升。 ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
5. 更关键的是，把更多 token 放进窗口，并不等于模型真的把这些信息转化成了可推理的长期记忆，结果是，榜单分数越刷越高，可在一些需要「深度脑暴」的复杂推理任务中，模型常常因为「记不住细节」，频频翻车…… ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
面对这一两难问题，近日，卡内基梅隆大学（CMU）联合马里兰大学等在一篇新论文中提出了有意思的视角： ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]
既然人类连续工作久了会变笨，大模型也一样，既然如此为什么不让 LLM 睡一觉呢？ ^[raw/articles/arxiv-2605-26099-ssm-attention-sleep-consolidation-cmu.md]

### 关联实体

- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]
- [[entities/karpathy-最新访谈从-vibe-coding-到-agentic-engineering]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏]]
- [[entities/ethan-he-cosmos-grok-imagine-latent-space-video-agent-20260606]]
- [[entities/karpathy-vibe-coding-agentic-engineering]]
- [[entities/agentops-operationalize-agentic-ai-at-scale-with-amazon-bedr]]

## 相关实体

- [[moc/mlops-training-inference|MOC]]
