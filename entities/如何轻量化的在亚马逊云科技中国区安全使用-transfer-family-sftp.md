---

title: 如何轻量化的在亚马逊云科技中国区安全使用 Transfer Family SFTP
created: 2026-07-10
updated: 2026-08-01
type: entity
tags: [fine-tuning, aws]
sources: [raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp]
review_value: 8
review_confidence: 8
review_recommendation: strong
review_stars: 4
confidence: medium
provenance_state: extracted
reviewed: 2026-09-07
review_verdict: keep
review_category: practice
---

# 如何轻量化的在亚马逊云科技中国区安全使用 Transfer Family SFTP

→ [[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp|原文存档]] ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

# 如何轻量化的在亚马逊云科技中国区安全使用 Transfer Family SFTP

摘要：在没有 Active Directory、没有自建 IDP，且中国区缺少 Cognito User Pool 的约束下，利用 AWS Transfer Family Custom Identity Provider、Secrets Manager 自动轮换密码和 IAM Roles Anywhere 证书认证，构建一套轻量化的且无长期凭据的安全 SFTP 文件传输方案。 ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

**目录**

01 一、背景与挑战

02 二、方案架构

03 三、核心设计决策

04 四、前提条件

05 五、实施步骤

06 六、端到端验证

07 七、安全分析

08 八、成本估算

09 九、生产化建议

10 十、总结

* * *

## **一、背景与挑战**

SFTP（SSH File Transfer Protocol）是企业间文件交换的常用协议。许多组织需要通过 SFTP 与合作伙伴、供应商进行安全的数据传输。[AWS Transfer Family](<https://www.amazonaws.cn/transfer-family/>) 提供了全托管的 SFTP 服务器，但在实际落地过程中，我们经常会遇到一些约束： ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

1. 没有 Active Directory：没有 AD 或 AD不由自己掌控，并且觉得单独为SFTP构建AD过重，排除了 Transfer Family 原生的 AD 集成方案。
  2. 没有自建 IDP：企业也没有 Okta、Entra ID 等第三方身份提供者。
  3. 中国区没有 Cognito User Pool：在海外区可以用 Cognito User Pool 做 M2M（client_credentials）Token 认证，但中国区只有 Cognito Identity Pool，无法签发 OAuth Token。
  4. 不想用固定凭据：传统的固定用户名+密码方案在程序对程序（M2M）场景下安全性不足，一旦泄露无法自动失效。
  5. 程序走公网访问：客户端程序部署在公网环境，需要通过 Internet 连接 SFTP 和 AWS API。 ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]



面对这些约束，本文介绍一套在中国区完全可落地的轻量化方案，通过三项服务的组合，最大程度保障 SFTP 的安全访问。 ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

## **二、方案架构**
    
    
    程序（公网部署）
     │  持有：X.509 客户端证书 + 私钥（唯一长期凭据）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │
     │ ① X.509 证书 → IAM Roles Anywhere^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     ▼
    IAM Roles Anywhere（Trust Anchor = 自签 CA）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │  返回：短时 AWS 临时凭据（AccessKey/Secret/SessionToken，1小时过期）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │
     │ ② 临时凭据 → Secrets Manager^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     ▼
    Secrets Manager（KMS 加密，每 4 小时自动轮换）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │  返回：当前有效的 SFTP 密码
     │
     │ ③ 用户名 + 密码 → SFTP
     ▼
    Transfer Family SFTP Server^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │  VPC Internet-Facing 端点 + Elastic IP（固定公网 IP 用于白名单）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │  安全组：仅允许客户端出口 IP 的 22 端口^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │  身份提供者：Custom Identity Provider（Lambda）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │      - 校验密码（同时接受 AWSCURRENT 和 AWSPREVIOUS 版本）^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     │      - 校验客户端源 IP
     │      - 返回 IAM 角色 + S3 逻辑目录映射^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

     ▼
    Amazon S3（后端存储，KMS 加密） ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

**核心组件**

组件 | 用途  
---|---  
Transfer Family | 全托管 SFTP 服务器  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

Custom Identity Provider (Lambda) | 自定义身份认证与授权  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

Secrets Manager | 存储 SFTP 密码，自动轮换  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

IAM Roles Anywhere | X.509 证书换取临时 AWS 凭据  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

Amazon EventBridge | 定时触发密码轮换  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

KMS | 加密 Secret 和 S3 数据 ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]
  
## **三、核心设计决策**

问题 | 决策 | 原因  
---|---|---  
中国区无 Cognito User Pool | 用 Secrets Manager 存密码 + Lambda 校验 | 不依赖任何外部身份服务  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

不想要固定凭据 | Secrets Manager 自动轮换密码 | 密码持续变化，泄露窗口小  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

程序侧不放长期 AWS 密钥 | IAM Roles Anywhere + X.509 证书 | 证书可短有效期+吊销，比 AccessKey 更安全  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

轮换切换瞬间不能断连 | Lambda 同时校验 AWSCURRENT + AWSPREVIOUS | 覆盖新旧密码过渡期  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

合作伙伴要固定 IP 白名单 | VPC Internet-Facing 端点 + EIP | 提供稳定的公网 IP  ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

多层纵深防御 | 安全组限源 IP + Lambda 限源 IP + KMS 加密 | 任何单点泄露不足以获取数据 ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]
  
## **四、前提条件**

1. 亚马逊云科技中国区账号（cn-north-1 或 cn-northwest-1）
  2. AWS CLI 已配置中国区 profile，本文中的profile为china
  3. Python 3.10+（测试用）
  4. OpenSSL（生成 CA 和客户端证书）
  5. Default VPC 可用（或自建 VPC） ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]



## **五、实施步骤**

### 5.1 第一步：创建基础资源

S3 桶（SFTP 后端存储）：
    
    
    aws s3api create-bucket \^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

      --bucket sftp-poc- \^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

      --create-bucket-configuration LocationConstraint=cn-north-1 \^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

      --profile china --region cn-north-1^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

    
    aws s3api put-public-access-block \^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

      --bucket sftp-poc- \^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

      --public-access-block-configuration \^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

      BlockPublicAcls=true,IgnorePublicAcls=true,Block ^[raw/articles/如何轻量化的在亚马逊云科技中国区安全使用-transfer-family-sftp.md]

---
## 关联
- 相关概念: [[concepts/harness-engineering-framework|Harness Engineering]]

