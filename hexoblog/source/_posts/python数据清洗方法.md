---
title: python数据清洗方法
tags: python
cover: true
categories: python
img: /images/python.jpg
abbrlink: 35380
date: 2022-06-30 21:02:09
---

## 01 重复值处理

数据录入过程、数据整合过程都可能会产生重复数据，直接删除是重复数据处理的主要方法。
pandas提供查看、处理重复数据的方法duplicated和drop_duplicates。以如下数据为例:


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
```


```python
sample = pd.DataFrame({'id':[1,1,1,3,4,5],
                       'name':['Bob','Bob','Mark','Miki','Sully','Rose'],
                       'city':['广州市','深圳市','上海市','重庆市','贵州市','北京市'],
                       'sex':['女','男','女','女','女','男'],
                       'score':[99,99,87,77,77,np.nan],
                       'group':[1,1,1,2,1,2],})
```

发现重复数据通过duplicated方法完成，如下所示，可以通过该方法查看重复的数据。


```python
sample[sample.duplicated()]

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>group</th>
      <th>id</th>
      <th>name</th>
      <th>score</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>

</div>



需要去重时，可drop_duplicates方法完成：



```python
sample.drop_duplicates() 
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>group</th>
      <th>id</th>
      <th>name</th>
      <th>score</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>广州市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>1</th>
      <td>深圳市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>男</td>
    </tr>
    <tr>
      <th>2</th>
      <td>上海市</td>
      <td>1</td>
      <td>1</td>
      <td>Mark</td>
      <td>87.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>3</th>
      <td>重庆市</td>
      <td>2</td>
      <td>3</td>
      <td>Miki</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>4</th>
      <td>贵州市</td>
      <td>1</td>
      <td>4</td>
      <td>Sully</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>5</th>
      <td>北京市</td>
      <td>2</td>
      <td>5</td>
      <td>Rose</td>
      <td>NaN</td>
      <td>男</td>
    </tr>
  </tbody>
</table>

</div>



drop_duplicates方法还可以按照某列去重，例如去除id列重复的所有记录：


```python
sample.drop_duplicates('id')
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>group</th>
      <th>id</th>
      <th>name</th>
      <th>score</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>广州市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>3</th>
      <td>重庆市</td>
      <td>2</td>
      <td>3</td>
      <td>Miki</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>4</th>
      <td>贵州市</td>
      <td>1</td>
      <td>4</td>
      <td>Sully</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>5</th>
      <td>北京市</td>
      <td>2</td>
      <td>5</td>
      <td>Rose</td>
      <td>NaN</td>
      <td>男</td>
    </tr>
  </tbody>
</table>

</div>



## 02 缺失值处理

缺失值是数据清洗中比较常见的问题，缺失值一般由NA表示，在处理缺失值时要遵循一定的原则。

首先，需要根据业务理解处理缺失值，弄清楚缺失值产生的原因是故意缺失还是随机缺失，再通过一些业务经验进行填补。一般来说当缺失值少于20%时，连续变量可以使用均值或中位数填补；分类变量不需要填补，单算一类即可，或者也可以用众数填补分类变量。

当缺失值处于20%-80%之间时，填补方法同上。另外每个有缺失值的变量可以生成一个指示哑变量，参与后续的建模。当缺失值多于80%时，每个有缺失值的变量生成一个指示哑变量，参与后续的建模，不使用原始变量。

### 1. 查看缺失情况

在进行数据分析前，一般需要了解数据的缺失情况，在Python中可以构造一个lambda函数来查看缺失值，该lambda函数中，sum(col.isnull())表示当前列有多少缺失，col.size表示当前列总共多少行数据：




```python
sample.apply(lambda col:sum(col.isnull())/col.size)
------------
miss = df.isnull()
df[miss.any(axis=1)==True].head(3)#任意一列都是True,等价于all
```




    city     0.000000
    group    0.000000
    id       0.000000
    name     0.000000
    score    0.166667
    sex      0.000000
    dtype: float64



### 2. 以指定值填补

pandas数据框提供了fillna方法完成对缺失值的填补，例如对sample表的列score填补缺失值，填补方法为均值,还可以以分位数等方法进行填补：


```python
sample.score.fillna(sample.score.mean())
```




    0    99.0
    1    99.0
    2    87.0
    3    77.0
    4    77.0
    5    87.8
    Name: score, dtype: float64




