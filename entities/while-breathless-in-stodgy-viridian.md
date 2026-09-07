---
title: "While Breathless in Stodgy Viridian"
created: 2026-05-08
updated: 2026-09-07
type: entity
tags: [rss, lm-theory, training-data, stochastic-parrot]
summary: "语言模型理论：训练语料决定模型行为，垃圾进垃圾出的思想实验"
sources: [raw/articles/while-breathless-in-stodgy-viridian]
review_value: 6
confidence: 0.7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---
## 核心洞察

### 1. Chomsky 句的真正论点：合语法性独立于语义与统计

文章以 Chomsky 的名句 "Colorless green ideas sleep furiously" 为起点：它与 "Furiously sleep ideas green colorless" 同为胡言，但任何英语母语者都只承认前一句是合语法的——**合语法性既不等同于语义，也不等同于统计频率**（在 1957 年前，两个词序的出现概率都趋近于零，唯一区别是其中一个服从英语句法）。这一区分正是后来讨论语言模型"学到的到底是什么"的理论地基。 ^[raw/articles/while-breathless-in-stodgy-viridian.md]

### 2. Jabberwocky 对照：无词义仍有可解读结构

《Jabberwocky》的生造词让诗句"因不知词义而胡言"，但读者仍能从句法轮廓读出"Jabberwock 是危险生物"的叙事弧线——**剥离全部词义后，结构本身仍携带可恢复的信息**。这与 Chomsky 句互为印证：句法骨架与词义层是可分离的两个信息通道。 ^[raw/articles/while-breathless-in-stodgy-viridian.md]

### 3. 对语言模型的引申：语料驱动的行为观

作者顺着这条线索走向"训练语料决定模型行为"的立场：模型的"理解"只能来自语料中可统计的模式，而非外部的真实世界指涉——垃圾进、垃圾出。文中那些" lavender equations snorkel wistfully "式的机器生成胡言，正是没有锚定词义层的统计系统的自然输出。 ^[raw/articles/while-breathless-in-stodgy-viridian.md]

## 与知识库的连接

- → [[entities/stochastic-parrot-language-models-and-meaning|Language Models and Meaning]]：同一议题的哲学面——LLM 缺少语言与外部世界的传统语义联结
- → [[concepts/mechanistic-interpretability|Mechanistic Interpretability]]：若语义不在语料统计里而在内部表征里，可解释性是检验"模型学到了什么"的对撞机

→ [[raw/articles/while-breathless-in-stodgy-viridian|原文存档]]
