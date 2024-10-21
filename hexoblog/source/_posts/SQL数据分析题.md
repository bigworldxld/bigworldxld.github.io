---
title: SQL数据分析题
categories: database
tags: SQL
img: /images/SQL数据分析题.jpg
abbrlink: 24233
date: 2022-12-11 21:08:39
---

## 题目1 找出每个部门工资第二高的员工

现有一张公司员工信息表employee，表中包含如下4个字段。

- **employee_id（员工ID）**：VARCHAR。
- **employee_name（员工姓名）**：VARCHAR。
- **employee_salary（员工薪资）**：INT。
- **department（员工所属部门ID）**：VARCHAR。

还有一张部门信息表department，表中包含如下两个字段。

- **department_id（部门ID）**：VARCHAR。
- **department_name（部门名称）**：VARCHAR。

数据导入的代码如下：

```sql
DROP TABLE IF EXISTS employee;
CREATE TABLE employee(
employee_id VARCHAR(8),
employee_name VARCHAR(8),
employee_salary INT(8),
department VARCHAR(8)
)
ENGINE = InnoDB
DEFAULT CHARSET = utf8;
INSERT INTO
employee (employee_id,employee_name,employee_salary,department)
VALUE ('a001','Bob',7000,'b1')
     ,('a002','Jack',9000,'b1')
     ,('a003','Alice',8000,'b2')
     ,('a004','Ben',5000,'b2')
     ,('a005','Candy',4000,'b2')
     ,('a006','Allen',5000,'b2')
     ,('a007','Linda',10000,'b3');

DROP TABLE IF EXISTS department;
CREATE TABLE department(
department_id VARCHAR(8),
department_name VARCHAR(8)
)
ENGINE = InnoDB
DEFAULT CHARSET = utf8;
INSERT INTO
department (department_id,department_name)
VALUE ('b1','Sales')
     ,('b2','IT')
     ,('b3','Product');
```

问题：查询每个部门薪资第二高的员工信息。

输出内容包括：

- **employee_id（员工ID）**
- **employee_name（员工姓名）**
- **employee_salary（员工薪资）**
- **department_id（员工所属部门名称）**

**可供参考的解题思路：**使用窗口函数根据部门ID分组，在组内按照员工薪资降序排列并记为ranking，然后将该处理后的表和部门信息表进行内连接，从而把部门名称关联进来，最后在连接后的表上使用ranking=2作为薪资第二高的条件进行WHERE筛选，选择需要的列，即可得到结果。

**涉及知识点：**窗口函数、子查询、多表连接。

```
SELECT  a.employee_id
       ,a.employee_name
       ,a.employee_salary
       ,b.department_id
FROM 
(
    SELECT  *
    ,RANK() OVER (PARTITION BY department
     ORDER BY employee_salary DESC) AS ranking
    FROM employee
) AS a
INNER JOIN department AS b
ON a.department = b.department_id
WHERE a.ranking = 2;
```

## 题目2：网站登录时间间隔统计

现有一张网站登录情况表login_info，该表记录了所有用户的网站登录信息，包含如下两个字段。

- **user_id（用户ID）**：VARCHAR。
- **login_time（用户登录日期）**：DATE。

**数据导入的代码如下：**

```sql
DROP TABLE IF EXISTS login_info;
CREATE TABLE login_info(
user_id VARCHAR(8),
login_time DATE
)
ENGINE = InnoDB
DEFAULT CHARSET = utf8;
INSERT INTO
login_info (user_id,login_time)
VALUE ('a001','2021-01-01')
,('b001','2021-01-01')
,('a001','2021-01-03')
,('a001','2021-01-06')
,('a001','2021-01-07')
,('b001','2021-01-07')
,('a001','2021-01-08')
,('a001','2021-01-09')
,('b001','2021-01-09')
,('b001','2021-01-10')
,('b001','2021-01-15')
,('a001','2021-01-16')
,('a001','2021-01-18')
,('a001','2021-01-19')
,('b001','2021-01-20')
,('a001','2021-01-23');
```

问题：计算每个用户登录日期间隔小于5天的次数。

输出内容包括：user_id	num

**可供参考的解题思路：**本题考查LEAD()函数在处理时间间隔问题上的使用方法，观察内层的查询部分，使用LEAD()函数在原有的login_time字段的基础上创造一列新的时间字段（即该用户下一次登录日期）

经过LEAD()函数处理后，数据会根据user_id字段分组后按照login_time字段排序。经过内层的处理后，只需在外层筛选出next_login_time与login_time字段的日期差小于5天的数据，即最终统计的目标数据，这里使用了TIMESTAMPDIFF(DAY, login_time, next_login_time)计算日期差，最后分组聚合统计不同user_id的记录个数，即每个用户登录日期间隔小于5天的次数。

**涉及知识点：**窗口函数、子查询、分组聚合、时间函数

```sql
SELECT a.user_id
    ,COUNT(*) AS num
FROM 
(
  SELECT user_id
      ,login_time
      ,LEAD(login_time,1) OVER (PARTITION BY user_id
      ORDER BY login_time) AS next_login_time
  FROM login_info
) AS a
WHERE TIMESTAMPDIFF(DAY, login_time, next_login_time) < 5 
GROUP BY user_id;
```

## 题目3：用户购买渠道分析

现有一张用户购买信息表purchase_channel，该表记录了用户在某购物平台的购物信息，该购物平台具有网页端（web）和手机端（app）两种访问方式，表中包含如下4个字段。

- **user_id（用户ID）：**VARCHAR。
- **channel（用户购买渠道）：**VARCHAR。
- **purchase_date（购买日期）：**DATE。
- **purchase_amount（购买金额）：**INT。

**数据导入代码如下：**

```sql
DROP TABLE IF EXISTS purchase_channel;
CREATE TABLE purchase_channel(
user_id VARCHAR(8),
channel VARCHAR(8),
purchase_date DATE,
purchase_amount INT(8)
)
ENGINE = InnoDB
DEFAULT CHARSET = utf8;
INSERT INTO
purchase_channel (user_id,channel,purchase_date,purchase_amount)
VALUE ('a001','app','2021-03-14',200)
     ,('a001','web','2021-03-14',100)
     ,('a002','app','2021-03-14',400)
     ,('a001','web','2021-03-15',3000)
     ,('a002','app','2021-03-15',900)
     ,('a003','app','2021-03-15',1000);
```

