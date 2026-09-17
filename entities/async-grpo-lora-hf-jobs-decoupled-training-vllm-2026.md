---
title: "Async GRPO with LoRA across HF Jobs：用 Storage Bucket 替代 NCCL 的解耦式异步 RL 训练"
created: 2026-09-14
updated: 2026-09-14
type: entity
tags: [post-training, grpo, lora, rl, vllm, inference, distributed-training, decoupled-training, huggingface, trl]
confidence: 0.75
provenance_state: extracted
sources: [raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026]
---

# Async GRPO with LoRA across HF Jobs：用 Storage Bucket 替代 NCCL 的解耦式异步 RL 训练

Hugging Face 团队在 TRL v1.14 为 `AsyncGRPOTrainer` 加了 LoRA 支持（PR #7017），并给出一个把**训练进程与推理进程彻底拆到不同机器**的参考实现：trainer 只把 LoRA adapter 同步给 vLLM，adapter 通过挂载在每个 Job 上的 Storage Bucket 传递，而不是走 NCCL。同一套配方五轮跑下来，500 步的训练时间从 3 小时 27 分压到 53 分钟。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

## 为什么 LoRA 让异步 RL 变得可行

异步 GRPO 的瓶颈在权重同步：如果每次更新都把完整策略发给推理 worker，每次同步都要搬运数 GB，这正是一般集群里 NCCL 存在的理由。LoRA 改变了这个算术——对 1.5B 模型，rank-1 adapter 只有几 MB，而完整模型约 3 GB。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

文章援引 Thinking Machines 的 *LoRA Without Regret*：LoRA 在 policy-gradient RL 上可以匹配全量微调，即使 rank=1。理由是优势函数每个 episode 只提供约 `O(1)` bit 的信息量，从总信息量角度看每一步没有那么多东西可学，rank-1 adapter 的容量足够吸收。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

系统层的后果同样重要：vLLM 可以同时加载多个 adapter，旧 rollout 用它开始时的策略跑完，新 rollout 用最新版本，天然形成策略版本的时间窗口。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

## 架构：三个 Job + 一个 Bucket + 一个 Proxy

`AsyncGRPOTrainer` 的 adapter-only 同步路径是：trainer 不向 vLLM 发送张量；每隔若干优化步，它把 adapter 存到 `<output_dir>/.vllm_lora/trl-policy-v{N}`，用原子 rename 发布目录，然后把该路径 POST 给 vLLM 的 `/v1/load_lora_adapter`；vLLM 从磁盘加载文件，rollout worker 之后就可以请求 `model="trl-policy-v{N}"`。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

这个端点接受的是路径而不是张量，因此 trainer 与 server 本来就需要共享文件系统——Slurm 集群上是网络文件系统，而在 HF Jobs 上通过把 Storage Bucket 挂载成卷来获得同样效果（底层是 `hf-mount` 把 bucket 暴露成容器内的 POSIX 文件系统）：

```sh
# every Job gets the same bucket at the same absolute path
hf jobs run ... -v hf://buckets/aminediroHF/asyncgrpo-lora-buckets:/lora ...
```

TRL 与 vLLM 都不需要为此改动：trainer 写 `/lora/<run>/.vllm_lora/`，server 从同一路径读，POST 里的路径在每个容器内都已合法。checkpoint 与最终 adapter 也存在 bucket 里，因此预抢占后的 trainer 可以续训。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

三个 Job 的分工：^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

- **trainer Job**：跑 `AsyncGRPOTrainer` + LoRA（外加 FSDP）；
- **两个 vLLM replica Job**：用官方 `vllm/vllm-openai` 镜像，只需开启 runtime LoRA 加载并留足 adapter 槽位；
- **Storage Bucket**：三个 Job 挂在同一路径，adapter 由此从 trainer 走到 server；
- **proxy server**：把每个 rollout 路由到最可能持有其 KV cache 的 replica，并把每次 adapter 更新广播到所有 replica。

## max_staleness 决定 adapter 槽位数

