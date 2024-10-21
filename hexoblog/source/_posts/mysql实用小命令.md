---
title: mysql实用小命令
tags: mysql
categories: database
img: /images/mysql.jpg
abbrlink: 24232
date: 2022-12-23 21:14:59
---

## 1.group_concat

在我们平常的工作中，使用`group by`进行分组的场景，是非常多的。

比如想统计出用户表中，名称不同的用户的具体名称有哪些？

具体sql如下：

```sql
select name from `user`
group by name;
```

但如果想把name相同的code拼接在一起，放到另外一列中该怎么办呢？

答：使用`group_concat`函数。

例如：

```sql
select name,group_concat(code) from `user`
group by name;
```

使用`group_concat`函数，可以轻松的把分组后，name相同的数据拼接到一起，组成一个字符串，用`逗号`分隔。

## 2.char_length

有时候我们需要获取字符的`长度`，然后根据字符的长度进行`排序`。

MYSQL给我们提供了一些有用的函数，比如：`char_length`。

通过该函数就能获取字符长度。

获取字符长度并且排序的sql如下：

```
select * from brand where name like '%苏三%' 
order by char_length(name) asc limit 5;
```

name字段使用关键字`模糊查询`之后，再使用`char_length`函数获取name字段的字符长度，然后按长度`升序`。

## 3.locate

有时候我们在查找某个关键字，比如：`苏三`，需要明确知道它在某个字符串中的位置时，该怎么办呢？

答：使用`locate`函数。

使用locate函数改造之后sql如下：

```sql
select * from brand where name like '%苏三%' 
order by char_length(name) asc, locate('苏三',name) asc limit 5,5;
```

先按长度排序，小的排在前面。如果长度相同，则按关键字从左到右进行排序，越靠左的越排在前面。

除此之外，我们还可以使用：`instr`和`position`函数，它们的功能跟`locate`函数类似

## 4.replace

我们经常会有替换字符串中部分内容的需求，比如：将字符串中的字符A替换成B。

这种情况就能使用`replace`函数。

例如：

```sql
update brand set name=REPLACE(name,'A','B') 
where id=1;
```

这样就能轻松实现字符替换功能。

也能用该函数去掉`前后空格`：

```sql
update brand set name=REPLACE(name,' ','') where name like ' %';
update brand set name=REPLACE(name,' ','') where name like '% ';
```

使用该函数还能替换`json格式`的数据内容，真的非常有用。

## 5.now

时间是个好东西，用它可以快速缩小数据范围，我们经常有获取当前时间的需求。

在MYSQL中获取`当前时间`，可以使用`now()`函数，例如：

```sql
select now() from brand limit 1;
```

它会包含`年月日时分秒`。

如果你还想返回`毫秒`，可以使用`now(3)`，例如：

```sql
select now(3) from brand limit 1;
```

## 6.insert into ... select

在工作中很多时候需要`插入数据`。

传统的插入数据的sql是这样的：

```sql
INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`) 
VALUES (5, '108', '苏三', '2022-09-02 19:42:21');
```

它主要是用于插入少量并且已经确定的数据。但如果有大批量的数据需要插入，特别是是需要插入的数据来源于，另外一张表或者多张表的结果集中。

例如：

```sql
INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`) 
select null,code,name,now(3) from `order` where code in ('004','005');
```

这样就能将order表中的部分数据，非常轻松插入到brand表中。

## 7.insert into ... ignore

不知道你有没有遇到过这样的场景：在插入1000个品牌之前，需要先根据name，判断一下是否存在。如果存在，则不插入数据。如果不存在，才需要插入数据。

如果直接这样插入数据：