问题：查询每天仅使用手机端的用户、仅使用网页端的用户和同时使用网页端和手机端（both）的不同用户人数和总购物金额，并且即使某天某渠道没有用户的购买信息，也需要展示。

**输出内容包括：**

purchase_date（日期）

channel（购买渠道）

sum_amount（总购买金额）

total_users（不同用户人数）

**可供参考的解题思路：**根据用户ID和日期进行分组，通过统计用户在各购买渠道购物的记录个数来判断某用户在某日期购物时采用的访问方式（web、app和both）。其中，web和app可以通过一个SELECT语句查询，both则可以通过另一个SELECT语句查询。将两部分使用UNION连接在一起，并将以上部分作为子查询内部，在子查询外部统计不同购买日期、购买渠道的总购买金额和总购买用户。

```sql
SELECT  purchase_date
       ,channel
       ,SUM(sum_amount) sum_amount
       ,SUM(total_users) total_users
FROM 
(
    SELECT  purchase_date
           ,MIN(channel) channel
           ,SUM(purchase_amount) sum_amount
           ,COUNT(DISTINCT user_id) total_users
    FROM purchase_channel
    GROUP BY  purchase_date
             ,user_id
    HAVING COUNT(DISTINCT channel) = 1 UNION
    SELECT  purchase_date
           ,'both' channel
           ,SUM(purchase_amount) sum_amount
           ,COUNT(DISTINCT user_id) total_users
    FROM purchase_channel
    GROUP BY  purchase_date
             ,user_id
    HAVING COUNT(DISTINCT channel) > 1 
) c
GROUP BY  purchase_date
         ,channel;
```

题目要求即使某天某渠道没有用户的购买信息，也需要展示。而想要展示更全的信息，则考虑使用最全的信息（所有日期和3个渠道的笛卡尔积）与刚查询出的结果数据表进行LEFT JOIN连接，即可得到两张表根据日期和渠道进行连接的结果。

**涉及知识点：**UNION、分组聚合、数据去重。

```sql
SELECT  t1.purchase_date
       ,t1.channel
       ,t2.sum_amount
       ,t2.total_users
FROM 
(
    SELECT  DISTINCT a.purchase_date
           ,b.channel
    FROM purchase_channel a,
    (
        SELECT  "app" AS channel
        UNION
        SELECT  "web" AS channel
        UNION
        SELECT  "both" AS channel
    ) b
) t1
LEFT JOIN 
(
SELECT 
purchase_date,
channel,
SUM(sum_amount) sum_amount,
SUM(total_users) total_users
FROM (
SELECT  purchase_date
           ,MIN(channel) channel
           ,SUM(purchase_amount) sum_amount
           ,COUNT(DISTINCT user_id) total_users
    FROM purchase_channel
    GROUP BY  purchase_date,user_id
    HAVING COUNT(DISTINCT channel) = 1 
    UNION
    SELECT  purchase_date
           ,'both' channel
           ,SUM(purchase_amount) sum_amount
           ,COUNT(DISTINCT user_id) total_users
    FROM purchase_channel
    GROUP BY  purchase_date,user_id
    HAVING COUNT(DISTINCT channel) > 1
)c GROUP BY purchase_date, channel
) t2
ON t1.purchase_date = t2.purchase_date AND t1.channel = t2.channel;
```

## 题目4：hive之行转列

指定数据库

```sql
show databases ;
show tables ;
use test_sql;
set hive.exec.mode.local.auto=true;
```

```sql
create table table2(year int,month int ,amount double) ;
insert overwrite table table2 values
                                  (1991,1,1.1),
                                  (1991,2,1.2),
                                  (1991,3,1.3),
                                  (1991,4,1.4),
                                  (1992,1,2.1),
                                  (1992,2,2.2),
                                  (1992,3,2.3),
                                  (1992,4,2.4);
select * from table2;
```

-- 扩展，当case when里面的分支只有2个，并且是互斥，则等价于if语句
-- 也就是 【case when month=1 then amount else 0 end】 等价于【if(month=1,amount,0)】
--行转列
--常规做法是，group by+sum(if())

--原始写法

```sql
select year,
       sum(if(month=1,amount,0)) as m1,
       sum(if(month=2,amount,0)) as m2,
       sum(if(month=3,amount,0)) as m3,
       sum(if(month=4,amount,0)) as m4
from table2
group by year;
--或者
select year,
       sum(case when month=1 then amount else 0 end) as m1,
       sum(case when month=2 then amount else 0 end) as m2,
       sum(case when month=3 then amount else 0 end) as m3,
       sum(case when month=4 then amount else 0 end) as m4
from table2 as t
group by year
```

-- 腾讯游戏
--建表

```sql
create table table1(DDate string, shengfu string) ;
insert overwrite table table1 values ('2015-05-09', "胜"),
                                     ('2015-05-09', "胜"),
                                     ('2015-05-09', "负"),
                                     ('2015-05-09', "负"),
                                     ('2015-05-10', "胜"),
                                     ('2015-05-10', "负"),
                                     ('2015-05-10', "负");
select * from table1;
```

```sql
select DDate,
       sum(case when shengfu='胜' then 1 else null end) as `胜`,
       sum(case when shengfu='负' then 1 else null end) as `负`
from  table1 as t
group by DDate;
```

-- 腾讯QQ

```sql
create table tableA(qq string, game string);
insert overwrite table tableA values
                                  (10000, 'a'),
                                  (10000, 'b'),
                                  (10000, 'c'),
                                  (20000, 'c'),
                                  (20000, 'd');

create table tableB(qq string, game string) ;
insert overwrite table tableB values
                                  (10000, 'a_b_c'),
                                  (20000, 'c_d');

select * from tableA;
select * from tableB;
```

