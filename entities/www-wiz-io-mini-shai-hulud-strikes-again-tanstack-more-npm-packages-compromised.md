---

title: "Mini Shai-Hulud Strikes Again: TanStack + more npm Packages Compromised"
type: entity
tags: [security, supply-chain, ai, cloud, npm, shai-hulud]
source: "[[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised|原文存档]]"
created: 2026-05-20
updated: 2026-09-07
sources: [raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised]
review_value: 8
review_confidence: 7
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

## 核心要点
- 2026年5月11日，TeamPCP 发动协同供应链攻击，同时入侵 npm 和 PyPI 生态系统
- 受影响命名空间：@tanstack（含 @tanstack/react-router，1200万周下载量）、@uipath、@mistralai
- 攻击通过劫持 CI/CD 发布系统和 OpenID Connect 令牌分发恶意包更新
- 5月13日分析发现 @uipath/ 和 @mistralai/ 包中的 payload 存在 bug 导致恶意功能失效

## 深度分析
Mini Shai-Hulud 攻击标志着 AI 开源生态系统的供应链攻击进入新阶段。与传统恶意包主要针对开发者工作站不同，此次攻击的深度和广度表明攻击者正在将目标扩展到更广泛的企业基础设施。 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]

**攻击技术特征**： ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]

- 利用 CI/CD 发布系统的合法发布机制分发恶意更新
- 劫持 OpenID Connect 令牌绕过身份验证
- 跨平台攻击（npm + PyPI 同时进行）
- 多命名空间协同攻击（TanStack、UiPath、Mistral AI） ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]

**受影响包的重要性**： ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]

- `@tanstack/react-router` 是 React 生态最流行的路由库之一
- `@mistralai/mistralai` 是 Mistral AI 官方 TypeScript 客户端
- `@uipath/apollo-core` 服务于企业自动化平台

**payload 技术细节**：恶意代码在包更新时执行，搜索 AWS/GCP/Azure/GitHub 凭据，有效凭据用于发布更多恶意包更新，无效凭据时执行本地环境擦除。 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]

## 实践启示
1. **供应链审计**：重新审视对 npm/PyPI 生态的依赖深度，实施依赖锁定策略 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]
2. **CI/CD 安全**：加强 OpenID Connect 令牌的颁发和轮换机制防护 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]
3. **包验证**：对关键依赖包实施签名验证，而非仅依赖包管理器的基本检查 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]
4. **内部镜像**：建立内部包镜像进行安全扫描，而非直接依赖官方仓库 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]
5. **凭据管理**：最小权限原则 + 定期轮换 + MFA，保护 CI/CD 环境中的所有凭据 ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]
## 相关实体
- [[entities/postmortem-tanstack-npm-supply-chain-compromise-tanstack-blog]]
- [[entities/npm-supply-chain-compromise-postmortem]]
- [[entities/teampcp-claims-sale-of-mistral-ai-repositories-amid-mini-shai-hulud-attack-1]]
- [[entities/cybersecurityreach-revoke-token-wipe-computer]]
- [[entities/thehackernews-fake-openai-privacy-filter]]

→ [[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised|原文存档]] ^[raw/articles/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised.md]
