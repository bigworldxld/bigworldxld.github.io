---
title: SQL与Excel相关的数据分析
date: 2023-01-14 15:41:07
tags: mysql
categories: database
img: /images/mysql.jpg
---

## Excel一样使用SQL进行数据分析

## **1  重复数据处理**

**查找重复记录**

```sql
SELECT * FROM user 
Where (nick_name,password) in
(
SELECT nick_name,password 
FROM user 
group by nick_name,password 
having count(nick_name)>1
);
```

查找id最大的记录

```sql
SELECT * FROM user 
WHERE id in
(SELECT max(id) FROM user
group by nick_name,password 
having count(nick_name)>1
);
```

删除重复记录

只保留id值最小的记录

```sql
DELETE  c1
FROM  customer c1,customer c2
WHERE c1.cust_email=c2.cust_email
AND c1.id>c2.id;
DELETE FROM user Where (nick_name,password) in
(SELECT nick_name,password FROM
    (SELECT nick_name,password FROM user 
    group by nick_name,password 
    having count(nick_name)>1) as tmp1
)
and id not in
(SELECT id FROM
    (SELECT min(id) id FROM user 
     group by nick_name,password 
     having count(nick_name)>1) as tmp2
);
```

## 2  缺失值处理

**查找缺失值记录**

```sql
SELECT * FROM customer
WHERE cust_email IS NULL;
```

**更新列填充空值**

```sql
UPDATE sale set city = "未知" 
WHERE city IS NULL;

UPDATE orderitems set 
price_new=IFNULL(price_new,5.74);
```

**查询并填充空值列**

```sql
SELECT AVG(price_new) FROM orderitems;

SELECT IFNULL(price_new,5.74) AS bus_ifnull
FROM orderitems;
```

## 3  计算列

**更新表添加计算列**

```sql
ALTER TABLE orderitems ADD price_new DECIMAL(8,2) NOT NULL;

UPDATE orderitems set price_new= item_price*count;
```

查询计算列

```sql
SELECT item_price*count as sales FROM orderitems;
```

## **4  排序**

**多列排序**

```sql
SELECT * FROM orderitems
ORDER BY price_new DESC,quantity;
```

**查询排名前几的记录**

```sql
SELECT * FROM orderitems
ORDER BY price_new DESC LIMIT 5;
```

**查询第10大的值**

```sql
SELECT DISTINCT price_new
FROM orderitems
ORDER BY price_new DESC LIMIT 9,1;
```

**排名**

数值相同的排名相同且排名连续

```sql
SELECT prod_price,
(SELECT COUNT(DISTINCT prod_price)
FROM products
WHERE prod_price>=a.prod_price
) AS rank
FROM products AS a
ORDER BY rank ;
```

## 5 字符串处理

**字符串替换**

```sql
UPDATE data1 SET city=REPLACE(city,'SH','shanghai');

SELECT city FROM data1;
```

**按位置字符串截取**

字符串截取可用于数据分列
MySQL 字符串截取函数：left(), right(), substring(), substring_index()

```sql
SELECT left('example.com', 3);
```

从字符串的第 4 个字符位置开始取，直到结束

```sql
SELECT substring('example.com', 4);
```

从字符串的第 4 个字符位置开始取，只取 2 个字符

```sql
SELECT substring('example.com', 4, 2);
```

**按关键字截取字符串**

取第一个分隔符之前的所有字符，结果是www

```sql
SELECT substring_index('www.google.com','.',1);
```

取倒数第二个分隔符之后的所有字符，结果是google.com;

```sql
SELECT substring_index('www.google.com','.',-2);
```

## **6 筛选**

**通过操作符实现高级筛选**

使用 AND OR IN NOT 等操作符实现高级筛选过滤

```sql
SELECT prod_name，prod_price FROM Products
WHERE vend_id IN('DLL01','BRS01');
SELECT prod_name FROM Products WHERE NOT vend_id='DLL01';
```

**通配符筛选**

常用通配符有% _ [] ^

```sql
SELECT * from customers WHERE country LIKE "CH%";
```

