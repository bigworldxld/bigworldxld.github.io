---
title: sql常见问题
date: 2023-03-05 16:35:59
tags: SQL
categories: database
img: /images/SQL数据分析题.jpg
---

## SQL常见错误指南

### 语法错误

标点错漏

e.g. 逗号多或缺，括号等不成对，漏写引号、多余的空格等；

e.g. case when … end函数，有时候少写end；

e.g. select含有 聚合函数(count, sum, avg)时，相应字段都要放入group by 后面；

重命名

如果有子查询，那么需要对子查询进行重命名；

表的重命名不要搞混；

数据拼接

e.g. union all时要求字段的名称和顺序都要保持一致；

e.g. join操作要求两边的字段格式一致；

e.g. join关联的时候注意是`1对1`映射还是`1对多`映射，小心出现**笛卡尔积**的情况；

null值

- 正常的数值和null值做四则运算，得到的结果还是null，建议用`isnull`、`coalesce`之类的函数对null值进行处理，或者计算的时候在where字句中过滤null值；
- `sum/avg(case when end)`操作时要加 `else 0` 不然会出现null的情况；
- join操作是最常见的出现null的情形(无匹配时)；
- join操作可能会因为null值产生数据倾斜。

### 函数错误

参数数量

e.g. 某函数需输入2个参数，结果只有1个

参数格式

e.g. `to_date(string timestamp)`，`select to_date('20161125')` 返回值为null，因为数据格式不是日期时间

e.g. 使用`between and`时还要注意字段和条件的颗粒度匹配，比如对某个timestamp字段(日期时间格式，带有时分秒的)时，如下代码

```sql
where order_time between '2020-09-01' and '2020-09-15'
```

判断条件给到的格式是日期，而字段是日期时间格式，`2010-09-15`对应的日期时间格式是`2020-09-15 00:00:00`，那么实际上9月15号0点后的数据实际是没有被选中的，对于这种情况，可以将原有的日期时间字段用`to_date`或者`substr`处理一下。

函数逻辑

e.g. `between 小值 and 大值`, 注意最小值在前,最大值在后，这个含义是`[小值,大值]`，是包含边界的；

e.g. 函数`datediff`中第1个参数是起始日期(通常是较小值)，第2个参数是结束日期(通常为较大值)

### 逻辑错误

数据重复

对于存在一对多关系的数据表关联后会产生数据重复，这种重复对于sum/avg等非去重的统计计算操作有影响，对`count(distinct *)`等去重计数操作没影响
e.g. 一张母订单可以对应多张子订单；
e.g. 一个用户可以对应多条交易记录；

无效筛选

**隐藏前提**

```sql
select a.col1,b.col2
from a
left join b on(a.id = b.id)
where b.tag = '1'
```

实际上`b.tag='1'` 这个筛选条件已经带有`b.tag is not null` 的“隐藏前提”了，所以这里用`left join` 和 `join`的效果是一样的。

及到转化率的时候，表的顺序和转化率的顺序是一致的，且不能在`where`子句中添加后续流程的筛选条件，不然“隐藏前提”会过滤掉一部分数据而导致结果有误。

标签重叠

建立标签的时候要符合MECE原则(相互独立，完全穷尽)；

一般来说建立标签的时候使用简单的逻辑，每个维度单独成列(基础标签)；e.g. 性别区分：男、女、未知；

编写sql进行分组统计时，不建议使用“复合逻辑”标签，复合标签不仅逻辑上容易出错(标签重叠)，维护成本也更高。e.g. 同时考虑会员等级和性别，然后对应的标签值就会是：(铁牌、铜牌、银牌、金牌、钻石、皇冠)*(男，女，未知)；

计算用户数量时，同一用户可能会有多个标签(行为标签、属性标签、不同时间段等)，这样同一用户会分别存在多个标签中，对各标签求和会大于实际用户数量。

此外，一个用户有多个标签时，可能会涉及到多个标签的“或、且、非”运算。

e.g. 一个用户在某一时刻，可能有多张优惠券，优惠券的状态可能是【已使用、已过期、未使用】等，现在要判断当前有“未使用”的优惠券。

时间错位

即数据匹配时要在时间维度上要对齐。