`AsyncGRPOTrainer` 里每次权重同步把 policy version 加一，`max_staleness` 是一个 rollout 样本在被丢弃前允许落后当前策略的版本数。若 `max_staleness=4`，在 `trl-policy-v3` 下生成的样本仍在 trainer 处于 `v7` 时用于训练；而一个在 `v3` 下开始的 rollout 必须能在 `v3` 下跑完。因此 vLLM 必须同时服务当前策略加上之前四个版本：trainer 注册 `max_staleness + 1` 个 adapter 版本并卸载更旧的，每次同步先加载新版本再卸载最旧版本，swap 期间需要额外一个槽位——于是 `--max-loras 6`。若只给 5 个，vLLM 会在每次同步时静默驱逐一个仍有 rollout 在飞的策略。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

## 为什么 adapter 必须带版本号

另一种设计是 trainer 只保留最新 adapter 并始终用同一个名字发布，文章明确否定了这条路：vLLM 的 prefix cache 以 adapter 名为键。单名之下，用旧权重算出的 KV block 在 swap 后仍然匹配，prefill 不会被重做，于是某个 rollout 可能前缀来自一个策略版本、decode 来自下一个版本。trainer 无从察觉，只会在指标上表现为 `ratio` 偏离 1。带版本号的名字让这件事不可能发生——一个名字永远只对应一组权重，缓存的 prefix 永远无法匹配更新的版本。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

实现上 vLLM 固定在 `v0.27.1`：上面这些 flag 与 runtime LoRA 端点正是该版本暴露的，因此版本号属于配方的一部分。replica 启动时开 `--enable-lora --max-lora-rank 1 --max-loras 6`，环境变量 `VLLM_ALLOW_RUNTIME_LORA_UPDATING=1` 打开 `/v1/load_lora_adapter`，`VLLM_SERVER_DEV_MODE=1` 打开 `/pause`、`/resume`、`/server_info`（TRL 三个都要）。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

## 数据集选型：把「静默失败」变成曲线上的可见信号

端到端验证用的是 `sail/Sanity-Test-R1D-1.5B`——来自 *Defeating the Training-Inference Mismatch via FP16*（Qi et al., 2025），复现代码在 `sail-sg/Precision-RL`。原始数据用 DeepSeek-R1-Distill-Qwen-1.5B 对每道 MATH 题生成 40 个回答，保留成功率在 20%–80% 之间的题，得到 1,460 道。作者选它的理由很工程化：题目既非已解也非无望，模型能拿到早期信号；规模足够小，两小时内可以跑完一轮；而且它是一个稳健的端到端测试——如果某个 vLLM replica 悄悄用基础模型顶替 adapter 名字，曲线会在几十步内暴露。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

超参直接取自该论文的 LoRA 脚本：`Qwen/Qwen2.5-Math-1.5B`，LoRA rank 1、alpha 2，学习率 4e-5，每 prompt 8 个样本，每步 128 个 completion，最多生成 3000 token，上下文 4096。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

## 实测：同配方 3h27m → 53min

`AsyncGRPO` 的指标显示瓶颈究竟在哪：同一套配方跑五次，500 步的耗时从 3 小时 27 分降到 53 分钟。^[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026.md]

## 要点

- **adapter-only sync** 把「权重同步」的搬运量从 GB 级降到 MB 级，是异步 RL 能跨机器跑的前提；
- **Storage Bucket 当共享 FS** 是对 NCCL 的工程替代——牺牲跨节点通信，换取 Job 级别的部署自由（Job 之间不共享 localhost、没有共享本地盘，也无法跨节点通信）；
- **版本化 adapter 名 + max_staleness+2 槽位** 是正确性约束，不是性能调优；
- **proxy** 承担两件事：按 KV prefix 亲和路由 rollout、广播 adapter 更新。

## 相关

- [[entities/agentic-rl-frameworks-practices-long-horizon-wolfe-2026]]
- [[entities/agent-lightning-v1-harnessed-agentic-rl-arxiv-2608-17528]]
- [[entities/parpo-personalized-agentic-rl-ustc-alibaba-2026]]
- [[concepts/grpo-policy-optimization-2026]]
- [[concepts/llm-rl-algorithms-ppo-dpo-grpo-marl-evolution-2026]]
- [[concepts/inference-optimization]]

→ [[raw/articles/async-grpo-lora-hf-jobs-decoupled-training-vllm-2026|原文存档]]