```sql
INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`) 
VALUES (123, '108', '苏三', now(3));
```

肯定不行，因为brand表的name字段创建了唯一索引，同时该表中已经有一条name等于苏三的数据了。

执行之后直接报错了

当然很多人通过在sql语句后面拼接`not exists`语句，也能达到防止出现重复数据的目的，比如：

```sql
INSERT INTO `brand`(`id`, `code`, `name`, `edit_date`) 
select null,'108', '苏三',now(3) 
from dual where  not exists (select * from `brand` where name='苏三');
```

可以使用`insert into ... ignore`语法。

例如：

```sql
INSERT ignore INTO `brand`(`id`, `code`, `name`, `edit_date`) 
VALUES (123, '108', '苏三', now(3));
```

这样改造之后，如果brand表中没有name为苏三的数据，则可以直接插入成功。

但如果brand表中已经存在name为苏三的数据了，则该sql语句也能正常执行，并不会报错。因为它会忽略异常，返回的执行结果影响行数为0，它不会重复插入数据。

## 8.select ... for update

MYSQL数据库自带了`悲观锁`，它是一种排它锁，根据锁的粒度从大到小分为：`表锁`、`间隙锁`和`行锁`。

在我们的实际业务场景中，有些情况并发量不太高，为了保证数据的正确性，使用悲观锁也可以。

比如：用户扣减积分，用户的操作并不集中。但也要考虑系统自动赠送积分的并发情况，所以有必要加悲观锁限制一下，防止出现积分加错的情况发生。

这时候就可以使用MYSQL中的`select ... for update`语法了。

例如：

```sql
begin;
select * from `user` where id=1 
for update;

//业务逻辑处理

