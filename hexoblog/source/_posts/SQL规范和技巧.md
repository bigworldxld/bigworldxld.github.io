---
title: SQL规范和技巧
date: 2023-03-21 11:08:18
tags: SQL
categories: database
img: /images/SQL数据分析题.jpg
---

## 命名规范

### 数据库对象全局命名规范

- 1、命名使用具有意义的英文词汇，词汇中间以下划线分隔
- 2、命名只能使用英文字母、数字、下划线，以英文字母开头
- 3、避免用MySQL的保留字如：backup、call、group等
- 4、所有数据库对象使用小写字母，实际上MySQL中是可以设置大小写是否敏感的，为了保证统一性，我们这边规范全部小写表示。

### 数据库命名规范

- 1、数据库命名尽量不超过30个字符。
- 2、数据库命名一般为项目名称+代表库含义的简写，比如IM项目的工作流数据库，可以是 im_flow。
- 3、数据库创建时必须添加默认字符集和校对规则子句。默认字符集为UTF8（已迁移dumbo的使用utf8mb4）
- 4、命名应使用小写。

### 表命名规范

- 1、常规表表名以t_开头，t代表table的意思，命名规则即 t + 模块（包含模块含义的简写）+ 表（包含表含义的简写），比如用户模块的教育信息表：t_user_eduinfo。
- 2、临时表（RD、QA或DBA同学用于数据临时处理的表），命名规则：temp前缀+模块+表+日期后缀：temp_user_eduinfo_20210719。
- 3、备份表（用于保存和归档历史数据或者作为灾备恢复的数据）命名规则，bak前缀+模块+表+日期后缀：bak_user_eduinfo_20210719。
- 4、同一个模块的表尽可能使用相同的前缀，表名称尽可能表达含义。
- 5、多个单词以下划线 _ 分隔。
- 6、常规表表名尽量不超过30个字符，temp表和bak表视情况而定，也尽量简短为宜，命名应使用小写。

### 字段命名规范

- 1、字段命名需要表示其实际含义的英文单词或简写，单词之间用下划线 _ 进行连接，如 service_ip、service_port。
- 2、各表之间相同意义的字段必须同名，比如a表和b表都有创建时间，应该统一为create_time，不一致会很混乱。
- 3、多个单词以下划线 _ 分隔
- 4、字段名尽量不超过30个字符，命名应该使用小写

### 索引命名规范

