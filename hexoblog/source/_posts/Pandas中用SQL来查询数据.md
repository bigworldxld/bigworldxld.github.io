---
title: Pandas中用SQL来查询数据
date: 2023-03-25 21:43:38
tags: pandas
categories: python
img: /images/pandas.jpg
---

## Pandas中用SQL来查询数据

`Pandas`和`SQL`之间的联用，我们其实也可以在`Pandas`当中使用`SQL`语句来筛选数据，通过`Pandasql`模块来实现该想法

```python
pip install pandasql
```

要是你目前正在使用`jupyter notebook`，也可以这么来下载

```python
!pip install pandasql
```

### 导入数据

我们首先导入数据

```python
import pandas as pd
from pandasql import sqldf
df = pd.read_csv("Dummy_Sales_Data_v1.csv", sep=",")
df.head()
```

数据集做一个初步的探索性分析，

```python
df.info()
```

output

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 9999 entries, 0 to 9998
Data columns (total 12 columns):
 #   Column               Non-Null Count  Dtype  
---  ------               --------------  -----  
 0   OrderID              9999 non-null   int64  
 1   Quantity             9999 non-null   int64  
 2   UnitPrice(USD)       9999 non-null   int64  
 3   Status               9999 non-null   object 
 4   OrderDate            9999 non-null   object 
 5   Product_Category     9963 non-null   object 
 6   Sales_Manager        9999 non-null   object 
 7   Shipping_Cost(USD)   9999 non-null   int64  
 8   Delivery_Time(Days)  9948 non-null   float64
 9   Shipping_Address     9999 non-null   object 
 10  Product_Code         9999 non-null   object 
 11  OrderCode            9999 non-null   int64  
dtypes: float64(1), int64(5), object(6)
memory usage: 937.5+ KB
```

数据集的列名做一个转换，代码如下

```python
df.rename(columns={"Shipping_Cost(USD)":"ShippingCost_USD",
                   "UnitPrice(USD)":"UnitPrice_USD",
                   "Delivery_Time(Days)":"Delivery_Time_Days"},
          inplace=True)
df.info()
```

output

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 9999 entries, 0 to 9998
Data columns (total 12 columns):
 #   Column              Non-Null Count  Dtype  
---  ------              --------------  -----  
 0   OrderID             9999 non-null   int64  
 1   Quantity            9999 non-null   int64  
 2   UnitPrice_USD       9999 non-null   int64  
 3   Status              9999 non-null   object 
 4   OrderDate           9999 non-null   object 
 5   Product_Category    9963 non-null   object 
 6   Sales_Manager       9999 non-null   object 
 7   ShippingCost_USD    9999 non-null   int64  
 8   Delivery_Time_Days  9948 non-null   float64
 9   Shipping_Address    9999 non-null   object 
 10  Product_Code        9999 non-null   object 
 11  OrderCode           9999 non-null   int64  
dtypes: float64(1), int64(5), object(6)
memory usage: 937.5+ KB
```

### 用`SQL`筛选出若干列来

我们先尝试筛选出`OrderID`、`Quantity`、`Sales_Manager`、`Status`等若干列数据，用`SQL`语句应该是这么来写的

```python
SELECT OrderID, Quantity, Sales_Manager, \
Status, Shipping_Address, ShippingCost_USD \
FROM df
```

与`Pandas`模块联用的时候就这么来写

```python
query = "SELECT OrderID, Quantity, Sales_Manager,\
Status, Shipping_Address, ShippingCost_USD \
FROM df"

df_orders = sqldf(query)
df_orders.head()
```

### `SQL`中带`WHERE`条件筛选

我们在`SQL`语句当中添加指定的条件进而来筛选数据，代码如下

```python
query = "SELECT * \
        FROM df_orders \
        WHERE Shipping_Address = 'Kenya'"
        
df_kenya = sqldf(query)
df_kenya.head()
```

而要是条件不止一个，则用AND来连接各个条件，代码如下

```python
query = "SELECT * \
        FROM df_orders \
        WHERE Shipping_Address = 'Kenya' \
        AND Quantity < 40 \
        AND Status IN ('Shipped', 'Delivered')"
df_kenya = sqldf(query)
df_kenya.head()
```

### 分组

同理我们可以调用`SQL`当中的`GROUP BY`来对筛选出来的数据进行分组，代码如下

```python
query = "SELECT Shipping_Address, \
        COUNT(OrderID) AS Orders \
        FROM df_orders \
        GROUP BY Shipping_Address"

df_group = sqldf(query)
df_group.head(10)
```

### 排序

而排序在`SQL`当中则是用`ORDER BY`，代码如下

```python
query = "SELECT Shipping_Address, \
        COUNT(OrderID) AS Orders \
        FROM df_orders \
        GROUP BY Shipping_Address \
        ORDER BY Orders"

df_group = sqldf(query)
df_group.head(10)
```

### 数据合并

我们先创建一个数据集，用于后面两个数据集之间的合并，代码如下

```python
query = "SELECT OrderID,\
        Quantity, \
        Product_Code, \
        Product_Category, \
        UnitPrice_USD \
        FROM df"
df_products = sqldf(query)
df_products.head()
```