e.g. T+1的用户标签匹配时，昨日的标签匹配今日的交易情况；

多行判断

假设订单表order_info有如下字段

| 字段名(En)  | 字段名                         |
| :---------- | :----------------------------- |
| order_id    | 订单号                         |
| user_id     | 用户ID                         |
| create_time | 订单生成时间                   |
| order_amt   | 订单金额(优惠前)               |
| fav_amt     | 优惠金额                       |
| pay_amt     | 实际支付金额=订单金额-优惠金额 |

注：

- 实际支付金额=订单金额-优惠金额
- 订单有使用优惠则fav_amt>0,否则其值为0

筛选第一单使用优惠且第二单没有使用优惠的用户ID，其中可能用到如下逻辑

```sql
(rn=1 and fav_amt>0)
or 
(rn=2 and fav_amt=0)
```

然后筛选`rn in (1,2)` 然后对符合条件的订单去重计数=2

筛选条件是针对一行一行的数据去匹配的，所以要注意多行条件判断时行与行之间的or关系。

### **1、LIMIT 语句**

一般 DBA 想到的办法是在 type, name, create_time 字段上加组合索引。这样条件排序都能有效的利用到索引，性能迅速提升。

```sql
SELECT * FROM  operation WHERE type = 'SQLStats' AND name = 'SlowLog' ORDER BY create_time LIMIT 1000000, 10; 
```

LIMIT 1000000,10 有索引也需要从头计算一次。出现这种性能问题

SQL 重新设计如下：

```sql
SELECT  * FROM   operation WHERE  type = 'SQLStats' AND   name = 'SlowLog' AND   create_time > '2017-03-16 14:00:00' ORDER BY create_time limit 10;
```

在新设计下查询时间基本固定，不会随着数据量的增长而发生变化

### **2、隐式转换**

SQL语句中查询变量和字段定义类型不匹配是另一个常见的错误。比如下面的语句：

```sql
explain extended SELECT * FROM  my_balance b WHERE b.bpn = 14000000123 AND b.isverified IS NULL; 
show warnings;
```

| Warning | 1739 | Cannot use ref access on index 'bpn' due to type or collation conversion on field 'bpn'

其中字段 bpn 的定义为 varchar(20)，MySQL 的策略是将字符串转换为数字之后再比较。函数作用于表字段，索引失效。

### **3、关联更新、删除**

比如下面 UPDATE 语句，MySQL 实际执行的是循环/嵌套子查询（DEPENDENT SUBQUERY)，其执行时间可想而知

```sql
UPDATE operation o SET  status = 'applying' WHERE o.id IN (SELECT id         FROM  (SELECT o.id,                o.status             FROM  operation o             WHERE o.group = 123                AND o.status NOT IN ( 'done' )             ORDER BY o.parent,                  o.id             LIMIT 1) t);
```

子查询的选择模式从 DEPENDENT SUBQUERY 变成 DERIVED，执行速度大大加快，从7秒降低到2毫秒。

```sql
UPDATE operation o    JOIN (SELECT o.id,               o.status           FROM  operation o           WHERE o.group = 123               AND o.status NOT IN ( 'done' )           ORDER BY o.parent,                o.id           LIMIT 1) t     ON o.id = t.id SET  status = 'applying' 
```

### **4、混合排序**

MySQL 不能利用索引进行混合排序。但在某些场景，还是有机会使用特殊方法提升性能的。

```sql
SELECT * FROM  my_order o INNER JOIN my_appraise a 
ON a.orderid = o.id 
ORDER BY a.is_reply ASC,      a.appraise_time DESC 
LIMIT 0, 20 
```

执行计划显示为全表扫描

由于 is_reply 只有0和1两种状态，我们按照下面的方法重写后，执行时间从1.58秒降低到2毫秒。

```sql
SELECT * FROM  
((SELECT *     FROM  my_order o INNER JOIN my_appraise a ON a.orderid = o.id AND is_reply = 0     ORDER BY appraise_time DESC LIMIT 0, 20)     
 UNION ALL     
(SELECT *     FROM  my_order o INNER JOIN my_appraise a ON a.orderid = o.id AND is_reply = 1     ORDER BY appraise_time DESC     LIMIT 0, 20)) t 
ORDER BY is_reply ASC,      
appraisetime DESC 
LIMIT 20;
```