MySQL中，Concat_WS() 函数 用来通过指定符号，将2个或多个字段拼接在一起，返回拼接后的字符串。

Concat_WS语法
Concat_WS( [SEPARATOR 拼接符号] , str1 , str2 , … )

参数说明
SEPARATOR
        拼接符号，默认的是 separator ：逗号,
str1 , str2 , …
        要拼接在一起的字段(字段不存在，MySQL将会报错)
返回值说明
若字段str1、str2中有null，则会被Concat_WS()函数忽略；
若字段只剩下一个字段有值（只有一个字段参与拼接），则直接返回该字段的值；

```sql
-- 行转列
select qq,
       concat_ws( '_', collect_list(game)) arr
from tableA group by qq;
-- 列转行 需要借助【侧视图+explode函数】
select qq,
       game2
from tableb lateral view explode(split(game,'_')) v1 as game2;
```

lateral view用于和split、explode等UDTF一起使用的，能将一行数据拆分成多行数据，在此基础上可以对拆分的数据进行聚合，lateral view首先为原始表的每行调用UDTF，UDTF会把一行拆分成一行或者多行，lateral view在把结果组合，产生一个支持别名表的虚拟表。

## 题目5: hive之连续N天登录

```sql
create table game(name string,`date` string);
insert overwrite table game  values
                       ('张三','2021-01-01'),
                       ('张三','2021-01-02'),
                       ('张三','2021-01-03'),
                       ('张三','2021-01-02'),

                       ('张三','2021-01-07'),
                       ('张三','2021-01-08'),
                       ('张三','2021-01-09'),

                       ('李四','2021-01-01'),
                       ('李四','2021-01-02'),
                       ('王五','2021-01-03'),
                       ('王五','2021-01-02'),
                       ('王五','2021-01-02');
```

```sql
--方案1
with t1 as (
    select distinct name,`date` from game
),
     t2 as (
         select *,
                row_number() over (partition by name order by `date`) as rn
         from t1
     ),
     t3 as (
         select *,
                date_sub(`date`,rn) as temp
         from t2
     ),
     t4 as (
         select name,temp ,
                count(1) as cnt
         from t3
         group by name,temp
         having count(1)>=3
     )
select distinct name from t4;

--方案2
with t1 as (
    select distinct name,`date` from game
),
    t2 as (
        select *,
               date_add(`date`,2) as date2,
               lead(`date`,2) over(partition by name order by `date`) as date3
          from t1
    )
select distinct name from t2 where date2=date3;
```

## 题目6: 窗口函数

```sql
create table score(cid int ,sname string,score int);
insert overwrite table score values
(1,'张三',60),
(1,'李四',70),
(1,'王五',80),
(1,'赵六',90),
(2,'安安',80),
(2,'平平',90);
```

### 第一类、 聚合类的窗口函数

​       -- sum() over()
​       -- count/avg/max/min
​       --让每个学生都知道，班级总分是多少
​       --sum(score) over (partition by cid ) as `班级总分`,

   --计算同一个班级内，每个同学和比他分数低的总分是多少
   --sum(score) over (partition by cid order by score )  as `累加分数1`,
   --上句等价于
   --sum(score) over (partition by cid order by score rows between unbounded preceding and current row ) as `累加分数2`,
   --思考：如何计算每个同学，在班级内，包括他自己，和比他低一名的，和比他高1名的，这3个分数求和
   --sum(score) over(partition by cid order by score rows between 1 preceding and 1 following) as `累加分数3`

### 第二类、 排序类的窗口函数

​       --同一个班内，按分数排序打上序号
​       row_number() over (partition by cid order by score)  as `分数序号排名`,
​       --考虑并列
​       rank()       over (partition by cid order by score)  as `分数序号排名2`,
​       dense_rank() over (partition by cid order by score)  as `分数序号排名3`

### 第三类、 偏移类的，跨行的

LAG(col,n,DEFAULT) 用于统计窗口内往上第n行值。

第一个参数为列名，第二个参数为往上第n行（可选，默认为1），第三个参数为默认值（当往上第n行为NULL时候，取默认值，如不指定，则为NULL）

LEAD(col,n,DEFAULT) 用于统计窗口内往下第n行值。

第一个参数为列名，第二个参数为往下第n行（可选，默认为1），第三个参数为默认值（当往下第n行为NULL时候，取默认值，如不指定，则为NULL）

FIRST_VALUE的使用：

取分组内排序后，截止到当前行，第一个值

LAST_VALUE的使用：

取分组内排序后，截止到当前行，最后一个值

一般使用的是 FIRST_VALUE 的倒序取出分组内排序最后一个值

CUME_DIST：小于等于当前值的行数/分组内总行数。  order默认顺序 ：正序

```sql
select *,

       -- lag / lead
       --同一班内，考得比自己低1名的分数是多少
       lag(score, 1) over (partition by cid order by score)     as `低一名的分数`,
       --同一班内，考得比自己低1名的分数是多少,如果找不到，则显示为0
       lag(score, 1, 0) over (partition by cid order by score)  as `低一名的分数2`,
       --同一班内，考得比自己低2名的分数是多少
       lag(score, 2) over (partition by cid order by score)     as `低2名的分数`,
       --同一班内，考得比自己高1名的分数是多少
       lead(score, 1) over (partition by cid order by score)    as `高一名的分数`
from score;
```

案例一：求每位学生的总分，排名，占全体总分的比率

```sql
create table emp(name string , month string, amt int);
insert overwrite table emp values ('张三', '01', 100),
                                  ('李四', '02', 120),
                                  ('王五', '03', 150),
                                  ('赵六', '04', 500),
                                  ('张三', '05', 400),
                                  ('李四', '06', 350),
                                  ('王五', '07', 180),
                                  ('赵六', '08', 400);
select * from emp;
with t1 as (
    select name,
           sum(amt) as sum_amt
        from emp
        group by name
),
    t2 as (
     select *,
            row_number() over (partition by null order by sum_amt desc) as rn, --排名
            sum(sum_amt) over(partition by null) as sum_all--全体总分
      from t1
    )
select *,
       -- || 两根竖线等价于 concat函数
       round(sum_amt*100/sum_all,2)||'%' as rate-- 占比
from t2;
```