两个数据集之间的交集，因此是`INNER JOIN`，代码如下

```python
query = "SELECT T1.OrderID, \
        T1.Shipping_Address, \
        T2.Product_Category \
        FROM df_orders T1\
        INNER JOIN df_products T2\
        ON T1.OrderID = T2.OrderID"

df_combined = sqldf(query)
df_combined.head()
```

### 与`LIMIT`之间的联用

在`SQL`当中的`LIMIT`是用于限制查询结果返回的数量的，我们想看查询结果的前10个，代码如下

```python
query = "SELECT OrderID, Quantity, Sales_Manager, \ 
Status, Shipping_Address, \
ShippingCost_USD FROM df LIMIT 10"

df_orders_limit = sqldf(query)
df_orders_limit
```

## `Pandas`与`SQL`语法不同

### 基础语法

在`SQL`当中，我们用`SELECT`来查找数据，`WHERE`来过滤数据，`DISTINCT`来去重，`LIMIT`来限制输出结果的数量，

输出数据集

```python
## SQL
select * from airports

## Pandas
airports
```

输出数据集的前三行数据，代码如下

```python
## SQL
select * from airports limit 3

## Pandas
airports.head(3)
```

对数据集进行过滤筛查

```py
## SQL
select id from airports where ident = 'KLAX'

## Pandas
airports[airports.ident == 'KLAX'].id
```

对于筛选出来的数据进行去重

```python
## SQL
select distinct type from airport

## Pandas
airports.type.unique()
```

### 多个条件交集来筛选数据

多个条件的交集来筛选数据，代码如下

```python
## SQL
select * from airports 
where iso_region = 'US-CA' and 
type = 'seaplane_base'

## Pandas
airports[(airports.iso_region == 'US-CA') & 
(airports.type == 'seaplane_base')]
```

或者是

```python
## SQL
select ident, name, municipality from airports 
where iso_region = 'US-CA' and
type = 'large_airport'

## Pandas
airports[(airports.iso_region == 'US-CA') &
(airports.type == 'large_airport')][['ident', 'name', 'municipality']]
```

### 排序

在`Pandas`当中默认是对数据进行升序排序，要是我们希望对数据进行降序排序，需要设定`ascending`参数

```python
## SQL*
select * from airport_freq
where airport_ident = 'KLAX'
order by type desc

*## Pandas*
airport_freq[airport_freq.airport_ident == 'KLAX']
.sort_values('type', ascending=False)
```

### 筛选出列表当中的数据

要是我们需要筛选出来的数据在一个列表当中，这里就需要用到`isin()`方法，代码如下

```python
## SQL
select * from airports 
where type in ('heliport', 'balloonport')

## Pandas
airports[airports.type.isin(['heliport', 'balloonport'])]
## SQL
select * from airports 
where type not in ('heliport', 'balloonport')

## Pandas
airports[~airports.type.isin(['heliport', 'balloonport'])]
```

### 删除数据

在`Pandas`当中删除数据用的是`drop()`方法，代码如下

```python
## SQL
delete from dataframe where type = 'MISC'

## Pandas
df.drop(df[df.type == 'MISC'].index)
```

### 更新数据

在`SQL`当中更新数据使用的是`update`和`set`方法，代码如下

```python
### SQL
update airports set home_link = 'aaa'
where ident == 'KLAX'

### Pandas
airports.loc[airports['ident'] == 'KLAX', 'home_link'] = 'aaa'
```

### 调用统计函数

对于给定的数据集，如下图所示

```python
runways.head()
```

我们调用`min()`、`max()`、`mean()`以及`median()`函数作用于`length_ft`这一列上面，代码如下

```python
## SQL
select max(length_ft), min(length_ft),
avg(length_ft), median(length_ft) from runways

## Pandas
runways.agg({'length_ft': ['min', 'max', 'mean', 'median']})
```

### 合并两表格

在`Pandas`当中合并表格用的是`pd.concat()`方法，在`SQL`当中则是`UNION ALL`，代码如下

```python
## SQL
select name, municipality from airports
where ident = 'KLAX'
union all
select name, municipality from airports
where ident = 'KLGB'

## Pandas
pd.concat([airports[airports.ident == 'KLAX'][['name', 'municipality']],
airports[airports.ident == 'KLGB'][['name', 'municipality']]])
```

### 分组

顾名思义也就是`groupby()`方法，代码如下

```python
## SQL
select iso_country, type, count(*) from airports
group by iso_country, type
order by iso_country, type

## Pandas
airports.groupby(['iso_country', 'type']).size()
```

### 分组之后再做筛选

在`Pandas`当中是在进行了`groupby()`之后调用`filter()`方法，而在`SQL`当中则是调用`HAVING`方法，代码如下

```python
## SQL
select type, count(*) from airports
where iso_country = 'US'
group by type
having count(*) > 1000
order by count(*) desc

## Pandas
airports[airports.iso_country == 'US']
.groupby('type')
.filter(lambda g: len(g) > 1000)
.groupby('type')
.size()
.sort_values(ascending=False)
```