update `user` set score=score-1 where id=1;
commit;
```

这样在一个事务中使用`for update`锁住一行记录，其他事务就不能在该事务提交之前，去更新那一行的数据。

需要注意的是for update前的id条件，必须是表的`主键`或者`唯一索引`，不然行锁可能会失效，有可能变成`表锁`。

## 9.on duplicate key update

是Mysql特有的语法，仅Mysql有效。

**作用：**当执行insert操作时，有已经存在的记录，执行update操作。

通常情况下，我们在插入数据之前，一般会先查询一下，该数据是否存在。如果不存在，则插入数据。如果已存在，则不插入数据，而直接返回结果。

防止重复数据的做法很多，比如：`加唯一索引`、`加分布式锁`等。

但这些方案，都没法做到让第二次请求也更新数据，它们一般会判断已经存在就直接返回了。

这种情况可以使用`on duplicate key update`语法。

该语法会在插入数据之前判断，如果主键或唯一索引不存在，则插入数据。如果主键或唯一索引存在，则执行更新操作。

具体需要更新的字段可以指定，例如：

```sql
INSERT  INTO `brand`(`id`, `code`, `name`, `edit_date`) 
VALUES (123, '108', '苏三', now(3))
on duplicate key update name='苏三',edit_date=now(3);
```

这样一条语句就能轻松搞定需求，既不会产生重复数据，也能更新最新的数据。

但需要注意的是，在高并发的场景下使用`on duplicate key update`语法，可能会存在`死锁`的问题

## 10.show create table

有时候，我们想快速查看某张表的字段情况，通常会使用`desc`命令，比如：

```sql
desc `order`;
```

想看创建了哪些索引，该怎么办呢？

答：使用`show index`命令。

比如：

```sql
show index from `order`;
```

但查看字段和索引数据呈现方式，总觉得有点怪怪的，有没有一种更直观的方式？

答：这就需要使用`show create table`命令了。

例如：

```sql
show create table `order`;
```

在create table时经常会碰到这样的语句，例如：password nvarchar(10)collate chinese_prc_ci_as null，

首先，collate是一个子句，可应用于数据库定义或列定义以定义排序规则，或应用于字符串表达式以应用排序规则投影。语法是collate collation_name

## 11.create table ... select

有时候，我们需要快速备份表。

通常情况下，可以分两步走：

1. 创建一张临时表
2. 将数据插入临时表

创建临时表可以使用命令：

```
create table order_2022121819 like `order`;
```

创建成功之后，就会生成一张名称叫：order_2022121819，表结构跟order一模一样的`新表`，只是该表的`数据为空`而已。

接下来使用命令：

```
insert into order_2022121819 select * from `order`;
```

执行之后就会将order表的数据插入到order_2022121819表中，也就是实现数据备份的功能。

但有没有命令，一个命令就能实现上面这两步的功能呢？

答：用`create table ... select`命令。

例如：

```
create table order_2022121820 
select * from `order`;
```

执行完之后，就会将order_2022121820表创建好，并且将order表中的数据自动插入到新创建的order_2022121820中。

一个命令就能轻松搞定`表备份`。

## 12.explain

很多时候，我们优化一条sql语句的性能，需要查看`索引`执行情况。

答：可以使用`explain`命令，查看mysql的`执行计划`，它会显示`索引的使用情况`。

例如：

```sql
explain select * from `order` where code='002';
```

通过这几列可以判断索引使用情况，执行计划包含列的含义

| id             | select_type | table  | partitions | type       | possible_keys  | key          | key_len      | ref            | rows             | filtered               | Extra    |
| -------------- | ----------- | ------ | ---------- | ---------- | -------------- | ------------ | ------------ | -------------- | ---------------- | ---------------------- | -------- |
| select唯一标识 | select类型  | 表名称 | 比配的分区 | 连接的类型 | 可能的索引选择 | 实际用的索引 | 实际索引长度 | 与索引比较的列 | 预计要检查的行数 | 按表条件过滤的行百分比 | 附加信息 |

​	sql语句没有走索引，排除没有建索引之外，最大的可能性是索引失效了。

下面说说索引失效的常见原因：

| 索                 | 引                   | 失              | 效             | 的                 | 常                 | 见                                                | 原              | 因                     |
| ------------------ | -------------------- | --------------- | -------------- | ------------------ | ------------------ | ------------------------------------------------- | --------------- | ---------------------- |
| 不满足最左前缀原则 | 范围索引列没有放最后 | 使用了select  * | 索引列上有计算 | 索引列上使用了函数 | 字符类型没有加引号 | 用is null和is   not null   没注意字段是否允许为空 | like查询左边有% | 使用or关键字时没有注意 |

## 13.show processlist

有些时候我们线上sql或者数据库出现了问题。比如出现了数据库连接过多问题，或者发现有一条sql语句的执行时间特别长。

可以使用`show processlist`命令查看`当前线程执行情况`。

从执行结果中，我们可以查看当前的连接状态，帮助识别出有问题的查询语句。

- id 线程id
- User 执行sql的账号
- Host 执行sql的数据库的ip和端号
- db 数据库名称
- Command 执行命令，包括：Daemon、Query、Sleep等。
- Time 执行sql所消耗的时间
- State 执行状态
- info 执行信息，里面可能包含sql信息。

如果发现了异常的sql语句，可以直接kill掉，确保数据库不会出现严重的问题。

## 14.mysqldump

有时候我们需要导出MYSQL表中的数据。

这种情况就可以使用`mysqldump`工具，该工具会将数据查出来，转换成insert语句，写入到某个文件中，相当于`数据备份`。

我们获取到该文件，然后执行相应的insert语句，就能创建相关的表，并且写入数据了，这就相当于`数据还原`。

mysqldump命令的语法为：`mysqldump -h主机名 -P端口 -u用户名 -p密码 参数1,参数2.... > 文件名称.sql`

备份远程数据库中的数据库：

```sh
mysqldump -h 192.22.25.226 -u root -p123456 dbname > backup.sql
```







## group by详解

### 1. 使用group by的简单例子

`group by`一般用于**分组统计**，它表达的逻辑就是`根据一定的规则，进行分组`。我们先从一个简单的例子，一起来复习一下哈。

假设用一张员工表，表结构如下：

```sql
CREATE TABLE `staff` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `id_card` varchar(20) NOT NULL COMMENT '身份证号码',
  `name` varchar(64) NOT NULL COMMENT '姓名',
  `age` int(4) NOT NULL COMMENT '年龄',
  `city` varchar(64) NOT NULL COMMENT '城市',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8 COMMENT='员工表';