案例二：求每年销售员工累计人数

```sql
--跨越速运
drop table if exists emp;
create table emp(empno string ,ename string,hiredate string,sal int ,deptno string);
insert overwrite table emp values
                               ('7521', 'WARD', '1981-2-22', 1250, 30),
                               ('7566', 'JONES', '1981-4-2', 2975, 20),
                               ('7876', 'ADAMS', '1987-7-13', 1100, 20),
                               ('7369', 'SMITH', '1980-12-17', 800, 20),
                               ('7934', 'MILLER', '1982-1-23', 1300, 10),
                               ('7844', 'TURNER', '1981-9-8', 1500, 30),
                               ('7782', 'CLARK', '1981-6-9', 2450, 10),
                               ('7839', 'KING', '1981-11-17', 5000, 10),
                               ('7902', 'FORD', '1981-12-3', 3000, 20),
                               ('7499', 'ALLEN', '1981-2-20', 1600, 30),
                               ('7654', 'MARTIN', '1981-9-28', 1250, 30),
                               ('7900', 'JAMES', '1981-12-3', 950, 30),
                               ('7788', 'SCOTT', '1987-7-13', 3000, 20),
                               ('7698', 'BLAKE', '1981-5-1', 2850, 30);
```

```sql
select *,
       sum(cnt) over(partition by null order by year1) as `累计人数`
from
(select year(hiredate) as year1,
       count(empno) as cnt
from emp
group by year(hiredate)) as t;
```

### 第四类、NTILE的使用

如果数据排序后分为三部分，业务人员只关心其中的一部分，如何将这中间的三分之一数据拿出来呢?NTILE函数即可以满足

ntile可以看成是：把有序的数据集合平均分配到指定的数量（num）个桶中, 将桶号分配给每一行。如果不能平均分配，则优先分配较小编号的桶，并且各个桶中能放的行数最多相差1

然后可以根据桶号，选取前或后 n分之几的数据。数据会完整展示出来，只是给相应的数据打标签；具体要取几分之几的数据，需要再嵌套一层根据标签取出。







## 题目7: 存留率

```sql
create table if not exists tb_cuid_1d
(
    cuid         string comment '用户的唯一标识',
    os           string comment '平台',
    soft_version string comment '版本',
    event_day    string comment '日期',
    visit_time    int comment '用户访问时间戳',
    duration     decimal comment '用户访问时长',
    ext          array<string> comment '扩展字段'
);
insert overwrite table tb_cuid_1d values
                                      (1,'android',1,'2020-04-01',1234567,100,`array`('')),
                                      (1,'android',1,'2020-04-02',1234567,100,`array`('')),
                                      (1,'android',1,'2020-04-08',1234567,100,`array`('')),
                                      (2,'android',1,'2020-04-01',1234567,100,`array`('')),
                                      (3,'android',1,'2020-04-02',1234567,100,`array`(''));
```

--写出用户表 tb_cuid_1d的 20200401 的次日、次7日留存的具体SQL ：
--一条sql统计出以下指标 （4.1号uv，4.1号在4.2号的留存uv，4.1号在4.8号的留存uv）;
--uv就是不同的用户数

--方案1 ，性能快，步骤稍微复杂

```sql
select '2020-04-01的留存情况' as type,
       count(cuid) as uv_4_1,
       count(case when cnt_4_2>0 then 1 else null end) as uv_4_2,
       count(case when cnt_4_8>0 then 1 else null end) as uv_4_8
from
(select cuid,
       count(case when event_day='2020-04-01' then 1 else null end) as cnt_4_1,
       count(case when event_day='2020-04-02' then 1 else null end) as cnt_4_2,
       count(case when event_day='2020-04-08' then 1 else null end) as cnt_4_8
from tb_cuid_1d
where event_day in ('2020-04-01','2020-04-02','2020-04-08')
group by cuid
having cnt_4_1>0) as t;
```

```sql
--方案2 ，性能慢，步骤比较简单
select count(a.cuid) as uv_4_1,
       count(b.cuid) as uv_4_2,
       count(c.cuid) as uv_4_8,
       count(b.cuid)/count(a.cuid) as rate_4_2,
       count(c.cuid)/count(a.cuid) as rate_4_8
from (select distinct cuid from tb_cuid_1d where event_day='2020-04-01') a
left join (select distinct cuid from tb_cuid_1d where event_day='2020-04-02') b on a.cuid=b.cuid
left join (select distinct cuid from tb_cuid_1d where event_day='2020-04-08') c on a.cuid=c.cuid
```

## 题目8: 分组内TopN

```sql
create table emp(empno string ,ename string,hiredate string,sal int ,deptno string);
insert overwrite table emp values
                               ('7521', 'WARD', '1981-2-22', 1250, 30),
                               ('7566', 'JONES', '1981-4-2', 2975, 20),
                               ('7876', 'ADAMS', '1987-7-13', 1100, 20),
                               ('7369', 'SMITH', '1980-12-17', 800, 20),
                               ('7934', 'MILLER', '1982-1-23', 1300, 10),
                               ('7844', 'TURNER', '1981-9-8', 1500, 30),
                               ('7782', 'CLARK', '1981-6-9', 2450, 10),
                               ('7839', 'KING', '1981-11-17', 5000, 10),
                               ('7902', 'FORD', '1981-12-3', 3000, 20),
                               ('7499', 'ALLEN', '1981-2-20', 1600, 30),
                               ('7654', 'MARTIN', '1981-9-28', 1250, 30),
                               ('7900', 'JAMES', '1981-12-3', 950, 30),
                               ('7788', 'SCOTT', '1987-7-13', 3000, 20),
                               ('7698', 'BLAKE', '1981-5-1', 2850, 30);

select * from emp order by deptno,sal desc;
```

--求出每个部门工资最高的前三名员工，并计算这些员工的工资占所属部门总工资的百分比。

