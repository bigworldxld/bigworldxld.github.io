---
title: mysql和oracle的区别
date: 2022-06-24 17:26:05
categories: database
tags: 
	- mysql
	- oracle
img: /images/oracle_mysql.jpg
---

## 一、mysql和oracle宏观上的区别：

1、mysql与oracle都是关系型数据库，应用于各种平台。mysql开源免费的，而oracle则是收费的，并且价格非常高。

mysql默认端口：3306，默认用户：root
oracle默认端口：1521，默认用户：system
mysql的安装卸载很简单，oracle很麻烦，安装所用的空间差别也是很大的，mysql安装后差不多一两百兆，而oracle则有3G左右，且使用的时候oracle占用特别大的内存空间和其他机器性能。

mysql登录：

```shell
mysql -hlocalhost -uroot -p密码
```

（h：host、u：user、p：password）
oracle登录：

```shell
sqlplus user_name/password@IP:port/instance_name;
```

（其中可以把IP地址，端口号，实例名写在一个TNS文件中取一个别名，登陆的时候输入这个别名就行了）
初学阶段，图形化工具，mysql可以使用Navicat，Oracle一般用PLSQL，也可以用sqlyog等；

mysql的管理工具较少，在Linux下的管理工具的安装有时需要安装额外的包（phpmyadmin，etc），有一定复杂性。
oracle有多重成熟命令行、图形界面、web管理工具，还有很多第三方的管理工具，管理极其方便高效。
oracle支持大并发，大访问量，是OLTP最好的工具。

## 二、操作区别：

### 1.数据库的层次结构：

mysql：默认用户是root，用户下可以创建好多数据库，每个数据库下还有好多表，一般情况下都是使用默认用户，不会创建多个用户；
oracle：创建一个数据库，数据库下有好多用户：sys、system、scott等，不同用户下有好多表，一般情况下只创建一个数据库用。

### 2.数据库中表字段的类型：

oracle：number（数值型），varchar2,varchar,char (字符型)，date 日期型 等

mysql：int,float,double等数值型，varchar,char字符型，date,datetime,time,year,timestamp等日期型。

其中char(2)这样定义，这个单位在oracle中2代表两个字节，mysql中代表两个字符。

其中varchar在mysql中 必须给长度例如varchar(10) 不然插入的时候出错。

MySQL具有CHAR和VARCHAR，最大长度允许为65,535字节（CHAR最多可以为255字节，VARCHAR为65.535字节）。

而，Oracle支持四种字符类型，即CHAR，NCHAR，VARCHAR2和NVARCHAR2; 所有四种字符类型都需要至少1个字节长; CHAR和NCHAR最大可以是2000个字节，NVARCHAR2和VARCHAR2的最大限制是4000个字节。可能会在最新版本中进行扩展。

### 3.主键递增操作：

oracle:可以借助序列

mysql:利用自增 auto_increment

### 4.单表sql语法：

（a）创建表：

oracle:

```sql
create table t_student(
sid int primary key ,
sname varchar(1) not null ,
enterdate date,
gender char(1),
mail unique, ---唯一约束
age number check (age>19 and age<30) -----检查约束
)
```

mysql:

```sql
create table t_student(
sid int primary key auto_increment,
sname varchar(1) not null ,
enterdate date,
gender char(1),
age int check (age>18 and age<40), ---检查约束，虽然语法可以通过，但是不好使
mail varchar(10) UNIQUE --- 唯一约束
)
```

（b）插入数据：

oracle:

```sql
insert into myuser values (序列名字.nextval,'nana','123','男',to_date('1990-3-4','YYYY-MM-DD'))
```

mysql:

(0)正确写法：null 自增

(1)日期不同 可以直接添加：'1990-3-4'

(2)单位(1)代表一个字符，字母汉字都是一个字符！ 性别 char(2) 代表 两个字符 ：a一个字符 男 一个字符

(3)日期没有sysdate , 要是：sysdate() now() 错误 没有to_date函数 insert into myuser values (NULL,'nana','123','男',to_date('1990-3-4','YYYY-MM-DD'))

(4)可以多条数据一起添加

(5)非空约束，唯一约束，主键约束，都可以，但是 检查约束不好使

下面语法都是可以的：

```sql
insert into t_student values (NULL,'a','1990-3-4','a')
insert into myuser values (NULL,'nana','123','男','1990-3-4')
insert into myuser values (NULL,'nana','123','男','1990/3/4')
insert into myuser values (NULL,'nana','123','男',sysdate())
insert into myuser values (NULL,'nana','123','男',now())
insert into myuser values (NULL,'nana','123','男',sysdate()),(NULL,'nana','123','男',sysdate())
```