### **5、EXISTS语句**

MySQL 对待 EXISTS 子句时，仍然采用嵌套子查询的执行方式。如下面的 SQL 语句：

```sql
SELECT *FROM  my_neighbor n LEFT JOIN my_neighbor_apply sra ON n.id =sra.neighbor_id AND sra.user_id = 'xxx' WHERE n.topic_status < 4 AND 
EXISTS(SELECT 1 FROM  message_info m  WHERE n.id = m.neighbor_id AND m.inuser = 'xxx') AND n.topic_type <> 5
```

去掉 exists 更改为 join，能够避免嵌套子查询，将执行时间从1.93秒降低为1毫秒。

```sql
SELECT *FROM  my_neighbor n    INNER JOIN message_info m        ON n.id = m.neighbor_id          AND m.inuser = 'xxx'    LEFT JOIN my_neighbor_apply sra        ON n.id = sra.neighbor_id         AND sra.user_id = 'xxx' WHERE n.topic_status < 4    AND n.topic_type <> 5 
```

### ** 6、条件下推**

外部查询条件不能够下推到复杂的视图或子查询的情况有：

1、聚合子查询；2、含有 LIMIT 的子查询；3、UNION 或 UNION ALL 子查询；4、输出字段中的子查询；

如下面的语句，从执行计划可以看出其条件作用于聚合子查询之后：

```sql
SELECT * FROM  
(SELECT target,Count(*) 
 FROM  operation     GROUP BY target) t WHERE target = 'rm-xxxx' 
```

确定从语义上查询条件可以直接下推后，重写如下：

```sql
SELECT target,    Count(*) FROM  operation WHERE target = 'rm-xxxx' GROUP BY target
```

### **7、提前缩小范围\****

先上初始 SQL 语句：

```sql
SELECT * 
FROM my_order o LEFT JOIN my_userinfo u ON o.uid = u.uid LEFT JOIN my_productinfo p ON o.pid = p.pid 
WHERE ( o.display = 0 )    AND ( o.ostaus = 1 ) 
ORDER BY o.selltime DESC 
LIMIT 0, 15 
```

该SQL语句原意是：先做一系列的左连接，然后排序取前15条记录。从执行计划也可以看出，最后一步估算排序记录数为90万，时间消耗为12秒

由于最后 WHERE 条件以及排序均针对最左主表，因此可以先对 my_order 排序提前缩小数据量再做左连接。SQL 重写后如下，执行时间缩小为1毫秒左右。

```sql
SELECT * FROM 
(SELECT * FROM  my_order o WHERE ( o.display = 0 )    AND ( o.ostaus = 1 ) ORDER BY o.selltime DESC LIMIT 0, 15) o   LEFT JOIN my_userinfo u ON o.uid = u.uid   LEFT JOIN my_productinfo p ON o.pid = p.pid 
ORDER BY o.selltime DESC
limit 0, 15
```

再检查执行计划：子查询物化后（select_type=DERIVED)参与 JOIN。虽然估算行扫描仍然为90万，但是利用了索引以及 LIMIT 子句后，实际执行时间变得很小

### **8、中间结果集下推**

再来看下面这个已经初步优化过的例子(左连接中的主表优先作用查询条件)：

```sql
SELECT  a.*,c.allocated 
FROM (SELECT resourceid
      FROM   my_distribute d WHERE isdelete = 0 AND cusmanagercode = '1234567'          ORDER BY salecode limit 20) a 
LEFT JOIN 
(SELECT resourcesid,sum(ifnull(allocation, 0)*12345) allocated FROM   my_resources          GROUP BY resourcesid) c 
ON a.resourceid = c.resourcesid
```

存在其它问题吗？不难看出子查询 c 是全表聚合查询，在表数量特别大的情况下会导致整个语句的性能下降。

其实对于子查询 c，左连接最后结果集只关心能和主表 resourceid 能匹配的数据。因此我们可以重写语句如下，执行时间从原来的2秒下降到2毫秒。

