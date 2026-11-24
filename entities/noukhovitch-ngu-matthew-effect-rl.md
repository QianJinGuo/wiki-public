---
title: "Learning to Solve Hard Problems in RL for LLMs by Never Give Up"
created: 2026-09-22
updated: 2026-11-09
type: entity
tags: [rl, post-training, evaluation, llm]
sources: [raw/articles/noukhovitch-ngu-matthew-effect-rl]
confidence: 0.7
---

# Learning to Solve Hard Problems in RL for LLMs by Never Give Up

## 摘要

RL post-training 的 Matthew Effect：增益集中在模型已能解决的简单问题上，pass@32=0 的难题保持 0——难度分层 eval 分析 + Never Give Up 方法论（配 arxiv + github）^[raw/articles/noukhovitch-ngu-matthew-effect-rl.md]

## 核心要点

- v*c=64（value=8, confidence=8, stars=4），newsletter ingest 2026-09-22
- 详细分析见原文存档

相关：[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]

→ [[raw/articles/noukhovitch-ngu-matthew-effect-rl|原文存档]]

## 深度分析

### The Matthew Effect: RL gains concentrate where the model is already competent

Noukhovitch's AIME 2025 eval curve for Olmo 3.1 7B RL-Zero Math training looked like textbook success — until the 30 questions were split into difficulty tiers by the *initial* model's performance (pass@32=0 → "hard"; the rest split evenly into "medium" and "easy" by pass-rate, starting at 0% / 3.8% / 22.7% pass@1). The aggregate gain was almost entirely the easy tier going from "somewhat solved" to "mostly solved"; the hard tier, where the base model could not produce a single correct answer in 32 samples, stayed pinned at pass@32=0 through training. A per-example heatmap ordered by initial difficulty makes it visceral: the top rows barely move. The paper names this the **Matthew Effect** ("the rich get richer"), borrowed from network science and economics, and states it as: RL improves performance on a task in proportion to the model's initial competence — easy tasks get easier while hard tasks often stay hard.

Crucially, this is not a math-RL quirk. The same difficulty-stratified analysis on code RL (DeepCoder on LCBv5, bucketed by the initial DeepSeek-R1-Distilled-Qwen-14B pass-rates) and agentic RL (DeepSWE on SWEBench, bucketed by task length/difficulty labels) shows the same pattern: RL gains are roughly proportional to how easy the problems were to begin with.

### Why larger k backfires: the real cause is wasted compute on easy problems, not undersampling

The obvious suspect is GRPO's zero-gradient property: if none of the k sampled completions for a prompt is correct, there is no advantage signal and no learning. The obvious fix — crank k up — turns out to be wrong. Training Qwen 2.5 0.5B Instruct with GRPO on GSM8k Platinum (difficulty buckets: easy 25%, medium 10%, hard 5%, extra-hard 0% initial pass@1, with k ∈ {4,8,16,32} at fixed batch size), **smaller k solves more hard problems than larger k**: n=64 prompts × k=4 beats n=8 × k=32.

The mechanism is in the training-batch composition. GRPO filters out prompts whose completions are all-correct or all-wrong, so the batch only contains mixed-outcome prompts. Larger k does find rare correct solutions for hard prompts — but it *also* finds rare incorrect solutions for easy prompts, keeping already-solved problems in the batch far longer (an easy problem exits the batch only when solved 32/32, versus 4/4 under k=4). The batch composition plot shows an inflection point around step 200: before it, k=32 trains on more hard problems; after it, the compute k=32 keeps burning on easy problems dominates, and k=4 overtakes on the hard tier. The paper's framing: the Matthew Effect is not just *signal loss* on hard problems (zero gradient) but poor **signal efficiency** — too much compute spent re-training competence the model already has. That reframing matters because it means the fix is compute reallocation, not just sampling harder.

### Never Give Up: geometric-distribution sampling as an implicit adaptive curriculum

NGU's goal is small-k efficiency on easy problems with large-k persistence on hard ones, implemented via asynchronous RL sampling (building on the authors' async RLHF line of work). The recipe: sample k completions. If the prompt is solved within the first k, train on it. If it's fully solved k/k, filter it out as mastered. The key branch: if *all* k completions are wrong, with probability p the prompt is re-queued for another k samples instead of being dropped — the model "never gives up." Old completions are kept, and once the problem is finally solved, training uses all k × rounds-of-NGU completions collected. The number of samples drawn follows a geometric distribution with expectation k/(1-p).

On GSM8k Platinum, NGU with k=4 and p=0.9 outperforms every fixed-k GRPO configuration at equal compute, with the margin largest exactly on the hardest subset — it matches large-k's early-training hard-problem solves and small-k's late-training efficiency simultaneously. The paper also draws a sharp contrast with curriculum learning: curricula pre-set problem difficulty offline, but difficulty is *nonstationary* during training (easy problems can harden relative to the batch, and vice versa). NGU's re-queuing rule is implicitly adaptive — it discovers which prompts deserve more compute from the live outcome distribution, which is why online adaptive scheduling beats a pre-set curriculum.

### The eval lesson: aggregate metrics hide distributional failure

Methodologically, the deepest takeaway predates the training fix: the rising AIME curve was *real* — the model genuinely improved — and it still concealed near-total stagnation on the hard tier. Bucketing eval questions by the initial model's pass-rate (a zero-cost relabeling using only the pre-RL checkpoint) is the instrument that converted an invisible failure mode into a measurable, actionable one. Any RLVR practitioner reporting a single aggregate eval number is vulnerable to the same illusion; the difficulty histogram of gains is the real training report.

## 实践启示

1. **Stratify every RL eval by initial difficulty before trusting the curve.** Relabel benchmark questions using the pre-RL checkpoint (pass@32=0 → hard; split the rest by pass@1). If your gains live only in the easy bucket, the Matthew Effect is active and the aggregate number is misleading.
2. **Do not reach for larger k to unlock hard problems.** At fixed compute, small k (k=4) beats large k (k=32) on hard subsets over a full training run, because large k traps easy problems in the batch via rare mistakes.
3. **Filter mastered problems aggressively.** A prompt solved k/k is pure compute waste; with small k it exits the batch almost immediately, and the freed generator capacity flows to harder prompts ("signal efficiency").
4. **Re-queue all-wrong prompts instead of dropping them.** An NGU-style rule — resample with probability p≈0.9, accumulate completions across rounds, train on everything once solved — gives hard prompts large-k persistence at small-k cost (expected samples k/(1-p)).
5. **Prefer online adaptive difficulty mechanisms over pre-set curricula.** Problem difficulty is nonstationary during training; NGU's implicit adaptation from live outcomes beat offline curriculum scheduling in the paper's comparisons.
6. **Audit domains beyond math with the same lens.** The effect was replicated on code RL (LCBv5/DeepCoder) and agentic RL (SWEBench/DeepSWE) — difficulty-bucketed reporting should be standard for any [[concepts/rlvr-reinforcement-learning-verified-reasoning|RLVR]] run, including agentic training stacks like [[entities/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026]].

相关概念：[[concepts/grpo-policy-optimization-2026]]、[[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]；训练侧同类问题见 [[entities/overcoming-reward-signal-challenges-verifiable-rewards-based-reinforcement-learn]]，评测基础设施见 [[entities/olmo-eval]]。
