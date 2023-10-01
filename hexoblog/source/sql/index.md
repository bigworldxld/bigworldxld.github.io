---
title: 每日sql
date: 2023-02-27 16:07:10
type: "sql"
layout: "sql"
---

## 每日sql题













### 分组条件过滤

有如下两张表 TBooks 表，TOrders 表

编写一个SQL查询，要求去年(今天减去一年的日期)销售少于10本的书籍，不包括从今天起1个月内可供使用的书籍。假设今天是2019-06-23。

结果表:

| BOOK_ID | NAME             |
| ------- | ---------------- |
| 1       | Kalia And Demna  |
| 2       | 28 Letters       |
| 5       | THe Hunger Games |

**测试数据**

```sql
CREATE TABLE TBooks
(
BOOK_ID INT,
NAME VARCHAR(50),
AVAILABLE_FROM DATE
)
INSERT INTO TBooks VALUES
(1,'Kalia And Demna','2010-01-01'),
(2,'28 Letters','2012-05-12'),
(3,'The Hobbit','2019-06-10'),
(4,'13 Reasons Why','2019-06-01'),
(5,'The Hunger Games','2008-09-21')

CREATE TABLE TOrders
(
ORDER_ID INT,
BOOK_ID INT,
QUANTITY INT,
DISPATCH_DATE DATE
)

INSERT INTO TOrders VALUES
(1,1,2,'2018-07-26'),
(2,1,1,'2018-11-05'),
(3,3,8,'2019-06-11'),
(4,4,6,'2019-06-05'),
(5,4,5,'2019-06-20'),
(6,5,9,'2009-02-02'),
(7,5,8,'2010-04-13')
```

```sql
select a.BOOK_ID,a.NAME
from TBooks a left join TOrders b on a.BOOK_ID=b.BOOK_ID 
where a.AVAILABLE_FROM<'2019-05-23' 
group by a.BOOK_ID,a.NAME
having ifnull(sum(case when b.DISPATCH_DATE<'2018-06-23' then 0 else QUANTITY end),0)<10
order by a.BOOK_ID
```

### 计数行转列

有如下两张表G0710A，G0710B，希望可以看到每个机房的主机汇总状态分别有多少个，预期结果如下：

| 机房  | 0    | 1    | 2    |
| ----- | ---- | ---- | ---- |
| 机房A | 0    | 2    | 1    |
| 机房B | 1    | 1    | 0    |
| 机房B | 0    | 0    | 0    |

其中，列名0,1,2为主机状态的值

**测试数据**

```sql
CREATE TABLE G0710A
(
ID INT,
机房 VARCHAR(20)
)
CREATE TABLE G0710B
(
ID INT,
机房ID INT,
主机名称 VARCHAR(20),
主机状态 INT
)
INSERT INTO G0710A VALUES (1,'机房A')
INSERT INTO G0710A VALUES (2,'机房B')
INSERT INTO G0710A VALUES (3,'机房C')
INSERT INTO G0710B VALUES (1,1,'主机A',1)
INSERT INTO G0710B VALUES (2,1,'主机B',1)
INSERT INTO G0710B VALUES (3,1,'主机C',2)
INSERT INTO G0710B VALUES (4,2,'主机D',0)
INSERT INTO G0710B VALUES (5,2,'主机E',1)
```

```sql
SELECT *
FROM
(SELECT a.机房,SUM(CASE WHEN b.主机状态=0 THEN 1 ELSE 0 END) OVER(PARTITION BY a.ID) AS '0',
SUM(CASE WHEN b.主机状态=1 THEN 1 ELSE 0 END) OVER(PARTITION BY a.ID) AS '1',
SUM(CASE WHEN b.主机状态=2 THEN 1 ELSE 0 END) OVER(PARTITION BY a.ID) AS '2'
FROM G0710A a LEFT JOIN G0710B b ON a.ID=b.机房ID) c
GROUP BY c.机房
```

### 标准递归

有如下两张表：部门表，职工表

要求：得到A部与B部（parentid=1)下的总人数，具体结果如下：

| 部门 | 人数 |
| ---- | ---- |
| A部  | 6    |
| B部  | 3    |

测试数据

```sql
CREATE TABLE 部门表(ID int,部门 VARCHAR(10),父ID int)
CREATE TABLE 职工表(部门 VARCHAR(10),姓名 VARCHAR(10))

INSERT INTO 部门表 VALUES(1,'A公司',null)
INSERT INTO 部门表 VALUES(2,'A部',1)
INSERT INTO 部门表 VALUES(3,'B部',1)
INSERT INTO 部门表 VALUES(4,'A部1',2)
INSERT INTO 部门表 VALUES(5,'A部2',2)
INSERT INTO 部门表 VALUES(6,'B部1',3)
INSERT INTO 部门表 VALUES(7,'A部11',4)
 
INSERT INTO 职工表 VALUES('A部','刘备')
INSERT INTO 职工表 VALUES('A部','关羽')
INSERT INTO 职工表 VALUES('A部1','张飞')
INSERT INTO 职工表 VALUES('A部1','曹操')
INSERT INTO 职工表 VALUES('A部2','孙权')
INSERT INTO 职工表 VALUES('B部', '周瑜')
INSERT INTO 职工表 VALUES('A部11','黄忠')
INSERT INTO 职工表 VALUES('B部','马超')
INSERT INTO 职工表 VALUES('B部','赵云')
```

```sql
WITH RECURSIVE cte AS(
SELECT *,部门 AS X FROM 部门表 WHERE 父ID=1
UNION ALL
SELECT b.ID,b.部门,b.父ID,X
FROM cte a JOIN 部门表 b ON a.ID=b.父ID
)
SELECT X AS 部门,COUNT(1) 人数
FROM cte a JOIN 职工表 b ON a.部门=b.部门
GROUP BY X
```

### 简单过滤

有如下一组数据G0706，希望得到如下结果：

| Docnum |
| ------ |
| 34     |
| 36     |
| 37     |

即只取状态为FULL的DOCNUM，如果有同时为NOFULL的DOCNUM则不取。

**测试数据**

```sql
CREATE TABLE G0706
(
DOCNUM INT,
STATUS VARCHAR(26)
)
INSERT INTO  G0706 VALUES (33,'FULL')
INSERT INTO  G0706 VALUES (33,'NOFULL')
INSERT INTO  G0706 VALUES (34,'FULL')
INSERT INTO  G0706 VALUES (35,'FULL')
INSERT INTO  G0706 VALUES (35,'NOFULL')
INSERT INTO  G0706 VALUES (36,'FULL')
INSERT INTO  G0706 VALUES (37,'FULL')
INSERT INTO  G0706 VALUES (38,'FULL')
INSERT INTO  G0706 VALUES (38,'NOFULL')
```

```sql
SELECT t.DOCNUM
FROM
(SELECT *,COUNT(1) OVER(PARTITION BY DOCNUM) AS rn
FROM G0706) t WHERE t.STATUS='FULL' AND t.rn=1
```

### 递归叠加

有如下一组数据G0705，

希望得到如下结果：

| NAME_1 | NAME_2 | NAME_3 |
| ------ | ------ | ------ |
| 安徽   |        |        |
| 安徽   | 合肥   |        |
| 安徽   | 宣城   |        |
| 浙江   |        |        |
| 浙江   | 杭州   |        |
| 浙江   | 杭州   | 萧山   |
| 浙江   | 杭州   | 滨江   |
| 浙江   | 杭州   | 富阳   |
| 浙江   | 宁波   |        |

即：按照省市区分别显示为3列，为省一级时，后面没有下一级，则后两列为空，为市一级，后面没有下一级，则后一列为空。

**测试数据**

```sql
CREATE TABLE G0705
(ID INT,
CODE INT,
NAME VARCHAR(20),
PARENTCODE INT
)
INSERT INTO G0705 VALUES (1,10000,'浙江',0)
INSERT INTO G0705 VALUES (2,20000,'安徽',0)
INSERT INTO G0705 VALUES (3,11000,'杭州',10000)
INSERT INTO G0705 VALUES (4,12000,'宁波',10000)
INSERT INTO G0705 VALUES (5,21000,'合肥',20000)
INSERT INTO G0705 VALUES (6,22000,'宣城',20000)
INSERT INTO G0705 VALUES (7,11100,'萧山',11000)
INSERT INTO G0705 VALUES (8,11300,'滨江',11000)
INSERT INTO G0705 VALUES (9,11300,'富阳',11000)
```

```sql
SELECT NAME_1,NAME_2,NAME_3
FROM
(
SELECT NAME AS NAME_1,'' AS NAME_2,'' AS NAME_3,1 AS LEVEL
FROM G0705 WHERE PARENTCODE=0
UNION ALL
SELECT a.NAME ,b.NAME ,'',2
FROM G0705 a JOIN G0705 b ON b.PARENTCODE=a.CODE AND a.PARENTCODE=0
UNION ALL
SELECT a.NAME,b.NAME ,c.NAME,3
FROM G0705 a
JOIN G0705 b ON a.CODE=b.PARENTCODE AND a.PARENTCODE=0
JOIN G0705 c ON b.CODE=c.PARENTCODE
) a ORDER BY NAME_1,NAME_2,LEVEL
```

### 中位数

G0704表包含所有员工，其中有三列：员工Id，公司名和薪水。

请编写SQL查询来查找每个公司的薪水中位数。

挑战点：你是否可以在不使用任何内置的SQL函数的情况下解决此问题

| Id   | Company | Salary |
| ---- | ------- | ------ |
| 2    | A       | 9410   |
| 6    | A       | 9513   |
| 10   | B       | 10345  |
| 9    | B       | 11540  |
| 15   | C       | 9000   |

中位数：若记录数为奇数，取一条，否则取两条，如记录数为7，按顺序直接取第4名即是中位数， 记录数为6，按顺序则是第3，4名是中位数。

**测试数据**

```sql
CREATE TABLE G0704
(
ID INT,
Company VARCHAR(10),
Salary INT
)
INSERT INTO G0704 VALUES
(1,'A',8341),
(2,'A',9410),
(3,'A',10050),
(4,'A',15314),
(5,'A',8451),
(6,'A',9513),
(7,'B',10005),
(8,'B',13000),
(9,'B',11540),
(10,'B',10345),
(11,'B',12210),
(12,'B',9234),
(13,'C',12000),
(14,'C',8900),
(15,'C',9000),
(16,'C',10100),
(17,'C',8000)
```

```sql
SELECT ID,Company,Salary
FROM
(SELECT *,ROW_NUMBER() OVER(PARTITION BY Company ORDER BY Salary) rk,
COUNT(*) OVER(PARTITION BY Company) AS num
FROM G0704) t
WHERE ((num%2=1 AND rk=FLOOR(num/2)+1)
OR (num%2=0 AND rk=FLOOR(num/2) OR rk=FLOOR(num/2)+1))     
```

### 条件筛选合并

有如下一张表G0703，

当001值是‘否’且002子项为‘空’时均不显示记录；

当001值是‘是’且002子项有记录时均不显示记录；

当001值是‘是’且002子项为‘空’时显示002该行空记录；

当001值是‘空’且002子项为‘空’时都显示记录；

最终理想的查询结果：

| ID   | CODE | QUESTION | ANSWER |
| ---- | ---- | -------- | ------ |
| 3    | 002  | 吸烟年龄 | (null) |
| 4    | 001  | 是否吸烟 | (null) |
| 4    | 002  | 吸烟年龄 | (null) |

**测试数据**

```sql
CREATE TABLE G0703
(
ID INT,
CODE VARCHAR(10),
QUESTION VARCHAR(20),
ANSWER VARCHAR(10)
)
INSERT INTO G0703 VALUES
(1,'001','是否吸烟','否'),
(1,'002','吸烟年龄',NULL),
(2,'001','是否吸烟','是'),
(2,'002','吸烟年龄','18'),
(3,'001','是否吸烟','是'),
(3,'002','吸烟年龄',NULL),
(4,'001','是否吸烟',NULL),
(4,'002','吸烟年龄',NULL)
```

```sql
SELECT *
FROM
(SELECT *
FROM G0703 a WHERE CODE='002' AND ANSWER IS NULL 
AND EXISTS(SELECT 1 FROM G0703 WHERE ID=a.ID AND CODE='001' AND (ANSWER='是' OR ANSWER IS NULL))
UNION ALL
SELECT *
FROM G0703 a WHERE CODE='001' AND ANSWER IS NULL 
AND EXISTS(SELECT 1 FROM G0703 WHERE ID=a.ID AND CODE='002' AND ANSWER IS NULL)
) b ORDER BY ID,CODE
```







### 错位比较

有如下一组数据G0630，

要求出同一台机器的状态转换次数，例如1001机器从，开机到关机算一次，从关机再到待机算一次，合计2次，希望得出以下结果

| MechineID | CNT  |
| --------- | ---- |
| 1001      | 2    |
| 1002      | 3    |

**测试数据**

```sql
CREATE TABLE G0630
(ID INT,
MechineID INT,
Status VARCHAR(10)
)
INSERT INTO G0630 VALUES
(1,1001,'开机'),
(2,1001,'开机'),
(3,1001,'关机'),
(4,1001,'关机'),
(5,1001,'待机'),
(6,1001,'待机'),
(7,1002,'开机'),
(8,1002,'开机'),
(9,1002,'关机'),
(10,1002,'待机'),
(11,1002,'开机')
```

```sql
--1 自连接，错位比较
select MechineID,sum(case when a.MechineID=b.MechineID and a.Status<>b.Status then 1 else 0 end) cnt
from G0630 a join G0630 b on a.ID=b.ID+1
 group by MechineID
 --2 lag()
 SELECT b.MechineID,SUM(CASE WHEN b.Status<>b.Status2 THEN 1 ELSE 0 END) cnt
 FROM 
 (SELECT a.MechineID,a.Status,
 LAG(a.Status,1) OVER(PARTITION BY a.MechineID ORDER BY a.ID) Status2
 FROM G0630 a)b GROUP BY b.MechineID
```

### 分组加时间

有如下一张表G0627，现在需要查询TripsID相同情况下，SetTime获取到一个月以后的最小最大数据

查询结果如下：

| TripsID | SetTime1   | SetTime2   |
| ------- | ---------- | ---------- |
| 1       | 2021-02-01 | 2021-04-01 |
| 2       | 2021-05-01 | 2021-07-01 |
| 3       | 2021-08-01 | 2021-12-01 |

**测试数据**

```sql
CREATE TABLE G0627(
ID INT,
SetTime DATE,
TripsID INT
)
INSERT INTO G0627 VALUES (1,'2021/1/1',1);
INSERT INTO G0627 VALUES (2,'2021/2/1',1);
INSERT INTO G0627 VALUES (3,'2021/3/1',1);
INSERT INTO G0627 VALUES (4,'2021/4/1',2);
INSERT INTO G0627 VALUES (5,'2021/5/1',2);
INSERT INTO G0627 VALUES (6,'2021/6/1',2);
INSERT INTO G0627 VALUES (7,'2021/7/1',3);
INSERT INTO G0627 VALUES (8,'2021/8/1',3);
INSERT INTO G0627 VALUES (9,'2021/9/1',3);
INSERT INTO G0627 VALUES (10,'2021/10/1',3);
INSERT INTO G0627 VALUES (11,'2021/11/1',3)
```

```sql
--1
select TripsID,date_add(min,interval 1 month) SetTime1,date_add(max,interval 1 month) SetTime2
from
(select TripsID,min(SetTime) min,max(SetTime) max
from G0627 group by TripsID) a
--2
SELECT TripsID,DATE_ADD(MIN(SetTime),INTERVAL 1 MONTH) SetTime1,DATE_ADD(MAX(SetTime),INTERVAL 1 MONTH) SetTime2
FROM G0627 GROUP BY TripsID
```

### *等分稠密排序

有如下一张表G0626,希望得到如下结果：

| 组号 | 药名 | 数   | 每次更换 |
| ---- | ---- | ---- | -------- |
| 1    | 药A  | 5    | 50mg     |
| 1    | 药B  | 5    | 20mg     |
| 2    | 药C  | 5    | 15mg     |
| 3    | 药D  | 3    | 25mg     |
| 1    | 药A  | 5    | 50mg     |
| 1    | 药B  | 5    | 20mg     |

即将每天次数大于1的药品，按数量进行均等分

**测试数据**

```sql
CREATE TABLE G0626
(组号 int NOT NULL ,
药品名 varchar(50) NOT NULL,
数量 int NOT NULL,
每天次数 int NOT NULL,
每次更换 VARCHAR(50) NOT NULL 
)
INSERT INTO G0626 VALUES (1,'药A',10,2,'50mg')
INSERT INTO G0626 VALUES (1,'药B',10, 2, '20mg')
INSERT INTO G0626 VALUES (2,'药C',5, 1, '15mg')
INSERT INTO G0626 VALUES (3,'药D',3, 1, '25mg')
```

```sql
WITH t1 AS
(
SELECT a.组号,a.药品名,ROUND(a.数量/a.每天次数,0) AS 数,
a.每次更换,b.help_topic_id
FROM G0626 a
LEFT JOIN mysql.help_topic b 
ON b.help_topic_id<=a.每天次数 AND b.help_topic_id>0
) -- 每天次数都是大于0的
,t2 AS(
SELECT 组号, help_topic_id,组号 AS N
FROM t1 GROUP BY 组号,help_topic_id
) --新建组号
,t3 AS(
SELECT *
FROM t2 WHERE help_topic_id=1
UNION ALL 
SELECT 组号,help_topic_id,
(SELECT MAX(组号) FROM t2)+ROW_NUMBER() OVER (ORDER BY 组号,help_topic_id) AS N
FROM t2 WHERE help_topic_id>1
) --确定需要等分的稠密排序
SELECT a.组号,a.药品名,a.数,a.每次更换
FROM t1 a LEFT JOIN t3 b ON a.组号=b.组号 AND a.help_topic_id=b.help_topic_id
ORDER BY N,药品名
```

### 递归连接加空格

有如下一张表G0621，希望得到如下结果：

| ID   | PARENTID | PRODUCTRUENAME | ORDERID |
| ---- | -------- | -------------- | ------- |
| 1    | (null)   | 汽车           | 1       |
| 2    | 1        | 车身           | 1->2    |
| 4    | 2        | 车门           | 1->2->4 |
| 5    | 2        | 驾驶舱         | 1->2->5 |
| 6    | 2        | 行李舱         | 1->2->6 |
| 3    | 1        | 发动机         | 1->3    |
| 7    | 3        | 气缸           | 1->3->7 |
| 8    | 3        | 活塞           | 1->3->8 |

即根据父ID（PARENTID）来逐级显示产品名和层级序号

**测试数据**

```sql
CREATE TABLE G0621 (
    ID INT,
    PRODUCTNAME VARCHAR(64),
    PARENTID INT
)
INSERT INTO G0621 VALUES ( 1,'汽车',NULL)
INSERT INTO G0621 VALUES ( 2,'车身',1)
INSERT INTO G0621 VALUES ( 3,'发动机',1)
INSERT INTO G0621 VALUES ( 4,'车门',2)
INSERT INTO G0621 VALUES ( 5,'驾驶舱',2)
INSERT INTO G0621 VALUES ( 6,'行李舱',2)
INSERT INTO G0621 VALUES ( 7,'气缸',3)
INSERT INTO G0621 VALUES ( 8,'活塞',3)
```

```sql
WITH RECURSIVE cte AS (
  SELECT a.ID,a.PARENTID, a.PRODUCTNAME, 0 AS ctelevel,CAST(a.ID AS CHAR(200)) AS ORDERID
  FROM G0621 a
  WHERE ID=1
  UNION ALL
  SELECT t.ID,t.PARENTID, t.PRODUCTNAME, cte.ctelevel+1,CONCAT(cte.ORDERID, '->', t.ID) AS ORDERID
  FROM G0621 t
  JOIN cte ON t.PARENTID = cte.ID
)
SELECT ID,PARENTID,CONCAT(SPACE(4*ctelevel),PRODUCTNAME) PRODUCTNAME,ORDERID
FROM cte ORDER BY ORDERID;
```

