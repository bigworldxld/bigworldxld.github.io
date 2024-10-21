---
title: Mysql真题
tags: SQL
categories: database
img: /images/mysql.jpg
abbrlink: 21735
date: 2022-06-12 21:14:59
---

# **MySQL面试真题**

## 统计薪水最高员工

货拉拉EMPLOYEE表里包含着所有货拉拉的员工信息，每个员工有自己的工号、姓名、薪水、和部门ID
![Image Name](https://cdn.kesci.com/upload/image/rau1u8vc8e.png)
货拉拉DEPARTMENT表里包含着货拉拉所有的部门
![Image Name](https://cdn.kesci.com/upload/image/rau1tq4b1h.png)
现在需要统计出货拉拉每个部门薪水最高的员工（包含department/name/salary字段）

```mysql
select
	d.department,
	a.name,
	a.salary
from
	(select
	*,
	max(salary) over (partition by departmentID) as "部门薪水最高"
	from
		employee) a
	join department d
		on a.departmentID=d.id
where a.salary=a.部门薪水最高
```

## 统计每个顾客每天总登录时长

某游戏数据后台设有“登录日志”和“登出日志”两张表。
游戏开服前两天（2022-08-13至2022-08-14）的角色登录和登出日志如下：

“登录日志”记录各玩家的登录时间和登录时的角色等级。
![Image Name](https://cdn.kesci.com/upload/image/rbnrhckcty.png)

“登出日志”记录各玩家的登出时间和登出时的角色等级。
![Image Name](https://cdn.kesci.com/upload/image/rbnrhuhccj.png)

其中，“角色id”字段唯一识别玩家。

一天中，玩家可以多次登录登出游戏，请使用SQL分析出以下业务问题：

分析开服首日（2022-08-13），游戏的DAU（日活跃玩家数）和次日留存率（次日仍登录的活跃玩家数/当日活跃玩家总数）

```mysql
select count(*) as DAU,
	sum(case when t1.首日玩家=t2.次日玩家 then 1 else 0 end)/count(*) as "次日留存率"
from
(select distinct d.角色id as '首日玩家'
from 登录日志 d
where d.日期='2022-08-13') t1
left join
(select distinct d.角色id as '次日玩家'
form 登录日志 d
where d.日期='2022-08-14') t2
on t1.首日玩家=t2.次日玩家
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rboavi6whr.png)

case搜索函数只返回第一个符合条件的值，剩下的case部分将会被自动忽略。

玩家在开服首日（2022-08-13）等级分布情况，即每个等级停留的角色数。（如玩家没有登出日志，则使用登录日志的等级信息。）

```mysql
select
	角色等级,
	count(角色id) as '角色数'
from
	(select
		*,
		row_number() over (partition by 角色id
	order by 
		c.登出时间 desc) as time_rank
	from
		登出日志 c
	where
		c.日期='2022-08-13'
		) t1

where
	time_rank=1
group by
	角色等级
order by 
	角色等级;
```

**ROW_NUMBER() OVER (ORDER BY xlh DESC)** 

简单的说row_number()从1开始，为每一条分组记录返回一个数字，这里的ROW_NUMBER() OVER (ORDER BY xlh DESC) 是先把xlh列降序，再为降序以后的没条xlh记录返回一个序号。

**row_number() OVER (PARTITION BY COL1 ORDER BY COL2)** 

 表示根据COL1分组，在分组内部根据 COL2排序，而此函数计算的值就表示每组内部排序后的顺序编号（组内连续的唯一的)

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rbobbl2lnh.png)

请根据玩家登录登出的时间，统计各玩家每天总在线时长情况（如玩家登录后没有对应的登出日志，可以使用当天23:59:59作为登出时间）

IFNULL(expr1,expr2)用法：

假如expr1不为NULL，则 IFNULL() 的返回值为expr1; 否则其返回值为 expr2

NULLIF(expr1,expr2)用法

  如果expr1 = expr2 成立，那么返回值为NULL，否则返回值为expr1。这和CASE  WHEN expr1 = expr2 THEN NULL ELSE  expr1 END相同。  

timestampdiff（unit，begin，end）

​    begin和end可以为DATE或[DATETIME](https://so.csdn.net/so/search?q=DATETIME&spm=1001.2101.3001.7020)类型，并且可允许参数为混合类型.

```mysql
with t1 as(
select *,row_number() over (partition by d.日期，d.角色id order by 登陆时间) as "log_rank"
from 登录日志 d
),
t2 as(
select *,row_number()over(partition by c.日期 ,c.角色id order by c.登出时间) as "out_rank"
from 登出日志 c
),
t3 as(
select t1.日期,t1.角色id,t1.登陆时间，ifnull(t2.登出时间,concat('t1.日期',' 23:59:59')) as "登出时间"
from t1 left join t2
on t1.log_rank=t2.out_rank
and t1.角色id=t2.角色id
and t1.日期=t2.日期
)，
t4 as (
select 日期,角色id,timestampdiff(second,登录时间,登出时间)/60 as "每次在线时长_min"
from t3
)
select 日期,角色id,round(sum(每次在线时长_min),2) as "各玩家每天总在线时长_min"
from t4
group by 日期,角色id
order by 日期,角色id;
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rbnrkkdl9i.png)

请根据玩家登录登出的时间，统计在开服首日各玩家在线时长分布。
如玩家登录后没有对应的登出日志，可以使用当天23:59:59作为登出时间。
区分在线时间段：0-30min，30min-1h，1-2h，2-3h，3-5h，5h以上；区间为左闭右开。

```sql
with t1 as(
select *,row_number() over (partition by d.日期,d.角色id order by d.登录时间) as "log_rank"
from 登录日志 d 
),
t2 as (
select *,row_number() over (partition by c.日期 ,c.角色id order by c.登出时间) as "out_rank"
from 登出日志 c
),
t3 as (select t1.日期,t1.角色id,t1.登录时间,ifnull(t2.登出时间,concat('t1.日期','23:59:59')) as "登出时间"
from t1 left join t2
on t1.log_rank=t2.out_rank
and t1.角色id=t2.角色id
and t1.日期=t2.日期            
)，
t4 as (
select 日期，角色id,timestampdiff(second,登陆时间，登出时间)/60 as "每次在线时长_min"
from t3    
),
t5 as (
select 角色id,round(sum(每次在线时长_min),2) as "各玩家首日总在线时长_min"
from t4
where by 角色id
order by 角色id    
)
t6 as(
case when 各玩玩家首日总在线时长_min>=0 and 各玩家首日总在线时长_min<30 then '0-30min'
when 各玩家首日总在线时长_min >=30 and 各玩家首日总在线时长_min <60 then '30min-1h'
when 各玩家首日总在线时长_min >=60 and 各玩家首日总在线时长_min <120 then '1-2h'
when 各玩家首日总在线时长_min >=120 and 各玩家首日总在线时长_min <180 then '2-3h'
when 各玩家首日总在线时长_min >=180 and 各玩家首日总在线时长_min <300 then '3-5h'
else '5h以上' end
)
select
t6 as "时间分布"，
count(角色id) as 玩家人数
from t5
group by t6
order by field(时间分布,'0-30min','30min-1h','1-2h','2-3h','3-5h')
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rbrvfvb92n.png)

## 查询入职以来的薪水涨幅

“雇员表“中记录了员工的信息。
![Image Name](https://cdn.kesci.com/upload/image/raku50uk0j.png)
“员工薪水表“中记录了对应员工发放的薪水。两表通过“雇员编号”关联。
![Image Name](https://cdn.kesci.com/upload/image/raku5abi2v.png)

查找当前所有雇员入职以来的薪水涨幅，给出雇员编号以及其对应的薪水涨幅，并按照薪水涨幅进行升序。（注：薪水表中结束日期为2004-01-01的才是当前员工，否则是已离职员工）

```sql
with t1 as(
select 雇员编号,
薪水 AS 当前工资
FROM 员工薪水表
WHERE 结束日期='2004-01-01'
),
t2 as(
SELECT 雇员编号,薪水 AS 入职工资
FROM 员工薪水表
JOIN 雇员表 
USING(雇员编号)
WHERE 起始日期=雇用日期
)
select 雇员编号,当前工资-入职工资 as 薪水涨幅
from t1
left join t2
useing(雇员编号)
order by 薪水涨幅 desc
```

WITH AS短语，也叫做子查询部分（subquery factoring），是用来定义一个SQL片断，该SQL片断会被整个SQL语句所用到。这个语句算是公用表表达式（CTE）

有一张“课程销售订单表”，包含4个字段：用户id、下单日期、下单id、学科。

![Image Name](https://cdn.kesci.com/upload/image/rakehufcb2.png)
**问题：查询每个用户第一个订单的记录，如果同时下单了包含多个课程的订单，则按照“语文、数学、英语”顺序排序。**

```sql
with t1 as(
select *,
dense_rank() over (partition by 用户id order by 下单日期) as rank
from 课程销售订单表
)
selet *
from t1
where rank=1
order by 用户id,field(学科，'语文','数学','英语')
```

FIELD()函数返回的索引（从1开始的位置）的str在str1，str2，STR3，...列表中。如果str没有找到，则返回0。就是用第一个参数str,跟后面的N个字符串参数中寻找，如果寻找到一模一样的字符串，则返回其索引位置。

## 快递场景

有一张“快递揽收表”，包含3列：运单号、客户id、创建日期。
![Image Name](https://cdn.kesci.com/upload/image/raji6h3q70.png)
问题：查询运单创建日期在0501-0531期间不同单量区间的客户分布。最终得出的数据如下：
![Image Name](https://cdn.kesci.com/upload/image/rakeqpjeuy.png)

```sql
with t1 as(
SELECT 
	客户id,
    COUNT(DISTINCT 运单号) AS 运单数量 
FROM
	快递揽收表 
    WHERE DATE_FORMAT(创建日期, '%Y/%m/%d') BETWEEN '2020/05/01' 
      AND '2020/05/31' 
    GROUP BY 客户id
),
t2 as(
SELECT 
    *,
    CASE
      WHEN 运单数量 BETWEEN 0 
      AND 5 
      THEN '0-5' 
      WHEN 运单数量 BETWEEN 6 
      AND 10 
      THEN '6-10' 
      WHEN 运单数量 BETWEEN 11 
      AND 20 
      THEN '11-20' 
      ELSE '20以上' 
    END AS 单量 
  FROM t1  
),
SELECT 
  单量,
  COUNT(DISTINCT 客户id) AS 客户数 
FROM t2
GROUP BY 单量
ORDER BY field(单量,'0-5','6-10','11-20','20以上')
```

## 找到特殊的电话号码

有一张“电话费用表”，包含3个字段：电话号码（8位数）、月份、月消费。
![Image Name](https://cdn.kesci.com/upload/image/raj505ndk5.png)
其中，月消费为0表明该月没有产生费用。第一行数据含义：电话号码（64262631）在月份（2017年11月）产生的月消费（30.6元的话费）。

查找2017年截止到10月31日所有四位尾数符合AABB或者ABAB或者AAAA的电话号

SQL MID() 函数用于得到一个字符串的一部分。这个函数被MySQL支持，但不被MS SQL Server和Oracle支持。在SQL Server， Oracle 数据库中，我们可以使用 SQL SUBSTRING函数或者 SQL SUBSTR函数作为替代。

MID() 函数语法为：

SELECT MID(ColumnName, Start [, Length])

FROM TableName

注：字符串从1开始，而非0，Length是可选项，如果没有提供，MID()函数将返回余下的字符串。

```sql
select distinct 电话号码
from 
(select *,
mid(电话号码，5，1) as 第五位数，
mid(电话号码，6，1) as 第六位数，
mid(电话号码，7，1) as 第七位数，
right(电话号码，1) as 第八位数，
from 电话费用表
where 月份<201711) a
where 第五位数=第六位数 and 第七位数=第八位数 and 第五位数<>第七位数
or 第五位数<>第六位数 and 第五位数=第七位数 and 第六位数=第八位数
or 第五位数=第六位数 and 第五位数=第七位数 and 第五位数=第八位数
)
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/raj52es5ee.png)

查找除去10月份重复数据的结果。

```sql
select *
from 电话费用表
where 月份<>201710
select distinct *
from 电话费用表
where 月份=201710
order by 月份
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/raj54ruqsi.png)

查找2017年6、7、8月都有话费，但是9、10月没有产生话费，并且6、7、8月话费均在51-100元之间的电话号码。

```sql
with t1 as(
select distinct 电话号码
from 
    电话费用表
where 月份 in ('201709'，'201710')
group by 电话号码
having max(月消费)<>0
),
t2 as(
select 电话号码
from 
    电话费用表  
where 月份 in ('201706','201707','201708')
group by 电话号码
having count(distinct 月份)=3
    and min(月消费)>=51
    and max(月消费)<=100
)
select *
from t2
where 电话号码 not in t1
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rajff2elpo.png)

