---
title: pgsql表继承和分区
date: 2023-03-09 15:27:37
categories: database
img: /images/pgsql分区.jpg
tags: pgsql
---

## PostgreSQL表继承和分区

### **一、表的继承：**

#### 1.第一个继承表：

   **CREATE TABLE** cities (  *--父表*
     name    text,
     population float,
     altitude   int
   );
   **CREATE TABLE** capitals ( *--子表*
     state   char(2)
   ) **INHERITS** (cities);

capitals表继承自cities表的所有属性。在PostgreSQL里，一个表可以从零个或多个其它表中继承属性，而且一个查询既可以引用父表中的所有行，也可以引用父表的所有行加上其所有子表的行，其中后者是缺省行为。

```sql
INSERT INTO cities values('Las Vegas', 1.53, 2174); 
```

--插入父表*
   INSERT 0 1

```sql
INSERT INTO cities values('Mariposa',3.30,1953);
```

   --插入父表*
   INSERT 0 1

```sql
INSERT INTO capitals values('Madison',4.34,845,'WI');
```

*--插入子表*
   INSERT 0 1

```sql
SELECT name, altitude FROM cities WHERE altitude > 500;
```

* --父表和子表的数据均被取出。*
    name   | altitude
   -----------+----------
   Las Vegas |   2174
   Mariposa  |   1953
   Madison  |   845
   (3 rows)

```sql
SELECT name, altitude FROM capitals WHERE altitude > 500;
```

* --只有子表的数据被取出。*
    name  | altitude
   ---------+----------
   Madison |   845
   (1 row)

**查询父表时，会将子表中的数据也显示出来；查询子表时，不会显示父表中的数据**   

如果希望只从父表中提取数据，则需要在SQL中加入ONLY关键字，如：

```sql
SELECT name,altitude FROM ONLY cities WHERE altitude > 500;
```

​    name   | altitude
   -----------+----------
   Las Vegas |   2174
   Mariposa  |   1953
   (2 rows)
  上例中cities前面的"ONLY"关键字表示该查询应该只对cities进行查找而不包括继承级别低于cities的表。许多我们已经讨论过的命令--SELECT，UPDATE和DELETE--支持这个"ONLY"符号。

 在执行整表数据删除时，如果直接truncate父表，此时父表和其所有子表的数据均被删除，如果只是truncate子表，那么其父表的数据将不会变化，只是子表中的数据被清空。

```sql
TRUNCATE TABLE cities; 
```

*--父表和子表的数据均被删除。*
   TRUNCATE TABLE

```sql
SELECT * FROM capitals;
```


   name | population | altitude | state
   ------+------------+----------+-------
   (0 rows)

#### 2、确定数据来源：

   有时候你可能想知道某条记录来自哪个表。在每个表里我们都有一个系统隐含字段**tableoid**，它可以告诉你表的来源：

```sql
SELECT tableoid, name, altitude FROM cities WHERE altitude > 500;
```

   tableoid |  name  | altitude
   ----------+-----------+----------
     16532 | Las Vegas |   2174
     16532 | Mariposa |   1953
     16538 | Madison  |   845
   (3 rows)
   以上的结果只是给出了**tableoid**，仅仅通过该值，我们还是无法看出实际的表名。要完成此操作，我们就需要和系统表**pg_class**进行关联，以通过tableoid字段从该表中提取实际的表名，见以下查询：

```sql
SELECT p.relname, c.name, c.altitude FROM cities c,pg_class p WHERE c.altitude > 500 and c.tableoid = p.oid;
```


   relname |  name  | altitude
   ----------+-----------+----------
   cities  | Las Vegas |   2174
   cities  | Mariposa  |   1953
   capitals | Madison  |   845
   (3 rows)

#### 3、数据插入的注意事项：

   继承并不自动从INSERT或者COPY中向继承级别中的其它表填充数据。在我们的例子里，下面的INSERT语句不会成功：

   我们可能希望数据被传递到capitals表里面去，但是这是不会发生的：INSERT总是插入明确声明的那个表。

```sql
INSERT INTO cities (name, population, altitude, state) VALUES ('New York', NULL, NULL, 'NY');
```

   我们可能希望数据被传递到capitals表里面去，但是这是不会发生的：INSERT总是插入明确声明的那个表。

#### 4、多表继承：

   一个表可以从多个父表继承，这种情况下它拥有父表们的字段的总和。子表中任意定义的字段也会加入其中。如果同一个字段名出现在多个父表中，或者同时出现在父表和子表的定义里，那么这些字段就会被"融合"，这样在子表里面就只有一个这样的字段。要想融合，字段必须是相同的数据类型，否则就会抛出一个错误。融合的字段将会拥有它所继承的字段的所有约束。