```sql
SELECT  a.*,c.allocated 
FROM (SELECT  resourceid FROM my_distribute d  WHERE isdelete = 0 AND cusmanagercode = '1234567' ORDER BY salecode limit 20) a LEFT JOIN  
(SELECT  resourcesid,sum(ifnull(allocation, 0) * 12345) allocated FROM my_resources r, (SELECT  resourceid FROM   my_distribute d WHERE  isdelete = 0 AND   cusmanagercode = '1234567' ORDER BY salecode limit 20) a     WHERE  r.resourcesid = a.resourcesid       GROUP BY resourcesid) c 
ON    a.resourceid = c.resourcesid
```

但是子查询 a 在我们的SQL语句中出现了多次。这种写法不仅存在额外的开销，还使得整个语句显的繁杂。使用 WITH 语句再次重写：

```sql
WITH a AS (SELECT  resourceid     FROM   my_distribute d     WHERE  isdelete = 0     AND   cusmanagercode = '1234567'     ORDER BY salecode limit 20)
SELECT  a.*,c.allocated FROM a LEFT JOIN   (SELECT  resourcesid,sum(ifnull(allocation, 0) * 12345) allocated FROM my_resources r,a WHERE r.resourcesid = a.resourcesid         GROUP BY resourcesid) c 
ON    a.resourceid = c.resourcesid
```

## SQL中去重的三种方法

在 MySQL 中通常是使用 distinct 或 group by子句，但在支持窗口函数的 sql（如Hive SQL、Oracle等等） 中还可以使用 row_number 窗口函数进行去重。

- `task_id`: 任务id;
- `order_id`: 订单id;
- `start_time`: 开始时间

> 注意：一个任务对应多条订单

我们需要求出任务的总数量，因为 task_id 并非唯一的，所以需要去重：

### distinct

```sql
-- 列出 task_id 的所有唯一值（去重后的记录）
-- select distinct task_id
-- from Task;

-- 任务总数
select count(distinct task_id) task_num
from Task;
```

distinct 通常效率较低。它不适合用来展示去重后具体的值，一般与 count 配合用来计算条数。

distinct 使用中，放在 select 后边，对后面所有的字段的值统一进行去重。

### group by

```sql
-- 列出 task_id 的所有唯一值（去重后的记录,null也是值）
-- select task_id
-- from Task
-- group by task_id;

-- 任务总数
select count(task_id) task_num
from (select task_id
      from Task
      group by task_id) tmp;
```

### row_number

row_number 是窗口函数，语法如下：

```
row_number() over (partition by <用于分组的字段名> order by <用于组内排序的字段名>)
```

其中 partition by 部分可省略。

```sql
-- 在支持窗口函数的 sql 中使用
select count(case when rn=1 then task_id else null end) task_num
from (select task_id
       , row_number() over (partition by task_id order by start_time) rn
   from Task) tmp;
```

| user_id | user_type |
| ------- | --------- |
| 1       | 1         |
| 1       | 2         |
| 2       | 1         |

*-- 下方的分号;用来分隔行*

**select** **distinct** user_id
**from** **Test**; 

返回

 1;2

**select** **distinct** user_id, user_type
**from** **Test**; 

 *-- 返回1, 1; 1, 2; 2, 1*

**select** user_id
**from** **Test**
**group** **by** user_id; 

 *-- 返回1; 2*

**select** user_id, user_type
**from** **Test**
**group** **by** user_id, user_type;

  *-- 返回1, 1; 1, 2; 2, 1*

**select** user_id, user_type
**from** **Test**
**group** **by** user_id;  
*-- Hive、Oracle等会报错，mysql可以这样写。*
*-- 返回1, 1 或 1, 2 ; 2, 1（共两行）。只会对group by后面的字段去重，就是说最后返回的记录数等于上一段sql的记录数，即2条*
*-- 没有放在group by 后面但是在select中放了的字段，只会返回一条记录（好像通常是第一条，应该是没有规律的）*

## 避免 MySQL 插入重复数据的 4 种方式

### 01 insert ignore into

即插入数据时，如果数据存在，则忽略此次插入，前提条件是插入的数据字段设置了主键或唯一索引，测试SQL语句如下，当插入本条数据时，MySQL数据库会首先检索已有数据（也就是idx_username索引），如果存在，则忽略本次插入，如果不存在，则正常插入数据：

```sql
insert ignore into user(username,sex,address) values ('Tom','male','China');
```

### 02 on duplicate key update