MySQL`SPACE(N)`函数用于在两个字符串之间添加空格,N是一个整数，指定我们要添加的空格数.

### *月第一天迭代过滤

有如下一张表G0620，希望得到如下结果：

| 姓名 | 月份 | 总天数 |
| ---- | ---- | ------ |
| 李四 | 5    | 9      |
| 李四 | 6    | 5      |
| 王五 | 4    | 6      |
| 王五 | 5    | 31     |
| 王五 | 6    | 5      |
| 张三 | 5    | 8      |
| 张三 | 6    | 1      |

解释：张三在5月请假天数为：6+2=8天
张三在6月请假天数为：1天
李四在5月请假天数为：2+7天
李四在6月请假天数为：6天
王五在4月请假天数为：6天
王五在5月请假天数为：31天
王五在6月请假天数为：6天

**测试数据**

```sql
CREATE TABLE G0620
(
请假时间 DATE,
请假时间到 DATE,
姓名 VARCHAR(20)
)

INSERT INTO G0620 VALUES ('2020-5-20','2020-5-25','张三');
INSERT INTO G0620 VALUES ('2020-5-30','2020-6-1','张三');
INSERT INTO G0620 VALUES ('2020-5-24','2020-5-25','李四');
INSERT INTO G0620 VALUES ('2020-5-25','2020-6-5','李四');
INSERT INTO G0620 VALUES ('2020-4-25','2020-6-5','王五');
```

```sql
SELECT 姓名,MONTH(请假时间) 月份,
SUM(TIMESTAMPDIFF(DAY,请假时间,请假时间到)+1) 总天数
FROM
(
SELECT 姓名,
CASE WHEN T2.help_topic_id=0 --3 请假时间和请假时间到的月份相同
THEN T1.请假时间 
ELSE DATE_ADD(T1.StartDay,INTERVAL T2.help_topic_id MONTH) END 请假时间,--4 下一月第一天
CASE WHEN DATE_ADD(DATE_ADD(T1.StartDay,INTERVAL T2.help_topic_id+1 MONTH),INTERVAL -1 DAY)<T1.请假时间到
THEN DATE_ADD(DATE_ADD(T1.StartDay,INTERVAL T2.help_topic_id+1 MONTH),INTERVAL -1 DAY) 
ELSE T1.请假时间到 END 请假时间到
FROM
(SELECT 请假时间,请假时间到,姓名,DATE_ADD(请假时间,INTERVAL 1-DAY(请假时间) DAY) StartDay --1 求请假时间月的第一天
FROM G0620) T1 JOIN
mysql.help_topic T2 ON T2.help_topic_id<=TIMESTAMPDIFF(MONTH,T1.请假时间,T1.请假时间到)+1 --2 使用help_topic_id，范围匹配
) T WHERE T.请假时间<=T.请假时间到 --5 过滤时间不符条件
GROUP BY 姓名,MONTH(请假时间) ORDER BY 1,2
```

### 组连接

有如下一张表G0616,希望得到如下结果：

| 品名 | 规格   | 颜色   | 数量 | IDs  |
| ---- | ------ | ------ | ---- | ---- |
| aaa  | a-1    | 红色   | 100  | 1,3  |
| bbb  | (Null) | 红色   | 120  | 2,5  |
| ccc  | c-1    | (Null) | 50   | 4    |

即相同品种，规格，颜色的数据行的数量进行汇总，同时合并他们汇总前的ID为IDs

**测试数据**

```sql
CREATE TABLE G0616
(
ID INT,
品名 VARCHAR(23),
规格 VARCHAR(23),
颜色 VARCHAR(22),
数量 INT
)
INSERT INTO G0616 VALUES(1,'AAA','A-1','红色',50) ;
INSERT INTO G0616 VALUES(2,'BBB',NULL,'红色',60) ;
INSERT INTO G0616 VALUES(3,'AAA','A-1','红色',50) ;
INSERT INTO G0616 VALUES(4,'CCC','C-1',NULL,50) ;
INSERT INTO G0616 VALUES(5,'BBB',NULL,'红色',60) ;
```

```sql
SELECT a.品名,a.规格,a.颜色,SUM(a.数量) AS 数量,GROUP_CONCAT(a.ID SEPARATOR ',') AS IDs
FROM G0616 a GROUP BY a.品名,a.规格,a.颜色
```

### 分组过滤关联

有如下一张表G0615，希望按工号查找日期(RecDate)相同的记录数>1，且Rectime=Time4

预计结果如下：

| workid | RecDate    | RecTime | Time4 |
| ------ | ---------- | ------- | ----- |
| 161181 | 2021-05-03 | 13:01   | 13:01 |

**测试数据**

```sql
CREATE TABLE G0615(
workid VARCHAR(20),
RecDate DATE,
RecTime VARCHAR(20),
Time4 VARCHAR(20)
)
INSERT INTO G0615 VALUES ('161181','2021-05-03','13:01','13:01');
INSERT INTO G0615 VALUES ('161181','2021-05-03','14:01','15:01');
INSERT INTO G0615 VALUES ('161181','2021-05-06','13:01','13:01');
INSERT INTO G0615 VALUES ('161181','2021-05-07','13:01','13:01');
INSERT INTO G0615 VALUES ('161181','2021-05-08','13:01','13:01');
INSERT INTO G0615 VALUES ('161181','2021-05-09','13:01','13:01');
```

```sql
SELECT b.* 
FROM 
G0615 b JOIN
(SELECT workid,RecDate
FROM G0615 
GROUP BY workid,RecDate HAVING COUNT(1)>1) a ON a.RecDate=b.RecDate AND a.workid=b.workid WHERE b.RecTime=b.Time4
```

### 串拆分和模糊匹配

有如下一张表G0614A

希望得到这个结果：

| 用户编号 | 用户地址   | 金额 | 微信用户订单号 |
| -------- | ---------- | ---- | -------------- |
| 001      | 花开村1号  | 10   | 90002          |
| 002      | 花开村2号  | 15   | 90002          |
| 003      | 花开村6号  | 15   | (null)         |
| 004      | 花开村5号  | 16   | 90002          |
| 005      | 花开村12号 | 8    | 90003          |

**测试数据**

```sql
CREATE TABLE G0614A
(
用户编号 VARCHAR(10),
用户地址 VARCHAR(20),
金额 INT
)

INSERT INTO G0614A VALUES
('001','花开村1号',10),
('002','花开村2号',15),
('003','花开村6号',15),
('004','花开村5号',16),
('005','花开村12号',8)

CREATE TABLE F0923B
(
微信用户订单号 VARCHAR(10),
用户编号 VARCHAR(50)
)

INSERT INTO F0923B VALUES
('90002','001,002,004'),
('90003','005')
```

```sql
--1 利用help_topic_id和substring_index()
SELECT d.用户编号,d.用户地址,d.金额,c.微信用户订单号
FROM G0614A d LEFT JOIN 
(SELECT  b.微信用户订单号,
    SUBSTRING_INDEX(SUBSTRING_INDEX(b.用户编号,',',help_topic_id+1),',',-1) AS num 
FROM 
    mysql.help_topic a,F0923B b
WHERE 
    help_topic_id < LENGTH(b.用户编号)-LENGTH(REPLACE(b.用户编号,',',''))+1) c
ON d.用户编号=c.num
--2 like匹配
SELECT 用户编号,用户地址,金额,
(
SELECT t2.微信用户订单号
FROM F0923B t2 WHERE CONCAT(',',t2.用户编号,',') LIKE CONCAT('%,',t1.用户编号,',%') LIMIT 1
) AS 微信用户订单号
FROM G0614A t1
```

select * from 表名 where 字段名 like 对应值(子串)

作用：主要是针对字符型字段的，在一个字符型字段列中检索包含对应子串的

Like主要支持两种通配符，分别是"_"和"%"。

1、"_"代表匹配1个任意字符，常用于充当占位符；

2、"%"代表匹配0个或多个任意字符

### 结束时间减一

有如下一张表G0613，

希望得到如下结果：

| CODE | INVTP | START_DATE | END_DATE   |
| ---- | ----- | ---------- | ---------- |
| 1001 | A     | 2021-01-01 | 2021/3/1   |
| 1001 | B     | 2021-03-02 | 2021-03-31 |
| 1001 | c     | 2021-04-01 | 9999-12-31 |
| 1002 | AA    | 2021-01-01 | 2021-02-27 |
| 1002 | BB    | 2021-02-28 | 9999-12-31 |
| 1003 | cc    | 2021-01-01 | 9999-12-31 |

该如何写这个SQL？

解释：同一个CODE组，DATE从小到大排序，需要实现每个日期到下一个日期的前一天作为结束日期，例如：CODE为1001组，第一个日期是2021-01-01，第二个日期是2021-03-02，那么第一个日期2021-01-01的结束日期就是2021-03-01，以此类推，如果是最后一个日期，那么结束日期默认为：9999-12-31

**测试数据**

```sql
CREATE TABLE G0613
(
CODE INT,
INVTP VARCHAR(10),
DATE DATE
)
INSERT INTO G0613 VALUES (1001,'A','2021/1/1');
INSERT INTO G0613 VALUES (1001,'B','2021/3/2');
INSERT INTO G0613 VALUES (1001,'C','2021/4/1');
INSERT INTO G0613 VALUES (1002,'AA','2021/1/1');
INSERT INTO G0613 VALUES (1002,'BB','2021/2/28');
INSERT INTO G0613 VALUES (1003,'CC','2021/1/1');
```

```sql
--1 使用lead（）
SELECT CODE,INVTP,DATE AS START_DATE,
CASE WHEN LEAD(DATE,1,STR_TO_DATE('9999-12-31','%Y-%m-%d')) OVER(PARTITION BY CODE ORDER BY DATE)= STR_TO_DATE('9999-12-31','%Y-%m-%d') 
THEN STR_TO_DATE('9999-12-31','%Y-%m-%d') ELSE
DATE_SUB(LEAD(DATE,1,STR_TO_DATE('9999-12-31','%Y-%m-%d')) OVER(PARTITION BY CODE ORDER BY DATE) ,INTERVAL 1 DAY) END AS END_DATE
FROM
G0613
--2 两表关联
WITH cte AS
(SELECT * ,ROW_NUMBER() OVER(ORDER BY CODE,INVTP) RN 
FROM G0613)
SELECT a.CODE,a.INVTP,a.DATE AS START_DATE,
IFNULL(DATE_SUB(b.DATE,INTERVAL 1 DAY),'9999-12-31') AS END_DATE
FROM CTE a LEFT JOIN CTE b
ON a.RN+1=b.RN AND b.CODE=a.CODE

```

### 两表条件过滤

有如下一张表G0612，挑选出带有选中字样的数据

规则如下： 同一类型的数据A_Type 按照A_NO的顺序 时间依次是变大的，如果时间有异常 ，则为条数异常的数据

如 id 为2的数据时间应该比id为3的数据小才对，但是比id为3的大则筛选出来，id为6，11都是同一规则，结果如下：

| ID   | A_Type | A_NO | A_Time | rid  |
| ---- | ------ | ---- | ------ | ---- |
| 2    | A      | 2    | 43492  | 2    |
| 6    | B      | 1    | 43610  | 1    |
| 11   | C      | 2    | 43633  | 2    |

**测试数据**

```sql
CREATE TABLE G0612(
    ID INT NULL,
    A_Type VARCHAR(50) NULL,
    A_NO INT NULL,
    A_Time DATETIME NULL
)
-- 插入语句
INSERT INTO G0612 VALUES (1,'A',1,'2019/1/21')
INSERT INTO G0612 VALUES (2,'A',2,'2019/1/27')
INSERT INTO G0612 VALUES (3,'A',3,'2019/1/23')
INSERT INTO G0612 VALUES (4,'A',4,'2019/1/24')
INSERT INTO G0612 VALUES (5,'A',5,'2019/1/25')
INSERT INTO G0612 VALUES (6,'B',1,'2019/5/25')
INSERT INTO G0612 VALUES (7,'B',2,'2019/5/22')
INSERT INTO G0612 VALUES (8,'B',3,'2019/5/23')
INSERT INTO G0612 VALUES (9,'B',4,'2019/5/24')
INSERT INTO G0612 VALUES (10,'C',1,'2019/6/12')
INSERT INTO G0612 VALUES (11,'C',2,'2019/6/17')
INSERT INTO G0612 VALUES (12,'C',3,'2019/6/14')
INSERT INTO G0612 VALUES (13,'D',1,'2019/7/8')
INSERT INTO G0612 VALUES (14,'D',2,'2019/7/9')
INSERT INTO G0612 VALUES (15,'D',3,'2019/7/10')
INSERT INTO G0612 VALUES (16,'D',4,'2019/7/11')
```

```sql
-- 1
SELECT a.ID,a.A_Type,a.A_NO,a.A_Time,a.A_NO AS rid
FROM G0612 a,G0612 b WHERE a.A_Type=b.A_Type AND a.A_NO=b.A_NO-1 AND a.A_Time>b.A_Time 
-- 2
SELECT a.ID,a.A_Type,a.A_NO,a.A_Time,a.A_NO AS rid
FROM G0612 a WHERE EXISTS(SELECT * FROM G0612 WHERE A_Type=a.A_Type AND A_NO=a.A_NO+1 AND A_Time<a.A_Time LIMIT 1)
```



### *时间差稠密排序序列

有如下一张表G0609，

以B列和C列为标准 首先判断B列相同 并且本次出现时间与上次时间大于24小时为有效 否则 标识为相同编号

1、2行 B列均为 aa 但是日期相差大于24小时 则标识不同 。
4、5行 B列不同 则标识不同。

查成这样结果:

| A    | B    | C                    | D    |
| ---- | ---- | -------------------- | ---- |
| 1    | aa   | 2018-01-25 01:26:06. | 1    |
| 2    | aa   | 2018-01-18 11:13:17, | 2    |
| 3    | aa   | 2018-01-06 17:30:56. | 3    |
| 4    | aa   | 2018-01-10 18:30:30. | 4    |
| 5    | as   | 2018-01-10 16:01:20. | 5    |
| 6    | as   | 2018-01-09 19:54:35. | 5    |
| 7    | bb   | 2018-01-10 22:43:40  | 6    |
| 8    | bb   | 2018-01-10 22:44:00  | 6    |
| 9    | bd   | 2018/1/10 22:43:50   | 7    |
| 10   | bd   | 2018-01-01 11:24:57  | 8    |

**测试数据**

```sql
CREATE TABLE G0609 (
A INT,
B VARCHAR(10),
C DATETIME
)
INSERT INTO G0609 VALUES (1,'aa','2018/1/25 01:26:06');
INSERT INTO G0609 VALUES (2,'aa','2018/1/18 11:13:17');
INSERT INTO G0609 VALUES (3,'aa','2018/1/6 17:30:56');
INSERT INTO G0609 VALUES (4,'aa','2018/1/10 18:30:30');
INSERT INTO G0609 VALUES (5,'as','2018/1/10 16:01:20');
INSERT INTO G0609 VALUES (6,'as','2018/1/9 19:54:35');
INSERT INTO G0609 VALUES (7,'bb','2018/1/10 22:43:40');
INSERT INTO G0609 VALUES (8,'bb','2018/1/10 22:44:00');
INSERT INTO G0609 VALUES (9,'bd','2018/1/10 22:43:50');
INSERT INTO G0609 VALUES (10,'bd','2018/1/1 11:24:57');
```

```sql
-- 可以实现还不完善，排序是dense_rank()的跳序
SELECT a.A,a.B,a.C,
CASE WHEN a.B=b.B AND ABS(TIMESTAMPDIFF(HOUR,a.C,b.C))<24 THEN c.topic-1 ELSE c.topic END AS D
FROM G0609 a LEFT JOIN G0609 b ON a.A-1=b.A
 LEFT JOIN 
(SELECT help_topic_id+1 AS topic
FROM mysql.help_topic ORDER BY help_topic_id) c ON a.A=c.topic
-- 先判断24内的标为0，将1的取行号，最后dense_rank()
WITH cte
AS
(SELECT *,CASE WHEN flag=1 THEN 
ROW_NUMBER() OVER (PARTITION BY B,flag ORDER BY A) END AS RN
FROM (SELECT *,
CASE WHEN ABS(TIMESTAMPDIFF(HOUR,LAG(C,1,C-1) OVER(PARTITION BY B ORDER BY A),C))<24 THEN 0 ELSE 1 END AS flag
FROM G0609) AS T)

SELECT a.A,a.B,a.C,DENSE_RANK() OVER(ORDER BY a.B,a.RN) AS D
FROM cte AS a;
```

### 条件合并

有如下一张表G0607：

求出每3个或2个相加的和等于10，结果如下：

| NUM1 | NUM2 | NUM3 |
| ---- | ---- | ---- |
| 5    | 3    | 2    |
| 5    | 4    | 1    |
| 6    | 3    | 1    |
| 7    | 2    | 1    |
| 6    | 4    |      |
| 7    | 3    |      |
| 8    | 2    |      |
| 9    | 1    |      |

**测试数据**

```sql
CREATE TABLE G0607
(
Num INT
)
INSERT INTO G0607 VALUES 
(1),(2),(3),(4),(5),(6),(7),(8),(9)
```

```sql
select a.NUM NUM1,b.NUM NUM2,c.NUM NUM3
from G0607 a,G0607 b,G0607 c
where (a.NUM+b.NUM+c.NUM=10) and a.NUM>b.NUM and b.NUM>c.NUM
union all
select a.NUM NUM1,b.NUM NUM2
from G0607 a,G0607 b
where (a.NUM+b.NUM=10) and a.NUM>b.NUM
```

### 简单行转列

怎么把下面的表 G0606,查成这样1个结果

| YEAR | M1   | M2   | M3   | M4   |
| ---- | ---- | ---- | ---- | ---- |
| 2020 | 9.1  | 9.2  | 9.3  | 9.4  |
| 2021 | 8.1  | 8.2  | 8.3  | 8.4  |

**测试数据**

```sql
CREATE TABLE G0606
(YEAR VARCHAR(10),
MONTH VARCHAR(10),
AMOUNT NUMERIC(18,1)
)
INSERT INTO G0606 VALUES
('2020','1',9.1),
('2020','2',9.2),
('2020','3',9.3),
('2020','4',9.4),
('2021','1',8.1),
('2021','2',8.2),
('2021','3',8.3),
('2021','4',8.4)
```

```sql
SELECT YEAR,
MAX(CASE WHEN MONTH='1' THEN AMOUNT END) AS M1,
MAX(CASE WHEN MONTH='2' THEN AMOUNT END) AS M2,
MAX(CASE WHEN MONTH='3' THEN AMOUNT END) AS M3,
MAX(CASE WHEN MONTH='4' THEN AMOUNT END) AS M4
FROM G0606 GROUP BY YEAR
```



### 求和与求累加和

有如下一张表G0602，求出NAME中每组累加/每组总数的比例大于0.6的ID和NAME 

解释：从题目意思可以看出关羽组的总数为14，从ID为1到5分别累加后的结果分别为1,3,9,11,14，只有9，11,14除以总数14才大于0.6，所以返回的结果ID为3,4,5，同样曹操组为7和8

返回结果：ID，NAME，NUM列

**测试数据**

```sql
CREATE TABLE G0602
(
ID INT,
NAME VARCHAR(10),
NUM INT
)
INSERT INTO G0602 VALUES
(1,'关羽',1),
(2,'关羽',2),
(3,'关羽',6),
(4,'关羽',2),
(5,'关羽',3),
(6,'曹操',2),
(7,'曹操',3),
(8,'曹操',3)
```

```sql
SELECT a.ID,a.NAME,a.NUM
FROM 
(SELECT (SUM(NUM) OVER(PARTITION BY NAME ORDER BY ID))/SUM(NUM) OVER(PARTITION BY NAME) rt,NAME,ID,NUM
FROM G0602) a WHERE a.rt>0.6
```