```sql
CREATE TABLE parent1 (FirstCol integer);
CREATE TABLE parent2 (FirstCol integer, SecondCol varchar(20));
CREATE TABLE parent3 (FirstCol varchar(200));
```

  *--子表child1将同时继承自parent1和parent2表，而这两个父表中均包含integer类型的FirstCol字段，因此child1可以创建成功。*

```sql
CREATE TABLE child1 (MyCol timestamp) INHERITS (parent1,parent2);
```

   *--子表child2将不会创建成功，因为其两个父表中均包含FirstCol字段，但是它们的类型不相同。*

```sql
CREATE TABLE child2 (MyCol timestamp) INHERITS (parent1,parent3);
```

  *--子表child3同样不会创建成功，因为它和其父表均包含FirstCol字段，但是它们的类型不相同。*

```sql
CREATE TABLE child3 (FirstCol varchar(20)) INHERITS(parent1);
```

#### 5、继承和权限：

   表访问权限并不会自动继承。因此，一个试图访问父表的用户还必须具有访问它的所有子表的权限，或者使用ONLY关键字只从父表中提取数据。在向现有的继承层次添加新的子表的时候，请注意给它赋予所有权限。    
   继承特性的一个严重的局限性是索引(包括唯一约束)和外键约束只施用于单个表，而不包括它们的继承的子表。这一点不管对引用表还是被引用表都是事实，因此在上面的例子里，如果我们声明cities.name为UNIQUE或者是一个PRIMARY KEY，那么也不会阻止capitals表拥有重复了名字的cities数据行。 并且这些重复的行缺省时在查询cities表的时候会显示出来。实际上，缺省时capitals将完全没有唯一约束，因此可能包含带有同名的多个行。你应该给capitals增加唯一约束，但是这样做也不会避免与cities的重复。类似，如果我们声明cities.name REFERENCES某些其它的表，这个约束不会自动广播到capitals。在这种条件下，你可以通过手工给capitals 增加同样的REFERENCES约束来做到这点。

注：1.父表的检查约束和非空约束会被子表继承。其他约束（如唯一约束，主键，外键）则不会被继承。

​      2.一个子表可以从多个父表继承，当同名字段出现在多个父表中（或者父表和子表中），这些字段会被融合（此时字段类型必须相同，否则会抛出一个错误）。

### **二、分区表：**

目前PostgreSQL支持的分区形式主要为以下两种：
  1). 范围分区: 表被一个或者多个键字字段分区成"范围"，在这些范围之间没有重叠的数值分布到不同的分区里。比如，我们可以为特定的商业对象根据数据范围分区，或者根据标识符范围分区。
   2). 列表分区: 表是通过明确地列出每个分区里应该出现那些键字值实现的。

#### 1、实现分区：

   1). 创建"主表"，所有分区都从它继承。

```sql
 CREATE TABLE measurement (            --主表
        city_id      int    NOT NULL,
        logdate     date  NOT NULL,
        peaktemp int,
    );   
```

 2). 创建几个"子"表，每个都从主表上继承。通常，这些"子"表将不会再增加任何字段。我们将把子表称作分区，尽管它们就是普通的PostgreSQL表。

```sql
CREATE TABLE measurement_yy04mm02 ( ) INHERITS (measurement);
    CREATE TABLE measurement_yy04mm03 ( ) INHERITS (measurement);
    ...
    CREATE TABLE measurement_yy05mm11 ( ) INHERITS (measurement);
    CREATE TABLE measurement_yy05mm12 ( ) INHERITS (measurement);
    CREATE TABLE measurement_yy06mm01 ( ) INHERITS (measurement);
```

 上面创建的子表，均已年、月的形式进行范围划分，不同年月的数据将归属到不同的子表内。这样的实现方式对于清空分区数据而言将极为方便和高效，即直接执行DROP TABLE语句删除相应的子表，之后在根据实际的应用考虑是否重建该子表(分区)。相比于直接DROP子表，PostgreSQL还提供了另外一种更为方便的方式来管理子表：

```sql
 ALTER TABLE measurement_yy06mm01 **NO INHERIT measurement;
```

  和直接DROP相比，该方式仅仅是使子表脱离了原有的主表，而存储在子表中的数据仍然可以得到访问，因为此时该表已经被还原成一个普通的数据表了。这样对于数据库的DBA来说，就可以在此时对该表进行必要的维护操作，如数据清理、归档等，在完成诸多例行性的操作之后，就可以考虑是直接删除该表(DROP TABLE)，还是先清空该表的数据(TRUNCATE TABLE)，之后再让该表重新继承主表，如：

