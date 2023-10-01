---
title: PostgreSQL与MySQL语法对比
date: 2023-01-15 15:42:09
categories: database
img: /images/pgsql和mysql对比.jpg
tags:
	- pgsql
	- mysql
---

## PostgreSQL与MySQL语法对比

PostgreSQL两种分页方法查询

第一种

```sql
SELECT * FROM test_table WHERE i_id>1000 limit 100;
```

第二种

```sql
SELECT * FROM test_table  limit 100 OFFSET 1000;
```

在3000W数据的时候,建议使用第一种.

mysql 的分页就非常简单了

```sql
SELECT * FROM test_table  limit 100, 10;
```

### PostgreSQL的数据类型:

名称	描述	存储大小	范围
smallint	存储整数，小范围	2字节	-32768 至 +32767
integer	存储整数。使用这个类型可存储典型的整数	4字节	-2147483648 至 +2147483647
bigint	存储整数，大范围。	8字节	-9223372036854775808 至 9223372036854775807
decimal	用户指定的精度，精确	变量	小数点前最多为131072个数字; 小数点后最多为16383个数字。
numeric	用户指定的精度，精确	变量	小数点前最多为131072个数字; 小数点后最多为16383个数字。
real	可变精度，不精确	4字节	6位数字精度
double	可变精度，不精确	8字节	15位数字精度
serial	自动递增整数	4字节	1 至 2147483647
bigserial	大的自动递增整数	8字节	1 至 9223372036854775807

**对比Mysql的区别很大,mysql 常用的就tinyint 和 int ,以及bigint** 

### 字符串数据类型

String数据类型用于表示字符串类型值。

数据类型	描述
char(size)	这里size是要存储的字符数。固定长度字符串，右边的空格填充到相等大小的字符。
character(size)	这里size是要存储的字符数。 固定长度字符串。 右边的空格填充到相等大小的字符。
varchar(size)	这里size是要存储的字符数。 可变长度字符串。
character varying(size)	这里size是要存储的字符数。 可变长度字符串。
text	可变长度字符串。
**比Mysql 拥有更加丰富的字符串类型**

### 日期/时间数据类型

日期/时间数据类型用于表示使用日期和时间值的列。

名称	描述	存储大小	最小值	最大值	解析度
timestamp [ (p) ] [不带时区 ]	日期和时间(无时区)	8字节	4713 bc	294276 ad	1微秒/14位数
timestamp [ (p) ]带时区	包括日期和时间，带时区	8字节	4713 bc	294276 ad	 
date	日期(没有时间)	4字节	4713 bc	5874897 ad	1微秒/14位数
time [ (p) ] [ 不带时区 ]	时间(无日期)	8字节	00:00:00	24:00:00	1微秒/14位数
time [ (p) ] 带时区	仅限时间，带时区	12字节	00:00:00+1459	24:00:00-1459	1微秒/14位数
interval [ fields ] [ (p) ]	时间间隔	12字节	-178000000年	178000000年	1微秒/14位数

###  特殊值