## 分析学员续费

某线上学习平台设置学员线上学习阶梯，新学员购买50节课为一个学习阶段，学习完想要进入下个阶段必须再次购买，即续费（假设所有学员只能续费一次）并且每个学员可选择不同老师进行学习。

表一：学员上课表
![Image Name](https://cdn.kesci.com/upload/image/ragqs5o6ce.png)
表二：购买表
![Image Name](https://cdn.kesci.com/upload/image/ragqsqf34p.png)

**问题：
1.现求出续费学员在续费前3个月内的总课量，3个月给学员上课老师数量，以及每个上课老师给学员的上课量。**

DATE_SUB() 函数从日期减去指定的时间间隔。

DATE_SUB(date,INTERVAL expr type)
date 参数是合法的日期表达式。expr 参数是您希望添加的时间间隔。

INTERVAL -31 DAY  表示未来31天

INTERVAL 31 DAY  表示过去的31天

```sql
#第一问前两解答：
select count(*) as 续费学员的总课量,
count(distinct 老师id) as 上课老师数量，
from
(select 续费时间,学员id
from 购买表
where 订单类型=2) a
join 学员上课表 b
on a.学员id=b.学员id
where 上课时间>date_sub(续费时间,interval 3 montch)
and 上课时间<续费时间
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ragqvns889.png)

```sql
#第一问最后一小问解答：
select 
count(*) as 每个上课老师给学员的上课量,
老师id，
FROM 
(SELECT 续费时间,学员id
FROM 购买表
WHERE 订单类型=2) a
JOIN 学员上课表 b
ON a.学员id=b.学员id
WHERE 上课时间 >DATE_SUB(续费时间,INTERVAL 3 MONTH) AND 上课时间<续费时间
GROUP BY 老师id
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ragqy4qaaw.png)

2.现求出每个续费学员在续费前的最后一节课的时间，以及对应的学员。

```sql
#第二问
SELECT a.学员id,MAX(上课时间) AS 续费前最后一节课的时间
FROM 
(SELECT 续费时间,学员id
FROM 购买表
WHERE 订单类型=2) a
JOIN 学员上课表 b
ON a.学员id=b.学员id
WHERE 上课时间<续费时间
GROUP BY a.学员id
```

结果如下：-
![Image Name](https://cdn.kesci.com/upload/image/ragr0logzr.png?imageView2/0/w/960/h/960)

## 各岗位编号中位数所在位置

学校每次考试完，都会有一个考试成绩表。例如，表中第1行表示编号为1的用户选择了C++岗位，该科目考了11001分。
![Image Name](https://cdn.kesci.com/upload/image/ragnj11je1.png)
问题：写一个sql语句查询每个岗位编号的中位数位置的范围，并且按岗位升序排序。

```sql
select 岗位，
floor((成绩个数+1)/2) as 起始位置，
ceil((成绩个数+1)/2) as 结束位置
from
(select 岗位,count(*) as 成绩个数
 from 考试成绩表
 group by 岗位) a
 order by 岗位 asc
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ragnpxfoj1.png)

## 银行数据分析实战

某理财银行有下面3个表。
交易表记录了每天交易的客户交易时间、客户号、消费类型和消费金额。其中，交易类型有两种值：消费和转账。
![Image Name](https://cdn.kesci.com/upload/image/rafpubcf42.png)
客户表记录了客户信息，包括客户号，客户名称和客户所属的银行分行号。
![Image Name](https://cdn.kesci.com/upload/image/rafpuw55a6.png)
银行分行记录表记录每个分行的信息，包括分行号、分行名称及对应上级分行。
![Image Name](https://cdn.kesci.com/upload/image/rafpw0cgss.png)

该理财银行要求对客户及销售额分析报告，要求如下：
1.计算2016年1-3月的消费总金额，生成如下格式的查询结果：
![Image Name](https://cdn.kesci.com/upload/image/rafpxkvt99.png)

```sql
with t1 as(
select date_format(交易时间,'%Y年%m月') as 年月,
    sum(交易金额) as 消费总金额
from 交易表
where 交易时间 between '2016-01-01' and '2016-03-31'
group by date_format(交易时间,'%Y年%m月')
)
select max(case when 年月='2016年01月' then 消费金额 end) as 2016年01月，
max(case when 年月='2016年02月' then 消费金额 end) as 2016年02月，
max(case when 年月='2016年03月' then 消费金额 end) as 2016年03月
from t1
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rafq1hexzf.png)

2.提取2016年3月消费金额大于等于1288的客户名单，并给出这些客户信息：
![Image Name](https://cdn.kesci.com/upload/image/rafpyxsw5v.png)

```sql
with t1 as(
select 客户名称，
sum(交易金额) as 2016年3月总消费金额
from 交易表 a
join 客户表 b
on a.交易客户=b.客户号
where 交易时间 between '2016-01-01' and '2016-03-31'
group by 客户名称
having sum(交易金额)>=1288
),
t2 as(
select *,
sum(交易金额) over (partition by 客户名称 order by 交易时间) as 累积消费
from 交易表 a
join 客户表 b
on a.交易客户=b.客户号
where 交易时间 between '2016-01-01' and '2016-03-31'
where 累计消费>1288
),
t3 as(
select 客户名称，交易时间
from (
select *,
row_number() over(PARTITION BY 客户名称 ORDER BY 交易时间) AS 排序
from t2
where 排序=1
)
)
select t1.*,t3.交易时间 as 2016年3月消费金额首次达到1288的时间
from t1 
left join t3
using(客户名称)
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rafq587qnh.png)

3.汇总各省分行（省分行下属支行也需要汇总至省分行）的2016年3月的总消费金额。

mysql语句中like用法：

1、常见用法：

(1)搭配%使用

%代表一个或多个字符的通配符，譬如查询字段name中以大开头的数据：

(2)搭配_使用

_代表仅仅一个字符的通配符，把上面那条查询语句中的%改为_，会发现只能查询出一条数据。

(3)[ ] 指定范围 ([a-f]) 或集合 ([abcdef]) 中的任何单个字符：

1，like'[CK]ars[eo]n' 将搜索下列字符串：Carsen、Karsen、Carson 和 Karson(如 Carson)。

2、like'[M-Z]inger' 将搜索以字符串 inger 结尾、以从 M 到 Z 的任何单个字母开头的所有名称(如 Ringer)。

 using()用于两张表的join查询，，要求using()指定的列在两个表中均存在，并使用之用于join的条件。 示例： select a.*, b.* from a left join b using(colA); 等同于： select a.*, b.* from a left join b on a.colA = b.colA;

using() 只是join中指定连接条件的简写，在简单的连接中常用。在列名称不同时或连接条件复杂时就无法用了

```sql
with t1 as(
select 分行号,分行名称，上级分行，sum(交易金额) as 分行消费总金额
    from 交易表 a
    join 客户表 b
    on a.交易客户=b.客户号
    join 银行分行对应表 c
    on b.所属分行=c.分行号
    where 交易时间 between '2016-03-01' and '2016-03-31' and 交易类型='消费'
    group by 分行号，分行名称，上级分行
)，
t2 as(
select a.分行号 as 省分行号,b.分行号 as 分行号,c.分行号 as 分行号1
    from 银行分行对应表 a,银行分行对应表 b,银行分行对应表 c
    where a.分行号=b.上级分行 and b.分行号=c.上级分行 and a.分行名称 like '%省分行'
)，
t3 as (
select 省分行号，分行号
    from t2
    union all
    select 省分行号，分行号
    from t2
    uinon all
    select 省分行号，分行号
    from t2
    order by 省分行号 
)，
t4 as (
select 省分行号,sum(分行消费总金额) as 各省分行2016年3月的总消费金额
    from t1
    left join t3
    using (分行号)
    group by 省分行号
)，
t5 as (
selct a.分行名称,各省分行2016年3月的总消费金额
    from t4
    left join 银行分行对应表 a
    on t4.省分行号=a.分行号
)
select * from t5
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rafq9l8tie.png)

## 交易记录分析实战

某商场为了分析用户购买渠道。表1是用户交易记录表，记录了用户id、交易日期、交易类型和交易金额。
![Image Name](https://cdn.kesci.com/upload/image/raf9rdxcdh.png)
表2是用户类型表，记录了用户支付类型（比如微信、支付宝、信用卡等），分别有type1、type2。
![Image Name](https://cdn.kesci.com/upload/image/raf9rqrjp6.png)

问题：
1.请在 type1的用户类型中，找出总交易金额最大的用户。

```sql
select 用户类型,用户id,sum(交易金额) as 总易金额
from 用户交易记录表
join 用户类型表  using (用户id)
where 用户类型='type1'
group by 用户id
having sum(交易金额) >=all
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/raf9ww5jpq.png)

2.筛选每个用户的第2笔交易记录。

```sql
select *
from (
select *,row_number() over(PARTITION BY 用户id order by 交易日期) as 第几笔交易记录
from 用户交易记录表    
) a
where 第几笔交易记录=2
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/raf9ywct9s.png)

3.如何实现如下表的数据格式？**
![Image Name](https://cdn.kesci.com/upload/image/raf9utarma.png)

将group by产生的同一个分组中的值连接起来，返回一个[字符串](https://so.csdn.net/so/search?q=字符串&spm=1001.2101.3001.7020)结果。

group_concat([distinct] 字段名 [order by 排序字段 asc/desc] [separator '分隔符'])

说明：

(1)使用distinct可以排除重复值；

(2)如果需要对结果中的值进行排序，可以使用order by子句；

(3)separator是一个字符串值，默认为逗号。

```sql
select id,group_concat(交易日期) as 交易日期,group_concat(交易类型) as 交易类型
from 用户交易记录表
group by 用户id
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/rafa151k40.png)

## 房产订单分析

“成交订单表”里记录了某房产平台（类似链家、贝壳等）每日房屋成交的明细。（贝壳面试题）
![Image Name](https://cdn.kesci.com/upload/image/raeysslash.png)
字段“成交客源渠道”中的值是“客源角色人”、“业主线上委托”、“null”表示线下渠道，其余的成交客源渠道是线上。
要求：
1．当月截止昨天二手线上成交单量占比（含车位）>=50%的门店可获奖；

```sql
with t1 as ( -- 求出是否线上
select *,case when 成交客源渠道 in('客源角色人','业主线上委托') or 成交客源渠道 is null then '否' else '是' end as 是否线上
    from 成交订单表
)，
t2 as( -- 求出各门店的线上成交占比
    select 签约经纪人门店，sum(case when 是否线上='是' then 1 else 0 end)/count(*) as 经纪人门店的线上成交占比
from t1
group by 签约经纪人门店
),
```

（线上成交占比=线上成交单量/总成交单量）
2．符合获奖条件的门店的第1单线上成交可获得200贝壳币（可以用于兑换奖金），第2单可获400贝壳币，第3单及以上可获800贝壳币，但车库不奖励（字段“房屋用途”中的值是”车位”、”车库”认为是车库）；

```sql
t3 as( -- 得出原表加上前面所求2个字段,并添加一个线上订单排序
select t1.*,t2.经纪人门店的线上成交占比,row_number() over(partition by 签约经纪人门店 order by 签约时间) as  订单顺序
    from t1
    join t2
    on t1.签约经纪人门店=t2.签约经纪人门店
    where 是否线上='是'
    and 经纪人门店的线上成交占比>=0.5
    and 房屋用途 is null
),
t4 as(
select *,case when 订单顺序=1 then '200' when 订单顺序=2 then '400' when 订单顺序>=3 then '800' else 0 end as 订单可获贝壳币
    from t3
)
```

3.在一个连续的SQL中实现以上需求，不能拆分成多个SQL，必须输出表格字段如下（可增加）；**
![Image Name](https://cdn.kesci.com/upload/image/raez0d965r.png)

```sql
select t1.*,t2.经纪人门店的线上成交占比,ifnull(t4.订单可获贝壳币,0) as 订单可获贝壳币
from t1
left join t2
on 签约经纪人门店=t2.签约经纪人门店
left join t4
on t1.协议id=t4.协议id
order by 签约经纪人门店,签约时间
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/raeyxaq041.png)

## 电影

某电影平台（类似豆瓣、猫眼电影）用3个表来记录电影信息。
“电影表”中是电影编号、电影名称、电影描述信息。
![Image Name](https://cdn.kesci.com/upload/image/raev4iwaud.png)
“类别表”是电影分类信息，类别包括：犯罪电影、爱情电影、科幻电影。
![Image Name](https://cdn.kesci.com/upload/image/raev51o9q9.png)
“电影类别表”是对应电影（电影表中的电影编号）属于哪一类（类别表中的电影类别编号）
![Image Name](https://cdn.kesci.com/upload/image/raev5jaoll.png)
**查找“电影表”电影描述信息包含“机器人”的电影，求出对应的电影类别名称和电影数目,还需该电影类别名称对应电影数量>=5部。**

```sql
select 电影类别名称，count(*) as 电影数目
from 电影表
left join 电影类别表 using(电影编号)
left join 类别表 using(电影类别编号)
where 电影描述信息 like '%机器人%'
group by 电影类别名称
having 电影类别名称 in
(select 电影类别名称
from 电影类别表
join 类别表 using(电影类别编号)
 group by 电影类别名称
 having count(*)>5
)
```

## 通讯运营商指标分析

现有表一：各城市用户ARPU值
![Image Name](https://cdn.kesci.com/upload/image/rad1h8z6eb.png)
表二：用户套餐费用表
![Image Name](https://cdn.kesci.com/upload/image/rad1i0mmxq.png)

业务要求：

1.各城市用户数、总费用（ARPU之和）是多少？

```sql
select 城市，count(distinct 用户id) as 各城市用户数,sum(ARPU) as 总费用
from 各城市用户ARPU值
group by 城市
```

2.各城市用户arpu值表中各城市ARPU(0,30)，[30,50)，[50-80)，[80以上)用户数分别是多少？

```sql
select sum(chase when ARPU值>0 and ARPU值<30 then 1 else 0 end ) as '[0,30)用户数',
sum(chase when ARPU值>=30 and ARPU值<50 then 1 else 0 end ) as '[30,50)用户数',
sum(chase when ARPU值>=50 and ARPU值<80 then 1 else 0 end ) as '[50,80)用户数',
sum(chase when ARPU值>=80 then 1 else 0 end ) as '80以上)用户数',
from 各城市用户ARPU值
group by 城市
```

3.用户套餐费用表中用户有重复的记录，找出重复的用户

```sql
select 用户id
from 用户套餐费用表
group by 用户id
having count(用户id)>1
```

## 打车业务分析

“订单信息表”里记录了巴西乘客使用打车软件的信息，包括订单呼叫、应答、取消、完单时间。（滴滴2020年笔试题）
![Image Name](https://cdn.kesci.com/upload/image/rabcwmga3.png)
字段释义：
订单id；呼叫订单识别号
乘客id：乘客识别号
呼叫时间：乘客从应用发起需要用车请求的时间（北京时间）
应答时间:司机点击接单的时间（北京时间）
取消时间：司机或乘客取消订单的时间（北京时间）
完单时间：司机点击到达目的地的时间（北京时间）

**注意：
（1）表中的时间是北京时间，巴西比中国慢11小时。
（2）应答时间列的数据值如果是“1970”年（巴西时间），表示该订单没有司机应答，属于无效订单。**

语法：date_sub(date,interval expr type)，函数从日期减去指定的时间间隔，

举例：Orders 表中 OrderDate 字段减去 2 天：

select OrderId,date_sub(OrderDate,interval 2 day) as OrderPayDate

from Orders

date_sub('2019-07-27', interval 30 day)表示往前推30天

订单的应答率，完单率分别是多少？

```sql
select count(distinct case when year(date_sub(t1.应答时间,interval 11 hour))<>1970 then t1.订单id end)/count(t1.订单id) as 应答率,
count(distinct case when year(date_sub(t1.完单时间,interval 11 hour))<>1970 then t1.订单id end)/count(t1.订单id) as 完单率
)
from 订单信息表 t1;
```

呼叫应答平均时间有多长？

MySQL

  语法：  timestampdiff（unit，begin，end）
   begin和end可以为DATE或DATETIME类型，并且可允许参数为混合类型。

DB2

  语法：timestampdiff（unit，char（begin-end））

begin和end需要为时间戳格式。若非时间戳类型，用timestamp（）函数转换。

   timestampdiff（unit，char（timestamp（begin）-timestamp（end）））


```sql
select avg(timestampdiff(minute,t1.呼叫时间,t1.应答时间)) as "呼叫应答平均时间(min)"
from 订单信息表 t1
where year(date_sub(t1.应答时间,interval 11 hour))<>1970;
```

从这一周的数据来看，呼叫量最高的是哪一个小时（当地时间）？呼叫量最少的是哪一个小时（当地时间）？

```sql
--呼叫量最高:
with  t1 as(
SELECT 
    COUNT(*) AS 呼叫量 
  FROM
    订单信息表 
  GROUP BY HOUR(
      DATE_SUB(呼叫时间, INTERVAL 11 HOUR) 
)
SELECT 
  HOUR(
    DATE_SUB(呼叫时间, INTERVAL 11 HOUR)
  ) AS 当地小时,
  COUNT(*) AS 呼叫量 
FROM 订单信息表
    GROUP BY HOUR(
    DATE_SUB(呼叫时间, INTERVAL 11 HOUR)
  ) 
HAVING COUNT(*) >= ALL 
 (t1)   
--呼叫量最低:
SELECT 
  HOUR(
    DATE_SUB(呼叫时间, INTERVAL 11 HOUR)
  ) AS 当地小时,
  COUNT(*) AS 呼叫量 
FROM
  订单信息表 
GROUP BY HOUR(
    DATE_SUB(呼叫时间, INTERVAL 11 HOUR)
  ) 
HAVING COUNT(*) <= ALL 
    (t1)
```

最高如下：
![Image Name](https://cdn.kesci.com/upload/image/rabdfesude.png)
最低如下：
![Image Name](https://cdn.kesci.com/upload/image/rabdfsazr3.png)

呼叫订单第二天继续呼叫的比例有多少？

```sql
with t2 as(
select t1.乘客id，day(data_sub(t1.呼叫时间 interval 11 hour)) as day
    from 订单信息表 t1
),
t3(
select *,
    lead(day) over (partiton by 乘客id order by day) as next_day
    from t2
)
select
sum(case when next_day-day=1 then 1 else 0 end)/count(*) as 次日呼叫比例
from
(t3)
```

 1.lag()

  lag(expr,N,default)

expr:它可以是列或任何内置函数。
N:它是一个正值，它确定当前行之前的行数。如果在查询中将其省略，则其默认值为1
default:如果在当前行之前没有行N行的情况下，它是函数返回的默认值。如果缺少，则默认为NULL。

**2.lead（）**

lead(expr,N,default)

expr:它可以是列或任何内置函数。

N:它是一个正值，它确定当前行之后的行数。如果在查询中将其省略，则其默认值为1

default:如果在当前行之后没有行N行的情况下，它是函数返回的默认值。如果缺少，则默认为NULL。

## 常见的分组比较

现在有三个表，“学生表”，“课程表”，“成绩表”。
“学生表”记录了学生的基本信息，有“学号”、“姓名”、“出生日期”、“性别”。
![Image Name](https://cdn.kesci.com/upload/image/ra80zbb9z4.png)
“成绩表”记录了学生选修课程的成绩，包括“学号”，选修的“课程号”以及对应课程的“成绩”。
![Image Name](https://cdn.kesci.com/upload/image/ra80zwxolw.png)
“课程表”记录了学生选修的课程信息，包括课程号、课程名称及其对应的“教师号”
![Image Name](https://cdn.kesci.com/upload/image/ra810x8qst.png)

**现在要查找出每门课程中成绩最好的学生的姓名和该学生的课程及成绩。
需要注意：可能出现并列第一的情况。**

```sql
--解法一：
with t1 as(
select *,dense_rank() over(partiton by 课程号 order by 成绩 decs) as 排名
	from 成绩表
    join 课程表
    using(课程号)
    join 学生表
    using(学号)   
)
select 姓名,课程名称,成绩
from t1
    where 排名=1
    order by 课程名称
--解法二：
select 姓名,课程名称,成绩
from 成绩表
join 课程表
using(课程号)
join 学生表
using(学号)
where (课程号,成绩) in (select 课程号,max(成绩) from 成绩表 group by 课程号)
order by 课程名称
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra815zvesp.png)

## 行列互换

表名：cook
![Image Name](https://cdn.kesci.com/upload/image/ra7xpmgpo7.png)

要求查询结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra7xrgqj0o.png?imageView2/0/w/960/h/960)

```sql
select 年,
max(case where 月=1 then 值 end) as m1,
max(case where 月=2 then 值 end) as m2,
max(case where 月=3 then 值 end) as m3,
max(case where 月=4 then 值 end) as m4,
from cook
group by 年
```

## 球赛分析

两只篮球队进行了激烈的比赛，比分交替上升。比赛结束后，你有一张两队分数的明细表：
![Image Name](https://cdn.kesci.com/upload/image/ra5ziz7z05.png)
该表记录了球队、球员号码、球员姓名、得分分数以及得分时间。现在球队要对比赛中表现突出的球员做出奖励。

问题：
**（1）请你写一个sql语句统计出：比赛中帮助各自球队反超比分的球员姓名以及对应时间**

```sql
with t1 as(
select *,
    case when 球队='A' then 得分分数 else 0 end as A队分数,
    case when 球队='B' then 得分分数 else 0 end as B队分数
    from 分数表
    order by 得分时间
)
t2 as(
select *,
    sum(A队分数) over(order by 得分时间) as A队累计,
    sum(B队分数) over(order by 得分时间) as B队累计,
    from
    t1
)
t3 as(
select *,
    case when A队累计>B队累计 then 'A'
    when A队累计<B队累计 then 'B'
    else '平' end as 结果
from t2
)
t4 as(
select *,
    lag(结果,1) over (order by 得分时间) as 上次结果
    lag(结果,2) over (order by 得分时间) as 前次结果
    from
    t3
)
t5 as(
select *,
    case when 结果<>'平' and 上次结果<>'平' and 结果<>上次结果 then 1
    when 结果<>'平' AND 上次结果='平' AND 结果<>前次结果 THEN 1 else 0 end as 判断
    from
    t4
)
select 球员姓名,得分时间
from 
t5
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra5zs2nqm2.png)

**（2）请你写一个sql语句统计出：连续三次（及以上）为球队得分的球员名单**

```sql
select distinct 球队,球员姓名
from 
(select *,
lead(球员姓名,1) over(partion by 球队 order by 得分时间) as 二次得分球员,
lead(球员姓名,2) over(partion by 球队 order by 得分时间) as 三次得分球员
 from 分数表
) a
where 球员姓名=二次得分球员
and 球员姓名=三次得分球员
```

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra6jh5ddhn.png)

## 选出当日浏览房源10套以上并且注册超过一年的用户

某房源平台有两张表来记录用户信息、和用户查看房源信息。
用户表（用户号、用户注册时间）。
![Image Name](https://cdn.kesci.com/upload/image/ra44ukgy0t.png)
浏览表，字段有日志号，用户号，房源号，浏览日期。
![Image Name](https://cdn.kesci.com/upload/image/ra44uvfyxm.png?imageView2/0/w/960/h/960)

问题：选出当日浏览房源10套以上并且注册超过一年的用户。

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra44xiytmy.png)

```sql
select 
浏览日期,
用户号,
count(房源号) as 房源数
from 浏览表 a
join 用户表 b using(用户号)
where data_sub(浏览日期,interval 1 year) >注册时间
group by 浏览日期，用户号
having count(房源号)>10
```

## 贷款逾期

一家金融贷款公司，需要了解用户贷款逾期未还的情况。该公司数据库中有一张用户"贷款逾期天数"表。
![Image Name](https://cdn.kesci.com/upload/image/ra1zox7pi3.png)

当逾期天数=0时，记为M0,

逾期天数在[1,30]区间时，记为M1,

逾期天数在[31,60]区间时, 记为M2,

逾期天数在[61,90]区间时，记为M3，

其他更高的逾期天数记为M4+，

现在需要在数据库中分析出每种逾期时段（M0、M1、M2、M3、M4+）的订单个数，如果是你，会如何分析呢？

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra1zz6agzk.png)

```sql
select sum(case when 逾期天数=0 then 1 else 0 end) as M0, 
	sum(case where 逾期天数>=1 and 逾期天数<=30 then 1 else 0 end) as M1,
	sum(case where 逾期天数>=31 and 逾期天数<=60 then 1 else 0 end) as M2, 
	sum(case where 逾期天数>=61 and 逾期天数<=90 then 1 else 0 end) as M3,
	sum(case where 逾期天数>=91 then 1 else 0 end) as M4
	from 贷款逾期天数
```

## 视频数据分析实战

"用户操作记录表"里记录着每天某短视频平台的用户点击访问情况，以便帮助公司内部分析师了解用户对于当前页面的点击偏好。

表包字段有：用户名、操作记录、操作时间。
![Image Name](https://cdn.kesci.com/upload/image/ra1x3y3hnz.png)

其中表内各字段含义如下：
用户名——表示用户在该短视频平台注册的唯一用户名。

操作记录——表示用户在该短视频平台点击的按钮名称。A表示用户点击“短视频”播放入口，B表示用户点击“长视频”播放入口。

操作时间——表示用户点击时候的时间，精确到秒。

现在运营人员找到作为数据分析师的你，想让你帮忙看看用SQL取两个数据，具体如下：

1.分析每天的访客数和他们的平均操作次数；
结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra1x8c9ghz.png)

```sql
select date(操作时间) as 日期，
count(distinct 用户名) as 每天访客数，
count(操作记录)/count(distinct 用户名) as 人均操作次数
from 用户操作记录表
group by date(操作时间)
```

2.统计每天符合以下条件的用户数：A操作之后是B操作，AB操作必须相邻；
结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra1xayhxjp.png)

```sql
with t1 as(
select *,
lead(操作记录) over(partiton by 用户名,date(操作时间) order by 操作时间) as 下次操作记录
 from 用户操作记录表
)

select date(操作时间) as 日期,
count(distinct case when 操作记录='A' and 下次操作记录='B' then 用户名 else null end) as 用户数
from t1
group by date(操作时间)
```

## 选出每个月有连续登录2天的用户名单

有一张“用户登陆记录表”，包含两个字段：用户id、日期。
![Image Name](https://cdn.kesci.com/upload/image/ra1vnz2026.png)

结果如下：
![Image Name](https://cdn.kesci.com/upload/image/ra1vrfcu3o.png)

查询2021年每个月，连续2天都有登陆的用户名单。

```sql
with t1 as(select *,lead(日期,1) over(partiton by 用户id,month(日期) order by 日期) as 本月第二次登录
from 用户登陆记录表
where year(日期)=2021        
)
select month(日期) as 月,
用户id
from t1
where datediff(本月第二次登录,日期)=1
order by 月
```

## 经营分析实战

某公司数据库里有3张表，销售订单表、产品明细表、销售网点表。

”销售订单表”记录了销售情况，每一张数据表示哪位顾客、在哪一天、哪个网点购买了什么产品，购买的数量是多少，以及对应产品的零售价。
![Image Name](https://cdn.kesci.com/upload/image/ra0x5m5lle.png)



“销售网点表”记录了公司的销售网点。
![Image Name](https://cdn.kesci.com/upload/image/ra0x791h7e.png)

**销售订单表和产品明细表通过“产品”字段关联，销售订单表和销售网点通过“交易网点”关联。**

分析在2020年度第一季度的购买人数，销售金额，客单价，客单件，人均购买频次

```sql
select count(distinct 顾客ID) as 购买人数,
sum(销售数量*零售价) as 销售金额,
sum(销售数量*零售价)/count(distinct 顾客ID) as 客单价,
sum(销售数量)/count(distinct 顾客ID) as 客单件,
count(distinct 订单号)/count(distinct 顾客ID)  as 人均购买频次
from 销售订单表
where 交易日期 between '2020-01-01' and '2020-03-31'
```

![Image Name](https://cdn.kesci.com/upload/image/ra0xb671u2.png)

分析品牌在2019.5-2020.4期间的复购率（复购率：在这期间下订单>1的人数/总购买人数）

```sql
select 顾客ID,sum(case when count(distinct 订单号)>1 then 1 else 0 end)/count(*) as 复购率
from 销售订单表
where 交易日期 between '2019-05-01' and '2020-04-31'
group by 顾客ID
--解法二
SELECT 
  SUM(
    CASE
      WHEN 每个顾客下单数 > 1 
      THEN 1 
      ELSE 0 
    END
  ) / COUNT(*) AS 复购率 
FROM
  (SELECT 
    顾客ID,
    COUNT(DISTINCT 订单号) AS 每个顾客下单数 
  FROM
    销售订单表 
  WHERE 交易日期 BETWEEN '2019-05-01' 
    AND '2020-04-30' 
  GROUP BY 顾客ID) a 
```

查找既购买过ProductA又购买过ProductB，但没有购买ProductC的顾客人数，并计算平均客单价

```sql
with t1 as(
    select 顾客ID,group_concat(distinct 产品) as 产品组合
from 销售订单表
group by 顾客ID
    )
select count(distinct 顾客ID) as 顾客人数,
sum(销售数量*零售价)/count(distinct 顾客ID) as 平均客单价
from 销售订单表
where 顾客ID in (
	select 顾客ID
    from t1
    where 产品组合 like '%ProductA%'
    and 产品组合 like '%ProductB%'
	and 产品组合 not like '%ProductC%'
)
group by 顾客ID
```

![Image Name](https://cdn.kesci.com/upload/image/ra0zh24pzw.png)

查找每个城市购买金额排名第二的客人，列出其购买城市、顾客ID、购买金额

```sql
with t1 as(
select *,
    sum(a.销售数量*a.零售价) as 购买金额
    dense_rank() over(partiton by b.城市 order by 购买金额 decs) as 购买金额排名
    from 销售订单表 a
    join 销售网点表 b
    using(交易网点)  
    group by b.城市,a.顾客ID
)
select 城市 as 购买城市,
顾客ID,
sum(销售数量*零售价) as 购买金额
from t1
group by 购买城市
where 购买金额排名=2
--解法2
SELECT 
  * 
FROM
  (SELECT 
    *,
    dense_rank () over (
      PARTITION BY 城市 
  ORDER BY 购买金额 DESC
  ) AS 排名 
  FROM
    (SELECT 
      b.城市,
      a.顾客ID,
      SUM(销售数量 * 零售价) AS 购买金额 
    FROM
      销售订单表 a 
      JOIN 销售网点表 b 
        ON a.交易网点 = b.交易网点 
    GROUP BY b.城市,
      a.顾客ID) a) b 
WHERE 排名 = 2
```

![Image Name](https://cdn.kesci.com/upload/image/ra0zz8cb5l.png)

计算每个城市的店铺数量及各个城市的生意汇总，输出包含无购买记录的城市

```sql
select 城市,
count(distinct 交易网点) as 城市店铺数量,
sum(销售数量*零售价) as 生意汇总
from 销售网点表
left join 销售订单表 using(交易网点)
group by 城市
```

![Image Name](https://cdn.kesci.com/upload/image/ra10eawfgg.png)

## 窗口函数运用

有一张“学生成绩表”，包含4个字段：班级id、学生id、课程id、成绩。
![Image Name](https://cdn.kesci.com/upload/image/ra0vlkla5p.png)

问题1：求出每个学生成绩最高的三条记录；

![Image Name](https://cdn.kesci.com/upload/image/ra0vq76ebg.png)

- rank()：是并列排序，会跳过重复序号
- dense_rank()：是并列排序，不会跳过重复序号
- row_number()：是顺序排序，不跳过任何一个序号，就是行号

```sql
select *,
row_number() over(partition by 学生id order by 成绩 decs) as 学生成绩排名
from 学生成绩表
where 学生成绩排名<=3
--解法二
SELECT 
  * 
FROM
  (SELECT 
    *,
    row_number () over (
      PARTITION BY 学生id 
  ORDER BY 成绩 DESC
  ) AS 学生成绩排名 
  FROM
    学生成绩表) a 
WHERE 学生成绩排名 <= 3 
```

问题2：找出每门课程都高于班级课程平均分的学生；
![Image Name](https://cdn.kesci.com/upload/image/ra0vum9yqi.png)

```sql
with t1 as(
select *,
    avg(成绩) over(partition by 课程id) as 班级课程平均分,
    case when 成绩>班级课程平均分 then 1 else 0 end as 是否高于平均分
    from 学生成绩表
    where 是否高于平均分=1
    group by 课程id,学生id
)
select 班级id,学生id
from
t1
group by 课程id,学生id
```

## 帕累托分析

**什么是二八定律？**

二八定律是说在任何一组东西中，最重要的只占一小部分，约20%。比如家店铺，卖的最多的商品数只占20%

**什么是ABC分类法？**

ABC分类方法是二八定律衍生出来的一种分类方法，由于它把对象分成A、B、C三类，所以叫做ABC分类法，也叫**帕累托分析**

ABC分类法计算步骤：

1）将分析对象由大到小排序

2）计算每一个对象及排在该对象之前的累计占比

3）将累计占比为0~60%的记为A类，60%~85%记为B类，85%以上记为C类

有一张“学生成绩表”，包含3个字段：学号、课程、成绩。
![Image Name](https://cdn.kesci.com/upload/image/ra05zij3mf.png)
**问题：
找出每门课程A类和B类的学生，判断标准是累计占比，0~60%的记为A类，60%~85%记为B类。**

累计占比:

比如课程A的累计占比=课程A的累计成绩/课程A的总成绩

课程A的累计成绩：
对课程A的学生成绩由大到小排序，累计成绩就是该学生的成绩加上排在他前面所有学生的成绩的总和；

课程A的总成绩：
所有学生课程A的成绩总和。

结果如图：

![Image Name](https://cdn.kesci.com/upload/image/ra0657xlao.png)

```sql
with t1 as(select *,
from 学生成绩表
group by 学号,课程
order by 成绩 decs
)
t2 as(
select *,
sum(成绩) over(partition by 课程 order by 成绩 decs rows between unbounded preceding and current row) as 累计成绩,    
sum(成绩) over(partition by 课程) as 科目总成绩
case when (累计成绩/科目总成绩)<=0.6  then "A"
    when (累计成绩/科目总成绩)>0.6 and (累计成绩/科目总成绩)<=0.85 then "B" else "C" end as 类别
from t1    
)
select *
from t2
where 类别 in ("A","B")
--解法二
with temp1 as (
select *,
sum(t1.成绩)over(partition by t1.课程 order by t1.成绩 desc rows between unbounded preceding and current row) as 累计成绩,
sum(t1.成绩)over(partition by t1.课程) as 科目总成绩
from 学生成绩表 t1
),
temp2 as (
select temp1.学号,
temp1.课程,
temp1.成绩,
case when (累计成绩/科目总成绩)<=0.6 then "A"
when (累计成绩/科目总成绩)>0.6 and (累计成绩/科目总成绩)<=0.85 then "B"
else "C" end as type_ABC
from temp1
)
select * from temp2
where type_ABC in ("A","B");
```

unbounded：无界限
preceding：从分区第一行头开始，则为 unbounded。 N为：相对当前行向前的偏移量
following ：与preceding相反,到该分区结束，则为 unbounded。N为：相对当前行向后的偏移量
current row：顾名思义，当前行，偏移量为0

## 除去部门最高&最低工资后的部门平均工资

薪水表中记录了员工的编号，所在部门编号，和薪水：
![Image Name](https://cdn.kesci.com/upload/image/r9y5thg6u8.png)
结果如下图：
![Image Name](https://cdn.kesci.com/upload/image/r9y5w04xle.png)

#查询出每个部门除去最高、最低薪水后的平均薪水，并保留整数。

```sql
with t1 as(
select *,
max(薪水) over(partition by 部门编号) as 最高薪水,
min(薪水) over(partition by 部门编号) as 最低薪水，
from 薪水表      
)
select 部门编号,
case when 薪水<>最高薪水 and 薪水<>最低薪水 then round(avg(薪水)) else null end as 平均薪水
from t1
group by 部门编号
--解法二
SELECT 
  部门编号,
  ROUND(AVG(薪水)) AS 平均薪水 
FROM
  (SELECT 
    *,
    MAX(薪水) over (PARTITION BY 部门编号) 部门最高薪水,
    MIN(薪水) over (PARTITION BY 部门编号) 部门最低薪水 
  FROM
    薪水表) a 
WHERE 薪水 <> 部门最高薪水 
  AND 薪水 <> 部门最低薪水 
GROUP BY 部门编号
```

## 分析红包领取情况

活跃用户表：
![Image Name](https://cdn.kesci.com/upload/image/r9x9kk1uhg.png)

领取红包表：

![Image Name](https://cdn.kesci.com/upload/image/r9x9ngn9ji.png)

1.计算2019年6月1日至今，每日DAU（活跃用户是指有登陆的用户）

```sql
select 登录日期,
count(distinct 用户id) as DAU
from 活跃用户表
where 登录日期>='20190601' 
group by 登录日期;
```

![Image Name](https://cdn.kesci.com/upload/image/r9x9qfk7um.png)

2.分析每天领取红包的用户数、人均领取金额、人均领取次数，要考虑用户属性及领取红包未登录情况。

```sql
select count(distinct case when t2.是新用户="1" then t1.用户id end) as 新用户数,
count(distinct case when t2.是新用户="0" then t1.用户id end) as 旧用户数,
count(distinct case when t2.是新用户 is null then t1.用户id end) as 未登录用户数,
sum(t1.金额)/count(distinct t1.用户ID) as 人均领取金额,
count(*)/count(distinct t1.用户ID) as 人均领取次数
from 领红包表 t1
left join 活跃用户表 t2
using(用户ID) and t1.抢红包日期=t2.登录日期
group by t1.抢红包日期;
```

![Image Name](https://cdn.kesci.com/upload/image/r9x9ut4nqg.png)

3.分析每个月红包领取天数、每个月领取红包的用户数、人均领取金额、人均领取次数。

```sql
select 
data_format(抢红包日期,'%Y%m') as 年月
count(distinct 抢红包日期) as 每个月红包领取天数,
count(distinct 用户id) as 每个月领取红包的用户数,
sum(金额)/count(distinct 用户ID) as 人均领取金额,
count(*)/count(distinct 用户ID) as 人均领取次数
from 领红包表
group by data_format(抢红包日期,'%Y%m');
```

4.分析每个月领过红包用户和未领红包用户的数量

```sql
with m1 as (
select 
data_format(t1.登录日期,'%Y%m') as 年月,
    t1.用户ID,
    t2.抢红包日期,
    min(t2.抢红包日期) over(partition by date_format(t1.登录日期),用户ID) as 各用户每月首次抢到红包日期
from 活跃用户表 t1
	left join 领取红包表 t2
	on t1.登录日期 =t2.抢红包日期 
	and t1.用户ID =t2.用户ID
)
select 年月,
count(case when 各用户每月首次抢到红包日期 is not null then 用户ID end) as 领过红包用户数,
count(case when 各用户每月首次抢到红包日期 is null then 用户ID end) as 未领过红包用户数
from m1
group by 年月;
```

DATE_FORMAT() 函数用于以不同的格式显示日期/时间数据。

语法

```
DATE_FORMAT(date,format)
```

*date* 参数是合法的日期。*format* 规定日期/时间的输出格式。

可以使用的格式有：

| 格式 | 描述                                           |
| :--- | :--------------------------------------------- |
| %a   | 缩写星期名                                     |
| %b   | 缩写月名                                       |
| %c   | 月，数值                                       |
| %D   | 带有英文前缀的月中的天                         |
| %d   | 月的天，数值(00-31)                            |
| %e   | 月的天，数值(0-31)                             |
| %f   | 微秒                                           |
| %H   | 小时 (00-23)                                   |
| %h   | 小时 (01-12)                                   |
| %I   | 小时 (01-12)                                   |
| %i   | 分钟，数值(00-59)                              |
| %j   | 年的天 (001-366)                               |
| %k   | 小时 (0-23)                                    |
| %l   | 小时 (1-12)                                    |
| %M   | 月名                                           |
| %m   | 月，数值(00-12)                                |
| %p   | AM 或 PM                                       |
| %r   | 时间，12-小时（hh:mm:ss AM 或 PM）             |
| %S   | 秒(00-59)                                      |
| %s   | 秒(00-59)                                      |
| %T   | 时间, 24-小时 (hh:mm:ss)                       |
| %U   | 周 (00-53) 星期日是一周的第一天                |
| %u   | 周 (00-53) 星期一是一周的第一天                |
| %V   | 周 (01-53) 星期日是一周的第一天，与 %X 使用    |
| %v   | 周 (01-53) 星期一是一周的第一天，与 %x 使用    |
| %W   | 星期名                                         |
| %w   | 周的天 （0=星期日, 6=星期六）                  |
| %X   | 年，其中的星期日是周的第一天，4 位，与 %V 使用 |
| %x   | 年，其中的星期一是周的第一天，4 位，与 %v 使用 |
| %Y   | 年，4 位                                       |
| %y   | 年，2 位                                       |

## 工资的众数、中位数

emp_salary表：
![Image Name](https://cdn.kesci.com/upload/image/r9wq47jgbv.png)

求员工工资的众数（众数是一组数据中出现次数最多的数值，有时众数在一组数中有好几个。）

having字句可以让我们筛选分组之后的各种数据，where字句在聚合前先筛选记录，也就是说作用在group by和having字句前。and字句在聚合前先筛选记录，也就是说and字句要在group by和having字句前使用。

any,all关键字必须与一个比较操作符一起使用。any关键词可以理解为“对于子查询返回的列中的任一数值，如果比较结果为true，则返回true”。all的意思是“对于子查询返回的列中的所有值，如果比较结果为true，则返回true”

any 可以与=、>、>=、结合起来使用，分别表示等于、大于、大于等于、小于、小于等于、不等于其中的任何一个数据。

all可以与=、>、>=、结合是来使用，分别表示等于、大于、大于等于、小于、小于等于、不等于其中的其中的所有数据。

举个例子：

select s1 from t1 where s1 > any (select s1 from t2);

假设any后面的s1返回了三个值，那其实就等价于

select s1 from t1 where s1 > result1 or s1 > result2 or s2 > result3

而all的用法相当于把上述语句的‘or’缓冲‘and’


```sql
select 
emp_salary
from emp_salary表
group by emp_salary
having count(emp_salary)>=ALL
(select 
count(emp_salary)
from emp_salary表
group by emp_salary
)
```

求员工工资的中位数（把所有观察值高低排序后找出正中间的一个作为中位数。如果观察值有偶数个，通常取最中间的两个数值的平均数作为中位数。）

```sql
select avg(emp_salary) as 工资中位数
from 
(select *,
row_number() over(order by emp_salary) 序号,
 count(*) as 最大序号
 from emp_salary表
)
where 序号 in(
floor((最大序号+1)/2),
ceil((最大序号+1)/2)
)
```

## 滴滴数据分析笔试题

现有四张表，分别是“司机数据”表，“订单数据”表，“在线时长数据”表，“城市匹配数据”表。
1、“司机数据”表，记录了日期、司机id、城市id、首次完成订单时间

![Image Name](https://cdn.kesci.com/upload/image/r9w9tc7get.png)

2、“订单数据”表，记录了日期、订单id、司机id、乘客id、产品线id、流水
产品线id: 1是表示专车，2表示企业，3表示快车，4表示企业快车

![Image Name](https://cdn.kesci.com/upload/image/r9w9xqnlm6.png)

3、“在线时长数据”表，记录了日期、司机id、在线时长

![Image Name](https://cdn.kesci.com/upload/image/r9wa0kr64m.png)

4、“城市匹配数据”表，记录了城市id、城市名称

![Image Name](https://cdn.kesci.com/upload/image/r9wa2r3rmk.png)

问题：

1. 提取2020年8月各城市每天的快车司机数、快车订单量和快车流水数据。

```sql
select t1.日期，
t3.城市名称,
count(distinct t1.司机id)  as 每天快车司机数,
count(t2.订单id) as 快车订单量,
sum(t2.流水) as 快车流水
from 司机数据 t1
left join 订单数据 t2
on t1.司机id=t2.司机id
left join 城市匹配数据 t3
on t1.城市id=t2.城市id
where 日期 between "2020-08-01" and "2020-08-31" and t2.产品线id=3
group by t1.日期;
```

第一问结果如图：
![Image Name](https://cdn.kesci.com/upload/image/raqd2numvv.png)

2. 提取2020年8月和9月，每个月的北京市新老司机（首单日期在当月为新司机）的司机数、在线时
   长和TPH（订单量/在线时长）数据。

```sql
select 
data_format(t1.日期,'%Y%m') as 日期年月,
data_format(t1.首次完成订单时间,'%Y%m') as 首次完成订单年月,
count(distinct case when t1.首次完成订单年月=t1.日期年月 then 司机id end) as 新司机数,
count(distinct case when t1.首次完成订单年月<>t1.日期年月 then 司机id end) as 老司机数,
sum(t4.在线时长) as 在线时长,
count(t2.订单id)/t4.在线时长 as TPH
from 司机数据 t1
join 订单数据 t2
on t1.司机id=t2.司机id
join 城市匹配数据 t3
on t1.城市id=t2.城市id
join 在线时长数据 t4
on t1.司机id=t4.司机id
where 日期年月="2020-08" and 日期年月="2020-09" and t3.城市名称="北京"
group by 日期年月
--解法二
with t1 as(
select substr(d.日期,1,7) as 年月,
d.司机id，
case when substr(d.日期,1,7)=substr(d.首次完成订单时间,1,7) then "新司机" else "老司机" end as "driver_state"   
from 司机数据 d
    where month(d.日期) in (8,9)
    and d.城市id =(select c.城市id from 城市匹配数据 c where c.城市名称="北京")
),
t2 as(
select 年月,
sum(case when driver_state="新司机" then 1 else 0 end) as "新司机数",
sum(case when driver_state="老司机" then 1 else 0 end) as "老司机数"
from t1
group by 年月
),
t3 as(
select substr(t.日期,1,7) as "年月",
sum(在线时长) as 在线时长 
from 在线时长数据 t
left join t1
on t.司机id =t1.司机id
where month(t.日期) in (8,9)
and t.司机id in (select t1.司机id from t1)
group by substr(t.日期,1,7)
),
t4 as(
select substr(o.日期,1,7) as "年月",
count(订单id) as 订单量
from 订单数据 o
where month(o.日期) in (8,9)
and o.司机id in (select t1.司机id from t1)
group by substr(o.日期,1,7)
)
select t2.*,t3.在线时长,t4.订单量/t3.在线时长 as "TPH"
from t2,t3,t4
where t2.年月=t3.年月
and t3.年月=t4.年月;
```

第二问结果如图：
![Image Name](https://cdn.kesci.com/upload/image/r9wfrzex2k.png)

3. 分别提取司机数大于20，司机总在线时长大于2小时，订单量大于1的城市名称数据。

```sql
with t1 as (
select *,
count(distinct 司机id) over(partition by 城市id) as 司机数    
from 司机数据 
group by 城市id   
),
t2 as(
select *,
sum(在线时长) over(distinct partition by 司机id) as 总在线时长,
from 在线时长数据    
join t1 
on t1.司机id=在线时长数据.司机id 
),
t3 as(
select *,
count(订单id) as 订单量  
from 订单数据
join t2
on t2.司机id=司机id    
),
select
城市名称,司机id,司机数,总在线时长,订单量
from 城市匹配数据
join t3
on t3.城市id=城市id
group by 城市名称
having 司机数>20 and 总在线时长>2 and 订单量>1
```

解法二：

（3.1）提取司机数大于20的城市名称数据

```sql
select c.城市名称 ,count(d.司机id) as 司机数
from 司机数据 d
join 城市匹配数据 c
on d.城市id =c.城市id
group by c.城市名称 
having count(d.司机id)>20;
```

![Image Name](https://cdn.kesci.com/upload/image/r9wg2e610r.png)

3.2）提取司机总在线时长大于2小时的城市名称数据

```sql
select c.城市名称 ,t1.*
from 
(select t.司机id ,sum(t.在线时长) as 总在线时长
from 在线时长数据 t
group by t.司机id
having sum(t.在线时长)>2) t1
left join 司机数据 d
on t1.司机id =d.司机id
left join 城市匹配数据 c
on d.城市id =c.城市id;
```

![Image Name](https://cdn.kesci.com/upload/image/r9wgg525ch.png)

（3.3）提取司机订单量大于1的城市名称数据

```sql
select c.城市名称 ,t1.*
from 
(select o.司机id ,count(o.订单id) as 司机订单量
from 订单数据 o
group by o.司机id
having count(o.订单id)>1)t1
left join 司机数据 d
on t1.司机id =d.司机id
left join 城市匹配数据 c
on d.城市id =c.城市id;
```

![Image Name](https://cdn.kesci.com/upload/image/r9wgyll82h.png)

## 连续打卡天数

现在有一张表t，这张表存储了每个员工每天的打卡情况
现在需要统计截止目前每个员工的连续打卡天数，表t如下表所示：

![Image Name](https://cdn.kesci.com/upload/image/r9vgwmrzqk.png)

结果如下：

![Image Name](https://cdn.kesci.com/upload/image/r9vhwmudll.png)

```sql
with t1 as(select *,     
from t
where is_flag="1"
group by uid
order by tdate           
),
select uid,
sum(case when lag(tdate,1) is not null then 1 else 0 end) as flag_days  
from t1   
group by uid    
--解法二
with t1 as(
select *,
lead(tdate) over (partition by uid order by tdate) as 下次打卡日期,
 max(tdate) over() as 截至日期
 from t
 where is_flag=1
),
t2 as(
select *,
lead(下次打卡日期) over(partition by uid order by tdate) as 在次打卡日期
from t1
where datediff(下次打卡日期,tdate)=1    
)
select uid,
sum(case when datediff(再次打卡日期,下次打卡日期)=1 and datediff(下次打卡日期,tdate)=1 then 2 else 0 end) as flag_days
from t2
group by uid;
```

## 连续天数

问题：请求出sales_record表中连续三天有销售记录的店铺

sales_record表：

![Image Name](https://cdn.kesci.com/upload/image/r9vgh3u4nw.png)

结果如下：

![Image Name](https://cdn.kesci.com/upload/image/r9vgrmgad4.png)

```sql
with t1 as(
select *,
	lead(td) over (distinct partition by shopid order by td) as 第二天销售记录
    from sales_record
    where sale is not null
    group by distinct shopid
),
t2 as(
select *,
	lead(第二天销售记录) over (distinct partition by shopid order by td) as 第三天销售记录
    from t1
    group by distinct shopid
),
selet distinct shopid,
from t2
where 第二天销售记录 is not null and 第三天销售记录 is not null
--解法二
select distinct shopid
from (select
     a.shopid,
     a.dt as day1,
     b.dt as day2,
     c.dt as day3
     from sales_record a
      left join sales_record b
      on a.shopid=b.shopid
      left join sales_revord c
      on b.shopid=c.shopid
     ) d
     where day2-day1=1
     and day3-day2=1
```

## N日留存率

【相机】是深受大家喜爱的应用之一，现在我们需要研究相机的活跃情况，需统计如下数据：
某日活跃的用户(uid)在后续的一周内的留存情况（计算次留、三留、七留）
指标定义：
某日活跃用户数：某日活跃的去重用户数
N日留存用户数：某日活跃的用户在之后的第N日活跃用户数
N日活跃留存率：N日留存用户数/某日活跃用户数

例：
20180501日去重用户数10000，这批用户20190503仍有7000人活跃，则3日活跃的留存率为7000/10000=70%

数据表名：act_user_info

![Image Name](https://cdn.kesci.com/upload/image/r9v2iupdd4.png)

要求：
1、活跃用户数为整数
2、该留存率表示为百分比，结果保留为两位小数
3、仅一条sql或hive sql完成
4、仅写出实现查询的代码即可

结果如下：

![Image Name](https://cdn.kesci.com/upload/image/r9v54yii44.png)

```sql
select date_format(登陆时间,%Y%m%d) as 日期，
count(distinct 用户id) over(partition by 日期 order by 日期) as 活跃用户数,
count(lead(distinct 活跃用户数,1) over(partition by 日期 order by 日期)) as 次日留存数，
count(lead(distinct 次日留存数,1) over(partition by 日期 order by 日期)) as 三日留存数，
count(lead(distinct 三日留存数,4) over(partition by 日期 order by 日期)) as 七日留存数，
concat(round(次日留存数/活跃用户数*100,2),'%') as 次日留存率,
concat(round(三日留存数/活跃用户数*100,2),'%') as 三日留存率,
concat(round(七日留存数/活跃用户数*100,2),'%') as 七日留存率,
from act_user_info
where 应用名称="相机"
group by 日期
--解法二
SELECT 时间1,
COUNT(DISTINCT 用户id) AS '活跃用户数',
COUNT(DISTINCT CASE WHEN 时间2-时间1=1 THEN 用户id ELSE NULL END) AS '次日留存数',
COUNT(DISTINCT CASE WHEN 时间2-时间1=2 THEN 用户id ELSE NULL END) AS '三日留存数',
COUNT(DISTINCT CASE WHEN 时间2-时间1=6 THEN 用户id ELSE NULL END) AS '七日留存数',
CONCAT(ROUND(COUNT(DISTINCT CASE WHEN 时间2-时间1=1 THEN 用户id ELSE NULL END)/COUNT(DISTINCT 用户id)*100,2),'%') AS '次日留存率',
CONCAT(ROUND(COUNT(DISTINCT CASE WHEN 时间2-时间1=2 THEN 用户id ELSE NULL END)/COUNT(DISTINCT 用户id)*100,2),'%') AS '三日留存率',
CONCAT(ROUND(COUNT(DISTINCT CASE WHEN 时间2-时间1=6 THEN 用户id ELSE NULL END)/COUNT(DISTINCT 用户id)*100,2),'%') AS '七日留存率'
FROM
(SELECT a.用户id,DATE_FORMAT(a.登陆时间,'%Y%m%d') AS 时间1,DATE_FORMAT(b.登陆时间,'%Y%m%d') AS 时间2
FROM act_user_info a
LEFT JOIN act_user_info b
ON a.用户id=b.用户id
WHERE a.应用名称='相机'
AND b.应用名称='相机')a
GROUP BY 时间1
```

## 统计不同月份用户的回购数

![Image Name](https://cdn.kesci.com/upload/image/r9v0tgdike.png)

问题：统计不同月份用户的回购数
（回购：上月购买用户中有多少用户本月又再次购买）

结果如图：

![Image Name](https://cdn.kesci.com/upload/image/r9v1pniqsm.png)

```sql
select date_format(buy_datetime,'%Y%m') as 日期,
count(distinct user_id) over(partition by 日期 order by 日期) as 当月首购数,
count(lead(当月首购数)) as 次月回购数,
count(lead(当月首购数,2)) as 三月回购数,
count(lead(当月首购数,3)) as 四月回购数,
count(lead(当月首购数,4)) as 五月回购数,
count(lead(当月首购数,5)) as 六月回购数,
from buy_data 
group by 日期
--解法二
with t1 as(
select
		a.user_id,
		a.buy_datetime as 时间1,
		b.buy_datetime as 时间2
	from
		buy_data a
	left join buy_data b 
      on
		a.user_id = b.user_id
),
select
	DATE_FORMAT(时间1, '%Y-%m') as 日期,
	COUNT(distinct user_id) as '当月首购数',

    count(distinct case when date_format(date_add(时间1,interval 1 month),'%Y-%m')=date_format(时间2,'%Y-%m') then user_id end) as '次月回购数',
    count(distinct case when date_format(date_add(时间1,interval 2 month),'%Y-%m')=date_format(时间2,'%Y-%m') then user_id end) as '三月回购数',
    count(distinct case when date_format(date_add(时间1,interval 3 month),'%Y-%m')=date_format(时间2,'%Y-%m') then user_id end) as '四月回购数',
	count(distinct case when date_format(date_add(时间1,interval 4 month),'%Y-%m')=date_format(时间2,'%Y-%m') then user_id end) as '五月回购数',
    count(distinct case when date_format(date_add(时间1,interval 5 month),'%Y-%m')=date_format(时间2,'%Y-%m') then user_id end) as '六月回购数',
from t1
group by 日期   
```

## 所有学生参加所有科目测试的次数

学生表: Students

![Image Name](https://cdn.kesci.com/upload/image/r9uzpxfhcc.png)

科目表: Subjects

![Image Name](https://cdn.kesci.com/upload/image/r9uzqgh0y.png)

考试表: Examinations

![Image Name](https://cdn.kesci.com/upload/image/r9uzqy1ppr.png)

考试表的每一行记录就表示学生表里的某个学生参加了一次科目表里某门科目测试。
要求写一段 SQL 语句，查询出每个学生参加每一门科目测试的次数，结果按 student_id 和subject_name 排序。
结果表需包含所有学生和所有科目（**即便测试次数为0**）

结果如下：

![Image Name](https://cdn.kesci.com/upload/image/r9v0kp9yq8.png)

```sql
select a.student_id,a.student_name,c.subject_name,
count(b.student_id) over(partition by subject_name) as attended_exams
from Students a
left join Examinations b
using(student_id)
join Subjects c
using(subject_name)
group by a.student_id,a.student_name,c.subject_name
order by a.student_id,c.subject_name
--解法二
SELECT 
  a.student_id,
  a.student_name,
  a.subject_name,
  COUNT(e.subject_name) AS attended_exams 
FROM
  (SELECT 
    * 
  FROM
    students 
    CROSS JOIN subjects) a --cross join：笛卡尔积
  LEFT JOIN examinations e 
    ON a.student_id = e.student_id
    AND a.subject_name = e.subject_name 
GROUP BY a.student_id,
  a.student_name,
  a.subject_name 
ORDER BY a.student_id,
  a.subject_name 
```

## 7天内是否玩过游戏

![Image Name](https://cdn.kesci.com/upload/image/r9uyu8xws8.png)

请查询出以下表。”<7 days” 表示此玩家每一行日期的前7天之内玩过游戏(不是今天之前的前7天，是玩家玩游戏日期的前7天）

![Image Name](https://cdn.kesci.com/upload/image/r9uywicozg.png)

```sql
with t1 as(select *,
lag(play_date) over(partition by player_name order by play_date) as 上次游戏时间
from play_time
),
select
*,
case when datediff(play_date,上次游戏时间)<7 then 'yes' else 'no' end as "<7 days"
from t1
order play_date;
```

## 用户复购率

现在数据库中有一张用户交易表 netease_order，其中有orderid（订单ID）、userid（用户ID）、
amount（消费金额）、paytime（支付时间）
请写出对应的SQL语句，查出每个月的新客数（新客指在严选首次支付的用户），当月有复购的新客数，新客当月复购率（当月有复购的新客数/月总新客数）。

![Image Name](https://cdn.kesci.com/upload/image/r9uwtyrw63.png)

得到结果如下图：

![Image Name](https://cdn.kesci.com/upload/image/r9uxsxq96w.png)

```sql
with t1 as(
select *,
    substr(paytime,1,6) as 年月,
    min(paytime) over(partition by userid) 首次支付时间,
    lead(paytime) over(partition by user_id order by paytime) as 下次支付时间
from netease_order 
),
select 年月,
count(userid) over(partition by 年月) 月总新客数,   
sum(case when date_format(首次支付时间,'%Y-%m')=date_format(下次支付时间,'%Y-%m') then 1 else 0 end) as 当月有复购的新客数,
sum(case when date_format(首次支付时间,'%Y-%m')=date_format(下次支付时间,'%Y-%m') then 1 else 0 end)/count(userid) as 当月有复购的新客数,    
from t1
where paytime=首次支付时间
group by 年月;   
```

## 用户行为分析

有订单事务表、收藏事务表,要求：请用一句SQL取出所有用户对商品的行为特征，
特征分为：
已购买、购买未收藏、收藏未购买、收藏且购买

订单事务表如下（redbk_orders）：
![Image Name](https://cdn.kesci.com/upload/image/rc22fn62e3.png)
收藏事务表如下（redbk_favorites）：
![Image Name](https://cdn.kesci.com/upload/image/rc22g66zmq.png)
最终结果如图：
![Image Name](https://cdn.kesci.com/upload/image/r9umz130yu.png)

```sql
select a.user_id,a.item_id,
case when a.par_time is not null then '1' else '0' end as 已购买,
case when a.par_time is not null and b.fav_time is null then '1' else '0' end as 购买未收藏,
case when a.par_time is null and b.fav_time is not null then '1' else '0' end as 收藏未购买,
case when a.par_time is not null and b.fav_time is not null then '1' else '0' end as 收藏且购买,
from redbk_orders a
join redbk_favorites b
using(user_id) and
using(item_id)
group by a.user_id,a.item_id
order by a.user_id,a.item_id
--解法二
SELECT o.user_id,o.item_id,
1 AS '已购买',
CASE WHEN f.item_id IS NULL THEN 1 ELSE 0 END AS '购买未收藏',
0 AS '收藏未购买',
CASE WHEN f.item_id IS NOT NULL  THEN 1 ELSE 0 END AS '收藏且购买'
FROM redbk_orders o
LEFT JOIN redbk_favorites f
ON o.user_id=f.user_id
AND o.item_id=f.item_id
union
SELECT f.user_id,f.item_id,
CASE WHEN o.item_id IS NOT NULL THEN 1 ELSE 0 END AS '已购买',
0 AS '购买未收藏',
CASE WHEN o.item_id IS NULL THEN 1 ELSE 0 END AS '收藏未购买',
CASE WHEN o.item_id IS NOT NULL THEN 1 ELSE 0 END AS '收藏且购买'
FROM redbk_favorites f
LEFT JOIN redbk_orders o
ON o.user_id=f.user_id
AND o.item_id=f.item_id
order by user_id,item_id;
```

题意分析

本题主要考察多表连接以及union的用法。分别将两个表作为主表进行左外连接，使用case when 取出相同的字段，最后通过union联结并过滤重复项得到最终结果。

(1)将订单表作为主表左外连接,连接条件不仅要user_id相同，item_id也要相同。

```sql
SELECT *
FROM redbk_orders o
LEFT JOIN redbk_favorites f
ON o.user_id=f.user_id
AND o.item_id=f.item_id
```

![Image Name](https://cdn.kesci.com/upload/image/r9uljyl55s.png)

(2)因为订单表作为主表，得到的是所有订单（无论是否收藏）的结果，所以已购买列全为1，收藏未购买列则全为0，通过判断收藏表的商品id(item_id)是否为null可判断是否收藏。

```sql
SELECT o.user_id,o.item_id,
1 AS '已购买',
CASE WHEN f.item_id IS NULL THEN 1 ELSE 0 END AS '购买未收藏',
0 AS '收藏未购买',
CASE WHEN f.item_id IS NOT NULL THEN 1 ELSE 0 END AS '收藏且购买'
FROM redbk_orders o
LEFT JOIN redbk_favorites f
ON o.user_id=f.user_id
AND o.item_id=f.item_id
```

得到结果如下：

![Image Name](https://cdn.kesci.com/upload/image/r9um38xljt.png)

同上分析，将收藏表作为主表左外连接，得到的是所有收藏（无论是否下单）的结果，则购买情况通过订单表的item_id是否为null进行判断。而购买未收藏则全列为0。

```sql
SELECT f.user_id,f.item_id,
CASE WHEN o.item_id IS NOT NULL THEN 1 ELSE 0 END AS '已购买',
0 AS '购买未收藏',
CASE WHEN o.item_id IS NULL THEN 1 ELSE 0 END AS '收藏未购买',
CASE WHEN o.item_id IS NOT NULL THEN 1 ELSE 0 END AS '收藏且购买'
FROM redbk_favorites f
LEFT JOIN redbk_orders o
ON o.user_id=f.user_id
AND o.item_id=f.item_id
```

结果如下：

![Image Name](https://cdn.kesci.com/upload/image/r9umnaj9m5.png)

上两步的最终结果因为字段数和字段名称都一致，通过union联结一起并剔除重复行。再对最终结果的user_id,item_id排序就得出最后结果。

MySQL UNION 操作符用于连接两个以上的 SELECT 语句的结果组合到一个结果集合中。多个 SELECT 语句会删除重复的数据。

## 用户取消率

Trips表中存所有货拉拉的行程信息。每段行程有唯一键 Id，Client_Id 和 Driver_Id。Status 是枚举类型，枚举内容为 (‘completed’,‘cancelled_by_driver’,‘cancelled_by_client’)。

![Image Name](https://cdn.kesci.com/upload/image/r9uiv8ek9a.png)

Users 表存所有用户。每个用户有唯一键 Users_Id。Banned 表示这个用户是否被禁止，Role 则是一个表示（‘client’, ‘driver’, ‘partner’）的枚举类型。

![Image Name](https://cdn.kesci.com/upload/image/r9uiy1iydu.png)

写一段 SQL 语句查出 2019年10月1日 至 2019年10月3日 期间非禁止用户的取消率。基于上表，你的SQL 语句应返回如下结果，取消率（Cancellation Rate）保留两位小数。

取消率即未完成订单（无所谓是司机还是顾客取消）/总订单，前提是这里订单都是不能有禁止用户参与的。

![Image Name](https://cdn.kesci.com/upload/image/r9ujuoa7im.png)

```sql
select distinct t.Request_at as "Day",
case when u.Banned='No' then round(sum(case when t.Status='cancelled_by_driver' or t.Status='cancelled_by_client' then 1 else 0 end)/count(*),2) else null end as "Cancellation_Rate"
from Trips t
join Users u
on u.Users_ID=t.Client_Id
where Day between '2019/10/1' and '2019/10/3' 
group by Day
order by  Day
--解法二
SELECT Request_at AS "Day",
ROUND(SUM(CASE WHEN STATUS='completed' THEN 0 ELSE 1 END)/COUNT(*),2) AS "Cancellation_Rate"
FROM
(SELECT *
FROM trips
WHERE client_ID NOT IN (SELECT Users_ID FROM users WHERE Banned='Yes')
AND Driver_ID NOT IN (SELECT Users_ID FROM users WHERE Banned='Yes')
AND Request_at BETWEEN '2019-10-01' AND '2019-10-03')a
GROUP BY Request_at
```

## 改变相邻座位

小马是货拉拉的行政同学，她有一张座位表，储存员工名字和与他们相对应的座位号id。其中第一列的id 是连续递增的因工作需要小马想改变相邻俩同事的座位。你能不能帮她写一个 SQL query 来输出小马想要的结果呢？

![Image Name](https://cdn.kesci.com/upload/image/r9uhelr58p.png)

结果图如下：

![Image Name](https://cdn.kesci.com/upload/image/r9uienrsfm.png)

```sql
select case when mod(ID,2)<>0 and ID<>最大ID then id+1
when mod(ID,2)<>0 and ID=最大ID then id else id-1 end as "ID",
FROM
  (select *,MAX(id) over () 最大ID 
  FROM Seat) a 
ORDER BY ID 
```

id为奇数的员工，如1、3、5的员工要换到2、4、6去，2、4、6员工则换到1、3、5去，这是当最大ID为偶数时的情况，当最大ID为奇数时，最大ID对应的员工就没有人换座位，则保持原位。而判断id为奇数还是偶数的方法，可以看它除以2的余数是否为0，为0的为偶数，不为0则是奇数。

## 统计薪水最高员工

货拉拉EMPLOYEE表里包含着所有货拉拉的员工信息，每个员工有自己的工号、姓名、薪水、和部门ID
![Image Name](https://cdn.kesci.com/upload/image/rau1u8vc8e.png)
货拉拉DEPARTMENT表里包含着货拉拉所有的部门
![Image Name](https://cdn.kesci.com/upload/image/rau1tq4b1h.png)
现在需要统计出货拉拉每个部门薪水最高的员工（包含department/name/salary字段）

```sql
select distinct b.DEPARTMENT,a.Name,
max(Salary) over (partition by DepartmentID) as 部门最高薪水
from EMPLOYEE a
join DEPARTMENT b
on a.ID=b.DepartmentID
where a.Salary=部门最高薪水
group by DEPARTMENT
```

