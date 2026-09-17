---
title: "Amazon Bedrock Managed Entitlements — 多账号模型访问治理"
created: 2026-07-01
updated: 2026-09-14
type: entity
tags: [aws, bedrock, multi-account, governance, license-manager, aws-marketplace, finops]
sources:
  - raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements
confidence: 0.80
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# Amazon Bedrock Managed Entitlements — 多账号模型访问治理

> **Background**: 本文基于 AWS 官方博客，介绍 Managed Entitlements for Amazon Bedrock 的功能设计、适用场景和部署流程。

## 核心问题

管理跨数十或数百个 AWS 账号的 AI 模型访问权限面临两难选择：要么广泛授予 AWS Marketplace 权限（治理风险），要么在每个账号手动启用订阅（运营开销）。对于使用 Anthropic Claude、Cohere 等第三方 AWS Marketplace 模型的组织，这一运营开销显著拖慢 AI 采用速度。 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

## Managed Entitlements 方案

Managed Entitlements for Amazon Bedrock 允许从一个中央账号订阅一次，然后通过 AWS License Manager 将模型访问权限分发到整个组织。工作账号无需 AWS Marketplace 权限。 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

### 模型分类

| 模型类别 | 示例 | 访问方式 |
|---------|------|---------|
| Amazon 自有模型 | Amazon Nova | 直接可用，仅需 Bedrock 权限 |
| Amazon 代售模型 | Meta, Mistral, DeepSeek | 直接可用，仅需 Bedrock 权限 |
| AWS Marketplace 模型 | Anthropic Claude, Cohere, Stability AI | 需 AWS Marketplace 订阅 |

Managed Entitlements 仅适用于第三类（AWS Marketplace 模型）。 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

### 四步部署流程

1. **前提条件**：启用 AWS Organizations（全功能）、管理账号访问权限、开通 AWS Marketplace 订阅 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]
2. **创建授权**：在管理账号的 License Manager 中创建 Managed Entitlement，指定模型和授权账号列表 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]
3. **成员账号接受**：成员账号在 Bedrock 控制台接受授权 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]
4. **验证**：确认成员账号可调用模型 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

### 关键考虑

- **Private Offer 定价**：Private Offer 定价绑定到订阅账号。如果管理账号订阅后分发，所有成员账号使用同一 Private Offer 定价 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]
- **区域行为**：授权按区域管理，需在每个使用区域分别创建 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]
- **不适用场景**：仅使用 Amazon/partner 模型（Nova/Llama/Mistral/DeepSeek）、单账号运营、或各账号自行管理订阅的场景 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

## 深度分析

### 治理与运营的结构性取舍

跨账号模型访问的矛盾根源在于：AWS Marketplace 订阅是账号级资源，而组织的治理诉求是集中式的。当账号数量从个位数增长到数十、上百时，两条传统路径同时失效——把 `aws-marketplace:Subscribe` 权限下放到每个工作账号，等于把采购与订阅权交给每个团队，既扩大了 Marketplace 的攻击面，也让成本归因与合规审计变得不可控；由中央团队逐个账号手动启用订阅，则把工程时间消耗在重复的机械操作上，成为 AI 采用率的实际瓶颈。Managed Entitlements 并不是在这两者之间取平衡，而是把「订阅」与「使用」拆成两个独立动作：订阅权收敛到管理账号，使用权通过 License Manager 的 grant 分发出去。这实际上重新划定了安全边界——从「谁拥有 Marketplace 权限」变成「谁被授予了许可证」，使工作账号在不持有任何采购权限的前提下仍能调用模型。 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

### 授权即分发：License Manager 作为控制平面