```python
sample.score.fillna(sample.score.median())
```




    0    99.0
    1    99.0
    2    87.0
    3    77.0
    4    77.0
    5    87.0
    Name: score, dtype: float64



也可以用pandas中的ffill函数对缺失值进行前向填补，但在前向填补时需要注意各个列数据的情况,
体重列的第一行未填补完成，而pandas中提供了bfill函数进行后向填补：


```python
sample.ffill()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>group</th>
      <th>id</th>
      <th>name</th>
      <th>score</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>广州市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>1</th>
      <td>深圳市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>男</td>
    </tr>
    <tr>
      <th>2</th>
      <td>上海市</td>
      <td>1</td>
      <td>1</td>
      <td>Mark</td>
      <td>87.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>3</th>
      <td>重庆市</td>
      <td>2</td>
      <td>3</td>
      <td>Miki</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>4</th>
      <td>贵州市</td>
      <td>1</td>
      <td>4</td>
      <td>Sully</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>5</th>
      <td>北京市</td>
      <td>2</td>
      <td>5</td>
      <td>Rose</td>
      <td>77.0</td>
      <td>男</td>
    </tr>
  </tbody>
</table>

</div>



### 3. 缺失值指示变量

pandas数据框对象可以直接调用方法isnull产生缺失值指示变量，例如产生score变量的缺失值指示变量：
若想转换为数值0，1型指示变量，可以使用apply方法，int表示将该列替换为int类型。


```python
sample.score.isnull()
```




    0    False
    1    False
    2    False
    3    False
    4    False
    5     True
    Name: score, dtype: bool




```python
sample.score.isnull().apply(int)
```




    0    0
    1    0
    2    0
    3    0
    4    0
    5    1
    Name: score, dtype: int64



### 4.缺失值删除

删除缺失值的情形，一般是在不会影响分析结果、造成的影响无伤大雅，或者难以填补的时候采用。

在pandas中，可以直接用dropna函数进行删除所有含有缺失值的行，或者选择性删除含有缺失值到的行：


```python
#how='any' 在给定的任何一列中有缺失值就删除
sample.dropna(subset=['score'],how='any')
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>group</th>
      <th>id</th>
      <th>name</th>
      <th>score</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>广州市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>1</th>
      <td>深圳市</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>男</td>
    </tr>
    <tr>
      <th>2</th>
      <td>上海市</td>
      <td>1</td>
      <td>1</td>
      <td>Mark</td>
      <td>87.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>3</th>
      <td>重庆市</td>
      <td>2</td>
      <td>3</td>
      <td>Miki</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
    <tr>
      <th>4</th>
      <td>贵州市</td>
      <td>1</td>
      <td>4</td>
      <td>Sully</td>
      <td>77.0</td>
      <td>女</td>
    </tr>
  </tbody>
</table>

</div>



### 5.数据类型转换

数据类型关乎后面的数据处理和数据可视化，不同的数据类型处理和进行可视化的用法都不一样，因此，事先把数据的类型转换好，利于后面的相关步骤。
在pandas中，可以用info和dtypes方法进行查看数据类型:



```python
sample.info()
sample.dtypes
#有多少行，多少列
sample.shape
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 6 entries, 0 to 5
    Data columns (total 6 columns):
    city     6 non-null object
    group    6 non-null int64
    id       6 non-null int64
    name     6 non-null object
    score    5 non-null float64
    sex      6 non-null object
    dtypes: float64(1), int64(2), object(3)
    memory usage: 368.0+ bytes





    (6, 6)



常用的数据类型包括str（字符型）、float（浮点型）和int（整型）。当某列数据的类型出现错误时，可通过astype函数进行强制转换数据类型。例如下面通过astype函数对数值型列转换为字符型：


```python
sample['city']=sample['city'].astype(str)
sample['score']=sample['score'].astype('float')
```

### 6.文本处理

在数据中，文本在某种程度上可以说是最‘脏’的数据，不管在录入的数据，还是爬取的数据，总会出现各种各样的‘脏’数据，处理难度非常高。在处理中，主要是切分字符串、值替换。
pandas提供了df.str.split.str()方法对字符串的切割，以下通过此方法获得地级市名称：
对于一些多数词，可以通过df.str.replace()方法进行增加、替换或者删除：


```python
sample['city']=sample['city'].str.split('市',expand=True)[0]

```


```python
sample['sex']=sample['sex'].str.replace('女','女性')

