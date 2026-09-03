---

title: "用 Amazon DynamoDB 构建无需双写的 Agent Memory 与语义检索"
created: 2026-09-01
updated: 2026-09-01
type: entity
tags: [ai, agent, harness, evaluation, memory, mcp]
sources: [raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索]
confidence: 0.75
provenance_state: extracted
---

# 用 Amazon DynamoDB 构建无需双写的 Agent Memory 与语义检索


# 用 Amazon DynamoDB 构建无需双写的 Agent Memory 与语义检索

摘要：本文用一个可运行的 Agent 长期记忆场景，说明如何把业务属性和 embedding 放进同一条 item，用一次 PutItem 完成写入，用 SearchVectors 完成语义召回；同时给出这套设计在真实项目里需要提前验证的几个点，以及它明确不适合的场景。  
  
**目录** ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

01 一、引言

02 二、双写从哪来

03 三、向量索引把这条链路收进了托管服务

04 四、数据模型：一张表承载两种读取

05 五、建表：把向量索引声明在 CreateTable 里

06 六、等索引可搜：要看两个字段

07 七、写入：一次 PutItem，业务数据和向量一起落地

08 八、插一段：文档向量和查询向量不是一回事

09 九、检索：语义召回

10 十、跑起来看效果

11 十一、实测细节：最终一致性有多久

12 十二、上生产前需要自己验证的几件事

13 十三、一个必须讲清楚的安全边界

14 十四、什么时候不该用它

15 十五、小结

* * *

## **一、引言**

[Amazon DynamoDB](<https://aws.amazon.com/cn/dynamodb/>) 现已支持原生实时向量搜索。对已经把会话状态、用户画像或 Agent 记忆放在 DynamoDB 里的团队来说，这个特性最实际的价值不是「多了一个向量库可选」，而是少了一条数据同步链路。本文用一个可运行的 Agent 长期记忆场景，说明如何把业务属性和 embedding 放进同一条 item，用一次 PutItem 完成写入，用 SearchVectors 完成语义召回；同时给出这套设计在真实项目里需要提前验证的几个点，以及它明确不适合的场景。 ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

## **二、双写从哪来**

一个带长期记忆的 Agent，通常同时需要两种读取方式：

  * 按时间读：最近几轮说了什么。这是标准的 key-value / range 查询。
  * 按语义读：三个月前用户提过的、和当前问题相关的事。这是向量检索。



在没有原生向量索引的年代，这两种读取往往落在两套存储上：事务数据在 DynamoDB，embedding 在一个独立的向量库。于是必须解决一系列问题： ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

1. 一致性：写完 DynamoDB 再写向量库，中间失败怎么办？向量库写成功但 DynamoDB 回滚了怎么办？
  2. 管道：用 DynamoDB Streams + Lambda 做异步复制，就要处理重试、乱序、幂等、[DLQ](<https://aws.amazon.com/cn/what-is/dead-letter-queue/>)、背压。
  3. 两份 schema：业务属性要在向量库里冗余一份做过滤，于是又多了一处需要同步的地方。
  4. 删除：item 删了，向量库里那条谁来删？TTL 过期的条目呢？
  5. 运维：两套容量规划、两套监控、两套灾备。 ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]



这些工作没有一项在产出业务价值，但每一项都会在半夜报警。

## **三、向量索引把这条链路收进了托管服务**

DynamoDB 的向量索引是一种新的索引类型，与 GSI、LSI 并列，但用的是近似最近邻（ANN）检索而不是精确匹配和范围查询。它通过你已经在用的 CreateTable / UpdateTable [API](<https://aws.amazon.com/cn/what-is/api/>) 管理，读取走一个新的 SearchVectors API。 ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

关键在于：索引与基表的同步由 DynamoDB 负责。应用只写基表一次。

| 向量索引 | 全局二级索引  
---|---|---  
查询类型 | 相似度搜索 | 精确匹配 + 范围  
读 API | SearchVectors | Query / Scan  
Schema | 向量属性 + 可选 SearchSchema（分区键、内联过滤） | 分区键 + 可选排序键  
每表上限 | 5 | 20  
容量模式 | 仅按需 | 按需或预置  
  
需要明确的一点：DynamoDB 存储和检索向量，但不生成向量。embedding 仍然由你调用嵌入模型产生，本文用 [Amazon Bedrock](<https://aws.amazon.com/cn/bedrock/>) 上的 Cohere Embed v4（多语言，输出维度可选 256 / 512 / 1024 / 1536，本文取 1024）。

## **四、数据模型：一张表承载两种读取**

这是整个设计的核心。我们让同一条 item 同时满足按时间读和按语义读。

属性 | 角色  
---|---  
pk = USER#<user_id> | 基表分区键  
sk = MEM#<类型>#<ISO 时间戳> | 基表排序键，让 Query 天然按时间倒序取最近几轮  
user_id | 向量索引分区键（HASH）  
mem_type | 向量索引内联过滤（INLINE_FILTER），取值 profile / conversation  
text | 记忆原文  
embedding | 1024 维向量  
embed_model | 生成该向量的模型 ID，用于检测模型变更  
expires_at | TTL，短期记忆自动过期  
  
**两个 schema 上的设计决策值得展开** ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

排序键前缀带上类型。MEM#conversation#<时间戳> 这样的结构，让「取最近 6 轮对话」变成一次 begins_with 的 Query，不需要额外的 GSI，也不需要过滤。 ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

向量索引分区键选 user_id。向量索引的分区键做两件事：把索引数据分片以便独立扩展，以及把每次搜索的范围限定在一个分区键值内。对多租 ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]

→ [[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索|原文存档]] ^[raw/articles/用-amazon-dynamodb-构建无需双写的-agent-memory-与语义检索.md]