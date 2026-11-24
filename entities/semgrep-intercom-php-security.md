---


title: "Semgrep Intercom Php Security"
type: entity
sha256: f6a7d8e21941c0a5b2364021ba32e21869c90ea61513bd239e79bba273180333
date: 2026-05-08
source: newsletter
tags: [security, architecture, ai]
created: 2026-05-10
updated: 2026-11-18
review_value: 6
sources: [raw/articles/semgrep-intercom-php-security]
review_confidence: 8
score_validated: 2026-09-05
reviewed: 2026-09-07
review_verdict: keep
review_category: tech
---

# Semgrep Intercom PHP Supply Chain Attack

## 摘要

2026年4月30日，Mini Shai-Hulud 供应链攻击行动从 npm/PyPI 扩展到 PHP 生态：攻击者劫持了 Packagist 上的 `intercom/intercom-php@5.0.2`，将其改造成一个 Composer 插件，在包安装阶段下载 Bun 运行时并执行混淆的凭证窃取载荷，窃取 GitHub token、SSH key、云凭证和环境变量后加密外传至 C2（zero.masscan.cloud）。同一天官方 Node.js SDK `intercom-client@7.0.4` 也通过 preinstall 钩子被植入相同载荷——两个包合计约 40 万周下载量，且常驻于 CI/CD 与后端服务中。事件的核心教训是：Packagist "tag 即发布" 的无门禁架构加上 Composer 插件系统的安装时执行能力，使一次 GitHub 账号劫持可以在数分钟内演化为全生态的凭证收割。

## 核心要点

- 受影响包：`intercom/intercom-php@5.0.2`（Packagist，Composer 插件化投毒）、`intercom-client@7.0.4`（npm，preinstall 钩子）
- 执行时机：安装时（install-time）而非使用时——`composer install` / `npm install` 一旦拉到恶意版本即已中招，与业务代码是否 import 无关
- C2：`zero.masscan.cloud`；落盘 IOC：`src/composerPlugin.php`、`setup-intercom.sh`、`router_runtime.js`、`.claude/router_runtime.js`、`.claude/setup.mjs`、`.vscode/setup.mjs`、`results/results-*.json`、`/tmp/tmp.987654321.lock`
- 同一攻击者当日先攻 PyPI（PyTorch Lightning），此前一周攻过 SAP 的 npm 包——是跨 npm/PyPI/Packagist 三生态的连续行动

## 深度分析

### 1. "Tag 即发布"模型与 npm/PyPI 的 provenance/attestation 演化差距

Packagist 的架构与 npm/PyPI 有本质差异：它根本不存储 tarball。作者只注册一个 VCS 仓库 URL，Packagist 爬取 `composer.json`、索引 tag 并向所有 `composer require` 请求提供元数据——真正的代码始终留在 GitHub/GitLab/Bitbucket 上。发布流程是"提交仓库 URL → GitHub OAuth 认证 → 添加 webhook"，之后每一次 `git push <tag>` 都会在几分钟内触发 webhook 重新索引并全网可拉取。这意味着攻击者只要劫持维护者的 GitHub 账号并推送一个恶意 tag，就完成了对整个 PHP 生态的分发，不存在任何预发布隔离区或人工审批环节。

对比之下，npm 和 PyPI 正是在经历多轮重大投毒事件后才逐步建立防御纵深：npm 引入了 provenance attestation（基于 Sigstore 的构建来源证明）、 granular access tokens、2FA 强制与维护者账号的发布保护；PyPI 推出了 Trusted Publishers（基于 OIDC 的无 token 发布）、上传隔离与恶意包扫描流水线。Packagist 至今缺少等价机制——这次事件把它打回 npm 生态在多起重大攻击之前的"等价状态"。更深一层的问题是 Composer 的依赖解析放大效应：只要任何一个流行库传递依赖了被污染的包，所有下游项目无需显式 require 就会自动拉入恶意版本。复制传播不是攻击者的"技巧"，而是 Composer 解析逻辑的内建特性——攻击面是架构决定的，而非实现偶然。