PostgreSQL 为方便起见支持几个特殊输入值， 如在 [Table 8-13](http://www.php100.com/manual/PostgreSQL8/datatype-datetime.html#DATATYPE-DATETIME-SPECIAL-TABLE) 里面显示的那样。 值`infinity` 和 `-infinity` 是特别在系统内部表示的，并且将按照同样的方式显示； 但是其它的都只是符号缩写，在读取的时候将被转换成普通的日期/时间值。 （特别是，`now` 和相关的字串在读取的时候就被转换成对应的数值。） 所有这些值在 SQL 命令里当作普通常量对待时，都需要写在单引号里面。

**Table 8-13. 特殊日期/时间输入**

| 输入字串    | 有效类型                    | 描述                                   |
| :---------- | :-------------------------- | :------------------------------------- |
| `epoch`     | `date`, `timestamp`         | 1970-01-01 00:00:00+00 (Unix 系统零时) |
| `infinity`  | `timestamp`                 | 比任何其它时间戳都晚                   |
| `-infinity` | `timestamp`                 | 比任何其它时间戳都早                   |
| `now`       | `date`, `time`, `timestamp` | 当前事务时间                           |
| `today`     | `date`, `timestamp`         | 今日午夜                               |
| `tomorrow`  | `date`, `timestamp`         | 明日午夜                               |
| `yesterday` | `date`, `timestamp`         | 昨日午夜                               |
| `allballs`  | `time`                      | 00:00:00.00 UTC                        |

### 一些其他数据类型

布尔类型：
名称	描述	存储大小
boolean	它指定true或false的状态。	1字节
货币类型：
名称	描述	存储大小	范围
money	货币金额	8字节	-92233720368547758.08 至 +92233720368547758.07
几何类型：
几何数据类型表示二维空间对象。最根本的类型：点 - 形成所有其他类型的基础。

名称	存储大小	表示	描述
point	16字节	在一个平面上的点	(x,y)
line	32字节	无限线(未完全实现)	((x1,y1),(x2,y2))
lseg	32字节	有限线段	((x1,y1),(x2,y2))
box	32字节	矩形框	((x1,y1),(x2,y2))
path	16+16n字节	封闭路径(类似于多边形)	((x1,y1),…)
polygon	40+16n字节	多边形(类似于封闭路径)	((x1,y1),…)
circle	24字节	圆	<(x，y)，r>(中心点和半径)

### 对数据库:

创建数据库:

```sql
CREATE DATABASE database_name;
```

删除数据库:

```sql
drop database database_name;
```

对表:

创建表:

```sql
CREATE TABLE table_name(

column1 datatype,

column2 datatype,

.....

columnN datatype,

PRIMARY KEY( one or more columns )

);
```


删除表:

```sql
drop table table_name;
```

插入:

```sql
INSERT INTO TABLE_NAME (column1, column2, column3,...columnN) VALUES (value1, value2, value3,...valueN);
```

查询:

```sql
SELECT "column1", "column2".."column" FROM "table_name";
```

更新:

```sql
UPDATE table_name SET column1 = value1, column2 = value2...., columnN = valueN WHERE [condition];
```

删除:

```sqlite
DELETE FROM table_name WHERE [condition];
```

**还有 order by / group by /  having / and   /  or  /  like  /  in /  not in    这些和Mysql 的没啥不同,一样用**

### 条件类型

Not

NOT  条件与WHERE子句一起使用以否定查询中的条件。

SELECT column1, column2, ..... columnN 

FROM  table_name

WHERE [search_condition] NOT [condition];

Between

BETWEEN条件与WHERE子句一起使用，以从两个指定条件之间的表中获取数据。

SELECT column1, column2, ..... columnN

FROM table_name

WHERE [search_condition] BETWEEN [condition];

示例:

```sql
SELECT * FROM EMPLOYEES WHERE AGE BETWEEN 24 AND 27;
```

### **事务:**

- `BEGIN TRANSACTION`：开始事务。
- `COMMIT`：保存更改，或者您可以使用`END TRANSACTION`命令。
- `ROLLBACK`：回滚更改。

其实也和mysql 的差不多.语法区别不是很大

{% asset_img pgsql和mysql.png This is an example image %}

## 数据库三范式

**第一范式: 确保每列保持原子性**

​		比如某些数据库系统中需要用到“地址”这个属性，本来直接将“地址”属性设计成一个数据库表的字段就行。但是如果系统经常会访问“地址”属性中的“城市”部分，那么就非要将“地址”这个属性重新拆分为省份、城市、详细地址等多个部分进行存储，这样在对地址中某一部分操作的时候将非常方便。这样设计才算满足了数据库的第一范式

**第二范式: 确保表中的每列都和主键相关**

第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。也就是说在一个数据库表中，一个表中只能保存一种数据，不可以把多种数据保存在同一张数据库表中。

比如要设计一个订单信息表，因为订单中可能会有多种商品，所以要将订单编号和商品编号作为数据库表的联合主键

这样就产生一个问题：这个表中是以订单编号和商品编号作为联合主键。这样在该表中商品名称、单位、商品价格等信息不与该表的主键相关，而仅仅是与商品编号相关。所以在这里违反了第二范式的设计原则。

而如果把这个订单信息表进行拆分，把商品信息分离到另一个表中，把订单项目表也分离到另一个表中，就非常完美了。

**第三范式: 确保每列都和主键列直接相关,而不是间接相关**

比如在设计一个订单数据表的时候，可以将客户编号作为一个外键和订单表建立相应的关系。而不可以在订单表中添加关于客户其它信息（比如姓名、所属公司等）的字段。