```sql
select *,
       round(sal*100/sum_sal)||'%' as rate
from
(select *,
       row_number() over (partition by deptno order by sal desc) as rn,--部门内的薪资排名
       sum(sal)over(partition by deptno) as sum_sal --部门内总薪资
from emp) as t
where rn<=3;
```

## 题目9:join系列

```sql
create table all_users(
                          id int comment '用户id',
                          name string comment '用户姓名',
                          sex string comment '性别',
                          age int comment '年龄'
) comment '银行用户信息表';
insert overwrite table all_users values
                                     (1,'张三','男',20),
                                     (2,'李四','男',29),
                                     (3,'王五','男',21),
                                     (4,'赵六','女',28),
                                     (5,'田七','女',22);
select * from all_users;
create table black_list(
                           user_id int comment '用户编号',
                           type string comment '风控类型'
)comment '银行黑名单信息表';
insert overwrite table black_list values
                                      (1,'诈骗'),
                                      (2,'逾期'),
                                      (3,'套现');
select * from black_list;
```

```sql
-- 观察inner join 的效果
select *
from all_users a
inner join black_list b
on a.id=b.user_id;
```

-- 使用left join对所有用户，如果也在黑名单中，则标记为YES，否则标记为NO。

```sql
set spark.sql.shuffle.partitions=4;
select a.*,
       case  when b.user_id is not null then 'YES'
             else 'NO'
       end as flag
from all_users a
left join black_list b
on a.id=b.user_id;
```

--对所有用户，如果也在黑名单中，则标记为YES，否则标记为NO。
--对上面的问题，使用right join再做一次。
--right join 特点是：

```sql
select a.*,
       if(b.user_id is not null,'YES','NO') as flag
from black_list b
right join all_users a
on b.user_id=a.id;
```

--使用left semi join对所有用户，如果也在黑名单中，则挑选出来。
-- left semi join结果只显示左表字段，不会显示右表字段

```sql
select *
from all_users a
left semi join black_list b on a.id = b.user_id;
```

--他跟left join的关系是:

```sql
select a.*
from all_users a
left join black_list b on a.id = b.user_id
where b.user_id is not null;
```

--也等价于

```sql
select a.*
from all_users a
join black_list b on a.id = b.user_id;
```

-- 使用left anti join对所有用户，如果不在黑名单中，则挑选出来。

```sql
select *
from all_users a
left anti join black_list b on a.id = b.user_id;
```

--他跟left join的关系是:

```sql
select a.*
from all_users a
left join black_list b on a.id = b.user_id
where b.user_id is null;
```

--用户的存款金额。

```sql
drop table if exists deposit;
create table deposit (
 user_id int comment '用户id',
 amount int comment '存款金额'
)comment '用户最新银行存款信息表';
insert overwrite table deposit values
                                   (1,2000),
                                   (2,2900),
                                   (3,2100);
select * from deposit;
```

--用户的负债金额。

```sql
drop table if exists debt;
create table debt (
  user_id int comment '用户id',
  amount int comment '负债金额'
)comment '用户最新银行负债信息表';
insert overwrite table debt values
                                (3,3400),
                                (4,2800),
                                (5,2200);
select * from debt;
```

select coalesce(success_cnt,period,1) from tableA
当success_cnt不为null，那么无论period是否为null，都将返回success_cnt的真实值（因为success_cnt是第一个参数），当success_cnt为null，而period不为null的时候，返回period的真实值。只有当success_cnt和period均为null的时候，将返回1。

```sql
--使用full join，展示用户的存款金额和负债金额。
select coalesce(a.user_id,b.user_id) as user_id_new,
       coalesce(a.amount,0) as deposit_amount,
       coalesce(b.amount,0) as debt_amount
from deposit a
full join debt b on a.user_id=b.user_id;
```

--coalesce返回第一个不为null的值
select coalesce(123,456) as x;
select coalesce(null,456) as x;
select nvl(null,0) as x;
select coalesce(null,null,456) as x;