### 交集条件

有一张成绩表G0601，表结构为G0601(StuID,CID,Course)，分部对应是学生ID，课程ID和学生成绩，有如下测试数据

查询出既学过'001'课程，也学过'003'号课程的学生ID，返回StuID列。要求用两种方法求解！

**测试数据**

```sql
CREATE TABLE G0601
(
StuID INT,
CID VARCHAR(10),
Course INT
)
INSERT INTO G0601 VALUES
(1,'001',67),
(1,'002',89),
(1,'003',94),
(2,'001',95),
(2,'002',88),
(2,'004',78),
(3,'001',94),
(3,'002',77),
(3,'003',90)
```

```sql
select StuID
from G0601 where CID='001' or CID='003' group by StuID having count(1)=2
--2
SELECT t1.StuID FROM
(SELECT StuID
FROM G0601 a WHERE a.CID='001') t1
INNER JOIN 
(SELECT StuID
FROM G0601 b WHERE b.CID='003') t2
ON t2.StuID=t1.StuID
```



### like连接

一个表G0531中有A，B两列，查询出B列中完全存在于A列的记录。返回结果：A，B两列

**测试数据**

```sql
CREATE TABLE G0531
(
A VARCHAR(100),
B VARCHAR(100)
)
INSERT INTO G0531 VALUES
('SQL数据库开发','数据库'),
('北京','中国'),
('新加坡城','新加坡')
```

```sql
SELECT A,B
FROM G0531 WHERE A LIKE CONCAT('%',B,'%')
```

### 整体和部分求平均值

给如下两个表，写一个查询语句，求出在每一个工资发放日，每个部门的平均工资与公司的平均工资的比较结果 （高 / 低 / 相同）。表：G0530A(工资表)其中employee_id 字段是下表G0530B(员工表)中 employee_id 字段的外键。

解释 在5月，公司的平均工资是 (9000+6000+10000)/3 = 8333.33... 由于部门 '1' 里只有一个 employee_id 为 '1' 的员工，所以部门 '1' 的平均工资就是此人的工资 9000 。因为 9000 > 8333.33 ，所以比较结果是 'higher'。第二个部门的平均工资为 employee_id 为 '2' 和 '3' 两个人的平均工资，为 (6000+10000)/2=8000 。因为 8000 < 8333.33 ，所以比较结果是 'lower' 。在4月用同样的公式求平均工资并比较，比较结果为 'same' ，因为部门 '1' 和部门 '2' 的平均工资与公司的平均工资相同，都是 7000 。

对于如上样例数据，结果为： 

| AY_MONTH | DEPARTMENT_ID | COMPARISON |
| -------- | ------------- | ---------- |
| 2021-04  | 1             | SAME       |
| 2021-05  | 1             | HIGHER     |
| 2021-04  | 2             | SAME       |
| 2021-05  | 2             | LOWER      |

**测试数据**

```sql
CREATE TABLE G0530A
(
ID INT,
Employee_ID INT,
Amount INT,
Pay_Date DATE
)
INSERT INTO G0530A VALUES
(1,1,9000,'2021-05-31'),
(2,2,6000,'2021-05-31'),
(3,3,10000,'2021-05-31'),
(4,1,7000,'2021-04-30'),
(5,2,6000,'2021-04-30'),
(6,3,8000,'2021-04-30')

CREATE TABLE G0530B
(
Employee_ID INT,
Department_ID INT
)
INSERT INTO G0530B VALUES
(1,1),
(2,2),
(3,2)
```

```sql
select DATE_FORMAT(a.Pay_Date,'%Y-%m') as PAY_MONTH,Department_ID,case when  
sum(Amount)/count(1)>(select sum(Amount)/count(1) as avgsumamount
from G0530A where a.Pay_Date=Pay_Date) then 'HIGHER' else (case when sum(Amount)/count(1)<(select sum(Amount)/count(1) as avgsumamount
from G0530A where a.Pay_Date=Pay_Date) then 'LOWER' else 'SAME' end) end 
as COMPARISON
from G0530A a left join G0530B b on a.Employee_ID=b.Employee_ID group by a.Pay_Date,b.Department_ID
--求公司平均工资
select sum(Amount)/count(1) as avgsumamount
from G0530A group by Pay_Date
--求部门平均工资
select sum(Amount)/count(1) as avgamount
from G0530A a left join G0530B b on a.Employee_ID=b.Employee_ID group by a.Pay_Date,b.Department_ID
--2
SELECT a.pay_month,a.department_id,CASE WHEN salary>salary_ttl THEN 'HIGHER'
WHEN salary=salary_ttl THEN 'SAME' ELSE 'LOWER' END AS comparsion
FROM
(SELECT LEFT(pay_date,7) pay_month,department_id,AVG(amount) salary
FROM g0530a a LEFT JOIN g0530b b ON a.employee_id=b.employee_id GROUP BY LEFT(pay_date,7),department_id) a
LEFT JOIN 
(SELECT LEFT(pay_date,7) pay_month,AVG(amount) salary_ttl
FROM g0530a a LEFT JOIN g0530b b ON a.employee_id=b.employee_id GROUP BY LEFT(pay_date,7)) b
ON a.pay_month=b.pay_month
```



### 求最大值编号名称

有如下两张表

表: G0529A(候选人)，表: G0529B(选票)

id 是自动递增的主键，CandidateId 是 T0604A 表中的 id. 请编写 sql 语句来找到当选者的名字，即选票最多的候选者。上面的例子将返回当选者 B，因为他获得了2票，其他人获得了1票或0票

返回结果：

NAME

B

注意:你可以假设没有平局，换言之，最多只有一位当选者。

**测试数据**

```sql
CREATE TABLE G0529A
(
ID INT,
Name VARCHAR(10)
)

INSERT INTO G0529A VALUES
(1,'A'),
(2,'B'),
(3,'C'),
(4,'D'),
(5,'E')

CREATE TABLE G0529B
(
ID INT,
CandidateID INT
)
INSERT INTO G0529B VALUES
(1,2),
(2,4),
(3,3),
(4,2),
(5,5)
```

```sql
select name
from
G0529A
where ID=
(select CandidateID
from  G0529B group by CandidateID order by count(1) desc limit 1)
```

### *求频数中位数

G0526表保存数字的值Num,及其个数Frq。

   在此表中，数字为 0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3，所以中位数是 (0 + 0) / 2 = 0。

请编写一个查询来查找所有数字的中位数并将结果命名为 median 。注意：什么是中位数？当一串数字是奇数个时，例如8,3,5,1,4。我们按顺序排列后为：1,3,4,5,8。那么4就是中位数 当一串数字为偶数个时，例如8,3,5,1,4,2。我们按顺序排列后为：1,2,3,4,5,8。那么(3+4)/2=3.5就是中位数。

**测试数据**

```sql
CREATE TABLE G0526
(
Num INT,
Frq INT
)
INSERT INTO G0526 VALUES
(0,7),
(1,1),
(2,3),
(3,1)
```

```sql
SELECT
AVG(t.Num) AS MEDIAN
FROM
(SELECT a.Num,a.Frq,
(SELECT SUM(Frq) FROM G0526 b WHERE b.Num<=a.Num) AS asc_frequency,
(SELECT SUM(Frq) FROM G0526 c WHERE c.Num>=a.Num) AS desc_frequency
FROM
G0526 a) t
WHERE t.asc_frequency>=(SELECT SUM(Frq) FROM G0526)/2 AND t.desc_frequency>=(SELECT SUM(Frq) FROM G0526)/2
```

### 分组条件

有如下一张表G0524：

写一条SQL查询语句获取合作过至少三次的演员和导演的 id 对 (actor_id, director_id) 预计结果：

| actor_id | director_id |
| -------- | ----------- |
| 1        | 1           |

唯一的 id 对是 (1, 1)，他们恰好合作了 3 次。

**测试数据**

```sql
CREATE TABLE G0524
(
actor_id INT,
director_id INT,
timestamps INT
)
INSERT INTO G0524 VALUES
(1,1,0),
(1,1,1),
(1,1,2),
(1,2,3),
(1,2,4),
(2,1,5),
(2,1,6)
```

```sql
SELECT actor_id,director_id
FROM
(SELECT actor_id,director_id
FROM G0524 GROUP BY actor_id,director_id HAVING COUNT(1)>=3) a
```

### 绝对差为1

几个朋友来到电影院的售票处，准备预约连续空余座位。你能利用表 F0831，帮他们写一个查询语句，获取所有空余座位，并将它们按照 seat_id 排序后返回吗？

注意：seat_id 字段是一个自增的整数，free 字段是布尔类型（'1' 表示空余， '0' 表示已被占据）。连续空余座位的定义是大于等于 2 个连续空余的座位。

对于如上样例，返回如下结果。 

seat_id 

3

4

5

**测试数据**

```sql
CREATE TABLE G0523
(
seat_id INT,
free INT
)

INSERT INTO G0523 VALUES 
(1,1),
(2,0),
(3,1),
(4,1),
(5,1)
```

```sql
SELECT DISTINCT a.seat_id AS SEAT_ID
FROM
(SELECT seat_id,free
FROM G0523) a,
(SELECT seat_id,free
FROM G0523) b WHERE ABS(a.seat_id-b.seat_id)=1 AND a.free=1 AND b.free=1 
ORDER BY a.seat_id
```

### 连续两天登录

编写一个SQL查询，报告在首次登录后第二天再次登录的玩家的分数， 四舍五入到小数点后两位。换句话说， 您需要从首次登录日期开始计算至少连续两天登录的玩家数，把这个数字除以玩家总数。

查询结果格式如下所示：

FRACTION

0.33

解释：只有ID为1的玩家在第一天登录后才重新登录，所以答案是1/3=0.33

**测试数据**

```sql
CREATE TABLE G0519
(
player_id INT,
device_id INT,
event_date DATE,
games_played INT
)

INSERT INTO G0519 VALUES
(1,2,'2021-03-01',5),
(1,2,'2021-03-02',6),
(2,3,'2021-05-13',1),
(3,1,'2021-03-02',0),
(3,4,'2021-04-08',5)
```

```sql
SELECT ROUND(
(SELECT COUNT(DISTINCT A.player_id)
FROM
G0519 A,
(SELECT MIN(event_date) AS min_date,player_id
FROM G0519 GROUP BY player_id) B WHERE B.player_id=A.player_id AND DATE_ADD(B.min_date ,INTERVAL 1 DAY)=A.event_date)
/(SELECT COUNT(DISTINCT player_id)FROM G0519),2) AS FRACTION
```

### in的使用

有如下一张表F0825

列AID和BID是一对多的关系，希望将BID中同时存在3和5的AID找出来，预计的结果如下：

AID

2

**测试数据**

```sql
CREATE TABLE G0518
(
AID INT,
BID INT
)
INSERT INTO G0518 VALUES (1,2);
INSERT INTO G0518 VALUES (1,5);
INSERT INTO G0518 VALUES (2,1);
INSERT INTO G0518 VALUES (2,3);
INSERT INTO G0518 VALUES (2,5);
INSERT INTO G0518 VALUES (3,4);
INSERT INTO G0518 VALUES (3,2);
INSERT INTO G0518 VALUES (3,5);
```

```sql
SELECT AID
FROM G0518 WHERE BID IN (3,5) GROUP BY AID HAVING COUNT(1)=2
```

### 分组union all

有如下一张表G0517

| 姓名 | 今日金额 | 本周金额 | 本月金额 | 分组 |
| ---- | -------- | -------- | -------- | ---- |
| 张三 | 100.00   | 900.00   | 2700.00  | 1    |
| 李四 | 120.00   | 680.00   | 2900.00  | 1    |
| 王五 | 110.00   | 850      | 3000     | 2    |
| 马六 | 120.00   | 790.00   | 2800.00  | 2    |

| 姓名 | 今日金额 | 本周金额 | 本月金额 |
| ---- | -------- | -------- | -------- |
| 张三 | 100.00   | 900.00   | 2700.00  |
| 李四 | 120.00   | 680.00   | 2900.00  |
| 1组  | 220.00   | 1580.00  | 5600.00  |
| 王五 | 110.00   | 850.00   | 3000.00  |
| 马六 | 120.00   | 790.00   | 2800.00  |
| 2组  | 230.00   | 1640.00  | 5800.00  |
| 总计 | 450.00   | 3220.00  | 11400.00 |

想要得到以上结果

**测试数据**

```sql
CREATE TABLE G0517 (
姓名 VARCHAR(20),
今日金额 NUMERIC(18,2),
本周金额 NUMERIC(18,2),
本月金额 NUMERIC(18,2),
分组 INT
)

INSERT INTO G0517 VALUES('张三',100,900,2700,1);
INSERT INTO G0517 VALUES('李四',120,680,2900,1);
INSERT INTO G0517 VALUES('王五',110,850,3000,2);
INSERT INTO G0517 VALUES('马六',120,790,2800,2);
```

```sql
select 姓名,今日金额,本周金额,本月金额
from G0517 where 分组=1 
union all
select '1组' as 姓名,sum(今日金额) 今日金额,sum(本周金额) 本周金额,sum(本月金额) 本月金额
from G0517 where 分组=1 
select 姓名,今日金额,本周金额,本月金额
from G0517 where 分组=2
union all
select '2组' as 姓名,sum(今日金额) 今日金额,sum(本周金额) 本周金额,sum(本月金额) 本月金额
from G0517 where 分组=2
union all
select '总计' as 姓名,sum(今日金额) 今日金额,sum(本周金额) 本周金额,sum(本月金额) 本月金额
from G0517
--2
SELECT 姓名,今日金额,本周金额,本月金额
FROM
(SELECT 姓名,SUM(今日金额) 今日金额,SUM(本周金额) 本周金额,SUM(本月金额) 本月金额,分组
FROM G0517 GROUP BY 姓名,分组
UNION ALL
SELECT CONCAT(TRIM(分组),'组') AS 姓名,SUM(今日金额) 今日金额,SUM(本周金额) 本周金额,SUM(本月金额) 本月金额,分组
FROM G0517 GROUP BY TRIM(分组)+'组',分组
UNION ALL
SELECT '总计' AS 姓名,SUM(今日金额) 今日金额,SUM(本周金额) 本周金额,SUM(本月金额) 本月金额,1000 AS 分组
FROM G0517) t
ORDER BY t.分组,姓名 DESC
```



### 行号求余相等

有如下两张表G0516A

要求G0516B按顺序与G0516A的第一个WEEK1依次有序的组合，直到依次组合完毕，预计结果如下：

| WEEKS  | TEAMS |
| ------ | ----- |
| WEEK1  | TEAM1 |
| WEEK2  | TEAM2 |
| WEEK3  | TEAM3 |
| WEEK4  | TEAM1 |
| WEEK5  | TEAM2 |
| WEEK6  | TEAM3 |
| WEEK7  | TEAM1 |
| WEEK8  | TEAM2 |
| WEEK9  | TEAM3 |
| WEEK10 | TEAM1 |

**测试数据**

```sql
CREATE TABLE G0516A
(
WEEKS VARCHAR(10)
)
INSERT INTO G0516A VALUES
('WEEK1'),('WEEK2'),('WEEK3'),
('WEEK4'),('WEEK5'),('WEEK6'),
('WEEK7'),('WEEK8'),('WEEK9'),
('WEEK10')
CREATE TABLE G0516B
(
TEAMS VARCHAR(10)
)
INSERT INTO G0516B VALUES
('TEAM1'),('TEAM2'),('TEAM3')
```

```sql
SELECT WEEKS,TEAMS
FROM
(SELECT ROW_NUMBER() OVER(ORDER BY CAST(WEEKS AS SIGNED)) AS aid,WEEKS
FROM G0516A) a
LEFT JOIN 
(SELECT ROW_NUMBER() OVER(ORDER BY CAST(TEAMS AS SIGNED)) AS bid,TEAMS
FROM G0516B) b
ON a.aid%3=b.bid%3 ORDER BY CAST(a.WEEKS AS SIGNED)
```



### *条件递归相加

有如下一张表G0511,要求

任意几个数的和大于95并且小于100。

例如：ID为2,9,10的数相加：18+62+18=98，符合上述要求。

ID为2,4,5,10的数相加：18+26+35+18=97，也符合上述要求。

故要求使用一个SQL查询求解出所有符合要求的数据（SQL语句可以执行多次）结果显示列为ID，NUM。且NUM相加都在指定范围内

**测试数据**

```sql
CREATE TABLE G0511
(
ID INT,
NUM INT
)
INSERT INTO G0511 VALUES (1,20)
INSERT INTO G0511 VALUES (2,18)
INSERT INTO G0511 VALUES (3,22)
INSERT INTO G0511 VALUES (4,26)
INSERT INTO G0511 VALUES (5,35)
INSERT INTO G0511 VALUES (6,42)
INSERT INTO G0511 VALUES (7,37)
INSERT INTO G0511 VALUES (8,76)
INSERT INTO G0511 VALUES (9,62)
INSERT INTO G0511 VALUES (10,18)
```