即插入数据时，如果数据存在，则执行更新操作，前提条件同上，也是插入的数据字段设置了主键或唯一索引，测试SQL语句如下，当插入本条记录时，MySQL数据库会首先检索已有数据（idx_username索引），如果存在，则执行update更新操作，如果不存在，则直接插入：

```sql
insert ignore into user(username,sex,address) values ('Tom','male','New Yerk') on duplicate key update sex='male',address='New Yerk';
```

### 03 replace into

即插入数据时，如果数据存在，则删除再插入，前提条件同上，插入的数据字段需要设置主键或唯一索引，测试SQL语句如下，当插入本条记录时，MySQL数据库会首先检索已有数据（idx_username索引），如果存在，则先删除旧数据，然后再插入，如果不存在，则直接插入：

```sql
replace into user(username,sex,address) values ('Tom','male','New Yerk');
```

### 04 insert if not exists

即 insert into … select … where not exist ... ，这种方式适合于插入的数据字段没有设置主键或唯一索引，当插入一条数据时，首先判断MySQL数据库中是否存在这条数据，如果不存在，则正常插入，如果存在，则忽略：

```sql
insert ignore into user(username,sex,address) values ('Tom','male','China') where not exists (select username from user where username='Tom');
```

### 例子

看看哪些数据重复了

```sql
SELECT name,count( 1 ) 
FROM
 student 
GROUP BY
NAME 
HAVING
 count( 1 ) > 1;
```

输出：

> name count(1) cat 2 dog 2

name为cat和dog的数据重复了，每个重复的数据有两条；

```sql
Select * From 表 Where 重复字段 In (Select 重复字段 From 表 Group By 重复字段 Having Count(1)>1)
```

删除全部重复数据，一条不留

直接删除会报错

```sql
DELETE 
FROM
 student 
WHERE
 NAME IN (
 SELECT NAME 
 FROM
  student 
 GROUP BY
 NAME 
HAVING
 count( 1 ) > 1)
```

报错：

> 1093 - You can't specify target table 'student' for update in FROM clause, Time: 0.016000s

原因是：更新这个表的同时又查询了这个表，查询这个表的同时又去更新了这个表，可以理解为死锁。mysql不支持这种更新查询同一张表的操作

解决办法：把要更新的几列数据查询出来做为一个第三方表，然后筛选更新。

```sql
DELETE 
FROM
 student 
WHERE
 NAME IN (
 SELECT
  t.NAME 
FROM
 ( SELECT NAME FROM student GROUP BY NAME HAVING count( 1 ) > 1 ) t)
```

删除表中删除重复数据，仅保留一条

在删除之前，查一下要删除的重复数据是啥样的

```sql
SELECT
 * 
FROM
 student 
WHERE
 id NOT IN (
 SELECT
  t.id 
 FROM
 ( SELECT MIN( id ) AS id FROM student GROUP BY `name` ) t 
 )
```

开始删除重复数据，仅留一条

很简单，刚才的select换成delete即可

```sql
DELETE 
FROM
 student 
WHERE
 id NOT IN (
 SELECT
  t.id 
 FROM
 ( SELECT MIN( id ) AS id FROM student GROUP BY `name` ) t 
 )
```

## 交并对称差补

1）表A和表B的交集：

```sql
SELECT a.cus_id from `表a` as a
INNER JOIN `表b` as b
on a.cus_id=b.cus_id；
```

（2）表A和表B的并集：

```sql
SELECT * from `表a`
UNION
SELECT * from `表b`；
```

（3）表A和表B的对称差：

```sql
SELECT * from `表a` 
where cus_id not in (SELECT * from `表b`)
UNION
SELECT * from `表b` 
where cus_id not in (SELECT * from `表a`)；
```

（4）表A中存在但表B中不存在：

```sql
SELECT * from `表a`
WHERE cus_id not in (SELECT cus_id from `表b`)；
```

## count(*) 和 count(1)和count(列名)区别*

执行效果上：

count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略为NULL的值。

count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略为NULL的值。

count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是指空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计。

执行效率上：

列名为主键，count(列名)会比count(1)快

列名不为主键，count(1)会比count(列名)快

