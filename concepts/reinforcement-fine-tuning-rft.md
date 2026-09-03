---
title: "Reinforcement Fine-Tuning (RFT)"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [fine-tuning, reinforcement-learning, training, model, alignment, rft, rlaif]
sources:
  - raw/articles/aws-reinforcement-fine-tuning-llm-as-judge
summary: RFT（强化微调）是通过自动化奖励信号对齐模型的训练方法，涵盖 RLVR 和 RLAIF（LLM-as-judge）两种核心范式，以及多维度评估、上下文感知反馈等优势。
---

# Reinforcement Fine-Tuning (RFT)

> RFT = Reinforcement Fine-Tuning，通过自动化奖励信号对齐模型的训练方法

## 一、为什么需要 RFT

### 1.1 LLM 的对齐问题

Large language models (LLMs) now drive the most advanced conversational agents, creative tools, and decision-support systems. However, their raw output often contains inaccuracies, policy misalignments, or unhelpful phrasing—issues that undermine trust and limit real-world utility.

### 1.2 传统对齐方法的局限

传统方法依赖：
- **人工标注**：成本高、速度慢、难以规模化
- **规则奖励**：设计困难，难以覆盖复杂场景
- **静态评分**：无法捕捉细微差异

## 二、RFT 的两种范式

### 2.1 RLVR（Reinforcement Learning with Verifiable Rewards）

通过可验证的奖励函数评分 LLM 生成：

```python
# RLVR 示例
def verify_reward(output):
    if "correct answer" in output:
        return 1.0
    elif "wrong answer" in output:
        return 0.0
    else:
        return -1.0  # 不确定
```

**适用场景**：有明确正确答案的领域（数学、代码、事实问答）

### 2.2 RLAIF（Reinforcement Learning with AI Feedback）

使用 LLM 作为评判者评估候选回复：

```python
# RLAIF 示例
def llm_judge(response_a, response_b):
    judge_prompt = f"""
    Compare these two responses and judge which is better:
    A: {response_a}
    B: {response_b}
    
    Consider: correctness, tone, safety, relevance
    """
    return judge_model.generate(judge_prompt)
```

## 三、LLM-as-Judge 的优势

### 3.1 vs 传统规则奖励

| 维度 | 传统规则奖励 | LLM-as-Judge |
|------|-------------|--------------|
| **评分粒度** | 粗糙（数字分数、关键词匹配） | 细腻（多维度推理） |
| **上下文感知** | 有限 | 完整上下文理解 |
| **领域适应** | 需要手工设计 | 自动适应 |
| **可解释性** | 有限 | 理由（rationales） |

### 3.2 多维度评估

LLM 法官可以从多个维度评估：
- **正确性**（Correctness）
- **语气**（Tone）
- **安全性**（Safety）
- **相关性**（Relevance）
- **专业性**（Professionalism）

### 3.3 内置可解释性

LLM-as-Judge 提供 **rationales**（理由）：

```
"Response A cites peer-reviewed studies, while Response B makes 
unsupported claims. Therefore A is more credible."
```

这种可解释性：
- 提供诊断信息，加速迭代
- 直接定位失败模式
- 减少隐藏的错位

## 四、RFT 的应用场景

### 4.1 代码生成对齐

```python
# 代码任务的奖励设计
def code_reward(code_output, expected_behavior):
    # 执行测试
    if passes_tests(code_output):
        # 检查代码质量
        if is_readable(code_output):
            return high_reward
        else:
            return medium_reward  # 能跑但质量一般
    else:
        return low_reward
```

### 4.2 对话系统对齐

```python
# 对话任务的 LLM-as-judge
judge_criteria = {
    "helpfulness": 0.3,
    "safety": 0.3,
    "relevance": 0.2,
    "coherence": 0.2
}

def dialogue_judge(response, context):
    scores = {}
    for criterion, weight in judge_criteria.items():
        scores[criterion] = llm_evaluate(response, context, criterion)
    return weighted_sum(scores, weights)
```

### 4.3 专业领域对齐

医学、法律、金融等领域需要：
- 引用权威来源
- 表达不确定性
- 遵守专业规范

## 五、实践考量

### 5.1 Judge 模型选择

| 选择 | 考量 |
|------|------|
| **相同模型** | 成本低，但可能与被评估模型有相同偏差 |
| **更强模型** | 更准确，但成本更高 |
| **专门微调的 Judge** | 针对特定领域优化，但需要额外训练 |

### 5.2 避免 Judge 偏差

常见偏差：
- **位置偏差**：倾向于选择 A 或 B
- **长度偏差**：偏好更长或更短的回复
- **自我偏好**：可能偏好与自己风格相似的输出

**缓解策略**：
- 对比交换（A vs B, B vs A）
- 长度控制
- Prompt 工程

### 5.3 迭代优化

```
初始 SFT 模型 → RFT 训练 → 评估 → 反馈 → 调整奖励 → RFT 训练 → ...
```

## 相关实体

- [[entities/aws-sagemaker-sft-dpo-tool-calling]]
- [[entities/agentic-rl-token-in-token-out]]
- [[entities/alphaxiv-reinforcement-learning-for-rlms]]
- [[entities/deepseek-v4-training-methodology]]

## 六、关联实体

- [[entities/aws-reinforcement-fine-tuning-llm-as-judge]] — AWS RFT 原文
- [[concepts/llm-pretraining-vs-sft]] — 预训练与 SFT 对比
- [[concepts/catastrophic-forgetting]] — 灾难性遗忘与微调

## 关联实体

**上游依赖**:
- [[entities/aws-sagemaker-sft-dpo-tool-calling]] — 提供基础理论/方法
- [[entities/agentic-rl-token-in-token-out]] — 提供基础理论/方法
- [[entities/alphaxiv-reinforcement-learning-for-rlms]] — 提供基础理论/方法

**下游应用**:
- [[entities/deepseek-v4-training-methodology]] — 具体应用场景
- [[entities/aws-reinforcement-fine-tuning-llm-as-judge]] — 具体应用场景
- [[entities/9to5google-gemini-app-extended-thinking]] — 具体应用场景

**平行协作**:
-  — 替代/补充方案
- [[entities/emo-pretraining-mixture-of-experts-for-emergent-modularity-ai2]] — 替代/补充方案
- [[entities/thinking-machines-interaction-models-ai-cold]] — 替代/补充方案


→ [[raw/articles/aws-reinforcement-fine-tuning-llm-as-judge|原文存档]]

## 新增关联实体
- [[entities/9to5google-gemini-app-extended-thinking]]
- [[entities/emo-pretraining-mixture-of-experts-for-emergent-modularity-ai2]]
- [[entities/thinking-machines-interaction-models-ai-cold]]
- [[entities/time-series-forecasting-augmentation-methods]]

## 所属 MOC

- [[moc/llm-core-technology|Llm Core Technology]]