### 2. 安装时执行 vs 使用时执行：CI/CD 成为第一受害者

传统依赖漏洞的威胁模型是"使用时"：代码存在缺陷，但只有当应用实际调用脆弱函数时才产生危害。这次攻击展现了完全不同的模型——恶意逻辑在**安装时**执行。`intercom/intercom-php` 被改造成 Composer 插件（`src/composerPlugin.php`），Composer 的插件系统允许包在安装阶段运行任意 PHP 代码；npm 侧的 `intercom-client` 则通过 preinstall 钩子运行 `setup.mjs`。两者的共同含义是：任何一台执行了 `composer install` / `npm install` 的机器——包括 CI/CD runner、开发者笔记本、部署宿主机——在拉取恶意版本的那一刻就已被攻破，与应用是否 import 该包、是否调用其 API 完全无关。

这个模型把攻击的最优目标从"应用代码"转移到了"构建基础设施"。CI/CD 服务器通常持有仓库写权限的 GitHub token、部署用的云凭证、SSH key 和大量环境变量，一台被攻破的 runner 可以直接外传整套部署钥匙链。此外，安装时执行还压缩了防御方的时间窗：从恶意版本发布到被发现并下架之间，每一次自动化安装都是一次成功的入侵，而依赖更新机器人（如 Dependabot/Renovate）恰恰会在这段时间内自动把恶意版本拉进所有启用自动更新的项目——防御机制的自动化反而成了攻击的加速器。

### 3. 开发者工具链（.claude/、.vscode/）成为独立的高价值目标类

落盘 IOC 清单中最值得警惕的是 `.claude/router_runtime.js`、`.claude/setup.mjs`、`.claude/settings.json` 和 `.vscode/setup.mjs`、`.vscode/tasks.json`——这些是 Claude Code 和 VS Code 的配置目录，与应用代码、用户主目录下的通用凭证存储都不在一个层次。攻击者在设计载荷时明确预判了目标开发环境中存在 AI 编程工具，并为此定制了持久化/劫持文件：污染 `.claude/` 意味着可以读取或篡改 AI 编程助手的配置（包括其中可能缓存的 API 密钥和 MCP 服务器凭据），污染 `.vscode/tasks.json` 则可以把恶意命令挂进开发者的日常任务执行链，实现跨包更新的持久化。

这预示着一个新的目标类别正在形成：**AI 编程工具链本身**。相比一次性收割的 GitHub token，AI 助手的会话上下文包含代码库全貌、架构讨论、凭证使用模式等高价值情报，而其 API 密钥具有长期有效、按量计费可变现的特点。Shai-Hulud 原始行动中大规模自我复制传播的"蠕虫"特征，加上这次对 `.claude/`、`.vscode/` 的定向落盘，表明攻击者已经把"开发环境的 AI 化"当作攻击面的一部分来经营——任何在开发者机器上拥有读写配置目录能力的 AI 工具，都同时是潜在的信息泄露通道和持久化驻留点。

### 4. 可达性分析作为噪音控制：从"有漏洞"到"可利用"的判定升级

Semgrep 对该事件的响应展示了依赖扫描工具的两层设计。第一层是已知漏洞：持续摄取安全公告，事件暴露 CVE 后自动生成新规则，PHP 的可达性分析覆盖 2017 年以来全部高危和严重等级 CVE。关键是可达性判定不只看 `composer.lock` 里是否存在问题版本，而是检查代码是否实际调用了有漏洞的函数，结果分为 reachable / unreachable / conditionally reachable 三档——一个存在 CVE 的依赖如果代码从未触达危险函数，就不会以严重级别告警。这种设计直接控制了告警噪音：在大型单体项目里，"版本匹配但不可达"的发现往往占绝大多数，不区分可达性会导致告警疲劳，进而让真正的可利用漏洞被淹没。