如果表多个列并且没有主键，则 count(1 的执行效率优于 count（*）

如果有主键，则 select count（主键）的执行效率是最优的

如果表只有一个字段，则 select count（*）最优。

## 定位数据库消耗 CPU 最高的 SQL 语句

### **定位线程**

```sql
pidstat -t -p <mysqld_pid> 1  5  
```

### **定位问题sql**

```sql
select * from performance_schema.threads where thread_os_id = xx ;  
select * from information_schema.`PROCESSLIST` where  id=threads.processlist_id  
```

### **查看问题sql执行计划**

explain+sql语句

## **SQL真题**

### **第一题**

- order订单表，字段为：goods_id， amount ；

- pv 浏览表，字段为：goods_id，uid；

- goods按照总销售金额排序，分成top10,top10~top20,其他三组

**求每组商品的浏览用户数（同组内同一用户只能算一次）**

```sql
create table if not exists test.nil_goods_category as 
select goods_id
,case when nn<= 10 then 'top10'
when nn<= 20 then 'top10~top20'
else 'other' end as goods_group
from
(
select goods_id
,row_number() over(partition by goods_id order by sale_sum desc) as nn
from
    (
select goods_id,sum(amount) as sale_sum
from order 
group by 1
    ) aa
) bb;
select b.goods_group,count(distinct a.uid) as num
from pv a
left join test.nil_goods_category b
on a.goods_id = b.goods_id
group by 1;
```

### **第二题**

商品活动表 goods_event，g_id（有可能重复），t1（开始时间）,t2（结束时间）

给定时间段（t3,t4)，**求在时间段内做活动的商品数**

```sql
select count(distinct g_id) as event_goods_num
from goods_event
where (t1<=t4 and t1>=t3)
or (t2>=t3 and t2<=t4)
-- 方法二
select count(distinct g_id) as event_goods_num
from goods_event
where (t1<=t4 and t1>=t3)
union all
```

### **第三题**

商品活动流水表，表名为event，字段：goods_id， time；

**求参加活动次数最多的商品的最近一次参加活动的时间**

```sql
select a.goods_id,a.time
from event a
inner join
(
select goods_id,count(*)
from event
group by gooods_id
order by count(*) desc
limit 1
) b
on a.goods_id = b.goods_id
order by a.goods_id,a.time desc
```

### 第四题

订单表，字段有订单编号和时间；

**取每月最后一天的最后三笔订单**

```sql
select *
from 
(
select *
  ,rank() over(partition by mm order by dd desc) as nn1
  ,row_number() over(partition by mm,dd order by inserttime desc) as nn2
from
  (
select 
cast(right(to_date(inserttime),2) as int) as dd,
month(inserttime) as mm,userid,inserttime
from koo.nil_temp0222
) aa
) bb
where nn1 = 1 and nn2<=3;
```

### **第五题**

数据库表Tourists，记录了某个景点7月份每天来访游客的数量如下：

userid dt num 1 2017-07-01 100 …… 非常巧，id字段刚好等于日期里面的几号。

**现在请筛选出连续三天都有大于100天的日期。**

上面例子的输出为：date 2017-07-01 ……

```sql
select a.*,b.num as num2,c.num as num3
from table  a
left join table b
on a.userid = b.userid
and a.dt = date_add(b.dt,-1)
left join table c
on a.userid = c.userid
and a.dt = date_add(c.dt,-2)
where b.num>100
and a.num>100
and c.num>100
```

### **第六题**

现有A表，有21个列，第一列id，剩余列为特征字段，列名从d1-d20，共10W条数据！

另外一个表B称为模式表，和A表结构一样，共5W条数据

请找到A表中的特征符合B表中模式的数据，并记录下相对应的id

有两种情况满足要求：

- 每个特征列都完全匹配的情况下

- 最多有一个特征列不匹配，其他19个特征列都完全匹配，但哪个列不匹配未知