```sql
 ALTER TABLE measurement_yy06mm01 INHERIT measurement;
```

3). 给分区表增加约束，定义每个分区允许的健值。同时需要注意的是，定义的约束要确保在不同的分区里不会有相同的键值。因此，我们需要将上面"子"表的定义修改为以下形式：

```sql
  CREATE TABLE measurement_yy04mm02 (
        CHECK ( logdate >= DATE '2004-02-01' AND logdate < DATE '2004-03-01')
    ) INHERITS (measurement);
    CREATE TABLE measurement_yy04mm03 (
        CHECK (logdate >= DATE '2004-03-01' AND logdate < DATE '2004-04-01')
    ) INHERITS (measurement);
    ...
    CREATE TABLE measurement_yy05mm11 (
        CHECK (logdate >= DATE '2005-11-01' AND logdate < DATE '2005-12-01')
    ) INHERITS (measurement);
    CREATE TABLE measurement_yy05mm12 (
        CHECK (logdate >= DATE '2005-12-01' AND logdate < DATE '2006-01-01')
    ) INHERITS (measurement);
    CREATE TABLE measurement_yy06mm01 (
        CHECK (logdate >= DATE '2006-01-01' AND logdate < DATE '2006-02-01')
    ) INHERITS (measurement);   
```

 4). 尽可能基于键值创建索引。如果需要，我们也同样可以为子表中的其它字段创建索引。

```sql
CREATE INDEX measurement_yy04mm02_logdate ON measurement_yy04mm02 (logdate);
    CREATE INDEX measurement_yy04mm03_logdate ON measurement_yy04mm03 (logdate);
    ...
    CREATE INDEX measurement_yy05mm11_logdate ON measurement_yy05mm11 (logdate);
    CREATE INDEX measurement_yy05mm12_logdate ON measurement_yy05mm12 (logdate);
    CREATE INDEX measurement_yy06mm01_logdate ON measurement_yy06mm01 (logdate);   
```

 5). 定义一个规则或者触发器，把对主表的修改重定向到适当的分区表。
   如果数据只进入最新的分区，我们可以设置一个非常简单的规则来插入数据。我们必须每个月都重新定义这个规则，即修改重定向插入的子表名，这样它总是指向当前分区。

```sql
  CREATE OR REPLACE RULE measurement_current_partition AS
    ON INSERT TO measurement
    DO INSTEAD
    INSERT INTO measurement_yy06mm01 VALUES (NEW.city_id, NEW.logdate, NEW.peaktemp);
```

**其中NEW是关键字，表示新数据字段的集合。这里可以通过点(.)操作符来获取集合中的每一个字段。**
   我们可能想插入数据并且想让服务器自动定位应该向哪个分区插入数据。我们可以用像下面这样的更复杂的规则集来实现这个目标

```sql
 CREATE RULE measurement_insert_yy04mm02 AS
    ON INSERT TO measurement WHERE (logdate >= DATE '2004-02-01' AND logdate < DATE '2004-03-01')
    DO INSTEAD
    INSERT INTO measurement_yy04mm02 VALUES (NEW.city_id, NEW.logdate, NEW.peaktemp);
    ...
    CREATE RULE measurement_insert_yy05mm12 AS
    ON INSERT TO measurement WHERE (logdate >= DATE '2005-12-01' AND logdate < DATE '2006-01-01')
    DO INSTEAD
    INSERT INTO measurement_yy05mm12 VALUES (NEW.city_id, NEW.logdate, NEW.peaktemp);
    CREATE RULE measurement_insert_yy06mm01 AS
    ON INSERT TO measurement WHERE (logdate >= DATE '2006-01-01' AND logdate < DATE '2006-02-01')
    DO INSTEAD
    INSERT INTO measurement_yy06mm01 VALUES (NEW.city_id, NEW.logdate, NEW.peaktemp); 
```

 **请注意每个规则里面的WHERE子句正好匹配其分区的CHECK约束。**
   可以看出，一个复杂的分区方案可能要求相当多的DDL。在上面的例子里我们需要每个月创建一次新分区，因此写一个脚本自动生成需要的DDL是明智的。除此之外，我们还不难推断出，分区表对于新数据的批量插入操作有一定的抑制，这一点在Oracle中也同样如此。

相比于基于Rule的重定向方法，基于触发器的方式可能会带来更好的插入效率，特别是针对非批量插入的情况。

由于Rule的额外开销是基于表的，而不是基于行的，因此效果会好于触发器方式。另一个需要注意的是，copy操作将会忽略Rules，如果我们想要通过COPY方法来插入数据，你只能将数据直接copy到正确的子表，而不是主表。这种限制对于触发器来说是不会造成任何问题的。