第二层是恶意包事件，与已知漏洞本质不同：没有 CVE 可等待，需要响应式的规则生成和主动检测。Semgrep 的处理也揭示了可达性分析的两条重要边界——传递依赖只做版本匹配、不跑全链路的代码级可达性分析，例外是"引入即可利用"型漏洞（无需任何函数调用）无论直接还是传递依赖都标记为 reachable，这正是供应链恶意包的判定逻辑；以及 PHP 特有的限制：不支持无 lockfile 扫描，`composer.lock` 必须存在且提交。这两条边界说明可达性分析不是普适银弹，它与生态的 lockfile 实践强耦合——lockfile 缺失时，连版本级判定都无从谈起。此外 Semgrep 的多生态联动响应（同一行动在 npm、PyPI、Packagist 三线同步出公告和规则）展示了自动化威胁情报分发在分钟级攻击窗口下的必要性。

## 实践启示

**对于 PHP 开发者：** 立即检查 `composer.lock` 和项目中是否存在 `intercom/intercom-php@5.0.2`。如果使用过受影响版本，假设凭证已泄露：立即轮换 GitHub token、SSH key、云服务商密钥，并审计 `~/.composer/` 目录是否存在异常文件。重点检查 `.claude/` 和 `.vscode/` 目录——如果发现非预期的 `setup.mjs`、`router_runtime.js` 或被篡改的 `tasks.json`/`settings.json`，立即隔离机器并按 IOC 清单（`src/composerPlugin.php`、`results/results-*.json`、C2 域名 `zero.masscan.cloud`）逐项排查，同时检查 AI 助手相关 API 密钥的用量异常。

**对于 DevSecOps 团队：** `composer.lock` 必须提交到版本控制（PHP 生态不支持无 lockfile 扫描，lockfile 是一切依赖审计的前提），并在 CI/CD 构建阶段强制依赖扫描。由于 Packagist 缺乏预发布安全门禁，锁定已知良好版本号（而非 `^5.0.2` 这类宽松范围）是防御供应链投毒的最低成本手段——它把"自动拉入未来恶意版本"的默认行为反转成"显式批准才能升级"。对高敏感项目，考虑使用私有 Packagist 镜像或 Artifactory 对上游依赖做镜像与审查，并为 CI runner 配置最小权限凭证与一次性 token，使安装时执行类攻击即使得手也拿不到长期有效的部署钥匙链。

**对于安全工具厂商：** Semgrep 的这次快速响应（多生态联动：PyPI + npm + Packagist 同步出公告与检测规则）展示了自动化威胁情报共享的价值。当同一攻击手法可以在数小时内跨三个包管理生态复制时，规则生成与分发速度成为竞争的关键维度；同时可达性分析（reachable/unreachable/conditionally reachable 三档判定）证明了"减少噪音"本身就是安全产品的核心能力——告警疲劳会吞掉真实威胁，精度即价值。

## 相关实体

- [[entities/semgrep-intercom-php-supply-chain|Semgrep Intercom PHP Supply Chain]]
- [[entities/www-wiz-io-mini-shai-hulud-strikes-again-tanstack-more-npm-packages-compromised|Mini Shai-Hulud Strikes Again: TanStack + more npm Packages Compromised]]
- [[entities/npm-supply-chain-compromise-postmortem|Postmortem: TanStack npm supply-chain compromise]]
- [[entities/teampcp-claims-sale-of-mistral-ai-repositories-amid-mini-shai-hulud-attack-1|TeamPCP Claims Sale of Mistral AI Repositories Amid Mini Shai-Hulud Attack]]
- [[entities/how-semgrep-cut-taint-analysis-time-by-75|How Semgrep Cut Taint Analysis Time by 75%]]

## 原文存档

- [[raw/articles/semgrep-intercom-php-security.md|原文存档]]
