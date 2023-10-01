---
title: 'Hive-sql,spark sql,sql区别'
date: 2022-10-01 17:27:57
img: /images/hivesql_sparksql.jpg
tags:
	- hive
	- spark
---

## Hive-sql和sql的区别是什么？

区别：1、Hive-sql不支持等值连接，而sql支持；2、Hive-sql不支持“Insert into 表 Values()”、UPDATA、DELETE操作，而sql支持；3、Hive-sql不支持事务，而sql支持。

**1、Hive不支持等值连接**

不支持等值连接，一般使用left join、right join 或者inner join替代。

SQL中内关联可以这样写： select * from a , b where a.key = b.key

Hive中应该这样写： select * from a join b on a.key = b.key

hive中不能使用省去join的写法。

**2、分号字符**

分号是sql语句的结束符号，在hive中也是，但是hive对分号的识别没有那么智能，有时需要进行转义 “；” --> “\073”

select concat(key,concat(';',key)) from dual;

但HiveQL在解析语句时提示：
     FAILED: Parse Error: line 0:-1 mismatched input '<EOF>' expecting ) in function specification
•解决的办法是，使用分号的八进制的ASCII码进行转义，那么上述语句应写成：
•select concat(key,concat('\073',key)) from dual;

**3、NULL**

sql中null代表空值，但是在Hive中，String类型的字段若是空（empty）字符串，即长度为0，那么对它 is null 判断结果为False

**4、Hive不支持将数据插入现有的表或分区中，**
仅支持覆盖重写整个表，示例如下：

INSERT OVERWRITE TABLE t1 

**5、hive不支持INSERT INTO 表 Values（）, UPDATE, DELETE操作**

6、hive支持嵌入mapreduce程序，来处理复杂的逻辑

**7、hive支持将转换后的数据直接写入不同的表，还能写入分区、hdfs和本地目录**
这样能免除多次扫描输入表的开销。

```sql
FROM t1  
INSERT OVERWRITE TABLE t2  
SELECT t3.c2, count(1)  
FROM t3  
WHERE t3.c1 <= 20  
GROUP BY t3.c2  
  
INSERT OVERWRITE DIRECTORY '/output_dir'  
SELECT t3.c2, avg(t3.c1)  
FROM t3  
WHERE t3.c1 > 20 AND t3.c1 <= 30  
GROUP BY t3.c2  
  
INSERT OVERWRITE LOCAL DIRECTORY '/home/dir'  
SELECT t3.c2, sum(t3.c1)  
FROM t3  
WHERE t3.c1 > 30  
GROUP BY t3.c2; 
```

## hive sql 和 spark sql 的对比

sql生成mapreduce程序必要的过程：解析（Parser）、优化（Optimizer）、执行（Execution）

|                | hive sql                    | spark sql                     | 备注                                                         |
| -------------- | --------------------------- | ----------------------------- | ------------------------------------------------------------ |
| 解析/优化/执行 | hive自有程序                | spark自有程序，主要是catalyst | 虽然hive sql spark sql 语法上和执行上看起来差别不大，但是因为spark 的提效目的，其实优化器中做了许多优化 |
| 计算引擎       | hadoop集群中默认是mapreduce | 默认是spark                   | hive sql可以通过配置更改其计算引擎                           |

catalyst的主要优化点

1、谓词下推

sql 语句

```sql
select a.*,b.* from 
tb1 a join tb2 b 
on a.id = b.id 
where a.c1 > 20 and b.c2< 100
```

会被优化为

```sql
select a.*,b.* from 
(select * from tb1 where c1>20) a 
join 
(select * from tb2 where c2<100) b 
on a.id = b.id
```

2、列裁剪

```sql
select a.name,b.salary from 
(select * from tb1 where c1>20) a 
join 
(select * from tb2 where c2<100) b 
on a.id = b.id
```

会被优化为

```sql
select a.name,b.salary from 
(select id,name from tb1 where c1>20) a 
join 
(select id,salary from tb2 where c2<100) b 
on a.id = b.id
```

执行前将不需要的列裁剪掉，减少数据量获取；

3、常量累加

sql语句

```sql
select 1+1 as cnt from tb
1
```

会被优化为

```sql
select 2 as cnt from tb
```

窗口函数中hive sql 若没有可以不用填写partition by order by ，会默认不指定执行，但是spark sql 中不支持，必须填写完整才能执行，示例

/*获取数据行号*/

```sql
select row_number() over() rownum from tb1;
```

如上两种写法，在hive中都可以正常运行，并且正确得到表的行号，而spark只支持下面那种写法

```sql
/*获取数据行号*/
select row_number() over(partition by 1 order by 1) rownum from tb1;
```

如上两种写法，在hive中都可以正常运行，并且正确得到表的行号，而spark只支持下面那种写法

![image-20221001173243455](E:\BLOG\hexoblog\source\_posts\Hive-sql-spark-sql-sql区别\image-20221001173243455.png)