```sql
1.
select aa.*
from 
(
select *,concat(d1,d2,d3……d20) as mmd
from table
) aa
left join 
(
select id,concat(d1,d2,d3……d20) as mmd
from table
) bb
on aa.id = bb.id
and aa.mmd = bb.mmd
2.
select a.*,sum(d1_jp,d2_jp……,d20_jp) as same_judge
from 
(
select a.*
  ,case when a.d1 = b.d1 then 1 else 0 end as d1_jp
  ,case when a.d2 = b.d2 then 1 else 0 end as d2_jp
  ,case when a.d3 = b.d3 then 1 else 0 end as d3_jp
  ,case when a.d4 = b.d4 then 1 else 0 end as d4_jp
  ,case when a.d5 = b.d5 then 1 else 0 end as d5_jp
  ,case when a.d6 = b.d6 then 1 else 0 end as d6_jp
  ,case when a.d7 = b.d7 then 1 else 0 end as d7_jp
  ,case when a.d8 = b.d8 then 1 else 0 end as d8_jp
  ,case when a.d9 = b.d9 then 1 else 0 end as d9_jp
  ,case when a.d10 = b.d10 then 1 else 0 end as d10_jp
  ,case when a.d20 = b.d20 then 1 else 0 end as d20_jp
  ,case when a.d11 = b.d11 then 1 else 0 end as d11_jp
  ,case when a.d12 = b.d12 then 1 else 0 end as d12_jp
  ,case when a.d13 = b.d13 then 1 else 0 end as d13_jp
  ,case when a.d14 = b.d14 then 1 else 0 end as d14_jp
  ,case when a.d15 = b.d15 then 1 else 0 end as d15_jp
  ,case when a.d16 = b.d16 then 1 else 0 end as d16_jp
  ,case when a.d17 = b.d17 then 1 else 0 end as d17_jp
  ,case when a.d18 = b.d18 then 1 else 0 end as d18_jp
  ,case when a.d19 = b.d19 then 1 else 0 end as d19_jp
from table a
left join table b
on a.id = b.id
) aa
where sum(d1_jp,d2_jp……,d20_jp) = 19
```

**第八题**

我们把用户对商品的评分用稀疏向量表示，保存在数据库表t里面：

- t的字段有：uid，goods_id，star。uid是用户id

- goodsid是商品id

- star是用户对该商品的评分，值为1-5

现在我们想要计算向量两两之间的内积，内积在这里的语义为：

对于两个不同的用户，如果他们都对同样的一批商品打了分，那么对于这里面的每个人的分数乘起来，并对这些乘积求和。

例子，数据库表里有以下的数据：

U0 g0 2
U0 g1 4
U1 g0 3
U1 g1 1

计算后的结果为：

U0 U1 2*3+4*1=10 ……

```sql
select aa.uid1,aa.uid2
,sum(star_multi) as result
from 
(
select a.uid as uid1
  ,b.uid as uid2
  ,a.goods_id
  ,a.star * b.star as star_multi
from t a
left join t b
on a.goods_id = b.goods_id
and a.uid<>b.uid
) aa
group by 1,2
```

### 第七题

给出一堆数和频数的表格 table(id,num)，统计这一堆数**中位数**

```sql
select a.*
,b.s_mid_n
,c.l_mid_n
,avg(b.s_mid_n,c.l_mid_n)
from 
(
select 
case when mod(count(*),2) = 0 then count(*)/2 else (count(*)+1)/2 end as s_mid
  ,case when mod(count(*),2) = 0 then count(*)/2+1 else (count(*)+1)/2 end as l_mid
from table 
) a
left join 
(
select id,num,row_number() over(partition by id order by num asc) nn
from table
) b
on a.s_mid = b.nn
left join 
(
select id,num,row_number() over(partition by id order by num asc) nn
from table
) c
on a.l_mid = c.nn
```

### 第八题

表koo.nil_temp0222有三个字段，店铺credit_level，订单时间inserttime，订单金额num

**查询2019-12月内每周都有销量的店铺**

```sql
select distinct credit_level
from 
(
select credit_level,count(distinct nn) as number
from
  (
select userid,credit_level,inserttime,month(inserttime) as mm
    ,weekofyear(inserttime) as week
    ,dense_rank() over(partition by credit_level,month(inserttime) order by weekofyear(inserttime) asc) as nn
from koo.nil_temp0222
where substring(inserttime,1,7) = '2019-12'
order by credit_level ,inserttime
  ) aa
group by 1  
) bb
where number = (select count(distinct weekofyear(inserttime))
from koo.nil_temp0222
where substring(inserttime,1,7) = '2019-12')
```