- 1、唯一[索引](http://mp.weixin.qq.com/s?__biz=MzI0MDQ4MTM5NQ==&mid=2247487492&idx=2&sn=319be0c2fdd5a89c9464c9f2775ed551&chksm=e91b7518de6cfc0ec75c032b1bdb9b6fe90c505d543c2b45c6d021accd72a8127e6f6b54ec1b&scene=21#wechat_redirect)使用uni + 字段名 来命名：`create unique index uni_uid on t_user_basic(uid)` 。
- 2、非唯一索引使用idx + 字段名 来命名：`create index idx_uname_mobile on t_user_basic(uname,mobile)` 。
- 3、多个单词以下划线 _ 分隔。
- 4、索引名尽量不超过50个字符，命名应该使用小写，组合索引的字段不宜太多，不然也不利于查询效率的提升。
- 5、多单词组成的列名，取尽可能代表意义的缩写，如 test_contact表member_id和friend_id上的组合索引：idx_mid_fid。
- 6、理解组合索引最左前缀原则，避免重复建设索引，如果建立了(a,b,c)，相当于建立了(a), (a,b), (a,b,c)。

### 视图命名规范

- 1、视图名以v开头，表示view，完整结构是v+视图内容含义缩写。
- 2、如果视图只来源单个表，则为v+表名。如果视图由几个表关联产生就用v+下划线（_）连接几个表名，视图名尽量不超过30个字符。如超过30个字符则取简写。
- 3、如无特殊需要，严禁开发人员创建视图。
- 4、命名应使用小写。

### 存储过程命名规范

- 1、存储过程名以sp开头，表示存储过程（storage procedure）。之后多个单词以下划线（_）进行连接。存储过程命名中应体现其功能。存储过程名尽量不能超过30个字符。
- 2、存储过程中的输入参数以i_开头，输出参数以o_开头。
- 3、命名应使用小写。

```sql
create procedure sp_multi_param(in i_id bigint,in i_name varchar(32),out o_memo varchar(100))  
```

### 函数命名规范

- 1、函数名以func开始，表示function。之后多个单词以下划线（_）进行连接，函数命名中应体现其功能。函数名尽量不超过30个字符。
- 2、命名应使用小写。

```
create function func_format_date(ctime datetime)
```

### 触发器命名规范

- 1、触发器以trig开头，表示trigger 触发器。
- 2、基本部分，描述触发器所加的表，触发器名尽量不超过30个字符。
- 3、后缀（_i,_u,_d）,表示触发条件的触发方式（insert,update或delete）。
- 4、命名应使用小写。

```sql
DROP TRIGGER IF EXISTS trig_attach_log_d;
CREATE TRIGGER trig_attach_log_d AFTER DELETE ON t_dept FOR EACH ROW; 
```

### 约束命名规范

- 1、唯一约束：uk_表名称_字段名。uk是UNIQUE KEY的缩写。比如给一个部门的部门名称加上唯一约束，来保证不重名，如下：`ALTER TABLE t_dept ADD CONSTRAINT un_name UNIQUE(name);`
- 2、外键约束：fk_表名，后面紧跟该外键所在的表名和对应的主表名（不含t_）。子表名和父表名用下划线(_)分隔。如下：`ALTER TABLE t_user ADD CONSTRAINT fk_user_dept FOREIGN KEY(depno) REFERENCES t_dept (id);`
- 3、非空约束：如无特殊需要，建议所有字段默认非空(not null)，不同数据类型必须给出默认值(default)。

```
`id` int(11) NOT NULL,
`name` varchar(30) DEFAULT '',
`deptId` int(11) DEFAULT 0,
`salary` float DEFAULT NULL, 
```

- 4、出于性能考虑，如无特殊需要，建议不使用外键。参照完整性由代码控制。这个也是我们普遍的做法，从程序角度进行完整性控制，但是如果不注意，也会产生脏数据。
- 5、命名应使用小写

### 用户命名规范

- 1、生产使用的用户命名格式为 code_应用
- 2、只读用户命名规则为 read_应用

## 设计规范

### 存储引擎的选择

- 1、如无特殊需求，必须使用innodb存储引擎。

可以通过 `show variables like 'default_storage_engine'`来查看当前默认引擎。主要有MyISAM 和 InnoDB，从5.5版本开始默认使用 InnoDB 引擎。

基本的差别为：MyISAM类型不支持事务处理等高级处理，而InnoDB类型支持。MyISAM类型的表强调的是性能，其执行速度比InnoDB类型更快，但是不提供事务支持，而InnoDB提供事务支持以及外部键等高级数据库功能。

### 字符集的选择

- 1、如无特殊要求，必须使用utf8或utf8mb4。

在国内，选择对中文和各语言支持都非常完善的utf8格式是最好的方式，MySQL在5.5之后增加utf8mb4编码，mb4就是most bytes 4的意思，专门用来兼容四字节的unicode。

所以utf8mb4是utf8的超集，除了将编码改为utf8mb4外不需要做其他转换。当然，为了节省空间，一般情况下使用utf8也就够了。

可以使用如下脚本来查看数据库的编码格式

```sql
SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';
-- 或
SHOW VARIABLES Like '%char%'; 
```

### 表设计规范

- 1、不同应用间所对应的数据库表之间的关联应尽可能减少，不允许使用外键对表之间进行关联，确保组件对应的表之间的独立性，为系统或表结构的重构提供可能性。目前业内的做法一般 由程序控制参照完整性。

- 2、表设计的角度不应该针对整个系统进行数据库设计，而应该根据系统架构中组件划分，针对每个组件所处理的业务进行数据库设计。

- 3、表必须要有PK，主键的优势是唯一标识、有效引用、高效检索，所以一般情况下尽量有主键字段。

- 4、一个字段只表示一个含义。

- 5、表不应该有重复列。

- 6、禁止使用复杂数据类型(数组,自定义等)，Json类型的使用视情况而定。

- 7、需要join的字段(连接键)，数据类型必须保持绝对一致，避免隐式转换。比如关联的字段都是int类型。

- 8、设计应至少满足第三范式,尽量减少数据冗余。一些特殊场景允许反范式化设计，但在项目评审时需要对冗余字段的设计给出解释。

- 9、TEXT字段作为大体量文本存储，必须放在独立的表中 , 用PK与主表关联。如无特殊需要，禁止使用TEXT、BLOB字段。

- 10、需要定期删除(或者转移)过期数据的表，通过分表解决，我们的做法是按照2/8法则将操作频率较低的历史数据迁移到历史表中，按照时间或者则曾Id做切割点。

- 11、单表字段数不要太多，建议最多不要大于50个。过度的宽表对性能也是很大的影响。

- 12、MySQL在处理大表时，性能就开始明显降低，所以建议单表物理大小限制在16GB，表中数据行数控制在2000W内。

- - 业内的规则是超过2000W性能开始明显降低。但是这个值是灵活的，你可以根据实际情况进行测试来判断，比如阿里的标准就是500W，百度的确是2000W。实际上是否宽表，单行数据所占用的空间都有起到作用的。

- 13、如果数据量或数据增长在前期规划时就较大，那么在设计评审时就应加入分表策略，数据拆分的做法：垂直拆分、水平拆分；

- 14、无特殊需求，严禁使用分区表

### 字段设计规范

- 1、INT：如无特殊需要，存放整型数字使用UNSIGNED INT型，整型字段后的数字代表显示长度。比如 id int(11) NOT NULL
- 2、DATETIME：所有需要精确到时间(时分秒)的字段均使用DATETIME,不要使用TIMESTAMP类型。

对于TIMESTAMP，它把写入的时间从当前时区转化为UTC（世界标准时间）进行存储。查询时，将其又转化为客户端当前时区进行返回。而对于DATETIME，不做任何改变，基本上是原样输入和输出。

另外DATETIME存储的范围也比较大：

```sql
timestamp所能存储的时间范围为：'1970-01-01 00:00:01.000000' 到 '2038-01-19 03:14:07.999999'。
datetime所能存储的时间范围为：'1000-01-01 00:00:00.000000' 到 '9999-12-31 23:59:59.999999'。
```

但是特殊情况，对于跨时区的业务，TIMESTAMP更为合适。

- 3、VARCHAR：所有动态长度字符串 全部使用VARCHAR类型,类似于状态等有限类别的字段,也使用可以比较明显表示出实际意义的字符串,而不应该使用INT之类的数字来代替；VARCHAR(N)，

N表示的是字符数而不是字节数。比如VARCHAR(255)，可以最大可存储255个字符（字符包括英文字母，汉字，特殊字符等）。但N应尽可能小，因为MySQL一个表中所有的VARCHAR字段最大长度是65535个字节，且存储字符个数由所选字符集决定。

如UTF8存储一个字符最大要3个字节，那么varchar在存放占用3个字节长度的字符时不应超过21845个字符。同时，在进行排序和创建临时表一类的内存操作时，会使用N的长度申请内存。(如无特殊需要，原则上单个varchar型字段不允许超过255个字符)

- 4、TEXT：仅仅当字符数量可能超过20000个的时候,才可以使用TEXT类型来存放字符类数据,因为所有MySQL数据库都会使用UTF8字符集。

所有使用TEXT类型的字段必须和原表进行分拆，与原表主键单独组成另外一个表进行存放，与大文本字段的隔离，目的是。如无特殊需要，不使用MEDIUMTEXT、TEXT、LONGTEXT类型

- 5、对于精确浮点型数据存储，需要使用DECIMAL，严禁使用FLOAT和DOUBLE。
- 6、如无特殊需要，尽量不使用BLOB类型
- 7、如无特殊需要，字段建议使用NOT NULL属性，可用默认值代替NULL
- 8、自增字段类型必须是整型且必须为UNSIGNED，推荐类型为INT或BIGINT，并且自增字段必须是主键或者主键的一部分

### 索引设计规范

- 1、索引区分度

索引必须创建在索引选择性（区分度）较高的列上，选择性的计算方式为:  `selecttivity = count(distinct c_name)/count(*) ;`如果区分度结果小于0.2，则不建议在此列上创建索引，否则大概率会拖慢SQL执行

- 2、遵循最左前缀

对于确定需要组成组合索引的多个字段，设计时建议将选择性高的字段靠前放。使用时，组合索引的首字段，必须在where条件中，且需要按照最左前缀规则去匹配。

- 3、禁止使用外键，可以在程序级别来约束完整性
- 4、Text类型字段如果需要创建索引，必须使用前缀索引
- 5、单张表的索引数量理论上应控制在5个以内。经常有大批量插入、更新操作表，应尽量少建索引，索引建立的原则理论上是多读少写的场景。
- 6、ORDER BY，GROUP BY，DISTINCT的字段需要添加在索引的后面，形成覆盖索引
- 7、正确理解和计算索引字段的区分度，文中有计算规则，区分度高的索引，可以快速得定位数据，区分度太低，无法有效的利用索引，可能需要扫描大量数据页，和不使用索引没什么差别。
- 8、正确理解和计算前缀索引的字段长度，文中有判断规则，合适的长度要保证高的区分度和最恰当的索引存储容量，只有达到最佳状态，才是保证高效率的索引。
- 9、联合索引注意最左匹配原则：必须按照从左到右的顺序匹配，MySQL会一直向右匹配索引直到遇到范围查询(>、<、between、like)然后停止匹配。

```
如：depno=1 and empname>'' and job=1 如果建立(depno,empname,job)顺序的索引，job是用不到索引的。
```

- 10、应需而取策略，查询记录的时候，不要一上来就使用*，只取需要的数据，可能的话尽量只利用索引覆盖，可以减少回表操作，提升效率。
- 11、正确判断是否使用联合索引（上面联合索引的使用那一小节有说明判断规则），也可以进一步分析到索引下推（IPC），减少回表操作，提升效率。
- 12、避免索引失效的原则：禁止对索引字段使用函数、运算符操作，会使索引失效。这是实际上就是需要保证索引所对应字段的”干净度“。
- 13、避免非必要的类型转换，字符串字段使用数值进行比较的时候会导致索引无效。
- 14、模糊查询'%value%'会使索引无效，变为全表扫描，因为无法判断扫描的区间，但是'value%'是可以有效利用索引。
- 15、索引覆盖排序字段，这样可以减少排序步骤，提升查询效率
- 16、尽量的扩展索引，非必要不新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。

举例子：比如一个品牌表，建立的的索引如下，一个主键索引，一个唯一索引

```sql
PRIMARY KEY (`id`),
UNIQUE KEY `uni_brand_define` (`app_id`,`define_id`)
```

当你同事业务代码中的检索语句如下的时候，应该立即警告了，即没有覆盖索引，也没按照最左前缀原则：

```sql
select brand_id,brand_name from  ds_brand_system where status=?  and define_id=?  and app_id=?
```

建议改成如下：

```sql
select brand_id,brand_name from  ds_brand_system where app_id=? and define_id=?  and  status=? 
```

### 约束设计规范

- 1、PK应该是有序并且无意义的，由开发人员自定义，尽可能简短，并且是自增序列。
- 2、表中除PK以外,还存在唯一性约束的,可以在数据库中创建以“uk_”作为前缀的唯一约束索引。
- 3、PK字段不允许更新。
- 4、禁止创建外键约束，外键约束由程序控制。
- 5、如无特殊需要，所有字段必须添加非空约束，即not null。
- 6、如无特殊需要，所有字段必须有默认值。

## 使用规范

### select 检索的规范性

- 1、尽量避免使用`select *，join`语句使用`select *`可能导致只需要访问索引即可完成的查询需要回表取数。

- 一种是可能取出很多不需要的数据，对于宽表来说，这是灾难；一种是尽可能避免回表，因为取一些根本不需要的数据而回表导致性能低下，是很不合算。

- 2、严禁使用 `select * from t_name` ，而不加任何where条件，道理一样，这样会变成全表全字段扫描。

- 3、MySQL中的text类型字段存储：

- - 3.1、不与其他普通字段存放在一起,因为读取效率低，也会影响其他轻量字段存取效率。
  - 3.2、如果不需要text类型字段，又使用了select *，会让该执行消耗大量io，效率也很低下

- 4、在取出字段上可以使用相关函数，但应尽可能避免出现 now() , rand() , sysdate() 等不确定结果的函数，在Where条件中的过滤条件字段上严禁使用任何函数，包括数据类型转换函数。大量的计算和转换会造成效率低下，这个在索引那边也描述过了。

- 5、分页查询语句全部都需要带有排序条件 , 否则很容易引起乱序

- 6、用in()/union替换or，效率会好一些，并注意in的个数小于300

- 7、严禁使用%前缀进行模糊前缀查询:如：`select a,b,c from t_name where a like ‘%name’;`可以使用%模糊后缀查询如：`select a,b from t_name where a like ‘name%’;`

- 8、避免使用子查询，可以把子查询优化为join操作

通常子查询在in子句中，且子查询中为简单SQL(不包含union、group by、order by、limit从句)时，才可以把子查询转化为关联查询进行优化。

### 子查询性能差的原因：

- 子查询的结果集无法使用索引，通常子查询的结果集会被存储到临时表中，不论是内存临时表还是磁盘临时表都不会存在索引，所以查询性能 会受到一定的影响；
- 特别是对于返回结果集比较大的子查询，其对查询性能的影响也就越大；
- 由于子查询会产生大量的临时表也没有索引，所以会消耗过多的CPU和IO资源，产生大量的慢查询。

### 操作的规范性

- 1、禁止使用不含字段列表的INSERT语句

```sql
如：insert into t_name values ('a','b','c');  应使用  insert into t_name(c1,c2,c3) values ('a','b','c'); 。
```

- 2、大批量写操作（`UPDATE、DELETE、INSERT`），需要分批多次进行操作

- - 大批量操作可能会造成严重的主从延迟，特别是主从模式下，大批量操作可能会造成严重的主从延迟，因为需要slave从master的binlog中读取日志来进行数据同步。
  - binlog日志为row格式时会产生大量的日志

## 常用技巧

### 小数转成百分数

我们在写SQL的时候有时候希望将小数转换成百分数显示，可以这样写：

```sql
SELECT CONVERT (
VARCHAR(20),CONVERT ( DECIMAL (18, 2),ROUND(A*100.0/B, 2) )
) + '%' AS Rate
```

代码解释：

ROUND(待四舍五入小数,四舍五入位数)：是四舍五入，但是并不会改变数字的长度。

CONVERT():第一个CONVERT，将四舍五入完的小数截取小数位数，通过DECIMAL(18,2)实现控制小数位数为2

CONVERT():第二个convert，将四舍五入并截取小数位数的数字转化为字符串类型，后加百分号，完成百分比显示

*注意两点：*

- 被除数不为0 
- 除数先转换成浮点型（这里我们使用100.0将2转换为了浮点型）

### **字符串与日期类型转换**

字符串和日期类型一般都可以相互转换，主要是使用CONVERT()函数来进行转换。

将字符串转换为DATETIME格式，

```sql
SELECT CONVERT(DATETIME,'2018-06-26 09:54:30.027');
```

将日期类型转换为字符串

```sql
SELECT CONVERT(VARCHAR(10),'2018-06-26 09:54:30.027',120)
--末尾的120是字符串显示格式的一种参数
```

### **常用字符串处理函数**

**CHARINDEX(SUBSTR,STR)**

返回子串 SUBSTR在字符串 STR中第一次出现的位置，如果字符SUBSTR在字符串STR中不存在，则返回0；

```sql
SELECT CHARINDEX('数据','SQL数据库开发')
--结果：4
```

**LEFT(STR, LENGTH)**   或**RIGHT(STR, LENGTH)**

从左边开始截取STR，LENGTH是截取的长度；

```sql
SELECT LEFT('SQL数据库开发',6)
--结果：SQL数据库
```

**SUBSTRING(STR,N ,M)**

返回字符串STR从第N个字符开始，截取之后的M个字符；

```sql
SELECT SUBSTRING('SQL数据库开发',4,3)
--结果：数据库
```

**REPLACE(STR, STR1, STR2)**

将字符串STR中的STR1字符替换成STR2字符；

```sql
SELECT REPLACE('SQL数据库开发', 'SQL', 'sql')
--结果：sql数据库开发
```

**REVERSE(STR)**

把字符串倒置；

```sql
SELECT REVERSE('SQL数据库开发')
--结果：发开库据数LQS
```

### **复制表**

```sql
--1复制表(只复制表结构,源表名：a 新表名：b)
--方法一 仅用于SQL Server：
select * into b from a where 1<>1
--方法二：
select top 0 * into b from a
```

```sql
--2、拷贝表(拷贝数据)
INSERT INTO TableName1 (field1, field2, field3)
SELECT field4, field5, field6 FROM TableName2
```

*注意：被复制的表的列和复制表的列数据类型需要一致*



**字母大小写的转换**

将大写字母改为小写字母,相反upper()

```sql
UPDATE TableName SET Field = LOWER (Field)
```

### **删除表/数据**

**DELETE FROM TableName**

- 只是删除表中某些数据，表结构还在.。
- DELETE 可以带WHERE子句来删除一部分数据，例如 DELETE FROM Student WHERE Age > 20
- 自动编号不恢复到初始值

**TRUNCATE TABLE TableName**

- TRUNCATE 语句不能跟where条件，无法根据条件来删除，只能全部删除数据。
- 自动编号恢复到初始值。
- 使用TRUNCATE 删除表中所有数据要比DELETE效率高的多，因为TRUNCATE 操作采用按最小方式来记录日志.
- TRUNCATE删除数据，不触发DELETE触发器。

**DROP TABLE  TableName**

- 删除表本身，即表中数据和表结构（列、约束、视图、键）全部删除

### SQL中的集合

SQL Server的集合包括交集（INTERSECT），并集（UNION），差集（EXCEPT）。

集合限制条件

- 子结果集要具有相同的结构。
- 子结果集的列数必须相同
- 子结果集对应的数据类型必须可以兼容。
- 每个子结果集不能包含order by 和 compute子句

**A：UNION 运算符** 
UNION 运算符通过组合其他两个结果表，并消去表中任何重复行而派生出一个结果表。当 ALL 随 UNION 一起使用时（即 UNION ALL），不消除重复行。两种情况下，派生表的每一行不是来自 TABLE1 就是来自 TABLE2。 
**B：EXCEPT 运算符 **EXCEPT运算符通过包括所有在 TABLE1 中但不在 TABLE2 中的行并消除所有重复行而派生出一个结果表。当 ALL 随 EXCEPT 一起使用时 (EXCEPT ALL)，不消除重复行。

**包括所有在 TableA中但不在 TableB和TableC中的行并消除所有重复行而派生出一个结果表**

```sql
(select a from tableA )
except 
(select a from tableB)
except 
(select a from tableC)
```

**C：INTERSECT 运算符**

INTERSECT运算符通过只包括 TABLE1 和 TABLE2 中都有的行并消除所有重复行而派生出一个结果表。当 ALL随 INTERSECT 一起使用时 (INTERSECT ALL)，不消除重复行。 
*注：使用运算词的几个查询结果行必须是一致的。*

取以上两个表的交集，我们可以这样写SQL

```sql
SELECT * FROM  City1
INTERSECT
SELECT * FROM  City2
```

和我们的内连接(INNER JOIN)有点类似，以上SQL也可以这样写

```sql
SELECT c1.* FROM City1 c1
INNER JOIN City2 c2
ON c1.Cno=c2.Cno AND c1.Name=c2.Name
```

**两张关联表，删除主表中已经在副表中没有的信息** 

```sql
delete from table1
where not exists (
select * from table2
where table1.field1=table2.field1
)
```

### 数据库分页

```sql
select top 10 b.*
from (
select top 20 主键字段,排序字段
from 表名 order by 排序字段 desc
) a,
表名 b
where b.主键字段 = a.主键字段
order by a.排序字段具体
```

实现：关于数据库分页：

```sql
declare @start int,@end int
@sql  nvarchar(600)
set @sql=’select top’+str(@end-@start+1)+’+from T
where rid not in(
select top’+str(@str-1)+’Rid from T where Rid>-1)’
exec sp_executesql @sql
```

### **随机取出10条数据**

```sql
select top 10 *
from tablename
order by newid()
```

**列出数据库里所有的表名**

```sql
use master
select name from sysobjects
where type='U' // U代表用户
```

**列出表里的所有的列名**

```sql
use master
select name 
from syscolumns
where id=object_id('TableName')
```

**where 1=1是表示选择全部，where 1=2全部不选**

**收缩数据库**

```sql
--重建索引
DBCC REINDEX
DBCC INDEXDEFRAG
--收缩数据和日志
DBCC SHRINKDB
DBCC SHRINKFILE
```

**压缩数据库**

```sql
dbcc shrinkdatabase(dbname)
```

**检查备份集**

```sql
RESTORE VERIFYONLY from disk='E:\dvbbs.bak'
```

案例：有如下表，要求就裱中所有沒有及格的成績，在每次增長0.1的基礎上，使他們剛好及格:

Name    score

Zhangshan  80

Lishi       59

Wangwu    50

Songquan   69

```sql
while((select min(score) from tb_table)<60)
begin
update tb_table set score =score*1.01
where score<60
if  (select min(score) from tb_table)>60
  break
 else
    continue
end
```

### 尽量避免使用 IN 和 NOT IN

1、效率低

2、容易出现问题，或查询结果有误 （不能更严重的缺点）

以 IN 为例。建两个表：test1 和 test2

```sql
create table test1 (id1 int)
create table test2 (id2 int)

insert into test1 (id1) values (1),(2),(3)
insert into test2 (id2) values (1),(2)
```

我想要查询，在test2中存在的  test1中的id 。使用IN的一般写法是：

```sql
select id1 from test1 
where id1 in (select id2 from test2)
```

结果：

id1

1

2

没问题

但是如果我一时手滑，写成了：

```sql
select id1 from test1 
where id1 in (select id1 from test2)
```

id1

1

2

3

单独查询 `select id1 from test2` 是一定会报错: 消息 207，级别 16，状态 1，第 11 行 列名 'id1' 无效。

然而使用了IN的子查询就是这么敷衍，直接查出 1 2 3

给test2插入一个空值：

```sql
insert into test2 (id2) values (NULL)
```

我想要查询，在test2中不存在的  test1中的id 。

```sql
select id1 from test1 
where id1 not in (select id2 from test2)
```

结果：

id1

空白！我们想要3。为什么会这样呢？

原因是：NULL不等于任何非空的值啊！如果id2只有1和2， 那么3<>1 且 3<>2 所以3输出了，但是 id2包含空值，那么 3也不等于NULL 所以它不会输出。

> 跑题一句：建表的时候最好不要允许含空值，否则问题多多

1、用 EXISTS 或 NOT EXISTS 代替

```sql
select *  from test1 
   where EXISTS (select * from test2  where id2 = id1 )

select *  FROM test1  
 where NOT EXISTS (select * from test2  where id2 = id1 )
```

2、用JOIN 代替

```sql
 select id1 from test1 
   INNER JOIN test2 ON id2 = id1 
   
 select id1 from test1 
   LEFT JOIN test2 ON id2 = id1 
   where id2 IS NULL
```

### 递归CTEs.

递归CTE是引用自己的CTE，就像Python中的递归函数一样。递归CTE尤其有用，它涉及查询组织结构图，文件系统，网页之间的链接图等的分层数据，尤其有用。

递归CTE有3个部分：

- 锚构件：返回CTE的基本结果的初始查询
- 递归成员：引用CTE的递归查询。这是所有与锚构件的联盟
- 停止递归构件的终止条件

以下是获取每个员工ID的管理器ID的递归CTE的示例：

```sql
with RECURSIVE org_structure as (
   SELECT id
          , manager_id
   FROM staff_members
   WHERE manager_id IS NULL
   UNION ALL
   SELECT sm.id
          , sm.manager_id
   FROM staff_members sm
   INNER JOIN org_structure os
      ON os.id = sm.manager_id
    )
```

### 临时函数

编写临时功能是重要的原因：

- 它允许您将代码的块分解为较小的代码块
- 它适用于写入清洁代码
- 它可以防止重复，并允许您重用类似于使用Python中的函数的代码

可以利用临时函数来捕获案例子句。

```sql
CREATE TEMPORARY FUNCTION get_seniority(tenure INT64) AS (
   CASE WHEN tenure < 1 THEN "analyst"
        WHEN tenure BETWEEN 1 and 3 THEN "associate"
        WHEN tenure BETWEEN 3 and 5 THEN "senior"
        WHEN tenure > 5 THEN "vp"
        ELSE "n/a"
   END
);
SELECT name
       , get_seniority(tenure) as seniority
FROM employees
```

### 计算Delta值

将不同时期的值进行比较。例如，本月和上个月的销售之间的三角洲是什么？或者本月和本月去年这个月是什么？

在将不同时段的值进行比较以计算Deltas时，这是Lead（）和LAG（）发挥作用时。

例子：

```sql
-- Comparing each month's sales to last month  
SELECT month  
       , sales  
       , sales - LAG(sales, 1) OVER (ORDER BY month)  
FROM monthly_sales  
-- Comparing each month's sales to the same month last year  
SELECT month  
       , sales  
       , sales - LAG(sales, 12) OVER (ORDER BY month)  
FROM monthly_sales
```

### 单引号、双引号的用法

**插入字符串型**

假如要插入一个名为张红的人，因为是字符串，所以Insert语句中名字两边要加单撇号，数值型可以不加单引号

如：

```sql
Insert into mytable(username) values('张红')
```

如果现在姓名是一个变量thename，则写成

```sql
strsql="Insert into mytable(username) values('" & thename & "')"
```

说明：&改为+号也可以吧，字符串连接

**插入数字型**

假如插入一个年龄为12的记录，要注意数字不用加单撇号

```sql
Insert into mytable(age) values(12)
```

如果现在年龄是一个变量theage，则为：

```sql
strsql="Insert into mytable(age) values(“ & theage & “)"
```

**插入日期型**

日期型和字符串型类似，但是要将单撇号替换为#号。（不过，access数据库中用单撇号也可以）

```sql
strsql=“Insert into mytable(birthday) values(#1980-10-1#)”
```

如果换成日期变量thedate

```sql
strsql=“Insert into mytable(birthday) values(#” & thedate & “#)”
```

### SQL自定义排序

按指定的顺序输出结果，比如按“北京，天津，上海，重庆……”这样的顺序。

这张表的数据是随机录进去的，下面我们希望按照我们指定的顺序输出为如下内容：

注意：这里既没有按照人口的多少排序，也没有按照GDP的多少排序，更加没有按照城市的拼音首字母排序，完全是按照我们自己的意愿进行排序。

**方法一** **ORDER BY CASE WHEN**

通过在ORDER BY的时候，我们对想要的输出顺序使用CASE WHEN，将文本转化为可排序的数字来进行间接排序，具体代码如下：

```sql
SELECT * FROM Citys
ORDER BY 
CASE WHEN City='北京' THEN 1      
     WHEN City='天津' THEN 2
     WHEN City='上海' THEN 3
     WHEN City='重庆' THEN 4
     WHEN City='广州' THEN 5
END
```

**方法二 UNION ALL**

使用UNION ALL的方法容易理解，但是代码会写的比较复杂，具体如下：

```sql
SELECT a.City,a.Population,a.GDP FROM
(
SELECT 1 Num,* FROM Citys WHERE City='北京'
UNION ALL
SELECT 2 Num,* FROM Citys WHERE City='天津'
UNION ALL
SELECT 3 Num,* FROM Citys WHERE City='上海'
UNION ALL
SELECT 4 Num,* FROM Citys WHERE City='重庆'
UNION ALL
SELECT 5 Num,* FROM Citys WHERE City='广州'
) a
ORDER BY a.Num
```

我们通过增加一列自定义的Num，给查询出来的每一行记录赋一个值，这个值是我们输出的顺序，再通过子查询对这个自定义的Num进行排序即可。时常用在比较复杂的查询语句中，且需要自定义排序的场景下。

**方法三 创建临时表**

相比上面两种方法，创建临时表的方法可以极大的减少代码量。我们可以先创建一个按照我们希望输出的顺序的临时表Temp，具体如下，字段有Num,City

当我们需要自定义排序输出时，可以直接关联该临时表，具体代码如下：

```sql
SELECT a.* FROM Citys a
JOIN Temp b ON a.City=b.City
ORDER BY b.Num
```