这套设计真正值得关注的是控制平面与访问面的分离。AWS License Manager 承担控制平面：管理账号订阅第三方模型（或接受 Private Offer）后，Marketplace 自动创建 license，license 代表整个组织对该模型的使用权；随后通过 grant 把这份权利共享给 OU、整个组织或指定账号，一份 license 可对应多个 grant。工作账号只感知 grant、不感知订阅，因此完全不需要 Marketplace IAM 权限。这个模型恰好解释了原文的三类划分：Amazon 自有模型（Nova）与 Amazon 代售模型（Llama、Mistral、DeepSeek）的访问面是 IAM 与 Bedrock 自身权限，根本不经过许可证平面；只有 Marketplace 模型（Claude、Cohere、Stability AI）才落在 License Manager 这条链路上，需要订阅→license→grant 三层对象。换言之，这不是通用的模型访问管理，而是专为许可证平面打的治理补丁——理解这点才能避免把它误用到本不需要许可证的模型上。 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

### 二阶后果：定价集中、区域重复与边界条件

集中订阅在带来便利的同时把风险一并集中。Private Offer 的定价、付款条款与支持协议绑定到订阅账号，一旦由管理账号统一接受，所有成员账号的用量都会流入这份协议——这既是原文强调的「跨账号一致费率与简化成本分摊」的收益，也意味着议价失误、承诺用量与实际用量错配的后果由整个组织共同承担。其次，license 在 us-east-1 创建、grant 也通过 us-east-1 端点管理，但授权按区域生效，多区域部署需要在每个使用区域重复创建 grant，把「一次订阅」变成了「一次订阅 + N 次区域配置」。最后是边界条件：单账号运营、各团队自行管理订阅、以及只使用 Amazon 自有或 Amazon 代售模型的组织都不需要该功能；而已有订阅的账号在被分发同一模型 grant 时，旧订阅的 entitlement 会被禁用并迁移到新 grant 上——这类隐性状态变更必须提前纳入变更管理与通知流程。 ^[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements.md]

## 实践启示

1. **先做账号清单 × 模型清单的比对**：把「哪些账号在用哪些模型」按三类（Amazon 自有 / Amazon 代售 / Marketplace）分类，只有依赖 Marketplace 订阅的部分才纳入 Managed Entitlements 范围，避免为不需要许可证的模型引入多余的治理层。
2. **前置条件逐项核验**：AWS Organizations 全功能模式、管理账号的 Marketplace 与 License Manager 权限、以及 License Manager / Marketplace 两个 service-linked role，缺任何一项流程都走不通，建议做成部署前的 checklist。
3. **为区域复制预留成本**：license 与 grant 都通过 us-east-1 端点管理，但授权按区域生效，多区域规划时必须把「每个使用区域重复创建 grant」计入部署工时；跨境与合规约束参见 [[entities/amazon-bedrock-cross-region-inference-cris-eu-gdpr|跨区域推理与合规]]。
4. **用 Cost Allocation Tags 做成本回摊**：模型用量统一计费在管理账号，必须按成员账号或团队打标签分摊，否则 FinOps 视角下所有 AI 成本会变成一笔无法归因的中央账单；标签体系可参考 [[concepts/ai-cost-optimization-framework|AI 成本优化框架]]。
5. **分阶段 rollout 而非一步到位**：先用 grant 只覆盖 pilot 账号验证成本、配额与合规，再把 target 扩展到 OU 或整个组织；整组织分发时新账号会自动继承访问，且 grant 默认处于 Disabled、需账号管理员显式激活——把这个「最后控制点」写进推广与安全评审流程。
6. **建立监控与撤销 SOP**：持续监控 grant 的激活状态与用量分布；退役模型时要记住取消订阅不会自动删除 grant，必须单独删除 grant 才能终止 private pricing，同时确认成员账号不再按私有费率计费。

## 相关实体

- [[entities/aws-budget-bedrock-cost-governance|AWS Budget Bedrock 成本治理]] — Bedrock 用量监控与预算告警
- [[entities/amazon-bedrock-application-inference-profile-per-bu-cost-alert|Bedrock Inference Profile 成本告警]] — 按业务单元追踪 Bedrock 成本
- [[entities/aws-devops-agent-mcp-china-partition-bridge|AWS DevOps Agent MCP 中国区桥接]] — 多账号场景的 DevOps Agent 部署

→ [[raw/articles/simplify-multi-account-access-to-amazon-bedrock-models-with-managed-entitlements|原文存档]]
