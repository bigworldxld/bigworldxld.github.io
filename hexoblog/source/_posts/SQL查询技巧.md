---
title: SQL查询技巧
date: 2022-11-30 16:29:36
tags: mysql
cover: true
categories: database
img: /images/mysql技巧.jpg
---

## 1.列转行

导入数据

```sql
DROP TABLE IF EXISTS `t_student`;
CREATE TABLE `t_student` (
  `id` int(20) NOT NULL AUTO_INCREMENT COMMENT '主键 id',
  `name` varchar(50) DEFAULT NULL COMMENT '姓名',
  `course` varchar(50) DEFAULT NULL COMMENT '课程',
  `score` int(3) DEFAULT NULL COMMENT '成绩',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8;

INSERT INTO `t_student` VALUES (1,'张三', '语文', 95);
INSERT INTO `t_student` VALUES (2,'李四', '语文', 99);
INSERT INTO `t_student` VALUES (3,'王五', '语文', 80);
INSERT INTO `t_student` VALUES (4,'张三', '数学', 86);
INSERT INTO `t_student` VALUES (5,'李四', '数学', 96);
INSERT INTO `t_student` VALUES (6,'王五', '数学', 81);
INSERT INTO `t_student` VALUES (7,'张三', '英语', 78);
INSERT INTO `t_student` VALUES (8,'李四', '英语', 88);
INSERT INTO `t_student` VALUES (9,'王五', '英语', 87);
INSERT INTO `t_student` VALUES (10,'张三', '历史', 98);
INSERT INTO `t_student` VALUES (11,'李四', '历史', 85);
INSERT INTO `t_student` VALUES (12,'王五', '历史', 89);
```

**t_student表 (学生成绩表)**

