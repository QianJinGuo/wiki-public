---
title: "Prompt Engineering Fundamentals"
created: 2026-05-21
updated: 2026-08-01
type: concept
tags: [prompt, prompting, llm, engineering, instruction, context]
sources:
  - raw/articles/ai-coding-入门指南-如何更好地让ai真正帮你干活-v2
summary: Prompt 工程基础，涵盖 prompt 结构设计、指令工程、上下文构建、Few-shot 示例等核心技法，以及在不同场景下的最佳实践。
---

# Prompt Engineering Fundamentals

> Prompt 工程是通过设计输入来引导 LLM 产生期望输出的技术和艺术。

## 一、Prompt 的基本结构

### 1.1 核心组件

| 组件 | 作用 | 示例 |
|------|------|------|
| **指令** | 告诉模型做什么 | "翻译以下文本" |
| **上下文** | 提供背景信息 | "这是一封商务邮件" |
| **输入** | 具体任务内容 | 待翻译的英文 |
| **输出格式** | 期望的输出形式 | "JSON格式，包含原文和译文" |

### 1.2 简单 vs 复杂 Prompt

**简单 Prompt**：
```
翻译：Hello, how are you?
```

**复杂 Prompt**：
```
你是一位专业的中英翻译专家，专门负责商务邮件翻译。

任务：将以下英文邮件翻译成中文

要求：
1. 保持正式语气
2. 专业术语使用标准译法
3. 保留邮件格式

邮件内容：
Hello,

Thank you for your inquiry regarding our product specifications...

输出格式：
{
  "original": "...",
  "translated": "...",
  "notes": ["术语说明"]
}
```

## 二、指令工程（Instruction Engineering）

### 2.1 清晰具体的指令

| 原则 | 错误示例 | 正确示例 |
|------|---------|---------|
| **模糊** | "处理这个" | "提取订单号、金额、日期" |
| **过长** | 500字描述 | 3-5个要点 |
| **歧义** | "尽快" | "24小时内" |
| **否定句** | "不要有错别字" | "确保文字准确" |

### 2.2 指令优先级

```
系统提示（最高优先级）
    ↓
任务指令
    ↓
格式要求
    ↓
示例（Few-shot）
    ↓
用户输入（最低优先级）
```

### 2.3 角色扮演（Role Playing）

```python
system_prompt = """
你是一位资深软件架构师，拥有15年以上的分布式系统设计经验。
你的专长包括：
- 微服务架构
- 云原生设计
- 性能优化

回答风格：
- 结构化、层次清晰
- 用图表辅助说明
- 提供具体案例
"""
```

## 三、上下文构建

### 3.1 上下文的来源

| 来源 | 描述 | 成本 |
|------|------|------|
| **对话历史** | 之前的交互轮次 | 高 |
| **系统提示** | 固定的角色和规则 | 低 |
| **知识库** | 检索到的相关信息 | 中 |
| **用户提供** | 用户主动提供的背景 | 取决于 |

### 3.2 上下文压缩

```python
def build_context(messages, max_tokens=50_000):
    # 1. 摘要早期对话
    summarized = summarize_early(messages[:-10])
    
    # 2. 保留最近对话
    recent = messages[-10:]
    
    # 3. 添加关键事实
    key_facts = extract_key_facts(messages)
    
    # 4. 组合
    return [summarized] + recent + key_facts
```

### 3.3 上下文相关的原则

- **相关性**：只包含与当前任务相关的上下文
- **时效性**：优先使用最新信息
- **简洁性**：避免冗余和重复
- **结构化**：使用清晰的组织方式

## 四、Few-shot Learning

### 4.1 Zero-shot vs Few-shot

| 模式 | 示例数量 | 适用场景 |
|------|---------|---------|
| **Zero-shot** | 0 | 简单任务、模型能力强 |
| **Few-shot** | 1-5 | 复杂任务、需要格式规范 |
| **Many-shot** | >5 | 特殊领域、复杂模式 |

### 4.2 Few-shot 示例设计

