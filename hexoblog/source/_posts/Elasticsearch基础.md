---
title: Elasticsearch基础
img: https://bigworldxld.github.io/picx-images-hosting/images/Elasticsearch.webp
categories: OPL
tags:
  - Elasticsearch
  - Kibana
abbrlink: 23237
date: 2024-11-10 14:36:17
---

## Elasticsearch基础

[Elasticsearch](https://github.com/elastic/elasticsearch/) 是一个基于 Apache Lucene 构建的分布式搜索和分析引擎、可扩展的数据存储和矢量数据库。

Elasticsearch 是 [Elastic Stack](https://www.elastic.co/guide/en/starting-with-the-elasticsearch-platform-and-its-solutions/current/stack-components.html) 的核心。 与 [Kibana](https://www.elastic.co/kibana) 结合使用

Elasticsearch 是 Elastic Stack 的核心组件，Elastic Stack 是一套用于收集、存储、搜索和可视化数据的产品

## 运用

**可观察性**

- **日志、指标和跟踪**
- **应用程序性能监控 （APM）
- **真实用户监控 （RUM）
- **OpenTelemetry**：重复使用现有检测

**搜索**

- **全文搜索**：使用倒排索引、分词元化和文本分析构建快速、相关的全文搜索解决方案
- **矢量数据库**：存储和搜索矢量化数据，并使用内置和第三方自然语言处理 （NLP） 模型创建矢量嵌入
- **语义搜索**：使用同义词、密集向量嵌入和学习的稀疏查询文档扩展等工具了解搜索查询背后的意图和上下文含义。
- **混合搜索**：使用最先进的排名算法将全文搜索与向量搜索相结合。
- **构建搜索体验**：向应用程序或网站添加混合搜索功能，或在组织的内部数据源上构建企业搜索引擎。
- **检索增强生成 （RAG）：**使用 Elasticsearch 作为检索引擎，为生成式 AI 模型提供更相关、最新或专有的数据，以用于一系列使用案例。
- **地理空间搜索**：使用地理空间查询搜索位置并计算空间关系。

**安全**

- **安全信息和事件管理 （SIEM）：**收集、存储和分析来自应用程序、系统和服务的安全数据。
- **端点安全**：监控和分析端点安全数据。
- **威胁搜寻**：搜索和分析数据以检测和响应安全威胁。

## 索引、文档和字段

索引是 Elasticsearch 中的基本存储单元，是一个用于存储具有相似特征的数据的逻辑命名空间。 部署 Elasticsearch 后，您将首先创建一个索引来存储数据。

索引是由名称或[别名](https://www.elastic.co/guide/en/elasticsearch/reference/current/aliases.html)唯一标识的文档集合。 此唯一名称非常重要，因为它用于在搜索查询和其他操作中定位索引。

一个密切相关的概念是[数据流](https://www.elastic.co/guide/en/elasticsearch/reference/current/data-streams.html)。 此索引抽象针对仅追加时间戳数据进行了优化，并且由隐藏的、自动生成的支持索引组成

### 文档和字段

Elasticsearch 以 JSON 文档的形式序列化和存储数据。 文档是一组字段，这些字段是包含数据的键值对。 每个文档都有一个唯一的 ID，您可以创建该 ID 或让 Elasticsearch 自动生成该 ID。

一个简单的 Elasticsearch 文档可能如下所示：

```json
{
  "_index": "my-first-elasticsearch-index",
  "_id": "DyFpo5EBxE8fzbb95DOa",
  "_version": 1,
  "_seq_no": 0,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "email": "john@smith.com",
    "first_name": "John",
    "last_name": "Smith",
    "info": {
      "bio": "Eco-warrior and defender of the weak",
      "age": 25,
      "interests": [
        "dolphins",
        "whales"
      ]
    },
    "join_date": "2024/05/01"
  }
}
```

### 元数据字段

索引文档包含数据和元数据。[元数据字段](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-fields.html)是存储有关文档的信息的系统字段。 在 Elasticsearch 中，元数据字段以下划线为前缀。 例如，以下字段是元数据字段：

- `_index`：存储文档的索引的名称。
- `_id`：文档的 ID。每个索引的 ID 必须是唯一的。

### 映射和数据类型

每个索引都有一个[映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)或架构，用于如何为文档中的字段编制索引。 映射定义每个字段的[数据类型](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html)、字段的索引方式、 以及应该如何存储。 将文档添加到 Elasticsearch 时，您有两个映射选项：

- [动态映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#mapping-dynamic)：让 Elasticsearch 自动检测数据类型并为您创建映射。动态映射可帮助您快速入门，但由于自动字段类型推理，可能会为您的特定使用案例产生次优结果。
- [显式映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#mapping-explicit)：通过为每个字段指定数据类型来预先定义映射。建议用于生产使用案例，因为您可以完全控制数据索引的方式以适应您的特定使用案例。

您可以对同一索引使用动态映射和显式映射的组合。 当数据中混合了已知字段和未知字段时，这非常有用。

## 搜索和分析数据

###  REST API

使用 REST API 管理 Elasticsearch 集群并建立索引 并搜索您的数据。 出于测试目的，您可以提交请求 直接从命令行或通过 Kibana 中的开发工具[控制台](https://www.elastic.co/guide/en/kibana/8.15/console-kibana.html)。 在您的应用程序中，您可以使用所选编程语言的[客户端](https://www.elastic.co/guide/en/elasticsearch/client/index.html)。

###  查询语言

Elasticsearch 提供了多种查询语言来与数据交互。

**查询 DSL** 是当今 Elasticsearch 的主要查询语言。

**ES|QL** 是一种新的管道查询语言和计算引擎，首次在 **8.11** 版中添加

###  查询 DSL

[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) 是一种功能齐全的 JSON 样式查询语言，支持复杂的搜索、筛选和聚合。 它是当今 Elasticsearch 的原始且最强大的查询语言。

[`_search` 终端节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-your-data.html)接受以 Query DSL 语法编写的查询。

### ES|QL 系列

[Elasticsearch 查询语言 （ES|QL）](https://www.elastic.co/guide/en/elasticsearch/reference/current/esql.html) 是一种管道查询语言，用于筛选、转换和分析数据。 ES|QL 构建在新的计算引擎之上，其中搜索、聚合和转换功能 直接在 Elasticsearch 本身中执行。 ES|QL 语法也可以在各种 Kibana 工具中使用

| 名字                                                         | 描述                                                         | 使用案例                                                     | API 终端节点                                                 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [查询 DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) | Elasticsearch 的主要查询语言。一种功能强大且灵活的 JSON 样式语言，支持复杂查询。 | 全文搜索、语义搜索、关键字搜索、筛选、聚合等。               | [`_search`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) |
| [ES\|QL 系列](https://www.elastic.co/guide/en/elasticsearch/reference/current/esql.html) | 在 **8.11** 中引入的 Elasticsearch 查询语言 （ES\|QL） 是一种管道查询语言，用于筛选、转换和分析数据。 | 最初针对处理日志和指标等时间序列数据进行了定制。 与 Kibana 的强大集成，用于查询、可视化和分析数据。 尚不支持全文搜索。 | [`_query`](https://www.elastic.co/guide/en/elasticsearch/reference/current/esql-rest.html) |
| [EQL](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql.html) | 事件查询语言 （EQL） 是一种基于事件的时间序列数据的查询语言。数据必须包含字段才能使用 EQL。`@timestamp` | 专为威胁搜寻安全使用案例而设计。                             | [`_eql`](https://www.elastic.co/guide/en/elasticsearch/reference/current/eql-apis.html) |
| [Elasticsearch SQL](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-sql.html) | 允许对 Elasticsearch 数据进行原生、类似 SQL 的实时查询。JDBC 和 ODBC 驱动程序可用于与商业智能 （BI） 工具集成。 | 使熟悉 SQL 的用户能够使用熟悉的 BI 和报告语法来查询 Elasticsearch 数据。 | [`_sql`](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-apis.html) |
| [Kibana 查询语言 （KQL）](https://www.elastic.co/guide/en/kibana/8.15/kuery-query.html) | Kibana 查询语言 （KQL） 是一种基于文本的查询语言，用于在您通过 Kibana UI 访问数据时对其进行筛选。 | 使用 KQL 筛选字段值存在、与给定值匹配或位于给定范围内的文档。 | 不适用                                                       |

## 使用多个节点和分片

服务器（*节点*）添加到集群中以增加容量，Elasticsearch 会自动分配您的数据和查询负载 跨所有可用节点。

Elastic 能够通过将索引细分为*多个分片*来跨节点分配数据。Elasticsearch 中的每个索引都是一个分组 一个或多个物理分片，其中每个分片都是一个自包含的 Lucene 索引，其中包含 索引。通过将索引中的文档分布到多个分片，并将这些分片分布到多个分片 节点，Elasticsearch 会增加索引和查询容量。

有两种类型的分片：*主分片*和*副本*分片。索引中的每个文档都属于一个主分片。副本 shard 是主分片的副本。副本在集群中的节点之间维护数据的冗余副本。 这可以防止硬件故障，并增加处理读取请求（如搜索或检索文档）的能力。

在创建索引时，索引中的主分片数量是固定的，但副本分片的数量可以 随时更改，而不会中断索引或查询操作

将节点共置在单个位置会使您面临单次中断导致整个集群离线的风险。自 保持高可用性，您可以通过实施 跨集群复制 （CCR）

**附录**

恢复被删除被格式化的数据工具：rcvPortable 

https://portableapps.com/apps/utilities/rcvportable

rcvPortable 允许您无需安装即可运行 Recuva®。Recuva 可以恢复丢失的图片、音乐、文档、视频、电子邮件或任何其他文件类型。它可以从您拥有的任何可重写介质中恢复：存储卡、外部硬盘驱动器、USB 记忆棒等