```


```python
#字典：旧列名和新列名对应关系
colNameDict = {'city':'籍贯'}
sample.rename(columns =colNameDict,inplace=True)
sample
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>籍贯</th>
      <th>group</th>
      <th>id</th>
      <th>name</th>
      <th>score</th>
      <th>sex</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>广州</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>女性</td>
    </tr>
    <tr>
      <th>1</th>
      <td>深圳</td>
      <td>1</td>
      <td>1</td>
      <td>Bob</td>
      <td>99.0</td>
      <td>男</td>
    </tr>
    <tr>
      <th>2</th>
      <td>上海</td>
      <td>1</td>
      <td>1</td>
      <td>Mark</td>
      <td>87.0</td>
      <td>女性</td>
    </tr>
    <tr>
      <th>3</th>
      <td>重庆</td>
      <td>2</td>
      <td>3</td>
      <td>Miki</td>
      <td>77.0</td>
      <td>女性</td>
    </tr>
    <tr>
      <th>4</th>
      <td>贵州</td>
      <td>1</td>
      <td>4</td>
      <td>Sully</td>
      <td>77.0</td>
      <td>女性</td>
    </tr>
    <tr>
      <th>5</th>
      <td>北京</td>
      <td>2</td>
      <td>5</td>
      <td>Rose</td>
      <td>NaN</td>
      <td>男</td>
    </tr>
  </tbody>
</table>

</div>



##  03噪声值处理

噪声值指数据中有一个或几个数值与其他数值相比差异较大，又称为异常值、离群值(outlier)。

#### 1. 盖帽法

盖帽法将某连续变量均值上下三倍标准差范围外的记录替换为均值上下三倍标准差值，即盖帽处理

Python中可自定义函数完成盖帽法。如下所示，参数x表示一个pd.Series列，quantile指盖帽的范围区间，默认凡小于百分之1分位数和大于百分之99分位数的值将会被百分之1分位数和百分之99分位数替代：




```python
def cap(x,quantile=[0.01,0.99]):
    """盖帽法处理异常值    Args：       
    x：pd.Series列，连续变量        
    quantile：指定盖帽法的上下分位数范围    """
    # 生成分位数    
    Q01,Q99=x.quantile(quantile).values.tolist()
    # 替换异常值为指定的分位数   
    if Q01 > x.min():        
        x = x.copy()        
        x.loc[x<Q01] = Q01    
        if Q99 < x.max():       
            x = x.copy()       
            x.loc[x>Q99] = Q99   
            return(x)

```

现生成一组服从正态分布的随机数，sample.hist表示产生直方图,未处理噪声时的变量直方图


```python
sample = pd.DataFrame({'normal':np.random.randn(1000)})
sample.hist(bins=50)
plt.show()
```

{% asset_img 直方图.png This is an example image %}




对pandas数据框所有列进行盖帽法转换，可以以如下写法，从直方图对比可以看出盖帽后极端值频数的变化。


```python
new = sample.apply(cap,quantile=[0.01,0.99])
new.hist(bins=50)
plt.show()
```


{% asset_img 直方图1.png This is an example image %}


#### 2. 分箱法

分箱法通过考察数据的“近邻”来光滑有序数据的值。有序值分布到一些桶或箱中。
分箱法包括等深分箱：每个分箱中的样本量一致；等宽分箱：每个分箱中的取值范围一致。直方图其实首先对数据进行了等宽分箱，再计算频数画图。
比如价格排序后数据为：4、8、15、21、21、24、25、28、34

将其划分为（等深）箱：
箱1：4、8、15 
箱2：21、21、24 
箱3：25、28、34 

将其划分为（等宽）箱：
箱1：4、8
箱2：15、21、21、24 
箱3：25、28、34 


分箱法将异常数据包含在了箱子中，在进行建模的时候，不直接进行到模型中，因而可以达到处理异常值的目的。
pandas的cut函数提供了分箱的实现方法，下面介绍如何具体实现。
等宽分箱：cut函数可以直接进行等宽分箱，此时需要的待分箱的列和分箱个数两个参数，如下所示，sample数据的int列为从10个服从标准正态分布的随机数：