```python
# 好的示例
examples = [
    {
        "input": "John bought 5 apples, Mary bought 3. How many?",
        "output": "8"
    },
    {
        "input": "Tom had 10 cookies, ate 4. How many left?",
        "output": "6"
    },
    {
        "input": "A store has 15 items, sold 7. Remaining?",
        "output": "8"  # 模型应该学会这个模式
    }
]

prompt = f"""
Solve the following math problems:

{examples[0]}
Q: {examples[0]['input']}
A: {examples[0]['output']}

{examples[1]['input']}
A: {examples[1]['output']}

Now solve this:
{test_input}
"""
```

### 4.3 示例选择策略

| 策略 | 方法 |
|------|------|
| **随机选择** | 随机选取 k 个示例 |
| **相似选择** | 选与输入最相似的示例 |
| **多样化选择** | 覆盖不同类型/难度 |
| **链式选择** | 用于 CoT，逐步引导 |

## 五、输出格式控制

### 5.1 常用格式

| 格式 | 适用场景 | 示例 |
|------|---------|------|
| **JSON** | 结构化数据 | API 返回、解析 |
| **Markdown** | 文档、说明 | 报告、指南 |
| **表格** | 对比、列表 | 规格对比 |
| **代码** | 编程任务 | Python、SQL |

### 5.2 格式指定技巧

```python
# 指定格式
prompt = """
提取以下文本中的关键信息。

输出格式（严格遵循）：
{
    "entity": "组织/人名",
    "action": "动词",
    "object": "受影响的对象",
    "sentiment": "positive/negative/neutral"
}

文本：...
"""

# 使用 XML 标签辅助
prompt = """
<instructions>提取订单信息</instructions>
<format>
<order>
    <order_id></order_id>
    <amount></amount>
    <date></date>
</order>
</format>
<text>...</text>
"""
```

## 六、Chain of Thought (CoT)

### 6.1 零样本 CoT

```python
prompt = """
Solve this problem step by step:

Problem: If a train travels 120km in 2 hours, what is its speed?

Let's think step by step:
1. Distance = 120km
2. Time = 2 hours
3. Speed = Distance / Time = 120 / 2 = 60 km/h

Answer: 60 km/h
"""
```

### 6.2 Few-shot CoT

```python
examples = [
    {
        "input": "Jim had 5 red balls, he bought 3 more, then gave 2 to Mike. How many?",
        "reasoning": "Step 1: Start with 5 balls. Step 2: Bought 3 more, total 8. Step 3: Gave 2 away, 6 left.",
        "output": "6"
    }
]
```

## 七、最佳实践

1. **从简单开始**：先用简单 prompt，根据需要逐步添加
2. **明确具体**：使用清晰、无歧义的语言
3. **结构化**：组织 prompt 的各个组成部分
4. **提供示例**：复杂任务使用 few-shot
5. **指定输出格式**：减少后处理成本
6. **测试迭代**：prompt 需要不断调优

## 相关实体

- [[entities/claude-code-harness-deep-dive-founder-park]]
- [[entities/别再把上下文当聊天记录]]
- [[entities/使用-agent-skills-做知识库检索能比传统-rag-效果更好吗]]
- [[entities/卡片式对话的协议方案探索和思考]]
- [[entities/agent-harness-context-management-working-set]]

## 八、关联实体

- [[concepts/context-management-agent-systems]] — 上下文管理
- [[concepts/inference-optimization]] — 推理优化
- [[concepts/llm-pretraining-vs-sft]] — LLM 预训练与微调

→ AI Coding 入门指南（已删除）

## 新增关联实体
- [[entities/evals-three-methods-of-ai-evaluation]]

## 关联实体

**上游依赖**:
- [[entities/claude-code-harness-deep-dive-founder-park]] — 提供基础理论/方法
- [[entities/别再把上下文当聊天记录]] — 提供基础理论/方法

**下游应用**:
- [[entities/使用-agent-skills-做知识库检索能比传统-rag-效果更好吗]] — 具体应用场景
- [[entities/卡片式对话的协议方案探索和思考]] — 具体应用场景

**平行协作**:
- [[entities/agent-harness-context-management-working-set]] — 替代/补充方案
- [[entities/evals-three-methods-of-ai-evaluation]] — 替代/补充方案

## 所属 MOC

- [[moc/layer-2-interaction|Layer 2 Interaction]]