（c）删除表

(1)删除表：这里不同

```sql
oracle: delate from myuser ; ---from 可有可无
mysql: delete from myuser; ---必须有from
```

(2)删除整个表：oracle,mysql一样

```sql
drop table myuser ;
```

(3)只删除数据 不删除表 :oracle,mysql一样

```sql
TRUNCATE table myuser ;
```

### 5.多表sql语法：

（1）oracle：创建学生表，班级表，添加外键关联：

--创建学生表：

```sql
create table t_student(
sid number primary key ,
sname varchar2(10),
gender char(3),
classid number
)
```

--创建班级表：

```sql
create table t_class(
cid number primary key,
cname varchar(10)
)
```

----学生表添加数据：

```sql
insert into t_student values (seq_emp.nextval,'lili','男',1);

insert into t_student values (seq_emp.nextval,'nana','男',2);

insert into t_student values (seq_emp.nextval,'feifei','男',3);
```

---班级表添加数据：

```sql
insert into t_class values (1,'java01');
insert into t_class values (2,'java02');
```

--添加学生表的外键约束：

```sql
alter table t_student add constraints fk_student foreign key (classid) references t_class (cid)on delete cascade;
```

（2）mysql创建学生表，班级表，添加外键关联：

\##创建学生表：

```sql
create table t_student(
sid int primary key auto_increment,
sname varchar(10),
gender char(3),
classid int
)
```

\##创建班级表：

```sql
create table t_class(
cid int primary key,
cname varchar(10)
)
```

\##学生表添加数据：

```sql
insert into t_student values (null,'lili','男',1);
insert into t_student values (null,'nana','男',2);
```

\##班级表添加数据：

```sql
insert into t_class values (1,'java01');
insert into t_class values (2,'java02');
```

\##添加学生表的外键约束：

```sql
alter table t_student add constraint fk_student foreign key (classid) references t_class (cid) on delete set null on update CASCADE;
```

注意哪里不同：

创建语法不同

外键约束：oracle是constraints,mysql是constraint级联操作：

oracle：on delete set null 或者on delete cascade

mysql : on delete set null on update CASCADE

更改班级表的主键的时候，学生表外键的值也随之更改

删除班级表的主键记录的时候，学生表外键的值置空

### 6.外连接：

oracle：92语法：可以内连接，外连接

99语法：可以内连接，外连接，全外连接

mysql：只支持 内连接，外连接 ，并且只能用类似oracle中99语法的格式写：

select * from t_class c,t_student s where c.cid(+)=s.classid; （不可以出错）

select * from t_class c right join t_student s on c.cid=s.classid;（只能这样写）

### 7.分页：

（1）oracle分页复杂：需要用到伪劣ROWNUM和嵌套查询

第2页:6-10条记录

```sql
select * from
(select rownum rr,a.* from (select * from emp order by sal) a )
where rr>5 and rr<=10;
```

（2）mysql分页简单一句话:

```sql
select * from help_category order by parent_category_id limit 10,5
```

10,5含义：

记录从0开始，所以其实10代表的是第11条记录，从第11条记录开始，取5个数据

### 8.单引号处理：

mysql里可以用双引号包起字符串，oracle只可以用单引号包起字符串。

### 9.对事务提交：

- mysql默认是自动提交，可以修改为手动提交
- oracle默认不自动提交，需要手动提交，需要在写`commit`指令或点击`commit`按钮。

### 10.对事务的支持：

mysql在innodb存储引擎的夯机所的情况下才支持事务，而oracle则完全支持事务

### 11.日期转换：

- mysql中日期转换用`dateformat()`函数；
- oracle用`to_date()`与`to_char()`两个函数。

### 12.保存数据的持久性：

- mysql默认提交sql语句，但如果更新过程中出现db或主机重启的问题，也许会丢失数据；
- oracle把提交的sql操作先写入了在线联机日志文件中，保持到了硬盘上，可以随时恢复。

### 13.性能诊断

mysql的诊断调优方法较少，主要有慢查询日志；

oracle有各种成熟的性能诊断调优工具，能实现很多自动分析、诊断功能。比如awr、addm、sqltrace、tkproof等。

### 14.逻辑备份：

mysql逻辑备份时要锁定数据，才能保证备份的数据是一致的，影响业务正常的dml使用，oracle逻辑备份时不锁定数据，且备份的数据是一致的。