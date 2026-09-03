---

title: "Teaching Everyone to Fish for Tokens：开源 AI 生态的 token 经济学"
type: entity
created: 2026-08-30
updated: 2026-08-30
tags: [open-source, llm, inference, economics, nvidia, meta, post-training, ecosystem]
sources:
  - raw/articles/teaching-everyone-to-fish-for-tokens
confidence: 0.8
---

# Teaching Everyone to Fish for Tokens：开源 AI 生态的 token 经济学

Nathan Lambert（Interconnects AI）分析了开源 AI 生态的经济可持续性问题。核心观点：开源模型生态正面临**存在性窗口期**——如果 Nvidia 的开源投资不能在几年内产生回报，或者没有其他开源模型公司建立起平台级财务反馈循环，开源生态将被迫转向效率、可修改性、专业化等差异化路径。^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

## 两种未来

### 未来一：开源"奏效"
Nvidia 投入 260 亿美元推动开源模型，目标是创造远超模型构建成本的芯片需求。如果成功，将形成"教所有人制造 token 机器"的生态，智能不会被垄断。关键假设：
- 开源 recipe 对 Nvidia 有效
- 推理需求覆盖训练成本
- 社区贡献改进数据/训练代码 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

### 未来二：开源分叉
如果财务正循环无法建立，开源模型将与前沿闭源模型分叉：
- **开源方向**：效率、可修改性、专业化（企业私有数据、重复业务任务）
- **闭源方向**：知识工作协作、药物发现、软件工程等高价值领域 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

Lambert 认为这是**最可能的结果**——开源模型仍有用，但在长尾生态系统中填补空白。

## Post-Training 的演变

当前开源生态由**后训练**（post-training）兴趣爆发推动：开发者使用 DeepSeek V4 Flash、Inkling Small、GLM 5.X 等模型进行任务特定微调（如 Tinker 平台）。 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

但训练正在变得**更复杂、更抽象化**：
- 基础模型训练的经济门槛持续提高
- 能够训练基础模型的开源构建者数量在减少
- 开源模型开始实验**收入分成许可证**（如 Kimi K3、Qwen3.8） ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

Lambert 预测训练 lexicon 将演变：`pretraining → reasoning training → post-training` ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

## Nvidia vs Meta 的不同策略

| | Nvidia | Meta |
|---|---|---|
| **策略** | 教所有人制造 token 机器 | 用开放权重冲垮竞争对手 |
| **目标** | 创造芯片需求 | 削弱 Anthropic/OpenAI 的 token 销售收入 |
| **方式** | 开源训练数据+代码 | 开放模型权重（如 Muse Spark 1.2） |
| **经济逻辑** | 推理需求 → 芯片销售 | 商品化互补品 → 竞争对手收入下降 |

两者都在**商品化自己的互补品**，但方式不同。

## 关键信号

1. **退出者**：Databricks、01.ai 等公司退出基础模型训练
2. **许可证实验**：Kimi K3、Qwen3.8 的收入分成模式
3. **后训练繁荣**：微调 API（Tinker）的流行
4. **Nvidia 投资规模**：260 亿美元的开源赌注 ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]

→ [[raw/articles/teaching-everyone-to-fish-for-tokens|原文存档]] ^[raw/articles/teaching-everyone-to-fish-for-tokens.md]