## **7 表联结**

SQL表连接可以实现类似于Excel中的Vlookup函数的功能

```sql
SELECT vend_id,prod_name,prod_price
FROM Vendors INNER JOIN Products
ON Vendors.vend_id=Products.vend_id;

SELECT prod_name,vend_name,prod_price,quantity
FROM OderItems,Products,Vendors
WHERE Products.vend_id=Vendors.vend_id
AND OrderItems.prod_id=Products.prod_id
AND order_num=20007;
--自联结 在一条SELECT语句中多次使用相同的表
SELECT c1.cust_od,c1.cust_name,c1.cust_contact
FROM Customers as c1,Customers as c2
WHERE c1.cust_name=c2.cust_name
AND c2.cust_contact='Jim Jones';
```

## 8 数据透视

数据分组可以实现Excel中数据透视表的功能

**数据分组**

group by 用于数据分组 having 用于分组后数据的过滤

```sql
SELECT order_num,COUNT(*) as items
FROM OrderItems
GROUP BY order_num HAVING COUNT(*)>=3;
```

## 9 交叉表

通过CASE WHEN函数实现

```sql
SELECT data1.city,
CASE WHEN colour = "A" THEN price END AS A,
CASE WHEN colour = "B" THEN price END AS B,
CASE WHEN colour = "C" THEN price END AS C,
CASE WHEN colour = "F" THEN price END AS F
FROM data1
```

## **01. 关联公式:Vlookup**

vlookup是excel几乎最常用的公式，一般用于两个表的关联查询等。所以我先创建一个新表：复制sale表并筛选出地区仅为广州的，命名为sale_guang。

```sql
create table sale_guang
SELECT * from sale where city="广州";
```

需求：根据订单明细号关联两表，并且sale_guang只有订单明细号与利润两列

```sql
SELECT * from sale a
inner JOIN
(SELECT ordernum,profit from sale_guang) b
on a.ordernum=b.ordernum
```

## **02. 对比两列差异**

需求：对比sale的订单明细号与sale_guang订单明细号的差异；

```sql
SELECT * from sale a
WHERE a.ordernum not in 
(SELECT b.ordernum from sale_guang b);
```

## **03. 去除重复值**

需求：去除业务员编码的重复值

```sql
SELECT * FROM sale
where salesnum not in 
(SELECT salesnum from sale
GROUP BY salesman
HAVING COUNT(salesnum)>1)
```

## 04. 缺失值处理

需求：用0填充缺失值或则删除有地区名称缺失值的行。

```
--用0填充：
update sale set city = 0 where city = NULL
--删除有缺失值的行：
delete from sale where city = NULL;
```

## **05. 多条件筛选**

需求：想知道业务员张爱，在北京区域卖的商品订单金额大于等于6000的信息。

```sql
SELECT * from sale
where salesman = "张爱" 
and city = "北京"
and orderaccount >=6000;
```

## **06. 模糊筛选数据**

需求:筛选存货名称含有"三星"或则含有"索尼"的信息。

```sql
SELECT * from sale
where inventoryname like "%三星%" 
or 存货名称 like "%索尼%";
```

## **07. 条件计算**

需求：存货名称含“三星字眼”并且税费高于1000的订单有几个？这些订单的利润总和和平均利润是多少？

```sql
--有多少个？
SELECT COUNT(*) from sale
where inventoryname like "%三星%"
and `tax` > 1000 ;

--这些订单的利润总和和平均利润是多少？
SELECT `ordernum`,SUM(profit),AVG(`profit`)
from sale
where inventoryname like "%三星%"
and `tax` > 1000 
GROUP BY `ordernum`;
```

## **08. 删除数据间的空格**

需求：删除存货名称两边的空格。

```sql
SELECT trim(inventoryname) from sale;
```

**10. 合并与排序列**

需求：计算每个订单号的成本并从高到低排序（成本 = 不含税金额 - 利润）

```sql
SELECT city,ordernum,
(Nontaxamount - profit) as cost 
from sale
order by cost DESC;
```





