---
title: "Every Frame Perfect"
created: 2026-06-16
updated: 2026-09-05
type: entity
tags: [article, newsletter]
source_url: "https://tonsky.me/blog/every-frame-perfect/"
sources: [raw/articles/tonsky-every-frame-perfect]
review_value: 7
review_confidence: 7
review_recommendation: strong
review_stars: 4
---

# Every Frame Perfect

> Source: [[raw/articles/tonsky-every-frame-perfect|原文存档]] ^[raw/articles/tonsky-every-frame-perfect.md]

## 核心要点

- **来源**: https://tonsky.me/blog/every-frame-perfect/
- **评分**: v=7, c=7, v×c=49, stars=4
- **评估理由**: Well-structured opinion piece on UI animation quality, drawing an insightful parallel between Wayland's technical goal and UI design. Provides a memorable heuristic ('every frame should be explainable') backed by concrete, well-chosen examples of common animation flaws (jank, desync, false transitio

## 内容提炼


A while ago I was reading about Wayland and this quote stuck with me: ^[raw/articles/tonsky-every-frame-perfect.md]

> A stated goal of Wayland is “[every frame is perfect](https://wayland-book.com/protocol-design/design-patterns.html)”.

And I think this is a goal we should all aspire to. Wayland is talking about the technical side of things (modern GPU stacks are very complex and Wayland is trying to take control back) but it could be applied to UI too. ^[raw/articles/tonsky-every-frame-perfect.md]

The rule of thumb is: ^[raw/articles/tonsky-every-frame-perfect.md]

If I take a screenshot of your app at any moment, you should be able to explain what I see ^[raw/articles/tonsky-every-frame-perfect.md]

EDIT: This used to say “..., it must make sense” but that doesn’t account for advanced animation techniques such as smear frames etc. ^[raw/articles/tonsky-every-frame-perfect.md]

Why care about every frame? It builds trust. Users can’t see the code, so UI is the only way for them to judge the quality of the app. If UI looks good, that means developers had time to polish it, which means that they probably spent a comparable amount of time to iron out the code. It’s a heuristic, but a reasonable one. ^[raw/articles/tonsky-every-frame-perfect.md]

Now, what does it mean in practice? I can think of a few things: ^[raw/articles/tonsky-every-frame-perfect.md]

*   No white flashes between screens.
*   No partially loaded content.
*   No relayout while cont

## 关键洞察

- No white flashes between screens.
- No partially loaded content.
- No relayout while content loads.
- Internally consistent. If one part of the UI says “1 update available”, another part should not say “Checking for updates...”

## 实践启示

- 文章的核心论点可在生产环境验证
- 与现有实体的差异化角度：本文来自 tonsky.me 视角
- 引用源：[[raw/articles/tonsky-every-frame-perfect]] ^[raw/articles/tonsky-every-frame-perfect.md]
## 相关实体
- [[entities/from-doer-to-director-the-ai-mindset-shift|from doer to director: the ai mindset shift]]
- [[entities/why-internally-built-ai-fails-fund-accounting-audits|why internally-built ai fails fund accounting audits]]
- [[entities/back-up-and-restore-your-amazon-eks-cluster-resources-using-velero-amazon-web-se|back up and restore your amazon eks cluster resources using ]]