### TOP N records

```python
## SQL 
select 列名 from 表名
order by size
desc limit 10

## Pandas
表名.nlargest(10, columns='列名')
```

## 数据分析神器：Polars

`Polars`，它在数据处理的速度上更快，当然里面还包括两种API，一种是`Eager API`，另一种则是`Lazy API`，其中`Eager API`和`Pandas`的使用类似，语法类似差不太多，立即执行就能产生结果

而`Lazy API`和`Spark`很相似，会有并行以及对查询逻辑优化的操作

### 模块的安装与导入

我们先来进行模块的安装，使用`pip`命令

```python
pip install polars
```

在安装成功之后，我们分别用`Pandas`和`Polars`来读取数据，看一下各自性能上的差异，我们导入会要用到的模块

```python
import pandas as pd
import polars as pl
import matplotlib.pyplot as plt
%matplotlib inline
```

使用%[matplotlib](https://so.csdn.net/so/search?q=matplotlib&spm=1001.2101.3001.7020)命令可以将matplotlib的图表直接嵌入到Notebook之中，或者使用指定的界面库显示图表，它有一个参数指定matplotlib图表的显示方式。inline表示将图表嵌入到Notebook中。

Python提供了许多魔法命令，使得在IPython环境中的操作更加得心应手。魔法命令都以%或者%%开头，以%开头的成为行命令，%%开头的称为单元命令。行命令只对命令所在的行有效，而单元命令则必须出现在单元的第一行，对整个单元的代码进行处理

%matplotlib inline 可以在Ipython编译器里直接使用，功能是可以内嵌绘图，并且可以省略掉plt.show()这一步。

### 用`Pandas`读取文件

本次使用的数据集是某网站注册用户的用户名数据，总共有360MB大小，我们先用`Pandas`模块来读取该`csv`文件

```python
%%time 
df = pd.read_csv("users.csv")
df.head()
```

可以看到用`Pandas`读取`CSV`文件总共花费了12秒的时间，数据集总共有两列，一列是用户名称，以及用户名称重复的次数“n”，我们来对数据集进行排序，调用的是`sort_values()`方法，代码如下

```python
%%time 
df.sort_values("n", ascending=False).head()
```

### 用`Polars`来读取操作文件

下面我们用`Polars`模块来读取并操作文件，看看所需要的多久的时间，代码如下

```python
%%time 
data = pl.read_csv("users.csv")
data.head()
```

可以看到用`polars`模块来读取数据仅仅只花费了730毫秒的时间，可以说是快了不少的，我们根据“n”这一列来对数据集进行排序，代码如下

```python
%%time
data.sort(by="n", reverse=True).head()
```

对数据集进行排序所消耗的时间为1.39秒，接下来我们用polars模块来对数据集进行一个初步的探索性分析，数据集总共有哪些列、列名都有哪些，我们还是以熟知“泰坦尼克号”数据集为例

```python
df_titanic = pd.read_csv("titanic.csv")
df_titanic.columns
```

output

```
['PassengerId',
 'Survived',
 'Pclass',
 'Name',
 'Sex',
 'Age',
 ......]
```

和`Pandas`一样输出列名调用的是`columns`方法，然后我们来看一下数据集总共是有几行几列的，

```python
df_titanic.shape
```

output

```
(891, 12)
```

看一下数据集中每一列的数据类型

```python
df_titanic.dtypes
```

output

```
[polars.datatypes.Int64,
 polars.datatypes.Int64,
 polars.datatypes.Int64,
 polars.datatypes.Utf8,
 polars.datatypes.Utf8,
 polars.datatypes.Float64,
......]
```

### 填充空值与数据的统计分析

我们来看一下数据集当中空值的分布情况，调用`null_count()`方法

```python
df_titanic.null_count()
```

相当于pandas的df.isnull().sum()

我们可以看到“Age”以及“Cabin”两列存在着空值，我们可以尝试用平均值来进行填充，代码如下

```python
df_titanic["Age"] = df_titanic["Age"].fill_nan(df_titanic["Age"].mean())
```

计算某一列的平均值只需要调用`mean()`方法即可，那么中位数、最大/最小值的计算也是同样的道理，代码如下

```python
print(f'Median Age: {df_titanic["Age"].median()}')
print(f'Average Age: {df_titanic["Age"].mean()}')
print(f'Maximum Age: {df_titanic["Age"].max()}')
print(f'Minimum Age: {df_titanic["Age"].min()}')
```

output

```
Median Age: 29.69911764705882
Average Age: 29.699117647058817
Maximum Age: 80.0
Minimum Age: 0.42
```

### 数据的筛选与可视化

我们筛选出年龄大于40岁的乘客有哪些，代码如下

```python
df_titanic[df_titanic["Age"] > 40]
```

绘制一张箱形图表，代码如下

```python
fig, ax = plt.subplots(figsize=(10, 5))
ax.boxplot(df_titanic["Age"])
plt.xticks(rotation=90)
plt.xlabel('Age Column')
plt.ylabel('Age')
plt.show()
```

参考链接：[polars官网](https://www.pola.rs/)