[Locate](https://so.csdn.net/so/search?q=Locate&spm=1001.2101.3001.7020)函数主要的作用是判断一个字符串是否包含另一个字符串

Locate(substr,str) > 0，表示str字符串包含substr字符串；

Locate(substr,str) = 0，表示str字符串不包含substr字符串。

```sql
WITH RECURSIVE CTE AS(
SELECT *,
CAST(ID AS CHAR(2000)) PATH,
num TOTAL FROM g0511
UNION ALL 
SELECT B.ID,B.NUM,
CONCAT(A.PATH,'-',RTRIM(b.ID)),
A.TOTAL+B.NUM
FROM CTE A JOIN g0511 B ON A.ID<B.ID AND A.TOTAL<100
)
-- select * from CTE;
SELECT
ID,NUM
FROM g0511,(SELECT PATH FROM CTE WHERE TOTAL>95 AND TOTAL<100 ORDER BY RAND() LIMIT 1) a
WHERE LOCATE(CONCAT('-',RTRIM(ID),'-'),CONCAT('-',RTRIM(PATH),'-'))>0
```



### *创临时表关联过滤条件

有如下一张表G0510，A，B，C三列的数据在1-10之间随机取数（数据可重复）。
如何取出T1118表中不存在于1-10之间的数？
上面那种情况预计的结果如下：

| id   |
| ---- |
| 1    |
| 4    |
| 5    |
| 6    |
| 7    |
| 8    |
| 10   |

**测试数据**

```sql
CREATE TABLE G0510 (
A INT,
B INT,
C INT
);
INSERT INTO G0510 VALUES(2,9,3);
```

```sql
CREATE TEMPORARY TABLE  IF NOT EXISTS tempA(id INT);
 INSERT INTO tempA VALUES(1),(2),(3),(4),(5),(6),(7),(8),(9),(10);
 SELECT b.id
 FROM
 (SELECT A FROM g0510 
 UNION ALL 
 SELECT B FROM g0510
UNION ALL 
 SELECT C FROM g0510
 ) a RIGHT JOIN tempA b ON a.A=b.id
 WHERE a.A IS NULL
```

### 间断连续

有如下一张表G0509，同一个代码，在不同日期有个不同的金额，统计连续增长天数和连续负增长天数，预计结果如下：

| 日期       | 代码 | 金额 | 金额差异值 | 增长方向 | 连续增长天数 |
| ---------- | ---- | ---- | ---------- | -------- | ------------ |
| 2020-07-01 | A    | 2.00 | .00        | 正       | 1            |
| 2020-07-06 | A    | 4.00 | 2.00       | 正       | 2            |
| 2020-07-12 | A    | 3.00 | -1.00      | 负       | 1            |
| 2020-07-13 | A    | 4.00 | 1.00       | 正       | 1            |
| 2020-07-13 | B    | 2.00 | .00        | 正       | 1            |
| 2020-07-19 | B    | 4.00 | 2.00       | 正       | 2            |

**测试语句**

```sql
CREATE TABLE G0509 (
日期 DATETIME NULL,
代码 NVARCHAR(999) NULL,
金额 DECIMAL(18, 2) NULL,
金额差异值 DECIMAL(19, 2) NULL
)
INSERT INTO G0509 VALUES('2020-07-01','A',2,0)
INSERT INTO G0509 VALUES('2020-07-06','A',4,2)
INSERT INTO G0509 VALUES('2020-07-12','A',3,-1)
INSERT INTO G0509 VALUES('2020-07-13','A',4,1)
INSERT INTO G0509 VALUES('2020-07-13','B',2,0)
INSERT INTO G0509 VALUES('2020-07-19','B',4,2)
```

```sql
WITH t AS(SELECT 日期,代码,金额,金额差异值,
CASE WHEN 金额差异值>=0 THEN '正' ELSE '负' END 增长方向,
ROW_NUMBER() OVER(PARTITION BY 代码 ORDER BY 日期) rn
FROM G0509)
 SELECT 日期,代码,金额,金额差异值,增长方向,
 a.rn-IFNULL((SELECT b.rn FROM t b WHERE a.代码=b.代码 AND b.rn<a.rn AND b.增长方向<>a.增长方向 ORDER BY b.rn DESC LIMIT 1),0) 连续增长天数
 FROM t a
```



### 计数判断

有如下一张表G0508

productBar为产品编码，step为组装序号，timeUsed为组装用时。

也就是一个产品必须要经过1,2,3,4个步骤，才算组装成功，像2001000001为成功，2001000002为失败。

查询出每个产品是否组装成功，以及他们的组装用时。

预计得到如下结果：

| 产品编码   | 组装序号状态 | 组装用时 |
| ---------- | ------------ | -------- |
| 2001000001 | 成功         | 53       |
| 2001000002 | 失败         | 65       |
| 2001000003 | 成功         | 89       |

**测试语句**

```sql
CREATE TABLE G0508(
    productBar VARCHAR(10),
    step INT,
    timeUsed INT   
)
INSERT INTO G0508 VALUES('2001000001',1,10);
INSERT INTO G0508 VALUES('2001000001',2,13);
INSERT INTO G0508 VALUES('2001000001',3,12);
INSERT INTO G0508 VALUES('2001000001',4,18);
INSERT INTO G0508 VALUES('2001000002',1,11);
INSERT INTO G0508 VALUES('2001000002',2,31);
INSERT INTO G0508 VALUES('2001000002',3,23);
INSERT INTO G0508 VALUES('2001000003',1,21);
INSERT INTO G0508 VALUES('2001000003',2,22);
INSERT INTO G0508 VALUES('2001000003',3,23);
INSERT INTO G0508 VALUES('2001000003',4,23);
```

```sql
select productBar as 产品编码,case when count(1)=4 then '成功' else '失败' end 组装序号状态,sum(timeUsed) as 组装用时,from G0508 group by productBar;
```



### 条件相加

有如下一张表G0506，希望用SQL 将每列数据中存在奇数的数据统计出来，结果如下：

| IP   | NUM1 | NUM2 | NUM3 | 奇数个数 |
| ---- | ---- | ---- | ---- | -------- |
| 1    | 3333 | 4442 | 221  | 2        |
| 2    | 65   | 24   | 96   | 1        |

**测试语句**

```sql
CREATE TABLE G0506(
IP INT,
NUM1 INT,
NUM2 INT,
NUM3 INT
)
INSERT G0506
SELECT 1,3333,4442,221 
UNION ALL
SELECT 2,65,24,96
```

```sql
select IP,NUM1,NUM2,NUM3,(case when NUM1%2=1 then 1 else 0 end)+(case then NUM2%2=1 then 1 else 0 end)+(case then NUM3%2=1 then 1 else 0 end) as 奇数个数
from G0506 
```

### 连续组连接

有如下一张表G0505

对上述进行排序后，把数据集合在一个字符串中输出，如果有不连续用 , 表示，如果连续显示 起始号-终止号  输出结果如下：

| Result                                            |
| ------------------------------------------------- |
| A10000001-A10000004,A10000006,A10000009-A10000012 |

**测试语句**

```sql
CREATE TABLE G0505
(
    val VARCHAR(50)
)
insert into G0505(val) values('A10000003')
insert into G0505(val) values('A10000001')
insert into G0505(val) values('A10000002')
insert into G0505(val) values('A10000011')
insert into G0505(val) values('A10000004')
insert into G0505(val) values('A10000006')
insert into G0505(val) values('A10000009')
insert into G0505(val) values('A10000010')
insert into G0505(val) values('A10000012')
```

```sql
SELECT 
GROUP_CONCAT(CASE WHEN mi=mx THEN mi ELSE CONCAT(mi,'-',mx) END)  AS '结果'
FROM(
SELECT  MIN(val) mi,MAX(val) mx
FROM
(SELECT val,CONVERT(SUBSTRING(val,2),SIGNED)-rn AS rnnum
FROM (SELECT val,ROW_NUMBER() OVER(ORDER BY CAST(SUBSTRING(val,2,8) AS SIGNED)) AS rn
FROM G0505 ORDER BY val) a) b GROUP BY b.rnnum
) c
```

SQL Server使用STRING_AGG(合并)：多行数据合并成一个字符串，以逗号隔开。

STRING_SPLIT(拆分):一个字符串，拆分成多行。

字符串转化为数字的三种方式

```sql
SELECT 'abd'+0  # 结果为 0 
SELECT 'abd5'+0  # 结果为 0 
SELECT '5abd'+0  # 结果为 5
SELECT '5abd5'+0 # 结果为 5
select '2FF351C4AC744E2092DCF08CFD314420' + 0 # 结果为0
SELECT '55'+0 # 结果为 55
SELECT CAST(SUBSTRING('A10000003',2,8) AS SIGNED) --结果为：10000003
SELECT CONVERT(SUBSTRING('A10000003',2,8),SIGNED)

```

在使用CAST函数转换类型时，可以转换的类型是有限制的。这个类型可以是以下值其中的一个。也就是说，UNSIGNED 可以替换成：

二进制，同带binary前缀的效果 : BINARY    
字符型，可带参数 : CHAR()     
日期 : DATE     
时间: TIME     
日期时间型 : DATETIME     
浮点数 : DECIMAL      
整数 : SIGNED     
无符号整数 : UNSIGNED 

其他转换varchar的方法：

```sql
SELECT CAST(8 AS UNSIGNED) 
select -(-字段名) from 表名;
select 字段名+0 from 表名;
select concat(8,'0') 
```



### 整除减余

有如下两张表G0504A，G0504B

希望得到一下结果：

| ID   | TYPE     | 平均 |
| ---- | -------- | ---- |
| 1    | 1-SERIES | 33   |
| 2    | 1-SERIES | 33   |
| 3    | 1-SERIES | 34   |
| 4    | 2-SERIES | 19   |
| 5    | 2-SERIES | 19   |
| 6    | 2-SERIES | 19   |
| 7    | 2-SERIES | 19   |
| 8    | 2-SERIES | 23   |

要求结果不是小数，且type的平均相加等于总数的和就行。

例如1-SERIES包含ID为1,2,3这3行，他们的和为1,2的QUANTITY的和（因为ID为3没有值），所以我们需要将1,2的和平均分给1,2，3。这样结果的平均值就为100/3=33.33，但是要求结果不为小数，那么我们需要将他们都取为整数。最接近的平均数且整数为100的就为33,33,34。

**测试数据**

```sql
CREATE TABLE G0504A(
ID INT,
TYPE NVARCHAR(28)
)
INSERT INTO G0504A VALUES (1,'1-SERIES')
INSERT INTO G0504A VALUES (2,'1-SERIES')
INSERT INTO G0504A VALUES (3,'1-SERIES')
INSERT INTO G0504A VALUES (4,'2-SERIES')
INSERT INTO G0504A VALUES (5,'2-SERIES')
INSERT INTO G0504A VALUES (6,'2-SERIES')
INSERT INTO G0504A VALUES (7,'2-SERIES')
INSERT INTO G0504A VALUES (8,'2-SERIES')

CREATE TABLE G0504B(
ID INT,
QUANTITY INT
)
INSERT INTO G0504B VALUES (1,50)
INSERT INTO G0504B VALUES (2,50)
INSERT INTO G0504B VALUES (4,33)
INSERT INTO G0504B VALUES (5,33)
INSERT INTO G0504B VALUES (6,33)
```

```sql
SELECT a.ID,a.TYPE,CASE WHEN a.rn<>a.countnum 
THEN FLOOR(SUM(b.QUANTITY) OVER(PARTITION BY a.TYPE)/countnum) 
ELSE SUM(b.QUANTITY) OVER(PARTITION BY a.TYPE)-FLOOR(SUM(b.QUANTITY) OVER(PARTITION BY a.TYPE)/countnum)*(rn-1) END AS 平均
FROM 
(SELECT ID,TYPE, ROW_NUMBER() OVER(PARTITION BY TYPE ORDER BY ID) AS rn ,COUNT(1) OVER(PARTITION BY TYPE) AS countnum FROM G0504A)
 a LEFT JOIN G0504B b ON a.ID=b.ID
 --2
  WITH t AS(
 SELECT a.ID,TYPE, 
 SUM(b.QUANTITY) OVER(PARTITION BY a.TYPE) AS sumnum,
 ROW_NUMBER() OVER(PARTITION BY TYPE ORDER BY ID) AS rn,
 COUNT(1) OVER(PARTITION BY TYPE) AS countnum 
 FROM G0504A a LEFT JOIN G0504B b ON a.ID=b.ID
 )
 SELECT t.ID, 
 t.TYPE,
 CASE WHEN rn<>t.countnum THEN FLOOR(t.sumnum/t.countnum) ELSE t.sumnum-FLOOR((t.sumnum/t.countnum))*(rn-1) END AS 平均
 FROM t
```

mysql中整除方法：

```sql
select 10 div 3
select floor(10/3)
select cast(10/3 as unsigned)
```

### *连续天数问题

下表记录了夺冠球队的名称及年份：

**请写出一条 SQL 语句，查询出在此期间连续获得冠军的有哪些，其连续的年份的起止时间是多少？**

查询结果：

之前我们有讲解如何求解连续多少天的问题，这个题有点类似，但是也有点不一样的地方。

问题分析
一般连续性的问题，我们都需要使用笛卡尔积进行错位匹配，就是类似a.ID=b.ID+1的这种。这一题我们也可以使用类似的方法‘’‘。

具体代码如下：

```sql
CREATE TABLE t(TEAM varchar(20), Y int)
INSERT t(TEAM,Y)  VALUES
('活塞',1990),
('公牛',1991),
('公牛',1992),
('公牛',1993),
('火箭',1994),
('火箭',1995),
('公牛',1996),
('公牛',1997),
('公牛',1998),
('马刺',1999),
('湖人',2000),
('湖人',2001),
('湖人',2002),
('马刺',2003),
('活塞',2004),
('马刺',2005),
('热火',2006),
('马刺',2007),
('凯尔特人',2008),
('湖人',2009),
('湖人',2010);
```

```sql
SELECT a.TEAM,MIN(a.Y) lyear,MAX(a.Y) ryear
FROM
(SELECT *,ROW_NUMBER() OVER(PARTITION BY TEAM ORDER BY Y) AS num
FROM t) a WHERE EXISTS(
SELECT 1 FROM
(SELECT *,ROW_NUMBER() OVER(PARTITION BY TEAM ORDER BY Y) AS num
FROM t) b WHERE a.TEAM=b.TEAM AND (b.Y=a.Y-1 OR b.Y=a.Y+1) 
) GROUP BY a.TEAM,a.Y-a.num
--方法二
SELECT a.TEAM,MIN(a.Y) lyear,MAX(a.Y) ryear
FROM 
(SELECT *,ROW_NUMBER() OVER(PARTITION BY TEAM ORDER BY Y) AS num
FROM t) a
LEFT JOIN 
(SELECT *,ROW_NUMBER() OVER(PARTITION BY TEAM ORDER BY Y) AS num
FROM t) b ON a.TEAM=b.TEAM AND (b.Y=a.Y-1 OR b.Y=a.Y+1)
GROUP BY a.TEAM,a.Y-a.num
HAVING MIN(a.Y)<>MAX(a.Y)
```

**首先是给这些数据添加一列(在相同team条件下，按y排序)自然数的num列**

**其次是将a进行自匹配，匹配的条件是TEAM名称相同(b.TEAM=a.TEAM)，并且年份Y与前后的年份进行匹配(b.Y=a.Y-1 OR b.Y=a.Y+1)。**

最后在进行分组的时候，不仅对球队TEAM进行了分组，而且还对Y-RN进行了分组。为什么要对Y-RN进行分组呢？

公牛和湖人中间间隔了几年才重新连续夺冠，但是这里因为没有对Y-RN进行分组，导致这个球队和夺冠年份在进行匹配时都满足了。

**b.Y=a.Y-1 OR b.Y=aY+1只要有一个满足即可判断是连续的年份，实际上经过我们处理后确实满足上述条件，所以需要加上Y-RN进行第二次分组来判断中间是否有间隔的年份。因为如果有间隔，那么Y-RN就不是同一个值了。**

### 固定时间差

有如下一组数据G0428，字段有UID ，CALLBACK_DATE 

需求：
第一次召回不计费，后续如果间隔满7天计费，否则不计费，再后续距离上一次计费满7天计费，否则不计费，该如何写这个需求？

预计结果如下：

| UID  | CALLBACK_DATE | CHARGE |
| ---- | ------------- | ------ |
| 1    | 2020-04-01    | 不计费 |
| 1    | 2020-04-05    | 不计费 |
| 1    | 2020-04-10    | 不计费 |
| 1    | 2020-04-19    | 计费   |
| 2    | 2020-04-01    | 不计费 |
| 2    | 2020-04-15    | 计费   |
| 2    | 2020-04-16    | 不计费 |
| 2    | 2020-04-20    | 不计费 |

**测试数据**

```sql
CREATE TABLE G0428
(
UID INT NOT NULL ,
CALLBACK_DATE DATE NOT NULL
)
INSERT INTO G0428 VALUES (1,'2020-4-1')
INSERT INTO G0428 VALUES (1,'2020-4-5')
INSERT INTO G0428 VALUES (1,'2020-4-10')
INSERT INTO G0428 VALUES (1,'2020-4-19')
INSERT INTO G0428 VALUES (2,'2020-4-1')
INSERT INTO G0428 VALUES (2,'2020-4-15')
INSERT INTO G0428 VALUES (2,'2020-4-20')
INSERT INTO G0428 VALUES (2,'2020-4-16')
```

```sql
SELECT a.UID,a.CALLBACK_DATE,
CASE WHEN TIMESTAMPDIFF(DAY,LAG(CALLBACK_DATE,1) OVER(PARTITION BY UID ORDER BY CALLBACK_DATE),a.CALLBACK_DATE)>7 THEN '计费' ELSE  '不计费'
END CHARGE 
FROM 
G0428 a ORDER BY a.UID,a.CALLBACK_DATE
--
SELECT a.UID,a.CALLBACK_DATE,
CASE WHEN LAG(CALLBACK_DATE,1) OVER(PARTITION BY UID ORDER BY CALLBACK_DATE) IS NULL OR TIMESTAMPDIFF(DAY,LAG(CALLBACK_DATE,1) OVER(PARTITION BY UID ORDER BY CALLBACK_DATE),a.CALLBACK_DATE)<7 THEN '不计费' ELSE  '计费'
END CHARGE 
FROM 
G0428 a ORDER BY a.UID,a.CALLBACK_DATE
```



### 求和和累加

有如下一组数据G0426，字段有日期，类型，金额，希望得到如下结果：

| 月份    | 借款金额 | 还款金额 | 累计借款 | 累计还款 |
| ------- | -------- | -------- | -------- | -------- |
| 2021-01 | 50       | 30       | 50       | 30       |
| 2021-02 | 50       | 30       | 100      | 60       |
| 2021-03 | 50       | 30       | 150      | 90       |

即：借还款金额=当月金额之和
    累计借还款金额=年内借还款之和（如：1月=1月，2月=1月+2月，3月=1月+2月+3月）

**测试数据**

```sql
CREATE TABLE G0427
(
日期 DATE,
类型 VARCHAR(10),
金额 INT
)
INSERT INTO G0427 VALUES ('2021-01-01','借款',100)
INSERT INTO G0427 VALUES ('2021-01-31','借款',-50)
INSERT INTO G0427 VALUES ('2021-01-20','还款',50)
INSERT INTO G0427 VALUES ('2021-01-23','还款',-20)
INSERT INTO G0427 VALUES ('2021-02-01','借款',100)
INSERT INTO G0427 VALUES ('2021-02-28','借款',-50)
INSERT INTO G0427 VALUES ('2021-02-20','还款',50)
INSERT INTO G0427 VALUES ('2021-02-23','还款',-20)
INSERT INTO G0427 VALUES ('2021-03-01','借款',100)
INSERT INTO G0427 VALUES ('2021-03-31','借款',-50)
INSERT INTO G0427 VALUES ('2021-03-20','还款',50)
INSERT INTO G0427 VALUES ('2021-03-23','还款',-20)
```

```sql
SELECT b.月份,b.借款金额,b.还款金额,
SUM(b.借款金额) OVER (ORDER BY b.月份) AS 累计借款,
SUM(b.还款金额) OVER (ORDER BY b.月份) AS 累计还款
FROM (SELECT a.月份,SUM(CASE WHEN a.类型='借款' THEN a.金额 ELSE 0 END)AS 借款金额,
SUM(CASE WHEN a.类型='还款' THEN a.金额 ELSE 0 END)AS 还款金额
FROM (SELECT DATE_FORMAT(日期,'%Y-%m') AS 月份,类型,金额  FROM G0427) a
GROUP BY a.月份
 ) b ;
```

### lag使用

有如下一组数据G0426，字段为Carrier_Name ，OrderNumber ，ReSumCost 

相同OrderNumber只取一条ReSumCost的结果，希望得出以下结果

| Carrier_Name | OrderNumber    | ReSumCost | NewReSumCost |
| ------------ | -------------- | --------- | ------------ |
| 临时         | JOY19080300003 | 170.00    | 170.00       |
| 张三         | JOY19080300003 | 170       | 0            |
| 临时         | JOY19080300004 | 196.5     | 196.5        |
| 张三         | JOY19080300004 | 196.50    | .00          |
| 临时         | JOY19080300006 | 458.80    | 458.80       |
| 张三         | JOY19080300006 | 458.80    | .00          |
| 李四         | JOY19080300007 | 272.00    | 272.00       |
| 李四         | JOY19080300008 | 315.00    | 315.00       |
| 临时         | JOY19080300008 | 315.00    | .00          |

测试数据

```sql
CREATE TABLE G0426
(Carrier_Name NVARCHAR(10) NOT NULL,
OrderNumber VARCHAR(20) NOT NULL,
ReSumCost DECIMAL(10, 2) NOT NULL
);
INSERT INTO G0426 VALUES('临时', 'JOY19080300003', 170);
INSERT INTO G0426 VALUES('张三', 'JOY19080300003', 170);
INSERT INTO G0426 VALUES('临时', 'JOY19080300004', 196.5);
INSERT INTO G0426 VALUES('张三', 'JOY19080300004', 196.5);
INSERT INTO G0426 VALUES('临时', 'JOY19080300006', 458.8);
INSERT INTO G0426 VALUES('张三', 'JOY19080300006', 458.8);
INSERT INTO G0426 VALUES('李四', 'JOY19080300007', 272);
INSERT INTO G0426 VALUES('李四', 'JOY19080300008', 315);
INSERT INTO G0426 VALUES('临时', 'JOY19080300008', 315);
```

```sql
select *,case when lag(OrderNumber,1) over(partition by OrderNumber order by OrderNumber)=OrderNumber then 0 else ReSumCost end NewReSumCost
from G0426
```

### 最大连续天数

有如下一张表G0425，字段有UID，LOADTIME.

如何查询各个用户最长的连续登陆天数？

**测试数据**

```sql
CREATE TABLE G0425 
(
    UID VARCHAR(10),
    LOADTIME DATE
)
INSERT INTO G0425
SELECT '201','2017/01/28' UNION ALL
SELECT '201','2017/01/29' UNION ALL
SELECT '202','2017/01/29' UNION ALL
SELECT '202','2017/01/30' UNION ALL
SELECT '203','2017/01/30' UNION ALL
SELECT '201','2017/01/31' UNION ALL
SELECT '202','2017/01/31' UNION ALL
SELECT '201','2017/02/01' UNION ALL
SELECT '202','2017/02/01' UNION ALL
SELECT '201','2017/02/02' UNION ALL
SELECT '203','2017/02/02' UNION ALL
SELECT '203','2017/02/03'
```

```sql
select c.UID,max(newtimenum) as CNT
from
(select UID,count(newtime) newtimenum
from
(select UID,LOADTIME-newnum as newtime
from
(select UID,LOADTIME,row_number() over(partition by UID order by LOADTIME) as newnum
from G0425)a )b group by UID,newtime)c
group by c.UID
```

```sql
SELECT c.UID,MAX(c.newtimenum) AS CNT
FROM
(SELECT b.UID,COUNT(b.newtime) newtimenum
FROM
(SELECT a.UID,DATE_SUB(a.LOADTIME,INTERVAL a.newnum DAY)AS newtime
FROM
(SELECT UID,LOADTIME,ROW_NUMBER() OVER(PARTITION BY UID ORDER BY LOADTIME) AS newnum
FROM G0425)a )b GROUP BY UID,newtime)c
GROUP BY c.UID
```

### 指定条件casewhen

有一个录取学生人数表，记录的是每年录取学生人数和入学学生的学制

以下是表结构：

  CREATE TABLE `admit` (    `id` int(11) NOT NULL AUTO_INCREMENT,    `year` int(255) DEFAULT NULL COMMENT  '入学年度',    `num` int(255) DEFAULT NULL COMMENT '录取学生人数',    `stu_len` varchar(255) DEFAULT NULL COMMENT  '学生学制',    PRIMARY KEY (`id`)   ) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8mb4 COMMENT='录取人数';  

year表示学生入学年度

num表示对应年度录取学生人数

stu_len表示录取学生的学制

说明：例如录取年度2018学制3，表示该批学生在校年份为2018~2019、2019~2020、2020~2021

以下是示例数据：

| id   | year | num  | stu_len |
| ---- | ---- | ---- | ------- |
| 1    | 2018 | 2000 | 3       |
| 2    | 2019 | 2000 | 3       |
| 3    | 2020 | 1000 | 4       |
| 4    | 2020 | 2000 | 3       |

根据以上示例计算出每年在校人数,写出SQL语句：

```sql
SELECT DISTINCT a.year,SUM(CASE WHEN b.id IS NULL  THEN a.num ELSE a.num-b.num END) OVER(ORDER BY a.year) AS stu_num  FROM (SELECT id,YEAR,num,stu_len,YEAR+stu_len AS  stu_outyear  FROM admit) a LEFT JOIN   (SELECT id,YEAR,num,stu_len,YEAR+stu_len AS  stu_outyear  FROM admit ORDER BY YEAR DESC) b ON  a.year=b.stu_outyear+1  
```



### 分组过滤条件

有如下一组数据G0420，列名有docnum,status

希望得到如下结果，即只取状态为FULL的DOCNUM，如果有同时为NOFULL的DOCNUM则不取。

**测试数据**

```sql
CREATE TABLE G0420
(
DOCNUM INT,
STATUS VARCHAR(26)
)
INSERT INTO  G0420 VALUES (33,'FULL')
INSERT INTO  G0420 VALUES (33,'NOFULL')
INSERT INTO  G0420 VALUES (34,'FULL')
INSERT INTO  G0420 VALUES (35,'FULL')
INSERT INTO  G0420 VALUES (35,'NOFULL')
INSERT INTO  G0420 VALUES (36,'FULL')
INSERT INTO  G0420 VALUES (37,'FULL')
INSERT INTO  G0420 VALUES (38,'FULL')
INSERT INTO  G0420 VALUES (38,'NOFULL')
```

```sql
select t.DOCNUM
from 
(select *,count(1) over (partition by DOCNUM) as rn
from G0420) t
where t.STATUS='FULL'
and rn=1
```

### 行转列

有如下一张表G0419，字段有No,NAME,age

希望得到如下结果，即对相同的No进行转置

**测试数据**

```sql
CREATE TABLE G0419 (
No INT,
NAME NVARCHAR(20),
age INT)

INSERT INTO G0419 SELECT 1,'张三','18'
UNION ALL SELECT 1,'李四','17'
UNION ALL SELECT 1,'王五','23'
UNION ALL SELECT 1,'赵六','40'
UNION ALL SELECT 2,'Tom','17'
UNION ALL SELECT 3,'Bob','19'
UNION ALL SELECT 3,'Tony','36'
UNION ALL SELECT 3,'Petter','25'
```

```sql
with cte as(
select row_number() over(partition by No order by (select 1) ) as rid,*
from G0419
)
select No,
Max(case when rid=1 then Name else null end) as Name1,
Max(case when rid=1 then Age else null end) as Age1,
Max(case when rid=2 then Name else null end) as Name2,
Max(case when rid=2 then Age else null end) as Age2,
Max(case when rid=3 then Name else null end) as Name3,
Max(case when rid=3 then Age else null end) as Age3,
Max(case when rid=4 then Name else null end) as Name4,
Max(case when rid=4 then Age else null end) as Age4
from cte
group by No

```



### 组链接

有如下一张表G0418，字段有ID，品名，规格，颜色，数量。希望得到如下结果，即相同品种，规格，颜色的数据行的数量进行汇总，同时合并他们汇总前的ID为IDs

**测试数据**

```sql
Create table G0418 
(
ID int,
品名 varchar(23),
规格 varchar(23),
颜色 varchar(22),
数量 int
)
Insert INTO G0418 VALUES(1,'aaa','a-1','红色',50) ;
Insert INTO G0418 VALUES(2,'bbb',null,'红色',60) ;
Insert INTO G0418 VALUES(3,'aaa','a-1','红色',50) ;
Insert INTO G0418 VALUES(4,'ccc','c-1',null,50) ;
Insert INTO G0418 VALUES(5,'bbb',null,'红色',60) ;
```

group_concat([distinct] 字段名 [order by 排序字段 asc/desc] [separator '分隔符'])

- distinct：可以排除重复值
- order by：对结果中的值进行排序
- separator：分隔符

```sql
select 品名,规格,颜色,sum(数量) as 数量,group_concat(ID) as IDs
from G0418
group by 品名,规格,颜色
```

### *归零累加和

有如下一张表G0412

需要统计每组（NAME）连续时间（TIME）内的连续完成数（COMPLETE），其中有某一时间的完成数为0就重新计算。将统计结果添加到新列Result上.

**测试数据**

```sql
CREATE TABLE G0412
(TIME INT,
 NAME VARCHAR(5),
 COMPLETE VARCHAR(5)
)
INSERT INTO G0412 VALUES(1,'1','0');
INSERT INTO G0412 VALUES(2,'1','1');
INSERT INTO G0412 VALUES(3,'1','1');
INSERT INTO G0412 VALUES(4,'1','1');
INSERT INTO G0412 VALUES(5,'1','0');
INSERT INTO G0412 VALUES(6,'1','0');
INSERT INTO G0412 VALUES(7,'1','1');
INSERT INTO G0412 VALUES(8,'1','1');
INSERT INTO G0412 VALUES(2,'3','1');
INSERT INTO G0412 VALUES(3,'3','1');
INSERT INTO G0412 VALUES(4,'3','1');
INSERT INTO G0412 VALUES(5,'3','0');
INSERT INTO G0412 VALUES(6,'3','1');
INSERT INTO G0412 VALUES(7,'3','1');
INSERT INTO G0412 VALUES(8,'3','0' );
```



```sql
WITH t1 AS(
SELECT *,ROW_NUMBER() OVER(PARTITION BY NAME ORDER BY TIME) AS num1
FROM G0412
),
t2 AS(
SELECT *,ROW_NUMBER() OVER(PARTITION BY NAME,COMPLETE ORDER BY num1) AS num2
FROM t1
)
SELECT t2.time,t2.name,t2.COMPLETE,
CASE WHEN COMPLETE='1' THEN  ROW_NUMBER() OVER(PARTITION BY NAME,COMPLETE,num1-num2 ORDER BY num1) ELSE 0 END AS result
FROM t2 ORDER BY NAME,TIME
```



### 条件子查询

有一张充值表G0411，字段有订单号，充值日期，充值金额，充值产品，有效天数，先需要根据财务的需求，根据充值日期、有效天数和充值金额分摊到2020年最后一天，即2020年12月31日。

需要得到如下的表，即在上表的基础上增加两列分摊金额和剩余金额。分摊金额时，包括充值日期和2020年12月31日这两天，即包括头尾日期。

| 订单号 | 充值日期   | 充值金额  | 充值产品           | 有效天数 | 分摊金额  | 剩余金额  |
| ------ | ---------- | --------- | ------------------ | -------- | --------- | --------- |
| 1001   | 2020/7/1   | 500.0000  | 初一数学提高班     | 90       | 500.0000  | 0.0000    |
| 1002   | 2020/8/4   | 1000.0000 | 成人英语口语突破班 | 30       | 1000.0000 | 0.0000    |
| 1003   | 2020/9/10  | 2000.0000 | 初三数学提高班     | 240      | 941.6629  | 1058.3371 |
| 1004   | 2020/11/15 | 3000.0000 | 高三语文作文提高班 | 360      | 391.6651  | 2608.3349 |
| 1005   | 2020/12/20 | 2000.0000 | 高一物理精讲班     | 60       | 399.9996  | 1600.0004 |

解释：例如2020-09-10这天充值了2000元，从2020-09-10到2020-12-31日这一天总共有113天，实际有效期为240天，那么到2020-12-31日这一天，需要分摊这2000元的金额计算方式为：2000/240*113=941.6629。如果有效天数小于到2020-12-31日这天的天数，那么就全部分摊。

**测试数据**

```sql
CREATE TABLE G0411
(
订单号 VARCHAR(10),
充值日期 DATE,
充值金额 double,
充值产品 VARCHAR(100),
有效天数 INT
)
INSERT INTO G0411 VALUES('1001','2020-07-01',500.00,'初一数学提高班',90);
INSERT INTO G0411 VALUES('1002','2020-08-04',1000.00,'成人英语口语突破班',30);
INSERT INTO G0411 VALUES('1003','2020-09-10',2000.00,'初三数学提高班',240);
INSERT INTO G0411 VALUES('1004','2020-11-15',3000.00,'高三语文作文提高班',360);
INSERT INTO G0411 VALUES('1005','2020-12-20',2000.00,'高一物理精讲班',60);
```

```sql
select a.订单号,a.充值日期,a.充值金额,a.充值产品,a.有效天数,
case when a.有效天数<=num then a.充值金额 else a.充值金额/a.有效天数*num end as 分摊金额,
case when a.有效天数<=num then '0.0000' else a.充值金额-a.充值金额/a.有效天数*num end as 剩余金额
from
(select *,TIMESTAMPDIFF(DAY,充值日期,'2020-12-31')+1 as num
from G0411) a
```

### select条件查询

有如下两张表G0410A，列名为A(比赛项目)，B(参赛人名)。表G0410B，列名为A(比赛项目),B（参赛人名）,C（输赢）。希望得到结果，列B为参赛人名，anum表示每个人参加的项目数，bnum表示每个人在各自项目中胜利的次数

该如何写这个查询？

**测试数据**

```sql
CREATE TABLE G0410A(
A VARCHAR(20),
B VARCHAR(20)
)
INSERT INTO G0410A VALUES('跑步','张三');
INSERT INTO G0410A VALUES('游泳','张三');
INSERT INTO G0410A VALUES('跳远','李四');
INSERT INTO G0410A VALUES('跳高','王五');
CREATE TABLE G0410B (
A VARCHAR(20),
B VARCHAR(20),
C VARCHAR(10)
)
INSERT INTO G0410B VALUES('跑步','张三','胜');
INSERT INTO G0410B VALUES('游泳','张三','胜');
INSERT INTO G0410B VALUES('跳高','王五','胜');
```

```sql
SELECT a.B,COUNT(1) AS anum,
IFNULL((SELECT SUM(CASE WHEN b.C='胜' THEN 1 ELSE 0 END) FROM G0410B b WHERE b.B=a.B),0) AS bnum
FROM G0410A a
GROUP BY a.B
--2
SELECT IFNULL(t1.B,t2.B) AS bid,
IFNULL(t1.anum,0) anum,
IFNULL(t2.bnum,0) bnum
FROM
(SELECT B,COUNT(1) AS anum
FROM G0410A GROUP BY B) t1
LEFT OUTER JOIN
(SELECT B,SUM(CASE WHEN C='胜' THEN 1 ELSE 0 END) AS bnum
FROM G0410B GROUP BY B) t2 ON t1.B=t2.B
```



### 嵌套条件查询

有如下一张表G0407，字段ID，Num

要求

当Num中的数据同时大于上下两行数据，返回是

当Num中的数据小于上下两行数据中的任何一行，返回否

例如：11大于5,11大于0，所以返回是

5小于11所以返回否

该如何写这个查询？

**测试数据**

```sql
CREATE TABLE G0407 (
ID INT,
Num INT
);
INSERT INTO G0407 VALUES(1,5);
INSERT INTO G0407 VALUES(2,11);
INSERT INTO G0407 VALUES(3,0);
INSERT INTO G0407 VALUES(4,-2);
INSERT INTO G0407 VALUES(5,2);
INSERT INTO G0407 VALUES(6,9);
INSERT INTO G0407 VALUES(7,1);
INSERT INTO G0407 VALUES(8,-4);
INSERT INTO G0407 VALUES(9,-7);
```

```sql
SELECT 
a.id,
a.num,
CASE WHEN a.num>IFNULL((SELECT b.num FROM g0407 b WHERE b.id=a.id-1),a.num-1)
AND
a.num>IFNULL((SELECT c.num FROM g0407 c WHERE c.id=a.id+1),a.num-1) 
THEN '是' ELSE '否' END AS result
FROM g0407 a
```



### *字符串拆分（使用mysql.help_topic表）

有如下一张表G0406,字段id，a1

希望希望将a1列里面的数据按照"."进行拆分，拆分后的结果如下：要求使用自定义函数进行求解

**测试语句**

```sql
CREATE TABLE G0406 (
id INT,
a1 VARCHAR(50)
)
INSERT INTO G0406 VALUES(1,'A.AC.BD.EFG');
INSERT INTO G0406 VALUES(2,'XY.FQ.AZY.ZLG');
```

```sql
SELECT 
 b.id,SUBSTRING_INDEX(SUBSTRING_INDEX(b.a1,'.',help_topic_id+1),'.',-1) AS num 
FROM
(SELECT 1 AS idx,help_topic_id
FROM mysql.help_topic) a 
CROSS JOIN 
(SELECT 1 AS idx,id,a1 FROM g0406) b
ON a.idx=b.idx
WHERE help_topic_id < LENGTH(b.a1)-LENGTH(REPLACE(b.a1,'.',''))+1
--2
SELECT
 b.id,SUBSTRING_INDEX(SUBSTRING_INDEX(b.a1,'.',help_topic_id+1),'.',-1) AS num 
FROM
mysql.help_topic a ,
g0406 b
WHERE help_topic_id < LENGTH(b.a1)-LENGTH(REPLACE(b.a1,'.',''))+1
```

利用 mysql 库的 help_topic 表的 help_topic_id 来作为变量，因为 help_topic_id 是自增的，当然也可以用其他表的自增字段辅助。

一、字符串拆分： SUBSTRING_INDEX（str, delim, count）

参数解说
参数名	解释
str	需要拆分的字符串
delim	分隔符，通过某字符进行拆分
count	当 count 为正数，取第 n 个分隔符之前的所有字符； 当 count 为负数，取倒数第 n 个分隔符之后的所有字符

------------------------------------------------

二、替换函数：replace( str, from_str, to_str)

1. 参数解说

| 参数名   | 解释                 |
| -------- | -------------------- |
| str      | 需要进行替换的字符串 |
| from_str | 需要被替换的字符串   |
| to_str   | 需要替换的字符串     |

[参考链接](https://blog.csdn.net/pjymyself/article/details/81668157?spm=1001.2101.3001.6650.15&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-16-81668157-blog-123268413.235%5Ev28%5Epc_relevant_t0_download&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-16-81668157-blog-123268413.235%5Ev28%5Epc_relevant_t0_download&utm_relevant_index=22)



### *嵌套case when

有如下2张表G0404A**发药信息表**，字段有iilszh(花药id),spmc(),fysl

G0404B,字段有iilszh,spmc,gjsl

某医院系统卖给患者阿莫西林X数量后去下购进记录的库存，购进记录可能是零散的。现在按iilszh排序，优先iilszh小的，依次下库存如何得到类似下面的结果集：

**药品名称**为spmc，**购进数量**为G0404B的gjsl，**本次下库存数量**为如果G0表的404A的fysl大于G0404B的gjsl，则为G0404B的gjsl，如果G0表的404A的fysl小于G0404B的gjsl，则为G0404A的fysl减去之前使用过G0404B的gjsl的数量，**剩余数量**为如果G0表的404A的fysl大于G0404B的gjsl，则为0，如果G0表的404A的fysl小于G0404B的gjsl，则为购进数量减去本次下库数量

**测试语句**

```sql
CREATE TABLE G0404A(
iilszh INT AUTO_INCREMENT,
spmc VARCHAR(100),
fysl NUMERIC(18,4),
PRIMARY KEY (iilszh)
)
INSERT INTO G0404A(spmc, fysl) VALUES ('阿莫西林胶囊',80);
INSERT INTO G0404A(spmc, fysl) VALUES ('人血白蛋白',100);

CREATE TABLE G0404B(
iilszh INT AUTO_INCREMENT,
spmc VARCHAR(100),
gjsl NUMERIC(18,4),
PRIMARY KEY (iilszh)
)
INSERT INTO G0404B(spmc, gjsl) VALUES ('阿莫西林胶囊',20);
INSERT INTO G0404B(spmc, gjsl) VALUES ('阿莫西林胶囊',50);
INSERT INTO G0404B(spmc, gjsl) VALUES ('阿莫西林胶囊',40);
INSERT INTO G0404B(spmc, gjsl) VALUES ('阿莫西林胶囊',30);
INSERT INTO G0404B(spmc, gjsl) VALUES ('人血白蛋白',120);
INSERT INTO G0404B(spmc, gjsl) VALUES ('人血白蛋白',80);
INSERT INTO G0404B(spmc, gjsl) VALUES ('人血白蛋白',100);
```

```sql
WITH t AS(
SELECT spmc,gjsl,
IFNULL((SELECT SUM(b.gjsl)
FROM g0404b b
WHERE b.spmc=a.spmc AND b.iilszh<=a.iilszh
),0) ls
FROM g0404b a
),
v AS(
SELECT t.spmc ,
t.gjsl,
CASE WHEN t.ls<=u.fysl THEN t.gjsl ELSE CASE WHEN (t.ls-t.gjsl)<u.fysl THEN u.fysl-(t.ls-t.gjsl) ELSE 0 END END AS  bc
FROM t JOIN g0404a u ON t.spmc=u.spmc
)
SELECT spmc AS 药名名称, gjsl AS 购进数量, bc AS 本次下库数量, gjsl-bc AS 剩余数量
FROM v
```



### 条件计数

有如下1张表G0403,字段为ID,NAME,MON,STATE,AMOUNT

其中Amount字段是每个人同一月份，不同状态的金额总和，需要对相同月份的数据均分Amount，如何写这个SQL

**测试语句**

```sql
CREATE TABLE G0403 
(ID int,
NAME varchar(22),
MON int,
STATE varchar(21),
AMOUNT int
);
INSERT INTO G0403 VALUES(1,'张三',201901,'A',9000);
INSERT INTO G0403 VALUES(2,'张三',201901,'B',9000);
INSERT INTO G0403 VALUES(3,'张三',201901,'E',9000);
INSERT INTO G0403 VALUES(4,'李四',201902,'A',1800);
INSERT INTO G0403 VALUES(5,'李四',201902,'C',1800);
INSERT INTO G0403 VALUES(6,'王五',201902,'C',30000);
INSERT INTO G0403 VALUES(7,'王五',201902,'F',30000);
```

```sql
--1
select a.ID,a.NAME,a.MON,STATE,b.AMOUNT from G0403 a
left join
(select AMOUNT/count(MON) as AMOUNT from G0403 group by NAME,MON 
) b on a.NAME=b.NAME and a.MON=b.MON
--2
select a.ID,a.NAME,a.MON,STATE,
AMOUNT/(select count(1) from G0403 b where a.MON=b.MON and b.NAME=a.NAME) as AMOUNT
from G0403 a
```

### 时间格式转化

有如下3张表

其中G0331C，这张为客户表，字段有客户编号，客户抬头

G0331X，这张为X业务应收金额表，字段有客户编号，日期，金额

G0331Y，这张为Y业务应收金额表，字段有客户编号，日期，金额

现在需要将每个客户按月生成汇总报表，用SQL该如何求解？（输出字段为日期，客户抬头，X业务金额，Y业务金额）

**测试语句**

```sql
Create table G0331C(
客户编号 varchar(24),
客户抬头 varchar(23)
)
Insert INTO G0331C VALUES('0001','A公司');
Insert INTO G0331C VALUES('0002','B公司');
Insert INTO G0331C VALUES('0003','C公司');

Create table G0331X(
客户编号 varchar(24),
日期 Date,
金额 int
)

Insert INTO G0331X VALUES('0001','2018-9-12',2000);
Insert INTO G0331X VALUES('0001','2018-9-16',1500);
Insert INTO G0331X VALUES('0001','2018-10-23',3000);
Insert INTO G0331X VALUES('0002','2018-9-15',3200);
Insert INTO G0331X VALUES('0002','2018-10-19',5000);


Create table G0331Y(
客户编号 varchar(24),
日期 Date,
金额 int
)
Insert INTO G0331Y VALUES('0001','2018-9-12',12000);
Insert INTO G0331Y VALUES('0001','2018-10-16',10000);
Insert INTO G0331Y VALUES('0001','2018-10-23',20000);
Insert INTO G0331Y VALUES('0002','2018-11-15',13200);
Insert INTO G0331Y VALUES('0002','2018-10-19',25000);
```

```sql
select isnull(t.日期,t1.日期) as 日期,a.客户抬头,t.金额 X业务金额,t1.金额 Y业务金额
from G0331C a
left join 
(select 客户编号,date_format(日期,'%Y-%m-%d') as 日期,sum(金额) X业务金额
G0331X group by 客户编号,date_format(日期,'%Y-%m-%d')
) t on a.客户编号=t.客户编号
left join 
(select 客户编号,date_format(日期,'%Y-%m-%d') as 日期,sum(金额) Y业务金额
G0331Y group by 客户编号,date_format(日期,'%Y-%m-%d')
) t1 on a.客户编号=t1.客户编号 and t.日期=t1.日期
where isnull(t.日期,t1.日期) is not null
order by 日期 desc,a.客户编号
```

date_format(date,'%Y-%m-%d')

date_format(date,'%Y-%m-%d %H:%i:%s')

str_to_date(date,'%Y-%m-%d')

str_to_date('2022-04-26 13:32:06','%Y-%m-%d %H:%i:%s')

cast``( <数据> ``as` `<数据类型> )
convert(<数据>,<数据类型>)

### 系统表拆分列

有如下一张表G0330，希望将每个Type按Num进行均等分，结果如下

| Type | Num  |
| ---- | ---- |
| A    | 1    |
| A    | 1    |
| B    | 1    |
| B    | 1    |
| B    | 1    |
| C    | 1    |
| C    | 1    |

测试语句

```sql
CREATE TABLE G0330(
Type VARCHAR(10),
Num  INT 
);
INSERT INTO G0330 VALUES('A',2);
INSERT INTO G0330 VALUES('B',3);
INSERT INTO G0330 VALUES('C',2);
```

Sql Server中master.dbo.spt_values是一个数据库常量表，表里都是一些枚举数据。

查询得知表里有五个字段：☞
name（名称）,number（值）,type（类型）,low（下限）,high（上限）,status（状态）

1. 取1-1000之间的数字

```sql
select number from master..spt_values 
where type='P' and number between 1 and 1000

-- 取当前日期往后的365个日期
select convert(varchar(10), dateadd(day,number,getdate()), 120) as [日期]  
from master..spt_values
where type = 'P' and number between 0 and 365  

```

```sql
select a.Type,1 Num
from G0330 a
join master.dbo.spt_values b on b.Type='P' anf a.Num>b.Number
```





### 合并数据最大值

表 G0329 存储了所有好友申请通过的数据记录，其中， requester_id 和 accepter_id 都是用户的编号。accept_date为接收日期。

写一个查询语句，求出谁拥有最多的好友和他拥有的好友数目。对于上面的样例数据，输出字段为id,num

**测试数据**

```sql
CREATE TABLE G0329
(
requester_id INT,
accepter_id INT,
accept_date DATE
)
INSERT INTO G0329 VALUES (1,2,'2016-6-3');
INSERT INTO G0329 VALUES (1,3,'2016-6-8');
INSERT INTO G0329 VALUES (2,3,'2016-6-8');
INSERT INTO G0329 VALUES (3,4,'2016-6-9');
```

```sql
select ids id,count(*) num
(select requester_id ids
from G0329
union all 
select accepter_id ids
from G0329) t
group by ids order by count(*) desc limit 1;
```

### 求临界点

表G0328如下字段为ID，金额。ID为自增长列，查询出第一条开始到第几条记录 的累计金额刚好超过100？

*要求：不能使用开窗函数*

**测试数据**

```sql
CREATE TABLE G0328 (
ID INT,
金额 INT
)
INSERT INTO G0328 VALUES (1,30);
INSERT INTO G0328 VALUES (2,30);
INSERT INTO G0328 VALUES (3,30);
INSERT INTO G0328 VALUES (4,9);
INSERT INTO G0328 VALUES (5,1);
INSERT INTO G0328 VALUES (6,1);
INSERT INTO G0328 VALUES (7,15);
INSERT INTO G0328 VALUES (8,33);
INSERT INTO G0328 VALUES (9,5);
INSERT INTO G0328 VALUES (10,8);
INSERT INTO G0328 VALUES (11,14);
INSERT INTO G0328 VALUES (12,3);
```

```sql
select min(ID) Result
from
(select b.ID,sum(金额)  as 金额
from G0328 a join G0328 b on a.id<=b.id 
group by b.ID having sum(a.金额)>100) t
```



### 累加和与总和的比例

有如下一张表G0327，字段为ID，NAME，NUM

求出NAME中每组累加/每组总数的比例大于0.6的ID和NAME 

预期的结果字段为ID，NAME，NUM

解释：从题目意思可以看出A组的总数为16，从ID为1到5分别累加后的结果分别为1,3,9,13,16，只有13和16除以总数16才大于0.6，所以返回的结果ID为4和5，同样B组为7和8

**测试数据**

```sql
CREATE TABLE G0327 
(
ID INT,
NAME VARCHAR(10),
NUM INT
)
INSERT INTO G0327 VALUES (1,'A',1);
INSERT INTO G0327 VALUES (2,'A',2);
INSERT INTO G0327 VALUES (3,'A',6);
INSERT INTO G0327 VALUES (4,'A',4);
INSERT INTO G0327 VALUES (5,'A',3);
INSERT INTO G0327 VALUES (6,'B',2);
INSERT INTO G0327 VALUES (7,'B',8);
INSERT INTO G0327 VALUES (8,'B',2);
```

```sql
select b.ID,b.NAME,b.NUM
from
(select a.ID,a.NAME,a.NUM,
sum(NUM) OVER(partition by NAME order by ID) A,
sum(NUM) OVER(partition by NAME order by ID) B,
sum(NUM) OVER(partition by NAME order by ID)*1.0/SUM(NUM) OVER(partition by NAME) C
from G0327 a) b
where b.C>0.6
```

### select嵌套查询

一张表G0324,字段LDate,Value1,Value2希望得到结果，描述： 如果某列的字段在该日期为空值，查询时结果显示为之前最接近日期的非空值。 

**测试数据**

```sql
CREATE TABLE G0324
(LDate DATE NOT NULL,
Value1 INT NULL,
Value2 INT NULL
)
INSERT INTO G0324 VALUES('2020-11-25', 500 ,200);
INSERT INTO G0324 VALUES('2020-11-24', Null, 200);
INSERT INTO G0324 VALUES('2020-11-23', Null, 250);
INSERT INTO G0324 VALUES('2020-11-22', 300 ,Null);
INSERT INTO G0324 VALUES('2020-11-21', 200 ,320)
```

```sql
SELECT
b.LDate,
(SELECT Value1
FROM G0324 a WHERE a.LDate<=b.LDate AND a.Value1 IS NOT NULL ORDER BY b.LDate LIMIT 1) Value1,
(SELECT Value2
FROM G0324 a WHERE a.LDate<=b.LDate AND a.Value2 IS NOT NULL ORDER BY b.LDate LIMIT 1) Value2
FROM G0324 b
```

### 上下关联表

有如下一张表F0623,字段ID,Cname,ParentID,OrgID,Type

希望得到一下结果：

| ID   | Cname | OrgName  | DeptName |
| ---- | ----- | -------- | -------- |
| 3    | 张三  | 公司总部 | 人事部   |
| 5    | 李四  | 公司总部 | 财务部   |

**测试数据**

```sql
CREATE TABLE G0323
(
ID INT,
Cname VARCHAR(20),
ParentID INT,
OrgID INT,
Type INT
)
INSERT INTO G0323 VALUES (1,'公司总部',0,0,1);
INSERT INTO G0323 VALUES (2,'人事部',1,1,2);
INSERT INTO G0323 VALUES (3,'张三',2,1,3);
INSERT INTO G0323 VALUES (4,'财物部',1,1,2);
INSERT INTO G0323 VALUES (5,'李四',4,1,3);
```

```sql
SELECT a.ID,a.Cname, b.Cname DeptName,c.Cname OrgName FROM G0323 a
,G0323 b,G0323 c
WHERE a.Type=3 AND a.ParentID=b.ID AND a.OrgID=c.ID
--2:
SELECT a.ID,a.Cname, b.Cname DeptName,c.Cname OrgName FROM G0323 a
LEFT JOIN G0323 b ON a.ParentID=b.ID LEFT JOIN G0323 c ON a.OrgID=c.ID
WHERE a.Type=3
```

### 累加和（不使用over）

有如下一张 G0322表,字段play_id,device_id,event_date,games_played,其中games_played是玩家登陆玩的游戏数量， 查询每个玩家每天累计玩的游戏数量有多少？结果如下,字段有play_id，event_date，games_played_so_far

解释：玩家1第一次玩了5个，所以是5，第二次是6个，所以累计就是5+6=11， 第三次是1个，累计就是5+6+1=12 玩家2类似

*要求：不能使用开窗函数*

**测试数据**

```sql
CREATE TABLE G0322
(
play_id INT,
device_id INT,
event_date DATE,
games_played INT
)

INSERT INTO G0322 VALUES(1,2,'2016/3/1',5);
INSERT INTO G0322 VALUES(1,2,'2016/5/2',6);
INSERT INTO G0322 VALUES(1,3,'2017/6/25',1);
INSERT INTO G0322 VALUES(3,1,'2016/3/2',0);
INSERT INTO G0322 VALUES(3,4,'2018/7/3',5);
```

```sql
SELECT a.play_id,a.event_date,SUM(b.games_played)  AS games_played_so_far 
FROM G0322 a, G0322 b WHERE a.play_id=b.play_id AND a.event_date>=b.event_date
GROUP BY a.play_id,a.event_date
ORDER BY a.play_id,a.event_date
--开窗函数
SELECT play_id,event_date,SUM(games_played) OVER(PARTITION BY play_id ORDER BY event_date)  AS games_played_so_far 
FROM G0322
```



### 嵌套连接查询

有如下三种表及表结构：G0321A(stuID,classID,stuName),分别对应学生编号，班级编号，学生姓名

G0321B(classID,className)，分别对应班级编号，班级名称

G0321C(stuID,course,score)，分别对应学生编号，课程名称，考试成绩

查询一班各科成绩最高的学生姓名？

**测试数据**

```sql
CREATE TABLE G0321A(
stuID INT,
classID VARCHAR(10),
stuName VARCHAR(20)
)
INSERT INTO G0321A VALUES(1,'A','zhang');
INSERT INTO G0321A VALUES(2,'A','li');
INSERT INTO G0321A VALUES(3,'B','wang');

CREATE TABLE G0321B
(
classID VARCHAR(10),
className VARCHAR(20)
)
INSERT INTO G0321B VALUES('A','一班');
INSERT INTO G0321B VALUES('B','二班');

CREATE TABLE G0321C
(
stuID INT,
course VARCHAR(20),
score INT
)
INSERT INTO G0321C VALUES (1,'语文',80);
INSERT INTO G0321C VALUES (1,'数学',90);
INSERT INTO G0321C VALUES (1,'英语',70);
INSERT INTO G0321C VALUES (2,'语文',89);
INSERT INTO G0321C VALUES (2,'数学',91);
INSERT INTO G0321C VALUES (2,'英语',88);
INSERT INTO G0321C VALUES (3,'语文',75);
INSERT INTO G0321C VALUES (3,'数学',77);
INSERT INTO G0321C VALUES (3,'英语',72);
```

先求一班里各课程的最高分数，在连学生和成绩表

```sql
SELECT f.stuName,d.score
FROM
(SELECT c.course,MAX(c.score) AS score
FROM g0321a a LEFT JOIN g0321b b ON a.classID=b.classID
LEFT JOIN g0321c c ON a.stuID=c.stuID
WHERE b.className='一班'
GROUP BY b.classID,c.course) d
LEFT JOIN g0321c e ON e.course=d.course AND e.score=d.score
LEFT JOIN g0321a f ON f.stuID=e.stuID
```

### 强转类型和连接

有一张表G0320，里面的字段ID，PID,像得到如下结果

| ID   | PID  | PATH       |
| ---- | ---- | ---------- |
| 1    | 0    | 1          |
| 2    | 1    | 1->2       |
| 3    | 2    | 1->2->3    |
| 4    | 3    | 1->2->3->4 |

**测试数据**

```sql
CREATE TABLE G0320 
(
ID INT,
PID INT
)
INSERT INTO G0320 VALUES (1,0);
INSERT INTO G0320 VALUES (2,1);
INSERT INTO G0320 VALUES (3,2);
INSERT INTO G0320 VALUES (4,3);
```

```sql
WITH RECURSIVE cte (ID, PID, PATH) AS (
  SELECT a.ID, a.PID, CAST(a.ID AS CHAR(200)) AS PATH
  FROM G0320 a
  WHERE ID=1
  UNION ALL
  SELECT t.ID, t.PID, CONCAT(cte.PATH, '->', t.ID) AS PATH
  FROM G0320 t
  JOIN cte ON t.PID = cte.ID
)
SELECT *
FROM cte;
```

mysql8.0使用CAST(a.ID AS NVARCHAR(200))报错

### 相邻行过滤

有这样一张表G0317，这只是其中一个单号的

希望按单号+工序排序，相邻行的部门编号相同的情况，取工序号最大的那一行记录？输出字段为单号，工序，部门编号，完成数量，NextDept

**测试数据**

```sql
CREATE TABLE G0317(
单号 VARCHAR(20) NOT NULL,
工序 VARCHAR(10) NOT NULL,
部门编号 INT NOT NULL,
完成数量 INT NOT NULL
)
INSERT INTO G0317 VALUES('2021090065','0010',222,1500);
INSERT INTO G0317 VALUES('2021090065','0020',223,1497);
INSERT INTO G0317 VALUES('2021090065','0030',223,1497);
INSERT INTO G0317 VALUES('2021090065','0040',213,1497);
INSERT INTO G0317 VALUES('2021090065','0050',224,1497);
INSERT INTO G0317 VALUES('2021090065','0060',224,1497);
INSERT INTO G0317 VALUES('2021090065','0070',220,1496);
INSERT INTO G0317 VALUES('2021090065','0080',220,1496);
INSERT INTO G0317 VALUES('2021090065','0090',224,0     );
```

```sql
with list as (
select *,
lead(部门编号,1,null) over (partition by 单号 order by 工序) NextDept
    from G0317
)
slect * from list where 部门编号<>NextDept or NextDept is null
--2
select a.单号,a.工序,a.部门编号,a.完成数量
from (select *,lead(部门编号,1,null) over(partition by 单号 order by 工序)  nextdept
from G0413) a where 部门编号<>nextdept or nextdept is null
```

### 递归添加

有如下一张表G0316,字段ID,EndTime

希望每条记录每次增加30分钟，增加3次,预计得到的结果,字段ID,EndTime,NUM

**测试数据**

```sql
CREATE TABLE G0316
(
ID INT,
EndTime DATETIME
)
INSERT INTO G0316 VALUES (1,'2020/5/26 16:00');
INSERT INTO G0316 VALUES (2,'2020/5/26 17:30');
```

```sql
WITH RECURSIVE  CTE AS (
SELECT ID,EndTime,0 AS NUM FROM G0316
UNION ALL
SELECT ID,DATE_ADD(c.EndTime,INTERVAL 30 MINUTE),c.NUM+1
FROM CTE c
WHERE NUM<3
)
SELECT * FROM CTE c ORDER BY c.ID,c.EndTime
```

### 条件求和

有如下一张表G0315，字段姓名，省份，城市，类别

根据客单类别表统计出每个省份每个城市的低客单数和高客单数，如果某城市无低客单记录或高客单记录，其统计数为0   要求：通过一条sql语句得到下列结果，字段有省份，城市，低客单数，高客单数

**测试数据**

```sql
CREATE TABLE G0315 (
姓名 VARCHAR(20),
省份 VARCHAR(20),
城市 VARCHAR(20),
类别 VARCHAR(20)
)
INSERT INTO G0315 VALUES ('张三','广东','广州','低客单');
INSERT INTO G0315 VALUES ('李四','广东','广州','高客单');
INSERT INTO G0315 VALUES ('王五','湖南','岳阳','高客单');
INSERT INTO G0315 VALUES ('赵六','湖南','长沙','低客单');
INSERT INTO G0315 VALUES ('钱七','广东','广州','高客单');
INSERT INTO G0315 VALUES ('孙九','湖南','长沙','低客单');
```

```sql
select 省份,城市,sum(case when 类别='低客单' then 1 else 0 end) as 低客单数,sum(case when 类别='高客单' then 1 else 0 end) as 高客单数
from G0315 group by 省份,城市
order by 省份,城市
```

### 时间间隔计数

表G0314中的字段时user_id，time（用户访问时间）

求每个用户相邻两次浏览时间之差小于三分钟的次数

预计结果：

| user_id | cnt  |
| ------- | ---- |
| 1       | 2    |
| 2       | 1    |
| 3       | 0    |
| 4       | 1    |

**测试数据**

```sql
CREATE TABLE G0314 (
user_id INT,
times DATETIME
)
INSERT INTO G0314 VALUES (1,'2020-12-7 21:13:07');
INSERT INTO G0314 VALUES (1,'2020-12-7 21:15:26');
INSERT INTO G0314 VALUES (1,'2020-12-7 21:17:44');
INSERT INTO G0314 VALUES (2,'2020-12-13 21:14:06');
INSERT INTO G0314 VALUES (2,'2020-12-13 21:18:19');
INSERT INTO G0314 VALUES (2,'2020-12-13 21:20:36');
INSERT INTO G0314 VALUES (3,'2020-12-21 21:16:51');
INSERT INTO G0314 VALUES (4,'2020-12-16 22:22:08');
INSERT INTO G0314 VALUES (4,'2020-12-2 21:17:22');
INSERT INTO G0314 VALUES (4,'2020-12-30 15:15:44');
INSERT INTO G0314 VALUES (4,'2020-12-30 15:17:57');
```

```sql
with T as (select row_number() over(partition by user_id order by times) rn,user_id,times
from G0314 group by user_id,times)

SELECT a.user_id,SUM(CASE WHEN DATE_SUB(a.times,INTERVAL 3 MINUTE)<=b.times THEN 1 ELSE 0 END) cnt
FROM T a LEFT JOIN T b ON a.rn=b.rn+1 AND a.user_id=b.user_id
GROUP BY a.user_id
```

### 去重留一

编写一个 SQL 查询，来删除表G0313字段ID,Phone,中所有重复的电话号码，重复的电话号码只保留 Id 最小 的那个.

Id 是这个表的主键

**测试数据**

```sql
CREATE TABLE G0313 (
ID INT,
Phone VARCHAR(20)
)

INSERT INTO G0313 VALUES (1,'13512345678');
INSERT INTO G0313 VALUES (2,'13512345678');
INSERT INTO G0313 VALUES (3,'13012345678');
```

```sql
--1:
delete from G0313 g1,G0313 g2 where g1.Phone=g2.Phone and g1.ID>g2.ID
--2:
delete from G0313 where id not in (select mim(id) from G0313 group by Phone)
```

### 当前天比较

给定一个表G0310包含字段ID,REDATE(日期),TEMP(温度），编写一个 SQL 查询，来查找与之前（昨天的）日期相比温度更高的所有日期的 Id。

**测试数据**

```sql
CREATE TABLE G0310 (
ID INT,
REDATE DATE,
TEMP INT)
INSERT INTO G0310 VALUES (1,'2020-1-1',10);
INSERT INTO G0310 VALUES (2,'2020-1-2',18);
INSERT INTO G0310 VALUES (3,'2020-1-3',15);
INSERT INTO G0310 VALUES (4,'2020-1-4',20);
```

答案

```sql
select b.ID
from G0310 a,G0310 b
where DATEDIFF(DAY,a.REDATE,b.REDATE)=1
AND a.TEMP>b.TEMP
```

### 空值填补

有如下一张合同表G0309包含字段ID，Type,MasterID,Amount,现在想获得每个直接合同的价格已经对应的补充合同的价格,字段为ID,TYPE,amount,amount2

**测试数据**

```sql
CREATE TABLE G0309
(
ID INT,
Type VARCHAR(20),
MasterID INT,
Amount INT
)
INSERT INTO G0309 VALUES (1,'直接合同',NULL,5000);
INSERT INTO G0309 VALUES (2,'补充合同',1,1000);
INSERT INTO G0309 VALUES (3,'补充合同',1,500);
INSERT INTO G0309 VALUES (4,'直接合同',NULL,6000);
INSERT INTO G0309 VALUES (5,'直接合同',NULL,4000);
INSERT INTO G0309 VALUES (6,'补充合同',5,1000);
```

答案

```sql
SELECT a.ID,a.Type,a.Amount AS amount,IFNULL(SUM(b.Amount),0) AS amount2
FROM g0309 a LEFT JOIN g0309 b ON b.MasterID=a.ID
WHERE a.Type='直接合同'
GROUP BY a.ID,a.Type,a.Amount
ORDER BY a.ID
```

### *新建表关联条件

有如下一张表G0308A字段有A，B，G0308B字段有C，D，E，两表之间无任何关联关系，且数据行数也不相同，希望将两表进行横向拼接 成一张表。

两表之间无任何关联关系，且数据行数也不相同，希望将两表进行拼接 成一张表。

**测试数据**

```sql
CREATE TABLE G0308A (A INT,
B INT)

INSERT INTO G0308A VALUES(1,2);
INSERT INTO G0308A VALUES(4,6);

CREATE TABLE G0308B (C INT,
D INT,
E INT)

INSERT INTO G0308B VALUES(7,4,3);
INSERT INTO G0308B VALUES(9,5,8);
INSERT INTO G0308B VALUES(11,15,18);
```

```sql
select a1.A,a1.B,b1.C,b1.D,b1.E
(select row_number() over(order A) id,A,B
from G0308A t1) a1
full join
(slect row_number() over (order C) id,C,D,E
from G0308B t2) b1 on a1.id=b1.id
```

### 去重计数

有如下一张表G0307字段A,B,C(值只有X或Y),想得到如下结果，即同一类A的B列进行求和，如果C列的两个值不同就取1，相同就取同类别的值.

**测试数据**

```sql
CREATE TABLE G0307
(
A VARCHAR(10),
B INT,
C VARCHAR(10)
)
INSERT INTO G0307 VALUES('a',1,'X');
INSERT INTO G0307 VALUES('a',2,'Y');
INSERT INTO G0307 VALUES('b',3,'X');
INSERT INTO G0307 VALUES('b',4,'X');
INSERT INTO G0307 VALUES('c',5,'Y');
INSERT INTO G0307 VALUES('c',6,'Y');
```

``` sql
slect a.A,a.B,case when D=2 then '1' else E end as C
from
(select A,sum(B) B,count(distinct C) D,min(C) E from G0307 t1 group by A) a 
```

### 求最大值

有如下两张表 G0306A 表字段project_id,employee_id。G0306B表字段employee_id,name,eperience_years

查询出每个项目中经验最丰富(experience_years最大)的员工.返回的结果如下：

| project_id | employee_id |
| ---------- | ----------- |
| 1          | 1           |
| 1          | 3           |
| 2          | 1           |

说明：员工1和3是project_id为1中exprerience_years最丰富的, 而project_id为2的项目，员工id为1的是exprerience_years最丰富

**测试数据**

```sql
CREATE TABLE G0306A
(project_id INT,
employee_id INT)

INSERT INTO G0306A VALUES
(1,1),
(1,2),
(1,3),
(2,1),
(2,4);
CREATE TABLE G0306B
(employee_id INT,
name VARCHAR(20),
experience_years INT
)
INSERT INTO G0306B VALUES
(1,'张三',3),
(2,'李四',2),
(3,'王五',3),
(4,'马六',2)
```

说明：员工1和3是project_id为1中exprerience_years最丰富的, 而project_id为2的项目，员工id为1的是exprerience_years最丰富

```sql
--1:两表关联
SELECT t1.project_id,t2.employee_id
FROM
(SELECT a.project_id,MAX(b.experience_years) AS YEAR
FROM G0306A a LEFT JOIN G0306B b ON a.employee_id=b.employee_id
GROUP BY a.project_id) t1
LEFT JOIN
(SELECT c.project_id,d.employee_id,d.experience_years
FROM G0306A c LEFT JOIN G0306B d ON c.employee_id=d.employee_id) t2
ON t1.year=t2.experience_years AND t1.project_id=t2.project_id
--2: where in（） 
SELECT c.project_id,c.employee_id
FROM G0306A c LEFT JOIN G0306B d ON c.employee_id=d.employee_id
WHERE d.experience_years IN 
(SELECT MAX(b.experience_years)
FROM G0306A a LEFT JOIN G0306B b ON a.employee_id=b.employee_id
GROUP BY a.project_id)
--3:dense_rank()
SELECT t1.project_id,t1.employee_id FROM
(SELECT a.project_id,a.employee_id,DENSE_RANK() OVER(PARTITION BY a.project_id ORDER BY b.experience_years DESC) AS num
FROM G0306A a LEFT JOIN G0306B b ON a.employee_id=b.employee_id) t1
WHERE t1.num=1
```

### 时间差函数

有如下一张表G0303字段TS，event。TS是时间戳 ，EVENTS是发生的事件编码。

查询某事件发生后及其1小时内的所有记录。

结果如下：

| TS                  | EVENTS  |
| ------------------- | ------- |
| 2023-03-01 09:45:00 | TH03001 |
| 2023-03-01 10:30:00 | TH03001 |
| 2023-03-01 10:45:00 | TH03001 |
| 2023-03-02 09:50:00 | TH03002 |
| 2023-03-02 10:30:00 | TH03002 |
| 2023-03-02 10:45:00 | TH03002 |

**测试语句**

```sql
CREATE TABLE G0303
(
TS DATETIME,
EVENTS VARCHAR(50)
)
INSERT INTO G0303 VALUES ('2023-03-01 09:45:00','TH03001');
INSERT INTO G0303 VALUES ('2023-03-01 10:30:00','TH03001');
INSERT INTO G0303 VALUES ('2023-03-01 10:45:00','TH03001');
INSERT INTO G0303 VALUES ('2023-03-01 11:00:00','TH03001');
INSERT INTO G0303 VALUES ('2023-03-02 09:50:00','TH03002');
INSERT INTO G0303 VALUES ('2023-03-02 10:30:00','TH03002');
INSERT INTO G0303 VALUES ('2023-03-02 10:45:00','TH03002');
INSERT INTO G0303 VALUES ('2023-03-02 11:00:00','TH03002');
```

```sql
SELECT 
a.ts,a.events
FROM g0303 a LEFT JOIN
(SELECT EVENTS,MIN(ts) AS mt
FROM g0303
GROUP BY EVENTS) b ON a.events=b.events
WHERE a.ts<=DATE_ADD(b.mt,INTERVAL 1 HOUR)
```

**mysql8.0 在datediff这个函数地方报错**

### 求累加和

有如下一张表G0302,字段playname为玩家姓名，amount为玩家的充值金额，logtime为具体充值时间。

需要计算玩家使用充值卡这个道具，累计使用满20元时，花费了多长时间？

| 玩家名字 | 时间差   |
| -------- | -------- |
| 曹操     | （null） |
| 关羽     | 13       |
| 刘备     | 9        |

**测试语句**

```sql
CREATE TABLE G0302
(
PLAYERNAME VARCHAR(50),
AMOUNT INT,
LOGTIME DATETIME
)
INSERT INTO G0302 VALUES ('曹操',5,'2023-02-01 15:29:59');
INSERT INTO G0302 VALUES ('刘备',5,'2023-01-06 10:09:44');
INSERT INTO G0302 VALUES ('刘备',10,'2023-01-11 18:23:56');
INSERT INTO G0302 VALUES ('刘备',10,'2023-01-15 20:41:11');
INSERT INTO G0302 VALUES ('刘备',5,'2023-01-29 11:33:26');
INSERT INTO G0302 VALUES ('刘备',5,'2023-02-01 14:30:34');
INSERT INTO G0302 VALUES ('关羽',10,'2023-01-06 16:19:33');
INSERT INTO G0302 VALUES ('关羽',5,'2023-01-17 18:22:45');
INSERT INTO G0302 VALUES ('关羽',5,'2023-01-19 16:36:11');
INSERT INTO G0302 VALUES ('关羽',5,'2023-01-31 19:46:23');
INSERT INTO G0302 VALUES ('关羽',5,'2023-02-05 14:58:34');
```

```sql
SELECT b.PLAYERNAME 玩家名称,MIN(时间差)
FROM 
(SELECT a.PLAYERNAME,
TIMESTAMPDIFF(DAY,MIN(a.LOGTIME) OVER (PARTITION BY a.PLAYERNAME),MIN(CASE WHEN a.cumsum>=20 THEN a.LOGTIME ELSE NULL END) OVER(PARTITION BY  a.PLAYERNAME))
时间差
FROM
(SELECT PLAYERNAME,AMOUNT,LOGTIME,SUM(amount) OVER(PARTITION BY PLAYERNAME ORDER  BY LOGTIME) cumsum
FROM G0302) a)b
GROUP BY PLAYERNAME
```

### select组连接

有如下一张表G0301,字段order_id,price,lkey,需要按订单号合并lkey值，得到如下结果输出结果如下：

| order_id | price | lkey   | newKey               |
| -------- | ----- | ------ | -------------------- |
| 1001     | 10    | 80-100 | 80-100,90-200        |
| 1001     | 20    | 90-200 | 80-100,90-200        |
| 1002     | 30    | 70-130 | 70-130,80-140,90-150 |
| 1002     | 40    | 80-140 | 70-130,80-140,90-150 |
| 1002     | 50    | 90-150 | 70-130,80-140,90-150 |

测试语句

```sql
CREATE TABLE G0301 (
order_id VARCHAR(10)
,price INT
,lkey VARCHAR(20)
)
INSERT INTO G0301 VALUES('1001','10','80-100')
INSERT INTO G0301 VALUES('1001','20','90-200')
INSERT INTO G0301 VALUES('1002','30','70-130')
INSERT INTO G0301 VALUES('1002','40','80-140')
INSERT INTO G0301 VALUES('1002','50','90-150')
```

```sql
--2:
SELECT *,(SELECT GROUP_CONCAT(b.lkey) FROM g0301 b WHERE a.order_id=b.order_id) AS newKey FROM g0301 a
```

SQL 中STUFF用法

删除指定长度的字符，并在指定的起点处插入另一组字符。

STUFF ( character_expression , start , length ,character_expression )

以下示例在第一个字符串 abcdef 中删除从第 2 个位置(字符 b)开始的三个字符，然后在删除的起始位置插入第二个字符串，从而创建并返回一个字符串

SELECT STUFF('abcdef', 2, 3, 'ijklmn')

aijklmnef

文本聚合函数(wm_concat, listagg, group_concat, string_agg

**实现目标**

 **1.聚合文本**

 **2.聚合文本(去重)**

 **3.聚合文本(去重),按照指定字段排序**

 **4.聚合文本(去重),按照指定字段排序,替换默认逗号分隔符**

**MySQL: group_concat**

```sql
SELECT pname As产品
, sum (pnumber) As 数量, group_comcat (pcity) As 发行区域1
, group_cchcat (distinct pcity) As 发行区域2
, group_concat (distinct pcity order by pnuber) As 发行区域3
, group_concat (distinct pcity order by pnumber separator 'll ') AS 发行区域
FROM subtext
GROUP BY pname ;
```

**Oracle: wm_concat(11g), listagg(12c)**

```sql
SELECT PNAME As 产品,SUM(PNUMBER) As 数量
-- wM_CONCAT无法直接实现按某个字段排序(ORACLE12c中默认无此函数)--LISTAGG无法直接实现去重
,wM_CONCAT (PCITY)
--文本聚合
,wM_CONCAT (DISTINCT PCITY)--文本聚合(去重)
,LISTAGG(PCITY, ',') WITHIN GROUP(ORDER BY 1) --文本聚合
,LISTAGG (PCITY,',' ) WITHIN GROUP (ORDER BY PNUMBER) --文本聚合(排序)
FROM SUBTEXT
GROUP BY PNAMEORDERBY PNAME

```

**SQL Server: for XML PATH**

```sql
SELECT M.PNAME,sUM(M.PNUMBER)
,(SELECT N.PCITY + ',' FROM SUBTEXT N WHERE N.PNAME= M.PNAME FOR XML PATH(''))
,(SELECTN.PCITY + ',' FROM SUBTEXT N WHERE N.PNAME =M.PNAME ORDER BY N.PNUNBER FOR XML PATH('')
 ,(SELECT DISTINCT N.PCITY + ',' FROM SUBTEXT N WHERE N.PNAME= M.PNAME FOR XML PATH(''))
，(SELECT DISTINCT M.PCITY + ',' FROM SUBTEXT NNHERE N.PNAME =M.PNAMNE ORDER BY N.PCITY + ',' FOR XME PATH ('')
FROM SUBTEXTM
GROUP BY .PNZME

```

**PostgreSQL: string_agg**

### 截取字符串

有如下两张表G0228，字段生产单号，类型，下单数量，成品入库数量，成品发货数量

说明：生产单号后缀有H的表示是回修改，Ｈ后面的数字表示回修次数。

要求：查询生产单号的下单数量、成品入库数量和成品发货数量

预计结果字段，生产单号，类型，下单数量，成品入库数量，成品发货数量

**测试数据**

```sql
CREATE TABLE G0228
(
生产单号 VARCHAR(25),
类型 VARCHAR(22),
下单数量 INT,
成品入库数量 INT,
成品发货数量 INT
)
INSERT INTO G0228 VALUES('001','新工单',100,80,80);
INSERT INTO G0228 VALUES('001H1','回修单',15,15,15);
INSERT INTO G0228 VALUES('001H2','回修单',5,5,5);
INSERT INTO G0228 VALUES('002','新工单',100,90,90);
INSERT INTO G0228 VALUES('002H1','回修单',10,10,10);
```

```sql
SELECT 
SUBSTRING(a.生产单号,1,3) 生产单号,SUM(a.下单数量) 下单数量,SUM(a.成品入库数量) 成品入库数量,SUM(a.成品发货数量) 成品发货数量
FROM G0228 a GROUP BY SUBSTRING(a.生产单号,1,3)
```

### 差集

有如下两张表G0227A（客户表）字段Id，Name,G0227B（订单表）字段Id,CustomerId

查询G0227B表（订单表）中找出从来没有买过商品的用户。返回Id,Name

**测试数据**

```sql
create table G0227A
(
Id int,
Name varchar(20)
)
insert into G0227A values (1,'曹操');
insert into G0227A values (2,'关羽');
insert into G0227A values (3,'刘备');
insert into G0227A values (4,'张飞');
create table G0227B
(
Id int,
CustomerId int
)
insert into G0227B values (1,3);
insert into G0227B values (2,1);
```

```sql
--1:
select a.Id,a.Name
from G0227A a where not exists (select * from G0227B b where a.id=b.CustomerId);
--2:
select a.Id,a.Name
from G0227A a where id in
(select a.Id
from G0227A a
except
 select b.CustomerId from G0227B b
);
--3:
select a.Id,a.Name
from G0227A a left join G0227B b on a.Id=b.CustomerId where b.CustomerId is null;
--4:
select a.Id,a.Name
from G0227A a where a.Id not in (select b.CustomerId from G0227B b);
```

## 每日sql技巧

### sql执行顺序：

(1)from笛卡尔积(3) join不符合on,逻辑表达式也添加(2) on主表保留(4) where无法聚合，无法使用别名(5)group by(开始使用select中的别名，后面的语句中都可以使用)(6) avg,sum…聚合计算(7)with,rollup,cube(8)having仅用于分组后(9) select选择列，表达式计算(10) distinct行去重(11) order by无法应用表达式（12）limit选择指定数量行

1 FROM 执行笛卡尔积
FROM 才是 SQL 语句执行的第一步，并非 SELECT 。对FROM子句中的前两个表执行笛卡尔积(交叉联接），生成虚拟表VT1，获取不同数据源的数据集。

FROM子句执行顺序为从后往前、从右到左，FROM 子句中写在最后的表(基础表 driving table)将被最先处理，即最后的表为驱动表，当FROM 子句中包含多个表的情况下，我们需要选择数据最少的表作为基础表。

2 ON 应用ON过滤器
对虚拟表VT1 应用ON筛选器，ON 中的逻辑表达式将应用到虚拟表 VT1中的各个行，筛选出满足ON 逻辑表达式的行，生成虚拟表 VT2 。

3 JOIN 添加外部行
如果指定了OUTER JOIN保留表中未找到匹配的行将作为外部行添加到虚拟表 VT2，生成虚拟表 VT3。保留表如下：

LEFT OUTER JOIN把左表记为保留表
RIGHT OUTER JOIN把右表记为保留表
FULL OUTER JOIN把左右表都作为保留表
在虚拟表 VT2表的基础上添加保留表中被过滤条件过滤掉的数据，非保留表中的数据被赋予NULL值，最后生成虚拟表 VT3。

如果FROM子句包含两个以上的表，则对上一个联接生成的结果表和下一个表重复执行步骤1~3，直到处理完所有的表为止。

4 WHERE 应用WEHRE过滤器
对虚拟表 VT3应用WHERE筛选器。根据指定的条件对数据进行筛选，并把满足的数据插入虚拟表 VT4。

由于数据还没有分组，因此现在还不能在WHERE过滤器中使用聚合函数对分组统计的过滤。
同时，由于还没有进行列的选取操作，因此在SELECT中使用列的别名也是不被允许的。
5 GROUP BY 分组
按GROUP BY子句中的列/列表将虚拟表 VT4中的行唯一的值组合成为一组，生成虚拟表VT5。如果应用了GROUP BY，那么后面的所有步骤都只能得到的虚拟表VT5的列或者是聚合函数（count、sum、avg等）。原因在于最终的结果集中只为每个组包含一行。

同时，从这一步开始，后面的语句中都可以使用SELECT中的别名。

6 AGG_FUNC 计算聚合函数
计算 max 等聚合函数。SQL Aggregate 函数计算从列中取得的值，返回一个单一的值。常用的 Aggregate 函数包涵以下几种：

AVG：返回平均值
COUNT：返回行数
FIRST：返回第一个记录的值
LAST：返回最后一个记录的值
MAX： 返回最大值
MIN：返回最小值
SUM： 返回总和
7 WITH 应用ROLLUP或CUBE
对虚拟表 VT5应用ROLLUP或CUBE选项，生成虚拟表 VT6。

CUBE 和 ROLLUP 区别如下：

CUBE 生成的结果数据集显示了所选列中值的所有组合的聚合。
ROLLUP 生成的结果数据集显示了所选列中值的某一层次结构的聚合。
8 HAVING 应用HAVING过滤器
对虚拟表VT6应用HAVING筛选器。根据指定的条件对数据进行筛选，并把满足的数据插入虚拟表VT7。

HAVING 语句在SQL中的主要作用与WHERE语句作用是相同的，但是HAVING是过滤聚合值，在 SQL 中增加 HAVING 子句原因就是，WHERE 关键字无法与聚合函数一起使用，HAVING子句主要和GROUP BY子句配合使用。

9 SELECT 选出指定列
将虚拟表 VT7中的在SELECT中出现的列筛选出来，并对字段进行处理，计算SELECT子句中的表达式，产生虚拟表 VT8。

10 DISTINCT 行去重
将重复的行从虚拟表 VT8中移除，产生虚拟表 VT9。DISTINCT用来删除重复行，只保留唯一的。

11 ORDER BY 排列
将虚拟表 VT9中的行按ORDER BY 子句中的列/列表排序，生成游标 VC10 ，注意不是虚拟表。因此使用 ORDER BY 子句查询不能应用于表达式。同时，ORDER BY子句的执行顺序为从左到右排序，是非常消耗资源的。

12 LIMIT/OFFSET 指定返回行
从VC10的开始处选择指定数量行，生成虚拟表 VT11，并返回调用者

### CUBE和ROLLUP实现数据多维汇总

【GROUP BY】加上【WITH ROLLUP】子句，看ROLLUP能不能提供更多的统计结果。前面说到多条件，其实说多维度更准确些。看个例子先：

--语句1 只用了【性别】一个维度进行汇总
SELECT 性别,  COUNT(学号) AS 数量
FROM STUDENT
GROUP BY 性别 WITH ROLLUP

--语句2 用了【性别】和【籍贯】两个维度进行汇总
SELECT 性别, 籍贯, COUNT(学号) AS 数量
FROM STUDENT
GROUP BY 性别, 籍贯 WITH ROLLUP

--语句3 用了【性别】、【籍贯】、【年龄】三个维度进行汇总
SELECT 性别, 籍贯, 年龄, COUNT(学号) AS 数量
FROM STUDENT
GROUP BY 性别, 籍贯, 年龄 WITH ROLLUP

`ROLLUP`计算了指定分组（就是汇总的维度）的多个层次的数量小计以及合计，先逐步创建高一级别的小计，最后再创建一行总计。整体结果都是以【性别】这一层次进行数据聚合（这也是与`CUBE`的不同之处）

【`GROUP BY`】+【`WITH CUBE`】

`CUBE`可以提供所选择列的所有组合的聚合。简单说，`CUBE`生成的结果是个多维数据集，就是包含各个维度的所有可能组合的交叉表格

*--语句只用了【性别】和【籍贯】两个维度进行汇总* SELECT 性别, 籍贯,  COUNT(学号) AS 数量 FROM STUDENT GROUP BY 性别, 籍贯 WITH CUBE

CUBE可以为指定的列创建各种不同组合的小计，是一种比 ROLLUP更细粒度的分组统计语句。如果将统计维度调整到三个维度，会与ROLLUP有更大的差异

`CUBE`和`ROLLUP`之间的区别在于：

`CUBE` 生成的结果集显示了所选列中值的所有组合的聚合。

`ROLLUP` 生成的结果集显示了所选列中值的某一层次结构的聚合。

感觉也可以这样来说：`ROLLUP`就是将`GROUP BY`后面的第一列名称求总和，而其他列并不要求，而`CUBE`则会将每一个列名称都求总和。

利用`sum()`和`with rollup`对学生信息表（studenttable）中的所有年龄生成小计

```sql
Select studentsex,sum(studentage) from studenttable group by [studentsex] with rollup
```

b. 利用`max()`和`with rollup` 对学生信息表(studenttable) 中的年龄最大值生成小计

```sql
Select studentsex,max(studentage) from studenttable group by [studentsex] with rollup
```

c．利用`min（）`和`with rollup`对学生信息表（studenttable）中的年龄最小值生成小计

```sql
select studentsex,min(studentage) from studenttable group by [studentsex] with rollup
1
```

d. 利用`avg（）`和`with rollup` 对学生表（studenttable）中的年龄平均值生成小计

```sql
Select studentsex,avg(studentage) from studenttable group by [studentsex] with rollup
```

e. 利用`count()`和`with rollup`对学生表（studenttable）中的记录数生成小计

```sql
Select studentsex,count(*) from studenttable group by [studentsex] with rollup
```

注意：
当`with rollup`与`sum（）`一起使用时得出的结果是分组后每组的和的和
当`with rollup`与`max（）`一起使用时得出的结果是分组后组的较大值
当`with rollup`与`min（）`一起使用时得出的结果是分组后组的较小值
当`with rollup`与`avg（）`一起使用时得出的结果是所以记录的平均值
当`with rollup`与`count（）`一起使用时得出的结果是分组后各组数量的总和

利用`sum（）`和`CUBE`对图书销售表（Booksales）中的销售额生成小计和总计

```sql
Select b_code,b_number,sum(sal_tot) from Booksales group by b_code,b_number with cube
```

b. 利用`max()`和`CUBE`对图书销售表(Booksales)中的销售额生成小计和总计

```sql
Select b_code,b_number,max(sal_tot) from Booksales group by b_code,b_number with cube
```

c. 利用`min()`和`CUBE`对图书销售表(Booksales)中的销售额生成小计和总计

```sql
Select b_code,b_number,min(sal_tot) from Booksales group by b_code,b_number with cube
```

d. 利用`avg()`和`cube`对图书销售表（Booksales）中的销售额生成小计和总计

```sql
Select b_code,b_number,avg(sal_tot) from Booksales group by b_code,b_number with cube
```

e. 利用`count（）`和`cube`对图书销售表（Booksales）中的销售额生成小计和总计

```sql
Select b_code,b_number,count（*）from Booksales group by b_code,b_number with cube
```

### `compute`与`compute by`的使用方法

说明:`compute`是对聚合函数生成小计的，一个`select`语句中可以有多个`compute`指定的聚合函数，注意在`compute`子句的聚合函数中不可以使用列别名，`compute`输出的结果为两行，
第一行：生成`select_list`列表中行的明细
第二行：`compute`子句中聚合函数的小计
`Compute by` 则是先实现分组然后统计`compute`子句中聚合函数小计的,在`compute by`语句中必须要有`order by`排序并且排序列和`by`分组列必须相同
第一行：生成`select_list`列表中行的明细
第二行：`compute`子句中聚合函数的小计
两个都是输出`compute`子句中聚合函数的小计,但是不相同的是`compute`输出的是聚合列在表中的所有值，而`compute by` 则是输出先按`by`子句中的字段分组以后每组的聚合列的值！
a． 查询图书销售表(Booksales)中的总销售额，销售个数

```sql
Select * from Booksales compute sum(sal_tot),sum(b_number)
```

b． 查询图书销售表(Booksales)中的销售额最小和最大并按图书编号升序排列

```sql
Select * from Booksales order by b_code compute max(sal_tot),min(sal_tot)
```

c． 查询图书销售表(Booksales)中的总销售额，销售个数按`b_code`分组

```sql
select * from Booksales order by b_code compute sum(sal_tot),sum(b_number) by b_code
```

### `group by` 和 `compute by` 的比较

说明：`group by`与`compute by`都可以进行分组，但是他们有一定的区别
`Group by` :生成单个结果集,每一行都只有分组依据列和聚合列,选择列表也只能包含分组依据列和聚合列，可排序也可不排序，如果要排序的话只能先分组后排序，排序列必须和分组列相同或者是分组列的子集或者为聚合函数产生的新列，聚合函数写在`select`列表中，在`group by` 子句中不可以使用聚合函数、在聚合函数聚合结果中可以使用列别名。
`Compute by`：生成多个结果集，一种结果集包括选择列表中的行的明细，另一种结果集则包含聚合以后的小计，在`compute by` 子句中必须排序，并且排序列必须和`compute by`的中分组列相同，聚合函数写在`compute`子句中，分组列写在`by`子句中，分组列可以是表中的任意列，但不可以使用列别名，在聚合函数聚合结果中不可以使用列别名



### **在使用left join时，on和where条件的区别如下：**

1、 on条件是在生成临时表时使用的条件，它不管on中的条件是否为真，都会返回左边表中的记录。

2、where条件是在临时表生成好后，再对临时表进行过滤的条件。这时已经没有left join的含义（必须返回左边表的记录）了，条件不为真的就全部过滤掉

left join,right join,full join的特殊性，不管on上的条件是否为真都会返回left或right表中的记录，full则具有left和right的特性的并集。而inner join没这个特殊性，则条件放在on中和where中，返回的结果集是相同的

### `group by and all`

说明：本实例中利用了`group by`子句和`all`关键字，在`group by` 子句中使用`all`关键字，只有在sql语句中包含`where`子句时，`all`才有意义。

a. 查询图书销售表(Booksales)中图书编号为1100010101的图书销售总额，且列出其他图书编号

```sql
Select b_code,sum(sal_tot) from Booksales where b_code=’1100010101’ group by all b_code
```





### TIMESTAMPDIFF

datediff

含义：计算两个日期之间的天数。正数表示前者大于后者，负数表示前者小于后者。

用法：datediff(date1,date2)

注意:**mysql8.0 在datediff这个函数地方报错**

用TIMESTAMPDIFF（``SECOND``,starttime,endtime）代替，函数获得时间差，得到的可以是DAY/天，HOUR/小时，MINUTE/分钟，SECOND/秒。

其中starttime为时间小的那个时间，endtime为时间大的时间。

### 常用日期函数

1、current_date

含义：获取当前日期。

2、current_timestamp

含义：获取当前时间。

3、date_format

含义：将日期格式化。

用法：date_format(date,格式)

4、to_date

含义：转为日期格式，默认为yyyy-MM-dd格式。

用法：to_date(time)

5、date_add

含义：日期加法函数，数字为正，则加多少天；为负，则减多少天。

用法：date_add(date,number)  

date_add(date,interval 3 DAY)

6、date_sub

含义：与date_add对应，日期减法函数，数字为正，则减多少天；为负，则加多少天。

用法：date_sub(date,number)

7、add_months

含义：日期加一个月。

用法：add_months(date,number)

8、next_day

含义：该日期的下一个周几所在的日期。（通俗理解：某日期的下周几是多少号）

用法：next_day(date,dayofweek)

9、last_day

含义：当月最后一天的日期。

用法：last_day(date)

11、dayofmonth

含义：日期所在月份的第多少天。

用法：dayofmonth(date)

12、weekofyear

含义：日期所在年份的第多少周。

用法：weekofyear(date)

案例：

1、取当月第1天

先获取当前日期在该月的第n天，然后当前日期减去第（n-1）天。

```sql
select date_sub('2022-09-13',dayofmonth('2022-09-13')-1);>> 2022-09-01
```

2、取当月第8天

先获取当前日期在该月的第n天，然后当前日期减去第（n-1）天，再增加（m-1）天。

```sql
select date_add(date_sub('2022-09-13',dayofmonth('2022-09-13')-1),8-1);>> 2022-09-08
```

3、查询下一个月的第一天

方式一：先获取最后一天，然后日期+1。

```sql
select date_add(last_day('2022-09-13'),1);>> 2022-10-01
```

方式二：先获取今天是当月第几天，算出当月第一天，然后加一个月。

```sql
select add_months(date_sub('2022-09-13',dayofmonth('2022-09-13')-1),1);>> 2022-10-01
```

4、获取本周一的日期

先获取下周一的日期，然后减去7天。

```sql
select date_add(next_day(current_date,"MO"),-7);>> 2022-09-12
```



### 使用Excel快速生成SQL语句

**写好模板语句**

我们可以先写一条插入语句，如下：

```sql
INSERT INTO Person VALUES(1,'吕布',25,'男','13500000001')
```

然后复制这条SQL语句打开Excel，选中表格后的一个单元格，在上方函数位置粘贴刚才的SQL语句并做修改，

```
="INSERT INTO Person VALUES("&A2&",'"&B2&"',"&C2&",'"&D2&"','"&E2&"')"
```

注意前面有个= 然后整个SQL用 ""包围住。

**生成SQL语句**

确认后就可以看到在单元格中会自动生成一条SQL语句。选中单元格下拉，会发现所有的行后面都会生成一条SQL语句

**执行SQL**

然后我们直接复制这些SQL语句到数据库的查询窗口执行。

比如要查询数据库中所有表中的记录数。

可以先从系统表中查询出所有的表名

```sql
SELECT TABLE_NAME FROM user_tables
```

将表名复制粘贴到Excel中，然后开始写查询语句

```
="SELECT '"&A2&"',count(1) num FROM "&B2&""
```

然后将这些代码复制粘贴到查询窗口即可查询出所有表中的记录数了。

### **01 删除指定列、重命名列**

**场景**：

多数情况并不是底表的所有特征（列）都对分析有用，这个时候就只需要抽取部分列，对于不用的那些列，可以删除。

重命名列可以避免有些列的命名过于冗长（比如Case When 语句），且有时候会根据不同的业务指标需求来命名。

```sql
删除列Python版：
df.drop(col_names, axis=1, inplace=True)

删除列SQL版：
1、select col_names from Table_Name

2、alter table tableName drop column columnName

重命名列Python版：
df.rename(index={'row1':'A'},columns ={'col1':'B'})

重命名列SQL版：
select col_names as col_name_B from Table_Name
```

**02 重复值、缺失值处理**

**场景**：比如某网站今天来了1000个人访问，但一个人一天中可以访问多次，那数据库中会记录用户访问的多条记录，而这时候如果想要找到今天访问这个网站的1000个人的ID并根据此做用户调研，需要去掉重复值给业务方去回访。

缺失值：NULL做运算逻辑时，返回的结果还是NULL，这可能就会出现一些脚本运行正确，但结果不对的BUG，此时需要将NULL值填充为指定值。

```sql
重复值处理Python版：
df.drop_duplicates()

重复值处理SQL版：
1、select distinct col_name from Table_Name

2、select col_name from Table_Name group bycol_name

缺失值处理Python版：
df.fillna(value = 0)

df1.combine_first(df2)

缺失值处理SQL版：
1、select ifnull(col_name,0) value from Table_Name

2、select coalesce(col_name,col_name_A,0) as value from Table_Name

3、select case when col_name is null then 0 else col_name end from Table_Name
```

**03 替换字符串空格、清洗\*%@等垃圾字符、字符串拼接、分隔等字符串处理**

**场景**：理解用户行为的重要一项是去假设用户的心理，这会用到用户的反馈意见或一些用研的文本数据，这些文本数据一般会以字符串的形式存储在数据库中，但用户反馈的这些文本一般都会很乱，所以需要从这些脏乱的字符串中提取有用信息，就会需要用到文字符串处理函数。

```sql
字符串处理Python版：
## 1、空格处理
df[col_name] = df[col_name].str.lstrip() 

## 2、*%d等垃圾符处理
df[col_name].replace(' &#.*', '', regex=True, inplace=True)

## 3、字符串分割
df[col_name].str.split('分割符')

## 4、字符串拼接
df[col_name].str.cat()

字符串处理SQL版：
## 1、空格处理
select ltrim(col_name) from Table_name 

## 2、*%d等垃圾符处理
select regexp_replace(col_name,正则表达式) from Table_name 

## 3、字符串分割
select split(col_name,'分割符') from Table_name 

## 4、字符串拼接
select concat_ws(col_name,'拼接符') from Table_name 
```

**04 合并处理**

**场景**：有时候你需要的特征存储在不同的表里，为便于清洗理解和操作，需要按照某些字段对这些表的数据进行合并组合成一张新的表，这样就会用到连接等方法。

```sql
合并处理Python版：

左右合并
1、pd.merge(left, right, how='inner', on=None, left_on=None, right_on=None,
         left_index=False, right_index=False, sort=True,
         suffixes=('_x', '_y'), copy=True, indicator=False,
         validate=None)
2、pd.concat([df1,df2])

上下合并
df1.append(df2, ignore_index=True, sort=False)

合并处理SQL版：

左右合并
select A.*,B.* from Table_a A join Table_b B on A.id = B.id

select A.* from Table_a A left join Table_b B on A.id = B.id

上下合并
## Union：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；
## Union All：对两个结果集进行并集操作，包括重复行，不进行排序；

select A.* from Table_a A 
union
select B.* from Table_b B 

# Union 因为会将各查询子集的记录做比较，故比起Union All ，通常速度都会慢上许多。一般来说，如果使用Union All能满足要求的话，务必使用Union All。 
```

**05、窗口函数的分组排序**

**场景**：假如现在你是某宝的分析师，要分析今年不同店的不同品类销售量情况，需要找到那些销量较好的品类，并在第二年中加大曝光，这个时候你就需要将不同店里不同品类进行分组，并且按销量进行排序，以便查找到每家店销售较好的品类。

Demo数据如上，一共a,b,c三家店铺，卖了不同品类商品，销量对应如上，要找到每家店卖的最多的商品。

```sql
窗口分组Python版：

df['Rank'] = df.groupby(by=['Sale_store'])['Sale_Num'].transform(lambda x: x.rank(ascending=False))

窗口分组SQL版：

select 
  * 
from
  (
  Select 
    *，
    row_number() over(partition by Sale_store order by Sale_Num desc) rk
  from 
    table_name
  ) b where b.rk = 1
```

### wm_concat()和group_concat()合并同列变成一行的用法

在oracle中，concat()函数和 “ || ” 这个的作用是一样的，是将不同列拼接在一起；那么wm_concat()是将同属于一个组的（group by）同一个字段拼接在一起变成一行。mysql是一样的，只不过mysql用的是group_concat()这个函数，用法是一样

oracle中：

concat只能连接两个字符串或者两个字段，|| 可以多次使用，拼接n个字符串或者字段

wm_concat(string <separator>, string <colname>)

- separator：必填。STRING类型常量，分隔符。
- colname：必填。STRING类型。如果输入为BIGINT、DOUBLE或DATETIME类型，会隐式转换为STRING类型后参与运算。

mysql中
concat()的使用，是可以连接多个字符串或者字段的。

group_concat(列名或与其他字符拼接 Separator ‘分隔符’)

```sql
--同一个同学的课程+成绩
select stuid,wm_concat(coursename || '(' || score||')')
from stu_score
group by stuid
```