```python
sample =pd.DataFrame({'normal':np.random.randn(10)})
sample

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }


    .dataframe thead th {
        text-align: left;
    }
    
    .dataframe tbody tr th {
        vertical-align: top;
    }

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>normal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-0.208894</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.864225</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-1.109015</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-1.729145</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1.151908</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.026209</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-0.921893</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-0.447447</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.033243</td>
    </tr>
    <tr>
      <th>9</th>
      <td>-0.140880</td>
    </tr>
  </tbody>
</table>

</div>



现分为5箱，可以看到，结果是按照宽度分为5份，下限中，cut函数自动选择小于列最小值一个数值作为下限，最大值为上限，等分为五分。结果产生一个Categories类的列，类似于R中的factor，表示分类变量列。
此外弱数据存在缺失，缺失值将在分箱后将继续保持缺失


```python
pd.cut(sample.normal,5)
```




    0    (-0.577, -0.000513]
    1         (0.576, 1.152]
    2       (-1.153, -0.577]
    3       (-1.732, -1.153]
    4         (0.576, 1.152]
    5     (-0.000513, 0.576]
    6       (-1.153, -0.577]
    7    (-0.577, -0.000513]
    8         (0.576, 1.152]
    9    (-0.577, -0.000513]
    Name: normal, dtype: category
    Categories (5, interval[float64]): [(-1.732, -1.153] < (-1.153, -0.577] < (-0.577, -0.000513] < (-0.000513, 0.576] < (0.576, 1.152]]



这里也可以使用labels参数指定分箱后各个水平的标签，如下所示，此时相应区间值被标签值替代：


```python
pd.cut(sample.normal,bins=5,labels=[1,2,3,4,5])
```




    0    3
    1    5
    2    2
    3    1
    4    5
    5    4
    6    2
    7    3
    8    5
    9    3
    Name: normal, dtype: category
    Categories (5, int64): [1 < 2 < 3 < 4 < 5]



标签除了可以设定为数值,也可以设定为字符，如下所示，将数据等宽分为两箱，标签为‘bad’，‘good’：


```python
pd.cut(sample.normal,bins=2,labels=['bad','good'])
```




    0    good
    1    good
    2     bad
    3     bad
    4    good
    5    good
    6     bad
    7     bad
    8    good
    9    good
    Name: normal, dtype: category
    Categories (2, object): [bad < good]



等深分箱：等深分箱中，各个箱的宽度可能不一，但频数是几乎相等的，所以可以采用数据的分位数来进行分箱。依旧以之前的sample数据为例，现进行等深度分2箱，首先找到2箱的分位数：


```python
sample.normal.quantile([0,0.5,1])
```




    0.0   -1.729145
    0.5   -0.174887
    1.0    1.151908
    Name: normal, dtype: float64



在bins参数中设定分位数区间，如下所示完成分箱，include_lowest=True参数表示包含边界最小值包含数据的最小值,也可以加入label参数指定标签


```python
pd.cut(sample.normal,bins=sample.normal.quantile([0,0.5,1]), include_lowest=True)
```




    0    (-1.73, -0.175]
    1    (-0.175, 1.152]
    2    (-1.73, -0.175]
    3    (-1.73, -0.175]
    4    (-0.175, 1.152]
    5    (-0.175, 1.152]
    6    (-1.73, -0.175]
    7    (-1.73, -0.175]
    8    (-0.175, 1.152]
    9    (-0.175, 1.152]
    Name: normal, dtype: category
    Categories (2, interval[float64]): [(-1.73, -0.175] < (-0.175, 1.152]]




```python
pd.cut(sample.normal,bins=sample.normal.quantile([0,0.5,1]), include_lowest=True,labels=['bad','good'])
```




    0     bad
    1    good
    2     bad
    3     bad
    4    good
    5    good
    6     bad
    7     bad
    8    good
    9    good
    Name: normal, dtype: category
    Categories (2, object): [bad < good]

#### 3. 多变量异常值处理-聚类法

通过快速聚类法将数据对象分组成为多个簇，在同一个簇中的对象具有较高的相似度，而不同的簇之间的对象差别较大。聚类分析可以挖掘孤立点以发现噪声数据，因为噪声本身就是孤立点。
对于聚类方法处理异常值，其步骤如下所示：

输入：数据集S（包括N条记录，属性集D：{年龄、收入}），一条记录为一个数据点，一条记录上的每个属性上的值为一个数据单元格。数据集S有N×D个数据单元格，其中某些数据单元格是噪声数据。

输出：孤立数据点如图所示。孤立点A是我们认为它是噪声数据，很明显它的噪声属性是收入，通过对收入变量使用盖帽法可以剔除A。