```

我们现在有这么一个需求：**统计每个城市的员工数量**。对应的 SQL 语句就可以这么写：

```sql
select city ,count(*) as num from staff group by city;
```

### 2. group by 原理分析

#### 2.1 explain 分析

我们先用`explain`查看一下执行计划

```sql
explain select city ,count(*) as num from staff group by city;
```

- Extra 这个字段的`Using temporary`表示在执行分组的时候使用了**临时表**
- Extra 这个字段的`Using filesort`表示使用了**排序**

`group by`怎么就使用到`临时表和排序`了呢？我们来看下这个SQL的执行流程

#### 2.2 group by 的简单执行流程

SQL的执行流程哈

1. 创建内存临时表，表里有两个字段`city`和`num`；
2. 全表扫描`staff`的记录，依次取出city = 'X'的记录。

- 判断**临时表**中是否有为 city='X'的行，没有就插入一个记录 (X,1);
- 如果临时表中有city='X'的行的行，就将x 这一行的num值加 1；

1. 遍历完成后，再根据字段`city`做**排序**，得到结果集返回给客户端。

临时表的排序是怎样的呢？

> 就是把需要排序的字段，放到sort buffer，排完就返回。在这里注意一点哈，排序分**全字段排序**和**rowid排序**
>
> - 如果是`全字段排序`，需要查询返回的字段，都放入`sort buffer`，根据**排序字段**排完，直接返回
> - 如果是`rowid排序`，只是需要排序的字段放入`sort buffer`，然后多一次**回表**操作，再返回。
> - 怎么确定走的是全字段排序还是rowid 排序排序呢？由一个数据库参数控制的，`max_length_for_sort_data`

### 3. where 和 having的区别

- group by + where 的执行流程
- group by + having 的执行流程
- 同时有where、group by 、having的执行顺序

#### 3.1 group by + where 的执行流程

有些小伙伴觉得上一小节的SQL太简单啦，如果加了**where条件**之后，并且where条件列加了索引呢，**执行流程是怎样**？

好的，我们给它加个条件，并且加个`idx_age`的索引，如下：

```sql
select city ,count(*) as num from staff where age> 30 group by city;
//加索引
alter table staff add index idx_age (age);
```

再来expain分析一下：

```sql
explain select city ,count(*) as num from staff where age> 30 group by city;
```

从explain 执行计划结果，可以发现查询条件命中了`idx_age`的索引，并且使用了`临时表和排序`

> **Using index condition**:表示索引下推优化，根据索引尽可能的过滤数据,然后再返回给服务器层根据where其他条件进行过滤。

执行流程如下：

1. 创建内存临时表，表里有两个字段`city`和`num`；
2. 扫描索引树`idx_age`，找到大于年龄大于30的主键ID
3. 通过主键ID，回表找到city = 'X'

- 判断**临时表**中是否有为 city='X'的行，没有就插入一个记录 (X,1);
- 如果临时表中有city='X'的行的行，就将x 这一行的num值加 1；

1. 继续重复2,3步骤，找到所有满足条件的数据，
2. 最后根据字段`city`做**排序**，得到结果集返回给客户端。

#### 3.2 group by + having 的执行

如果你要查询每个城市的员工数量，获取到员工数量不低于3的城市，having可以很好解决你的问题，SQL酱紫写：

```sql
select city ,count(*) as num from staff  group by city having num >= 3;
```

`having`称为分组过滤条件，它对返回的结果集操作。

#### 3.3 同时有where、group by 、having的执行顺序

如果一个SQL同时含有`where、group by、having`子句，执行顺序是怎样的呢。

比如这个SQL：

```sql
select city ,count(*) as num from staff  where age> 19 group by city having num >= 3;
```

1. 执行`where`子句查找符合年龄大于19的员工数据
2. `group by`子句对员工数据，根据城市分组。
3. 对`group by`子句形成的城市组，运行聚集函数计算每一组的员工数量值；
4. 最后用`having`子句选出员工数量大于等于3的城市组。

#### 3.4 where + having 区别总结

- `having`子句用于**分组后筛选**，where子句用于**行**条件筛选
- `having`一般都是配合`group by`和聚合函数一起出现如(`count(),sum(),avg(),max(),min()`)
- `where`条件子句中不能使用聚集函数，而`having`子句就可以。
- `having`只能用在group by之后，where执行在group by之前

### 4. 使用 group by 注意的问题

group by 就是**分组统计**的意思，一般情况都是配合聚合函数`如（count(),sum(),avg(),max(),min())`一起使用。

group by 后面跟的字段一定要出现在select中嘛。

不一定，比如以下SQL：

```
select max(age)  from staff group by city;
```

`group by`使用不当，很容易就会产生慢SQL   问题。因为它既用到**临时表**，又默认用到**排序**。有时候还可能用到**磁盘临时表**。

> - 如果执行过程中，会发现内存临时表大小到达了**上限**（控制这个上限的参数就是`tmp_table_size`），会把**内存临时表转成磁盘临时表**。
> - 如果数据量很大，很可能这个查询需要的磁盘临时表，就会占用大量的磁盘空间

### 5. group by的一些优化方案

从哪些方向去优化呢？

- 方向1：既然它默认会排序，我们不给它排是不是就行啦。
- 方向2：既然临时表是影响group by性能的X因素，我们是不是可以不用临时表？

我们一起来想下，执行`group by`语句为什么需要临时表呢？`group by`的语义逻辑，就是统计不同的值出现的个数。如果这个**这些值一开始就是有序的**，我们是不是直接往下扫描统计就好了，就不用**临时表来记录并统计结果**啦?

- group by 后面的字段加索引
- order by null 不用排序
- 尽量只使用内存临时表
- 使用SQL_BIG_RESULT

#### 5.1 group by 后面的字段加索引

如何保证`group by`后面的字段数值一开始就是有序的呢？当然就是**加索引**啦。

我们回到一下这个SQL

```sql
select city ,count(*) as num from staff where age= 19 group by city;
```

如果我们给它加个联合索引`idx_age_city（age,city）`

```sql
alter table staff add index idx_age_city(age,city);
```

再去看执行计划，发现既不用排序，也不需要临时表啦。

**加合适的索引**是优化`group by`最简单有效的优化方式。

#### 5.2 order by null 不用排序

并不是所有场景都适合加索引的，如果碰上不适合创建索引的场景，我们如何优化呢？

> 如果你的需求并不需要对结果集进行排序，可以使用`order by null`。

```sql
select city ,count(*) as num from staff group by city order by null
```

执行计划如下，已经没有`filesort`啦

#### 5.3 尽量只使用内存临时表

如果`group by`需要统计的数据不多，我们可以尽量只使用**内存临时表**；因为如果group by 的过程因为内存临时表放不下数据，从而用到磁盘临时表的话，是比较耗时的。因此可以适当调大`tmp_table_size`参数，来避免用到**磁盘临时表**。

#### 5.4 使用SQL_BIG_RESULT优化

使用`SQL_BIG_RESULT`这个提示直接用磁盘临时表。MySQl优化器发现，磁盘临时表是B+树存储，存储效率不如数组来得高。因此会直接用数组来存

示例SQl如下：

```sql
select SQL_BIG_RESULT city ,count(*) as num from staff group by city;
```

执行计划的`Extra`字段可以看到，执行没有再使用临时表，而是只有排序

执行流程如下：

1. 初始化 sort_buffer，放入city字段；
2. 扫描表staff，依次取出city的值,存入 sort_buffer 中；
3. 扫描完成后，对 sort_buffer的city字段做排序
4. 排序完成后，就得到了一个有序数组。
5. 根据有序数组，统计每个值出现的次数。