NVL2(表达式A，表达式B，表达式C）

如果表达式A为空，则返回表达式C的值；如果表达式A不为空，则返回表达式B的值。

求出：2021-01-01到2021-01-07每个服务器ID，角色ID，角色使用付费和免费购买对应道具消耗的免费元宝数量总和，以及对应比率

```sql
create table if not exists dm_paid_buy
(
    `time`    bigint comment '#购买的时间戳',
    server_id string comment '#服务器ID',
    role_id   int comment '#角色ID',
    cost      int comment '#购买对应道具消耗的付费元宝数量',
    item_id   int comment '#购买对应道具的id',
    amount    int comment '#购买对应道具的数量',
    p_date    string comment '#登录日期, yyyy-MM-dd'
) comment '角色使用付费元宝购买商城道具时候记录一条';
insert overwrite table dm_paid_buy values
                                       (1234567,120,10098,2,3,4,'2021-01-01'),
                                       (1234567,120,10098,4,3,5,'2021-01-01'),
                                       (1234567,123,10098,3,3,2,'2021-01-02'),
                                       (1234567,123,10098,2,3,2,'2021-01-02');
                                       
-- 查看表结构
desc dm_paid_buy;

create table if not exists dm_free_buy
(
    `time`    bigint comment '#购买的时间戳',
    server_id string comment '#服务器ID',
    role_id   int comment '#角色ID',
    cost      int comment '#购买对应道具消耗的免费元宝数量',
    item_id   int comment '#购买对应道具的id',
    amount    int comment '#购买对应道具的数量',
    p_date    string comment '#登录日期, yyyy-MM-dd'
) comment '角色使用免费元宝购买商城道具时候记录一条';
insert overwrite table dm_free_buy values
(1234567,123,10098,8,3,4,'2021-01-01'),
(1234567,123,10098,5,3,5,'2021-01-01'),
(1234567,120,10098,6,3,4,'2021-01-01'),
(1234567,120,10098,9,3,5,'2021-01-01'),
(1234567,123,10098,18,3,2,'2021-01-02'),
(1234567,123,10098,7,3,2,'2021-01-02');
```

```sql
--使用with as 来重构上面的逻辑
with a as (
    select p_date,server_id,role_id,
           sum(cost) as cost
    from dm_paid_buy
    where p_date between '2021-01-01' and '2021-01-07'
    group by p_date,server_id,role_id
),
    b as(
        select p_date,server_id,role_id,
               sum(cost) as cost
        from dm_free_buy
        where p_date between '2021-01-01' and '2021-01-07'
        group by p_date,server_id,role_id
    )
select coalesce(a.p_date,b.p_date) as p_date,
       coalesce(a.server_id,b.server_id) as server_id,
       coalesce(a.role_id,b.role_id) as role_id,
       coalesce(a.cost,0) as a_cost,
       coalesce(b.cost,0) as b_cost,
       coalesce(a.cost,0)/coalesce(b.cost,0) as rate
from  a
full join b
on a.p_date=b.p_date and a.server_id=b.server_id and a.role_id=b.role_id
;
```

题目10：截取字符串

使用or，因为or自带去重，而union等价于or，但union all 可以不去重

1、LOCATE(substr , str )：返回子串 substr 在字符串 str 中第一次出现的位置，如果字符substr在字符串str中不存在，则返回0；

2、POSITION(substr IN str )：返回子串 substr 在字符串 str 中第一次出现的位置，如果字符substr在字符串str中不存在，与LOCATE函数作用相同；

3、LEFT(str, length)：从左边开始截取str，length是截取的长度；

4、RIGHT(str, length)：从右边开始截取str，length是截取的长度；

5、SUBSTRING_INDEX(str ,substr ,n)：返回字符substr在str中第n次出现位置之前的字符串;

6、SUBSTRING(str ,n ,m)：返回字符串str从第n个字符截取到第m个字符；

7、REPLACE(str, n, m)：将字符串str中的n字符替换成m字符；

8、LENGTH(str)：计算字符串str的长度

例子：str=[www.wikibt.com](http://www.wikibt.com/)

   substring_index(str,'.',1)

   结果是：www

   substring_index(str,'.',2)

   结果是：www.wikibt

   如果count是正数，那么就是从左往右数，第N个分隔符的左边的所有内容
   如果count是负数，那么就是从右往左数，第N个分隔符的右边的所有内容
   substring_index(str,'.',-2)

   结果为：wikibt.com

   有人会问，如果我要中间的的wikibt怎么办？

   很简单的，两个方向：

   从右数第二个分隔符的右边全部，再从左数的第一个分隔符的左边：

substring_index(substring_index(str,'.',-2),'.',1);

### GROUPING SETS、GROUPING__ID、CUBE、ROLLUP



## SQL 之 case 表达式

CASE表达式 是不依赖于具体数据库的技术，所以可以提高 SQL 代码的可移植性。

**基本格式如下**

```sql
-- 简单 CASE表达式
CASE 列(或表达式)
     WHEN <匹配值1> THEN <表达式>
     WHEN <匹配值2> THEN <表达式>
     ......
     ELSE <表达式>
END

-- 搜索 CASE表达式
CASE WHEN <判断表达式> THEN <表达式>
     WHEN <判断表达式> THEN <表达式>
     WHEN <判断表达式> THEN <表达式>
     ......
     ELSE <表达式>
END


-- 简单 CASE表达式 示例
CASE sex
    WHEN '1' THEN '男'
    WHEN '2' THEN '女'
    ELSE '其他' 
END

-- 搜索CASE表达式 示例
CASE WHEN sex = '1' THEN '男'
     WHEN sex = '2' THEN '女'
     ELSE '其他' 
END
```

CASE表达式 的 ELSE子句 可以省略，但推荐不要省略

### **行转列**

可能我们用的更多的是 IF（MySQL）或 DECODE（Oracle），但这两者都不是标准的 SQL，更推荐大家用 CASE表达式，移植性更高

有如下表，以及如下数据

```sql
CREATE TABLE t_customer_credit (
    id INT(11) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '自增主键',
    login_name VARCHAR(50) NOT NULL COMMENT '登录名',
    credit_type TINYINT(1) NOT NULL COMMENT '额度类型，1:自由资金,2:冻结资金,3:优惠',
    amount DECIMAL(22,6) NOT NULL DEFAULT '0.00000' COMMENT '额度值',
    create_by VARCHAR(50) NOT NULL COMMENT '创建者',
    create_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
    update_by VARCHAR(50) NOT NULL COMMENT '修改者',
  PRIMARY KEY (id)
);
INSERT INTO `t_customer_credit` VALUES (1, 'zhangsan', 1, 550.000000, 'system', '2019-7-7 11:30:09', '2019-7-8 20:21:05', 'system');
INSERT INTO `t_customer_credit` VALUES (2, 'zhangsan', 2, 0.000000, 'system', '2019-7-7 11:30:09', '2019-7-7 11:30:09', 'system');
INSERT INTO `t_customer_credit` VALUES (3, 'zhangsan', 3, 0.000000, 'system', '2019-7-7 11:30:09', '2019-7-7 11:30:09', 'system');
INSERT INTO `t_customer_credit` VALUES (4, 'lisi', 1, 0.000000, 'system', '2019-7-7 11:30:09', '2019-7-7 11:30:09', 'system');
INSERT INTO `t_customer_credit` VALUES (5, 'lisi', 2, 0.000000, 'system', '2019-7-7 11:30:09', '2019-7-7 11:30:09', 'system');
INSERT INTO `t_customer_credit` VALUES (6, 'lisi', 3, 0.000000, 'system', '2019-7-7 11:30:09', '2019-7-7 11:30:09', 'system');
```

如果我们要一行显示用户的三个额度，而不是 3 条记录显示 3 个额度，我们应该怎么做，方式有很多种，这里提供如下 3 种

```sql
-- 1、最容易想到的IF，不具备移植性，不推荐
SELECT login_name,
    MAX(IF(credit_type=1, amount, 0)) freeAmount,
    MAX(IF(credit_type=2, amount, 0)) freezeAmount,
    MAX(IF(credit_type=3, amount, 0)) promotionAmount
FROM t_customer_credit GROUP BY login_name;

-- 2、CASE表达式，标准的 SQL 规范，具备移植性，推荐使用
SELECT login_name,
    MAX(CASE WHEN credit_type = 1 THEN amount ELSE 0 END) freeAmount,
    MAX(CASE WHEN credit_type = 2 THEN amount ELSE 0 END) freezeAmount,
    MAX(CASE WHEN credit_type = 3 THEN amount ELSE 0 END) promotionAmount
FROM t_customer_credit GROUP BY login_name;

-- 3、自连接，数据量大的情况下，结合索引，效率不错，具备移植性
SELECT
    a.login_name,a.amount freeAmount,
    b.amount freezeAmount,
    c.amount promotionAmount
FROM (
    SELECT login_name, amount FROM t_customer_credit WHERE credit_type = 1
)a
LEFT JOIN t_customer_credit b ON a.login_name = b.login_name AND b.credit_type = 2
LEFT JOIN t_customer_credit c ON a.login_name = c.login_name AND c.credit_type = 3;
```

自连接是效率最高的，不管在不在 login_name 上加索引

### **转换统计**

将已有编号方式转换为新的方式并统计

```sql
DROP TABLE t_province_population;
CREATE TABLE t_province_population (
 id tinyint(2) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
 province_name varchar(50) NOT NULL COMMENT '省份名',
 sex tinyint(1) NOT NULL COMMENT '性别，1:男,2:女',
 population int(11) NOT NULL COMMENT '人口数',
 PRIMARY KEY (id)
);
INSERT INTO t_province_population(province_name,sex,population)
VALUES
("黑龙江", 1 ,20),
("黑龙江", 2 ,18),
("内蒙古", 1 ,7),
("内蒙古", 2 ,8),
("海南", 1 ,20),
("海南", 2 ,22),
("西藏", 1 ,8),
("西藏", 2 ,7),
("浙江", 1 ,35),
("浙江", 2 ,35),
("台湾", 1 ,26),
("台湾", 2 ,23),
("河南", 1 ,40),
("河南", 2 ,38),
("湖北", 1 ,27),
("湖北", 2 ,24);
```

需要按各个省所在的位置，统计出东南西北中，各个区域内的人口数量

东：浙江、台湾，西：西藏，南：海南，北：黑龙江、内蒙古，中：湖北、河南

```sql
SELECT CASE province_name
    WHEN '浙江' THEN '东'
    WHEN '台湾' THEN '东'
    WHEN '海南' THEN '南'
    WHEN '西藏' THEN '西'
    WHEN '黑龙江' THEN '北'
    WHEN '内蒙古' THEN '北'
    WHEN '河南' THEN '中'
    WHEN '湖北' THEN '中'
    ELSE '其他' END district,
    SUM(population) populations
FROM t_province_population
GROUP BY district;
```

需要对各个省份做一个人口数级别的统计，统计出各个级别的数量

level_1：population < 20，level_2：20 <= population < 50 ，level_3：50 <= population < 70 ，level_4：>= 70；统计出 level_1 ~ level_4 的数量各有多少

SQL 与执行结果如下

```sql
SELECT 
    CASE WHEN population < 20 THEN 'level_1'
        WHEN population >= 20 AND population < 50 THEN 'level_2'
        WHEN population >= 50 AND population < 70 THEN 'level_3'
        WHEN population >= 70 THEN 'level_4'
        ELSE NULL 
    END pop_level,
    COUNT(*) cnt
FROM (
    SELECT province_name,SUM(population) population FROM t_province_population GROUP BY province_name
)a
GROUP BY pop_level
```

### **条件分支**

**SELECT 条件分支**

还是以上面的 t_province_population 为例，如果我们想要直观的知道各个省份的男、女数量情况

```sql
-- 1、CASE表达式 集合 GROUP BY
SELECT province_name,
    SUM(CASE WHEN sex = 1 THEN population ELSE 0 END) c,
    SUM(CASE WHEN sex = 2 THEN population ELSE 0 END) f_pops
FROM t_province_population
GROUP BY province_name;

-- 2、自关联
SELECT t.province_name, SUM(t.population) m_pops, SUM(a.population) f_pops
FROM t_province_population t
LEFT JOIN t_province_population a
ON t.province_name = a.province_name
WHERE t.sex = 1 AND a.sex = 2;
GROUP BY province_name;
```

**UPDATE 条件分支**

我们有一张薪资表，如下

```sql
CREATE TABLE t_user_salaries(
  id int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '自增主键',
  name varchar(50) NOT NULL COMMENT '姓名',
    sex tinyint(1) NOT NULL COMMENT '性别，1:男,2:女',
  salary int(11) NOT NULL COMMENT '薪资',
  PRIMARY KEY (id)
);
INSERT INTO t_user_salaries(name, sex,salary) VALUES
("张三", 1, 30000),
("李四", 1, 27000),
("王五", 1, 22000),
("菲菲", 2, 24000),
("赵六", 1, 29000);
SELECT * FROM t_user_salaries;
```

假设现在需要根据以下条件对该表的数据进行更新：1、对当前工资为 30000 元以上的员工，降薪 10%，2、对当前工资为 25000 元以上且不满 28000 元的员工，加薪 20%。

```sql
UPDATE t_user_salaries SET salary = 
    CASE WHEN salary >= 30000 THEN salary * 0.9
            WHEN salary >= 25000 AND salary < 28000 THEN salary * 1.2
            ELSE salary
    END;

SELECT * FROM t_user_salaries;
```

### **CHECK 约束**

注意：CHECK 是标准的 SQL，但是 MySQL 却没有实现它，所以 CHECK 在 MySQL 中是不起作用的！

薪资表，假设某个公司有这样一个无理的规定：女性员工的工资不得高于50000

```sql
-- 创建表的时候增加约束
CREATE TABLE t_user_salaries_check(
  name varchar(50) NOT NULL COMMENT '姓名',
    sex tinyint(1) NOT NULL COMMENT '性别，1:男,2:女',
  salary int(11) NOT NULL COMMENT '薪资',
  PRIMARY KEY (id),
    CONSTRAINT chk_sex_salary CHECK(
        CASE WHEN sex = 2 THEN 
                        CASE WHEN salary <= 50000 THEN 1 
                                ELSE 0 
                        END
                ELSE 1 
        END = 1 )
);

-- 若t_user_salaries_check已创建，则补充上约束
ALTER TABLE t_user_salaries_check
ADD CONSTRAINT chk_sex_salary CHECK(
    CASE WHEN sex = 2 THEN 
                        CASE WHEN salary <= 50000 THEN 1 
                                ELSE 0 
                        END
                ELSE 1 
        END = 1 
);
```

## order by 的高级用法

### **ORDER BY返回的是游标而不是集合**

**对于带有排序作用的ORDER BY子句的查询，它返回的是一个对象，其中的行按特定的顺序组织在一起，我们把这种对象称为****游标。**

### **ORDER BY子句是唯一能重用列别名的一步**

这里涉及SQL语句的语法顺序和执行顺序了，我们常见的SQL语法顺序如下：

> SELECT DISTINCT <Top Num> <select list>
> FROM [left_table]
> <join_type> JOIN <right_table>
> ON <join_condition>
> WHERE <where_condition>
> GROUP BY <group_by_list>
> WITH <CUBE | RollUP>
> HAVING <having_condition>
> ORDER BY <order_by_list> 

而数据库引擎在执行SQL语句并不是从SELECT开始执行，而是从FROM开始，具体执行顺序如下(关键字前面的数字代表SQL执行的顺序步骤)：

> (**8**)SELECT (**9**)DISTINCT (**11**)<Top Num> <select list>
> (**1**)FROM [left_table]
> (**3**)<join_type> JOIN <right_table>
> (**2**)    ON <join_condition>
> (**4**)WHERE <where_condition>
> (**5**)GROUP BY <group_by_list>
> (**6**)WITH <CUBE | RollUP>
> (**7**)HAVING <having_condition>
> (**10**)ORDER BY <order_by_list> 



从上面可以看到SELECT在HAVING后才开始执行，这个时候SELECT后面列的别名只对后续的步骤生效，而对SELECT前面的步骤是无效的。所以如果你在WHERE，GROUP BY，或HAVING后面使用列的别名均会报错。

**ORDER BY子句是唯一能重用列别名的一步。**

### **谨慎使用ORDER BY 后面接数字的方式来进行排序**

```sql
SELECT 
姓名 AS Name,
地址 AS Address,
城市 AS City
FROM Customers
ORDER BY 1,2,3
```

ORDER BY后面的数字1,2,3分别代表SELECT后面的第1，第2，第3个字段(也就是Name，Address，City)

### **表表达式不能使用ORDER BY排序**

表表达式包括视图，内联表值函数，派生表(子查询)和公用表表达式(CTE)。

下面的视图是无效的

```sql
CREATE VIEW V_Customers AS
SELECT 
客户ID AS ID,
姓名 AS Name,
地址 AS Address,
城市 AS City
FROM Customers
ORDER BY ID,Name,Address
```

### **T-SQL中表表达式加了TOP可以使用ORDER BY**

```sql
SELECT 
客户ID AS ID,
姓名 AS Name,
地址 AS Address,
城市 AS City
FROM
(SELECT TOP 3 *
FROM Customers
ORDER BY 城市) Customers
ORDER BY ID,Name,Address
```

因为T-SQL中**带有ORDER BY的表表达式加了TOP后返回的是一个没有固定顺序的表**。因此，在这种情况下，ORDER BY子句只是为TOP选项定义逻辑顺序，就是下面这个逻辑子句

```sql
SELECT TOP 3 *
FROM Customers
ORDER BY 城市
```

而不保证结果集的排列顺序，因为表表达式外面至少还有一层才是我们最终需要的结果集。

这里的ORDER BY只对当前的子查询生效，到了主查询是不起作用的。必须在主查询末尾继续添加一个ORDER BY子句才能对结果集生效，就像我们例子中写的那样。

除非逻辑要求，一般情况下并不推荐大家这样巧妙的避开**子查询中不能使用ORDER BY的限制**。

## WITH(NOLOCK)用法

SQL在每次新建一个查询，就相当于创建了一个会话。在不同的查询窗口操作，会影响到其他会话的查询。当某张表正在写数据时，这时候去查询很可能就会一直处于阻塞状态

我们这里使用事务来往某张表里写数据，我们知道事务在写完表必须提交（COMMIT）或回滚（ROLLBACK）才能释放表，否则会一直处于阻塞状态。

使用BEGIN TRAN就可以开始一个事务

事务里的数据虽然还没有提交，但是它实际上已经存在内存里面了，这个时候我们使用NOLOCK查询到的结果，实际上还没存储到硬盘。

从上面的两个测试可以看出，**NOLOCK的作用其实就是为了防止查询时被阻塞，只是这样会产生脏读（未提交的数据）。**

那么一般什么情况下使用NOLOCK呢？

通常是一些被频繁写的表，不管是插入，更新还是删除。这样的表在查询时，使用NOLOCK是非常有效的。

**WITH(NOLOCK)和NOLOCK的区别**

```sql
SELECT * FROM A NOLOCK
SELECT * FROM A (NOLOCK);
SELECT * FROM A WITH(NOLOCK);
```

-  (NOLOCK)这样的写法，NOLOCK其实只是别名的作用，而没有任何实质作用。所以不要粗心将(NOLOCK)写成NOLOCK
-  (NOLOCK)与WITH(NOLOCK)其实功能上是一样的。(NOLOCK)只是WITH(NOLOCK)的别名,但是在SQL Server 2008及以后版本中，(NOLOCK)不推荐使用了，"不借助 WITH 关键字指定表提示”的写法已经过时了。
-  在使用链接服务器的SQL当中，(NOLOCK)不会生效，WITH(NOLOCK)才会生效。

```sql
--这样会提示用错误
select * from [IP].[dbname].dbo.tableName with (nolock)
--这样就可以
select * from [dbname].dbo.tableName with(nolock)
```