基于Rule的重定向方式还存在另外一个问题，就是当插入的数据不在任何子表的约束中时，PostgreSQL也不会报错，而是将数据直接保留在主表中。

#### 2、添加新分区：

   这里将介绍两种添加新分区的方式，第一种方法简单且直观，我们只是创建新的子表，同时为其定义新的检查约束，如：

```sql
CREATE TABLE measurement_y2008m02 (
        CHECK ( logdate >= DATE '2008-02-01' AND logdate < DATE '2008-03-01' )
    ) INHERITS (measurement);
```

 第二种方法的创建步骤相对繁琐，但更为灵活和实用。见以下四步：
   */\* 创建一个独立的数据表(measurement_y2008m02)，该表在创建时以将来的主表(measurement)为模板，包含模板表的缺省值(DEFAULTS)和一致性约束(CONSTRAINTS)。\*/*

```sql
  CREATE TABLE measurement_y2008m02
        (LIKE measurement INCLUDING DEFAULTS INCLUDING CONSTRAINTS);
```

 */\* 为该表创建未来作为子表时需要使用的检查约束。\*/*

```sql
  ALTER TABLE measurement_y2008m02 ADD CONSTRAINT y2008m02
        CHECK (logdate >= DATE '2008-02-01' AND logdate < DATE '2008-03-01');
```

   */\* 导入数据到该表。下面只是给出一种导入数据的方式作为例子。在导入数据之后，如有可能，还可以做进一步的数据处理，如数据转换、过滤等。\*/*

```sql
 copy measurement_y2008m02 from 'measurement_y2008m02'
```

   */\* 在适当的时候，或者说在需要的时候，让该表继承主表。\*/*

```sql
 ALTER TABLE measurement_y2008m02 INHERIT measurement;
```

  7). 确保postgresql.conf里的配置参数constraint_exclusion是打开的。没有这个参数，查询不会按照需要进行优化。这里我们需要做的是确保该选项在配置文件中没有被注释掉。
   */> pwd*
   /opt/PostgreSQL/9.1/data
   */> cat postgresql.conf | grep "constraint_exclusion"*
   constraint_exclusion = partition    # on, off, or partition

#### 3、分区和约束排除：

   约束排除(Constraint exclusion)是一种查询优化技巧，它改进了用上面方法定义的表分区的性能。比如

```sql
   SET constraint_exclusion = on;
    SELECT count(*) FROM measurement WHERE logdate >= DATE '2006-01-01';
```

QUERY PLAN
   \-----------------------------------------------------------------------------------------------
   Aggregate (cost=63.47..63.48 rows=1 width=0)
    -> Append (cost=0.00..60.75 rows=1086 width=0)
       -> Seq Scan on measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)
       -> Seq Scan on measurement_yy06mm01 measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)

```sql
 SET constraint_exclusion = off;
    EXPLAIN SELECT count(*) FROM measurement WHERE logdate >= DATE '2006-01-01';   
```

​        QUERY PLAN

   \-----------------------------------------------------------------------------------------------
   Aggregate (cost=158.66..158.68 rows=1 width=0)
    -> Append (cost=0.00..151.88 rows=2715 width=0)
       -> Seq Scan on measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)
       -> Seq Scan on measurement_yy04mm02 measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)
       -> Seq Scan on measurement_yy04mm03 measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)
   ...
       -> Seq Scan on measurement_yy05mm12 measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)
       -> Seq Scan on measurement_yy06mm01 measurement (cost=0.00..30.38 rows=543 width=0)
          Filter: (logdate >= '2006-01-01'::date)

请注意，约束排除只由CHECK约束驱动，而不会由索引驱动。
   目前版本的PostgreSQL中该配置的缺省值是**partition**，该值是介于on和off之间的一种行为方式，即规划器只会将约束排除应用于基于分区表的查询，而on设置则会为所有查询都进行约束排除

   约束排除在使用时注意事项：
  1). 约束排除只是在查询的WHERE子句包含约束的时候才生效。一个参数化的查询不会被优化，因为在运行时规划器不知道该参数会选择哪个分区。因此像CURRENT_DATE这样的函数必须避免。把分区键值和另外一个表的字段连接起来也不会得到优化。
  2). 在CHECK约束里面要避免跨数据类型的比较

 3). 在主表上的UPDATE和DELETE命令并不执行约束排除。
  4). 在规划器进行约束排除时，主表上的所有分区的所有约束都将会被检查

 5). 在执行ANALYZE语句时，要为每一个分区都执行该命令，而不是仅仅对主表执行该命令