![img](https://img-blog.csdnimg.cn/a999f42ea93d4b1e89af6a11126be151.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_6,color_FFFFFF,t_70,g_se,x_16)

### 1.1MAX(CASE WEHN)方法

```sql
SELECT name as '姓名',
       MAX(CASE WHEN course = '语文' THEN score ELSE 0 END) AS '语文',
       MAX(CASE WHEN course = '数学' THEN score ELSE 0 END) AS '数学',
       MAX(CASE WHEN course = '英语' THEN score ELSE 0 END) AS '英语',
       MAX(CASE WHEN course = '历史' THEN score ELSE 0 END) AS '历史'
FROM t_student
GROUP BY name;
```

### 1.2 SUM(IF(条件,列值,0))

```sql
SELECT name as '姓名',
       SUM(IF(course = '语文',score,0)) AS '语文',
       SUM(IF(course = '数学',score,0)) AS '数学',
       SUM(IF(course = '英语',score,0)) AS '英语',
       SUM(IF(course = '历史',score,0)) AS '历史'
FROM t_student
GROUP BY name;
```

结果展示:

![img](https://img-blog.csdnimg.cn/7976ae377b0d4e04be629090d5102e20.png)

案例：

pivot函数实现某列的值转为列名,行转列

unpivot列转行

```sql
create table test2
(id int,name varchar(20), Q1 int, Q2 int, Q3 int, Q4 int)
insert into test2 values(1,'苹果',1000,2000,4000,5000)
insert into test2 values(2,'梨子',3000,3500,4200,5500)
select * from test2
```

```sql
--列转行
select id,name,quarter,number
from
test2
unpivot
(
number
for quarter in
([Q1],[Q2],[Q3],[Q4])
)
as unpvt
```

## 2.行转列

案例一：导入数据

```sql
DROP TABLE IF EXISTS `t_course`;
CREATE TABLE `t_course` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(50) DEFAULT NULL COMMENT '用户名',
  `chinese` double DEFAULT NULL COMMENT '语文成绩',
  `math` double DEFAULT NULL COMMENT '数学成绩',
  `english` double DEFAULT NULL COMMENT '英语成绩',
  `history` double DEFAULT NULL COMMENT '历史成绩',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

INSERT INTO t_course VALUES ('1', '张三', '95', '86', '78', '98');
INSERT INTO t_course VALUES ('2', '李四', '99', '96', '88', '85');
INSERT INTO t_course VALUES ('3', '王五', '80', '81', '87', '89');
```

**t_course表**

![img](https://img-blog.csdnimg.cn/2aa832f09e1645bcba3bf4e679c11873.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_14,color_FFFFFF,t_70,g_se,x_16)

行转列的过程， 其实就是列转行的逆过程

-- 列转行:通过UNION或UNION ALL实现

```sql
SELECT user_name,'语文' AS course,chinese AS score FROM t_course
UNION ALL
SELECT user_name,'数学' AS course,math AS score FROM t_course
UNION ALL
SELECT user_name,'英语' AS course,english AS score FROM t_course
UNION ALL
SELECT user_name,'政治' AS course,history AS score FROM t_course
ORDER BY user_name;
```

部分结果展示:

![img](https://img-blog.csdnimg.cn/542796f0ea9547888c5d162f13ab3441.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_7,color_FFFFFF,t_70,g_se,x_16)

UNION 与 UNION ALL的区别:

1.对重复结果的处理: UNION会去掉重复记录，UNION ALL不会

2.对排序的处理: UNION会排序，UNION ALL只是简单地将两个结果集合并

3.效率方面的区别: 因为UNION会做去重和排序处理，因此效率比UNION ALL慢很多

案例二：

```sql
CREATE table test
(id int,name nvarchar(20),quarter int,number int)
insert into test values(1,N'苹果',1,1000)
insert into test values(1,N'苹果',2,2000)
insert into test values(1,N'苹果',3,4000)
insert into test values(1,N'苹果',4,5000)
insert into test values(2,N'梨子',1,3000)
insert into test values(2,N'梨子',2,3500)
insert into test values(2,N'梨子',3,4200)
insert into test values(2,N'梨子',4,5500)
select * from test
```

```sql
select ID,NAME,
[1] as '一季度',
[2] as '二季度',
[3] as '三季度',
[4] as '四季度'
from
test
pivot
(
sum(number)
for quarter in
([1],[2],[3],[4])
)
as pvt
```





## 3.列转换成字符串

在某些场景下，对单列或者多列转换成字符串，实现这个需求需要使用到 GROUP_CONCAT函数

语法格式

GROUP_CONCAT([DISTINCT] 要连接的字段 [ORDER BY 排序字段 ASC/DESC] [SEPARATOR '分隔符'])
t_student表 (学生成绩表)

![img](https://img-blog.csdnimg.cn/a999f42ea93d4b1e89af6a11126be151.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_6,color_FFFFFF,t_70,g_se,x_16)

**问题:**实现t_student表中课程和成绩拼接为一个字符串的功能

SELECT name, GROUP_CONCAT(course, ":", score) AS '课程:成绩'
FROM t_student
GROUP BY name;结果展示:

![img](https://img-blog.csdnimg.cn/9ef1626898b64c7fbc80a756e73702c4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_9,color_FFFFFF,t_70,g_se,x_16)

## 4.字符串转换成列

在某些场景下，我们需要把某一列的字符串转成多列

导入数据

**t_user_order表 (用户订单表)**

```sql
DROP TABLE IF EXISTS `t_user_order`;
CREATE TABLE `t_user_order` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键 id',
  `user_id` varchar(50) DEFAULT NULL COMMENT '用户 id',
  `order_id` varchar(100) DEFAULT NULL COMMENT '订单 ids',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

INSERT INTO t_user_order VALUES ('1', 'user1', '1,3,5,19,20');
INSERT INTO t_user_order VALUES ('2', 'user2', '2,4,6,8,30,50');
INSERT INTO t_user_order VALUES ('3', 'user3', '11,15,29,31,33');
```

结果展示:

![img](https://img-blog.csdnimg.cn/f3a6c34fea1e4778bced3ae1cece1568.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_8,color_FFFFFF,t_70,g_se,x_16)

从上表可以看出用户ID (user_id)和订单ID (order_id)之间的关系是一对多关系,用户ID对应的订单 ID是一个字符串

问题: 将order_id中的字符串转换成列

思路: 利用help_topic表把以逗号分隔的字符串转换成行

-- 字符串转换成列: 利用SUBSTRING_INDEX和mysql.help_topic实现

```sql
SELECT a.user_id,
       SUBSTRING_INDEX(SUBSTRING_INDEX(a.order_id, ',', b.help_topic_id + 1 ), ',',- 1 )AS order_id
FROM t_user_order AS a
LEFT JOIN mysql.help_topic AS b 
ON b.help_topic_id < (length(a.order_id) - length(REPLACE(a.order_id, ',', '' )) + 1);
```

部分结果展示:

![img](https://img-blog.csdnimg.cn/4aed4d66661a495ca4c3cce6ff88611d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bGx6Iy26Iqx5byA5pe244CC,size_5,color_FFFFFF,t_70,g_se,x_16)

## **5、分页**

**方案一：利用NOT IN和SELECT TOP分页语句形式**

```sql
SELECT TOP 10 * FROM TestTable
WHERE ID NOT IN
(SELECT TOP 20 ID FROM TestTable ORDER BY ID)
ORDER BY ID
```

 **方案二：利用ID大于多少和SELECT TOP分页语句形式**

```sql
SELECT TOP 10 * FROM TestTable
WHERE ID > (
SELECT MAX(id) FROM 
(SELECT TOP 20 id FROM 
TestTable ORDER BY id) AS T)
ORDER BY ID
```

**方案三：利用SQL Server中的特性ROW_NUMBER进行分页** 

```sql
SELECT * FROM (
  SELECT ROW_NUMBER() OVER(ORDER BY ID DESC) AS ROWID,*
  FROM TestTable
) AS mytable where ROWID between 21 and 40
```

## **6、随机排序**

```sql
SELECT * FROM TestTable ORDER BY NEWID()
```

还可以结合TOP取随机的前N条记录

```sql
SELECT TOP 100 * FROM TestTable ORDER BY NEWID()
```

## **7、以任意符号分隔取两边数据**

例如我们以逗号(,)来分割数据，将如下数据

R

A,B

NULL

,

A,C

B,C

C,D

分割成如下图所示：

R	R1	R2

A,B	A	B

NULL	NULL	NULL

,	NULL	NULL

A,C	A	C

B,C	B	C

C,D	C	D

```sql
SELECT R,
CASE WHEN  CHARINDEX(',',R)>1 THEN  LEFT(R,CHARINDEX(',',R)-1) ELSE NULL END AS R1 ,
CASE WHEN CHARINDEX(',',R)>1 THEN RIGHT(R,(LEN(R) - CHARINDEX(',',R))) ELSE NULL END AS R2
FROM  t
```

**CHARINDEX ( expressionToFind , expressionToSearch [ , start_location ] )**

   expressionToFind ：目标字符串，就是想要找到的字符串，最大长度为8000 。

　　expressionToSearch ：用于被查找的字符串。

   start_location：开始查找的位置，为空时默认从第一位开始查找。

sql的left()函数表示的是从字符表达式最左边一个字符开始返回指定数目的字符.若 b 的值大于 a 的长度,则返回字符表达式的全部字符a.如果 b 为负值或 0,则返回空字符串.

## **8、WAITFOR延时执行**

例 等待1 小时2 分零3 秒后才执行SELECT 语句

```sql
WAITFOR DELAY '01:02:03'
SELECT * FROM Employee
```

其中 DELAY是在延时多长时间后才开始执行。

例 等到晚上11 点零8 分后才执行SELECT 语句

```sql
WAITFOR TIME '23:08:00'
SELECT * FROM Employee
```

其中TIME是等到具体某个时刻才开始执行

## 9、where 1=1

如果在where语句后面加上 or 1=1会是什么后果？

即：

```sql
delete from customers where name='张三' or 1=1
```

本来只要删除张三的记录，结果因为添加了or  1=1的永真条件，会导致整张表里的记录都被删除了

```java
String sql="select * from table_name where 1=1";
if( condition 1) {
  sql=sql+"  and  var2=value2";
}
if(condition 2) {
  sql=sql+"  and var3=value3";
}
```

这里写上where 1=1 是为了避免where 关键字后面的第一个词直接就是 “and”而导致语法错误，加上1=1后，不管后面有没有and条件都不会造成语法错误了

**SQL注入**

加不加where 1=1，查询不都一样吗？例如：

```sql
select * from customers;
与
select * from customers where 1=1;
```

查询出来的结果完全没有区别呀。

**拷贝表** 

在我们进行数据备份时，也经常使用到where 1=1，当然其实这两可以不写，写上之后如果想过滤一些数据再备份会比较方便，直接在后面添加and条件即可。

```sql
create table  table_name
as   
select * from  Source_table
where   1=1;
```

**复制表结构** 

有1=1就会有1<>1或1=2之类的永假的条件，这个在拷贝表的时候，加上where 1<>1，意思就是没有任何一条记录符合条件，这样我们就可以只拷贝表结构，不拷贝数据了。

```sql
create table  table_name
as   
select  * from   
Source_table where   1 <> 1;
```

## **10、字符串替换SUBSTRING/REPLACE**

```sql
SELECT REPLACE('abcdefg',SUBSTRING('abcdefg',2,4),'**')
```

返回a**fg

## **11、表复制**

语法1：Insert INTO table(field1,field2,...) values(value1,value2,...)

语法2：Insert into Table2(field1,field2,...) select value1,value2,... from Table1

（要求目标表Table2必须存在，由于目标表Table2已经存在，所以我们除了插入源表Table1的字段外，还可以插入常量。）

语法3：SELECT vale1, value2 into Table2 from Table1

（要求目标表Table2不存在，因为在插入时会自动创建表Table2，并将Table1中指定字段数据复制到Table2中。）

语法4：使用导入导出功能进行全表复制。

## **12、利用带关联子查询Update语句更新数据**

```sql
--方法1：
Update Table1
set c = (select c from Table2 where a = Table1.a)
where c is null

--方法2：
update  A
set  newqiantity=B.qiantity
from  A,B
where  A.bnum=B.bnum

--方法3：
update
(select A.bnum ,A.newqiantity,B.qiantity from A
left join B on A.bnum=B.bnum) AS C
set C.newqiantity = C.qiantity
where C.bnum ='001'
```



**8、连接远程服务器**

```sql
--方法1：
select *  from openrowset(
'SQLOLEDB',
'server=192.168.0.1;uid=sa;pwd=password',
'SELECT * FROM dbo.test')

--方法2：
select *  from openrowset(
'SQLOLEDB',
'192.168.0.1';
'sa';
'password',
'SELECT * FROM dbo.test')
```

当然也可以参考以前的示例，建立DBLINK进行远程连接



## **13、Date 和 Time 样式 CONVERT**

CONVERT() 函数是把日期转换为新数据类型的通用函数。 

CONVERT() 函数可以用不同的格式显示日期/时间数据。

语法

> CONVERT(data_type(length),data_to_be_converted,style)

data_type(length) 规定目标数据类型（带有可选的长度）。data_to_be_converted 含有需要转换的值。style 规定日期/时间的输出格式。

可以使用的 style 值：

| Style ID    | Style 格式                            |
| :---------- | :------------------------------------ |
| 100 或者 0  | mon dd yyyy hh:miAM （或者 PM）       |
| 101         | mm/dd/yy                              |
| 102         | yy.mm.dd                              |
| 103         | dd/mm/yy                              |
| 104         | dd.mm.yy                              |
| 105         | dd-mm-yy                              |
| 106         | dd mon yy                             |
| 107         | Mon dd, yy                            |
| 108         | hh:mm:ss                              |
| 109 或者 9  | mon dd yyyy hh:mi:ss:mmmAM（或者 PM） |
| 110         | mm-dd-yy                              |
| 111         | yy/mm/dd                              |
| 112         | yymmdd                                |
| 113 或者 13 | dd mon yyyy hh:mm:ss:mmm(24h)         |
| 114         | hh:mi:ss:mmm(24h)                     |
| 120 或者 20 | yyyy-mm-dd hh:mi:ss(24h)              |
| 121 或者 21 | yyyy-mm-dd hh:mi:ss.mmm(24h)          |
| 126         | yyyy-mm-ddThh:mm:ss.mmm（没有空格）   |
| 130         | dd mon yyyy hh:mi:ss:mmmAM            |
| 131         | dd/mm/yy hh:mi:ss:mmmAM               |

```sql
SELECT CONVERT(varchar(100), GETDATE(), 0)
--结果：
12  7 2020  9:33PM
SELECT CONVERT(varchar(100), GETDATE(), 1)
--结果：
12/07/20
SELECT CONVERT(varchar(100), GETDATE(), 2)
--结果：
20.12.07
SELECT CONVERT(varchar(100), GETDATE(), 3)
--结果：
07/12/20
SELECT CONVERT(varchar(100), GETDATE(), 4)
--结果：
07.12.20
SELECT CONVERT(varchar(100), GETDATE(), 5)
--结果：
07-12-20
SELECT CONVERT(varchar(100), GETDATE(), 6)
--结果：
07 12 20
SELECT CONVERT(varchar(100), GETDATE(), 7)
--结果：
12 07, 20
SELECT CONVERT(varchar(100), GETDATE(), 8)
--结果：
21:33:18
SELECT CONVERT(varchar(100), GETDATE(), 9)
--结果：
12  7 2020  9:33:18:780PM
SELECT CONVERT(varchar(100), GETDATE(), 10)
--结果：
12-07-20
SELECT CONVERT(varchar(100), GETDATE(), 11)
--结果：
20/12/07
SELECT CONVERT(varchar(100), GETDATE(), 12)
--结果：
201207
SELECT CONVERT(varchar(100), GETDATE(), 13)
--结果：
07 12 2020 21:33:18:780
SELECT CONVERT(varchar(100), GETDATE(), 14)
--结果：
21:33:18:780
SELECT CONVERT(varchar(100), GETDATE(), 20)
--结果：
2020-12-07 21:33:18
SELECT CONVERT(varchar(100), GETDATE(), 21)
--结果：
2020-12-07 21:33:18.780
SELECT CONVERT(varchar(100), GETDATE(), 22)
--结果：
12/07/20  9:33:18 PM
SELECT CONVERT(varchar(100), GETDATE(), 23)
--结果：
2020-12-07
SELECT CONVERT(varchar(100), GETDATE(), 24)
--结果：
21:33:18
SELECT CONVERT(varchar(100), GETDATE(), 25)
--结果：
2020-12-07 21:33:18.780
SELECT CONVERT(varchar(100), GETDATE(), 100)
--结果：
12  7 2020  9:33PM
SELECT CONVERT(varchar(100), GETDATE(), 101)
--结果：
12/07/2020
SELECT CONVERT(varchar(100), GETDATE(), 102)
--结果：
2020.12.07
SELECT CONVERT(varchar(100), GETDATE(), 103)
--结果：
07/12/2020
SELECT CONVERT(varchar(100), GETDATE(), 104)
--结果：
07.12.2020
SELECT CONVERT(varchar(100), GETDATE(), 105)
--结果：
07-12-2020
SELECT CONVERT(varchar(100), GETDATE(), 106)
--结果：
07 12 2020
SELECT CONVERT(varchar(100), GETDATE(), 107)
--结果：
12 07, 2020
SELECT CONVERT(varchar(100), GETDATE(), 108)
--结果：
21:33:18
SELECT CONVERT(varchar(100), GETDATE(), 109)
--结果：
12  7 2020  9:33:18:780PM
SELECT CONVERT(varchar(100), GETDATE(), 110)
--结果：
12-07-2020
SELECT CONVERT(varchar(100), GETDATE(), 111)
--结果：
2020/12/07
SELECT CONVERT(varchar(100), GETDATE(), 112)
--结果：
20201207
SELECT CONVERT(varchar(100), GETDATE(), 113)
--结果：
07 12 2020 21:33:18:780
SELECT CONVERT(varchar(100), GETDATE(), 114)
--结果：
21:33:18:780
SELECT CONVERT(varchar(100), GETDATE(), 120)
--结果：
2020-12-07 21:33:18
SELECT CONVERT(varchar(100), GETDATE(), 121)
--结果：
2020-12-07 21:33:18.780
```

