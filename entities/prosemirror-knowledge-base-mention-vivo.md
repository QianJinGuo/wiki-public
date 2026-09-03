---

title: "知识库问答 @文档：从 DOM 方案到 ProseMirror 落地"
created: 2026-06-10
updated: 2026-08-29
tags: [agent, architecture, data, k8s, rag, search]
review_value: 7
review_confidence: 7
type: entity
sources:
  - raw/articles/prosemirror-knowledge-base-mention-vivo
---

# 知识库问答 @文档：从 DOM 方案到 ProseMirror 落地

→ [[raw/articles/prosemirror-knowledge-base-mention-vivo|原文存档]] ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]

## 深度分析

知识库问答 @文档：从 DOM 方案到 ProseMirror 落地 涉及agent领域的核心技术议题。 ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
### 核心观点
1. # 知识库问答 @文档：从 DOM 方案到 ProseMirror 落地 ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
> 作者：vivo 互联网项目团队 · Ding Junjie
> 原文：https://mp.
2. com/s/7db3l9s9MfMonr0BYwyouQ ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
> 背景：知识库问答输入框的 @文档 mention 能力 —— 表面是"输入 @ 后选一个文档"，实则是编辑器稳定性的工程问题
## 一句话定位
**从 DOM 方案转向 ProseMirror** 是因为"文本 + 原子节点"混排后，复杂度会从"能不能插进去"转移到"能不能一直稳定"——光标恢复、IME、`innerHTML` 污染 undo 栈、临时交互态混入文档，每一项都让裸 `contenteditable` 不可维护。 ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
3. ## 为什么不用 DOM 方案 ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
1. ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
4. `contenteditable` 容器监听输入 ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
2. ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
5. 识别光标前的 `@query` ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]
3. ^[raw/articles/prosemirror-knowledge-base-mention-vivo.md]

### 关联实体

- [[entities/存之有序治之有矩agent-记忆系统的工程实践与演进]]
- [[entities/scale-robot-reinforcement-learning-with-nvidia-isaac-lab-on-]]
- [[entities/nvidia-isaac-lab-sagemaker-robot-rl-humanoid]]
- [[entities/qy_zacztcs1ql3bifmbmgg]]
- [[entities/天猫新品营销技术团队ai编码实战指南上-v2]]
- [[entities/openclaw-完全指南这可能是全网最新最全的系统化教程了32w字建议收藏-v2]]

