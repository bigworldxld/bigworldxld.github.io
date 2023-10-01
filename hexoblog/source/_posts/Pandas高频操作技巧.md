---
title: Pandas高频操作技巧
date: 2023-03-08 11:39:25
tags: pandas
categories: python
img: /images/pandas1.jpg
---

## Pandas 高频操作技巧

## **01、复杂查询**

### 1、逻辑运算

```python
# Q1成绩大于36
df.Q1> 36
# Q1成绩不小于60分，并且是C组成员
~(df.Q1< 60) & (df['team'] == 'C')
```

### 2、逻辑筛选数据

切片（[ ]）、.loc[ ] 左闭右开 和.iloc[ ] 左闭右闭均 支持上文所介绍的逻辑表达式。

**以下是切片（[ ]）的逻辑筛选示例：**

```python
df[df['Q1']== 8] # Q1等于8
df[~(df['Q1']== 8)] # 不等于8
df[df.name== 'Ben'] # 姓名为Ben
df[df.Q1> df.Q2]
# 表达式与切片一致
df.loc[df['Q1']> 90,'Q1'] # Q1大于90，只显示Q1
df.loc[(df.Q1> 80) & (df.Q2 < 15)] # and关系
df.loc[(df.Q1> 90) | (df.Q2 < 90)] # or关系
df.loc[df['Q1']== 8] # 等于8
df.loc[df.Q1== 8] # 等于8
df.loc[df['Q1']> 90, 'Q1':] # Q1大于90，显示Q1及其后所有列
```

### 3、函数筛选

```python
# 查询最大索引的值
df.Q1[lambda s: max(s.index)] # 值为21
# 计算最大值
max(df.Q1.index)
# 99
df.Q1[df.index==99]
```

### 4、比较函数

```python
# 以下相当于 df[df.Q1 == 60]
df[df.Q1.eq(60)]
df.ne() # 不等于 !=
df.le() # 小于等于 <=
df.lt() # 小于 <
df.ge() # 大于等于 >=
df.gt() # 大于 >
```

### 5、查询df.query()

```python
df.query('Q1 > Q2 > 90') # 直接写类型SQL where语句
```

还支持使用@符引入变量

```python
# 支持传入变量，如大于平均分40分的
a = df.Q1.mean()
df.query('Q1 > @a+40')
df.query('Q1 > `Q2`+@a')
```

df.eval()与df.query()类似，也可以用于表达式筛选。

```python
# df.eval()用法与df.query类似
df[df.eval("Q1 > 90 > Q3 >10")]
df[df.eval("Q1 > `Q2`+@a")]
```

### 6、筛选df.filter()

```python
df.filter(items=['Q1', 'Q2']) # 选择两列
df.filter(regex='Q', axis=1) # 列名包含Q的列
df.filter(regex='e$', axis=1) # 以e结尾的列
df.filter(regex='1$', axis=0) # 正则，索引名以1结尾
df.filter(like='2', axis=0) # 索引中有2的
# 索引中以2开头、列名有Q的
df.filter(regex='^2',axis=0).filter(like='Q', axis=1)
```

**7、按数据类型查询**

```python
df.select_dtypes(include=['float64']) # 选择float64型数据
df.select_dtypes(include='bool')
df.select_dtypes(include=['number']) # 只取数字型
df.select_dtypes(exclude=['int']) # 排除int类型
df.select_dtypes(exclude=['datetime64'])
```

## **02、数据类型转换**

```python
# 对所有字段指定统一类型
df = pd.DataFrame(data, dtype='float32')
# 对每个字段分别指定
df = pd.read_excel(data, dtype={'team':'string', 'Q1': 'int32'})
```

### 1、推断类型

```python
# 自动转换合适的数据类型
df.infer_objects() # 推断后的DataFrame
df.infer_objects().dtypes
```

### 2、指定类型

```python
# 按大体类型推定
m = ['1', 2, 3]
s = pd.to_numeric(s) # 转成数字
pd.to_datetime(m) # 转成时间
pd.to_timedelta(m) # 转成时间差
pd.to_datetime(m, errors='coerce') # 错误处理
pd.to_numeric(m, errors='ignore')
pd.to_numeric(m,errors='coerce').fillna(0) # 兜底填充
pd.to_datetime(df[['year', 'month', 'day']])# 组合成日期
```

### 3、类型转换astype()

```python
df.Q1.astype('int32').dtypes
# dtype('int32')
df.astype({'Q1': 'int32','Q2':'int32'}).dtypes
```

## **03、数据排序**

### 1、索引排序df.sort_index()

```python
s.sort_index() # 升序排列
df.sort_index() # df也是按索引进行排序
df.team.sort_index(ascending=False)# 降序排列
s.sort_index(inplace=True) # 排序后生效，改变原数据
# 索引重新0-(n-1)排，很有用，可以得到它的排序号
s.sort_index(ignore_index=True)
s.sort_index(na_position='first') # 空值在前，另'last'表示空值在后
s.sort_index(level=1) # 如果多层，排一级
s.sort_index(level=1, sort_remaining=False) #这层不排
# 行索引排序，表头排序
df.sort_index(axis=1) # 会把列按列名顺序排列
```

### 2、数值排序sort_values()

```python
df.Q1.sort_values()
df.sort_values('Q4')
df.sort_values(by=['team','name'],ascending=[True, False])
```

对于numpy array,

a[np.argsort(a[:,1])]

这里argsort(a[:，1])计算使a的第二列按升序排序的排列，然后a[…]相应地对a的行重新排序。

其他方法：

```python
s.sort_values(ascending=False) # 降序
s.sort_values(inplace=True) # 修改生效
s.sort_values(na_position='first') # 空值在前
# df按指定字段排列
df.sort_values(by=['team'])
df.sort_values('Q1')
# 按多个字段，先排team，在同team内再看Q1
df.sort_values(by=['team', 'Q1'])
# 全降序
df.sort_values(by=['team', 'Q1'], ascending=False)
# 对应指定team升Q1降
df.sort_values(by=['team', 'Q1'],ascending=[True, False])
# 索引重新0-(n-1)排
df.sort_values('team', ignore_index=True)
```

### 3、混合排序

```python
df.set_index('name', inplace=True) # 设置name为索引
df.index.names = ['s_name'] # 给索引起名
df.sort_values(by=['s_name', 'team']) # 排序
```

### 4、按值大小排序nsmallest()和nlargest()

```python
s.nsmallest(3) # 最小的3个
s.nlargest(3) # 最大的3个
# 指定列
df.nlargest(3, 'Q1')
df.nlargest(5, ['Q1', 'Q2'])
df.nsmallest(5, ['Q1', 'Q2'])
```

## **04、添加修改**

### 1、修改数值

```python
df.iloc[0,0] # 查询值
# 'Liver'
df.iloc[0,0] = 'Lily' # 修改值
df.iloc[0,0] # 查看结果
# 'Lily'

# 将小于60分的成绩修改为60
df[df.Q1 < 60] = 60
# 查看
df.Q1

# 生成一个长度为100的列表
v = [1, 3, 5, 7, 9] * 20
```

### 2、替换数据

```python
s.replace(0, 5) # 将列数据中的0换为5
df.replace(0, 5) # 将数据中的所有0换为5
df.replace([0, 1, 2, 3], 4) # 将0～3全换成4
df.replace([0, 1, 2, 3], [4, 3, 2, 1]) # 对应修改
s.replace([1, 2], method='bfill') # 向下填充
df.replace({0: 10, 1: 100}) # 字典对应修改
df.replace({'Q1': 0, 'Q2': 5}, 100) # 将指定字段的指定值修改为100
df.replace({'Q1': {0: 100, 4: 400}}) # 将指定列里的指定值替换为另一个指定的值
```

### 3、填充空值

```python
df.fillna(0) # 将空值全修改为0
# {'backfill', 'bfill', 'pad', 'ffill',None}, 默认为None
df.fillna(method='ffill') # 将空值都修改为其前一个值
values = {'A': 0, 'B': 1, 'C': 2, 'D': 3}
df.fillna(value=values) # 为各列填充不同的值
df.fillna(value=values, limit=1) # 只替换第一个
```

### 4、修改索引名

```python
df.rename(columns={'team':'class'})
```

常用方法如下：

```python
df.rename(columns={"Q1":"a", "Q2": "b"}) # 对表头进行修改
df.rename(index={0: "x", 1:"y", 2: "z"}) # 对索引进行修改
df.rename(index=str) # 对类型进行修改
df.rename(str.lower, axis='columns') # 传索引类型
df.rename({1: 2, 2: 4}, axis='index')

# 对索引名进行修改
s.rename_axis("animal")
df.rename_axis("animal") # 默认是列索引
df.rename_axis("limbs",axis="columns") # 指定行索引

# 索引为多层索引时可以将type修改为class
df.rename_axis(index={'type': 'class'})

# 可以用set_axis进行设置修改
s.set_axis(['a', 'b', 'c'], axis=0)
df.set_axis(['I', 'II'], axis='columns')
df.set_axis(['i','ii'],axis='columns',inplace=True)
```

### 5、增加列

```python
df['foo'] = 100 # 增加一列foo，所有值都是100
df['foo'] = df.Q1 + df.Q2 # 新列为两列相加
df['foo'] = df['Q1'] + df['Q2'] # 同上
# 把所有为数字的值加起来
df['total'] =df.select_dtypes(include=['int']).sum(1)
df['total']=df.loc[:,'Q1':'Q4'].apply(lambda x: sum(x), axis='columns')
df.loc[:, 'Q10'] = '我是新来的' # 也可以
# 增加一列并赋值，不满足条件的为NaN
df.loc[df.num >= 60, '成绩'] = '合格'
df.loc[df.num < 60, '成绩'] = '不合格'
```

对于numpy array,

np.column_stack((a,a[:,1]/a[:,2]))等价于df[‘price’]=df.price/df.weight

### 6、插入列df.insert()

```python
# 在第三列的位置上插入新列total列，值为每行的总成绩
df.insert(2, 'total', df.sum(1))
```

### 7、指定列df.assign()

在DataFrame添加一列常用4种方法，分别是1、直接赋值法; 2、apply函数:3、assign函数; 4、条件筛选,
此外,concat函数
也可以灵活的添加列,insert函数可以插入列

```python
# 增加total列
df.assign(total=df.sum(1))
# 增加两列
df.assign(total=df.sum(1), Q=100)
df.assign(total=df.sum(1)).assign(Q=100)
其他使用示例：
df.assign(Q5=[100]*100) # 新增加一列Q5
df = df.assign(Q5=[100]*100) # 赋值生效
df.assign(Q6=df.Q2/df.Q1) # 计算并增加Q6
df.assign(Q7=lambda d: d.Q1 * 9 / 5 + 32) # 使用lambda# 添加一列，值为表达式结果：True或False
df.assign(tag=df.Q1>df.Q2)
# 比较计算，True为1，False为0
df.assign(tag=(df.Q1>df.Q2).astype(int))
# 映射文案
df.assign(tag=(df.Q1>60).map({True:'及格',False:'不及格'}))
# 增加多个
df.assign(Q8=lambda d: d.Q1*5,
          Q9=lambda d: d.Q8+1) # Q8没有生效，不能直接用df.Q8
```

### 8、执行表达式df.eval()

```python
# 传入求总分表达式
df.eval('total = Q1+Q3+Q3+Q4')
```

**其他方法：**

```python
df['C1'] = df.eval('Q2 + Q3')
df.eval('C2 = Q2 + Q3') # 计算
a = df.Q1.mean()
df.eval("C3 =`Q3`+@a") # 使用变量
df.eval("C3 = Q2 > (`Q3`+@a)") #加一个布尔值
df.eval('C4 = name + team', inplace=True) # 立即生效
```

### 9、增加行

```python
# 新增索引为100的数据
df.loc[100] = ['tom', 'A', 88, 88, 88, 88]
```

**其他方法：**

```python
df.loc[101]={'Q1':88,'Q2':99} # 指定列，无数据列值为NaN
df.loc[df.shape[0]+1] = {'Q1':88,'Q2':99} # 自动增加索引
df.loc[len(df)+1] = {'Q1':88,'Q2':99}
# 批量操作，可以使用迭代
rows = [[1,2],[3,4],[5,6]]
for row in rows:
    df.loc[len(df)] = row
```

### 10、追加合并

```python
df = pd.DataFrame([[1, 2],[3,4]],columns=list('AB'))
df2 = pd.DataFrame([[5, 6], [7, 8]],columns=list('AB'))
df.append(df2)
```

### 11、删除

```python
# 删除索引为3的数据
s.pop(3)
# 93s
s
```

### 12、删除空值

```python
df.dropna() # 一行中有一个缺失值就删除
df.dropna(axis='columns') # 只保留全有值的列
df.dropna(how='all') # 行或列全没值才删除
df.dropna(thresh=2) # 至少有两个空值时才删除
df.dropna(inplace=True) # 删除并使替换生效
```

## **05、高级过滤**

### 1、df.where()

```python
# 数值大于70
df.where(df > 70)
```

### 2、np.where()

```python
# 小于60分为不及格
np.where(df>=60, '合格', '不合格')
```

### 3、df.mask()

```python
# 符合条件的为NaN
df.mask(s > 80)
```

### 4、df.lookup()

```python
# 行列相同数量，返回一个array
df.lookup([1,3,4], ['Q1','Q2','Q3']) # array([36, 96, 61])
df.lookup([1], ['Q1']) # array([36])
```

## **06、数据迭代**

### 1、迭代Series

```python
# 迭代指定的列
for i in df.name:
      print(i)
# 迭代索引和指定的两列
for i,n,q in zip(df.index, df.name,df.Q1):
    print(i, n, q)
```

### 2、df.iterrows()

```python
# 迭代，使用name、Q1数据
for index, row in df.iterrows():
    print(index, row['name'], row.Q1)
```

### 3、df.itertuples()

```python
for row in df.itertuples():
    print(row)
```

### 4、df.items()

```python
# Series取前三个
for label, ser in df.items():
    print(label)
    print(ser[:3], end='\n\n')
```

### 5、按列迭代

```python
# 直接对DataFrame迭代
for column in df:
    print(column)
```

### 07、函数应用

### 1、pipe()

**应用在整个DataFrame或Series上。**

```python
# 对df多重应用多个函数
f(g(h(df), arg1=a), arg2=b, arg3=c)
# 用pipe可以把它们连接起来
(df.pipe(h)
    .pipe(g, arg1=a)
    .pipe(f, arg2=b, arg3=c)
)
```

### 2、apply()

**应用在DataFrame的行或列中，默认为列。**

```python
# 将name全部变为小写
df.name.apply(lambda x: x.lower())
```

### 3、applymap()

**应用在DataFrame的每个元素中。**

```python
# 计算数据的长度
def mylen(x):
    return len(str(x))
df.applymap(lambda x:mylen(x)) # 应用函数
df.applymap(mylen) # 效果同上
```

### 4、map()

**应用在Series或DataFrame的一列的每个元素中。**

```python
df.team.map({'A':'一班', 'B':'二班','C':'三班', 'D':'四班',})# 枚举替换
df['name'].map(f)
```

### 5、agg()

```python
# 每列的最大值
df.agg('max')
# 将所有列聚合产生sum和min两行
df.agg(['sum', 'min'])
# 序列多个聚合
df.agg({'Q1' : ['sum', 'min'], 'Q2' : ['min','max']})
# 分组后聚合
df.groupby('team').agg('max')
df.Q1.agg(['sum', 'mean'])
```

### 6、transform()

```python
df.transform(lambda x: x*2) # 应用匿名函数
df.transform([np.sqrt, np.exp]) # 调用多个函数
```

### 7、copy()

```python
s = pd.Series([1, 2], index=["a","b"])
s_1 = s
s_copy = s.copy()
s_1 is s # True
s_copy is s # False
```

### Series 和 Index

在Pandas中，我们做了大量工作来统一所有支持的数据类型对NaN的使用。根据定义(在CPU级别上强制执行)，nan+anything会得到nan。所以

```python
np.sum([1, np.nan, 2])  
nan
```

但是

```python
pd.Series([1, np.nan, 2]).sum()  
3.0
```

一个公平的比较是使用np.nansum代替np.sum,用np.nanmean而不是np.mean等

如果您100%确定列中没有缺失值，则使用df.column.values.sum()而不是df.column.sum()可以获得x3-x30的性能提升。在存在缺失值的情况下，Pandas的速度相当不错，甚至在巨大的数组(超过10个同质元素)方面优于NumPy

pdi(代表pandas illustrated)是github上的一个开源库，具有本文所需的这个和其他功能。

对Series做了补丁，让它看起来更好

Footer在这里被禁用了，但它可以用于显示dtype，特别是分类类型。

还可以使用pdi.sidebyside(obj1, obj2，…)并排显示多个Series或dataframe

```shell
import pdi
pdi.patch_series_repr(footer=False)

pip install pandas-illustrated
```

### 按值查找元素

Series内部由一个NumPy数组和一个类似数组的结构index组成

```python
s.index[s.tolist().find(x)]      *# faster for len(s) < 1000* 
s.index[np.where(s.values==x)[0][0]] *# faster for len(s) > 1000*
```

### 插值函数

pandas其实自带一个很强大的插值函数：interpolate

DataFrame.interpolate(method=‘linear’, axis=0, limit=None, inplace=False, limit_direction=None, limit_area=None, downcast=None, **kwargs)

method ： str，默认为‘linear’使用插值方法。
可用的插值方法：
‘linear’：忽略索引，线性等距插值。这是MultiIndexes支持的唯一方法。
‘time’: 在以天或者更高频率的数据上插入给定的时间间隔长度数据。
‘index’, ‘values’: 使用索引的实际数值。
‘pad’：使用现有值填写NaN。
‘nearest’, ‘zero’, ‘slinear’, ‘quadratic’, ‘cubic’, ‘spline’, ‘barycentric’, ‘polynomial’: 传递给 scipy.interpolate.interp1d。这些方法使用索引的数值。‘polynomial’ 和 ‘spline’ 都要求您还指定一个顺序（int），例如 ，df.interpolate(method=‘polynomial’, order=5)
‘krogh’，‘piecewise_polynomial’，‘spline’，‘pchip’，‘akima’：包括类似名称的SciPy插值方法。
‘from_derivatives’：指 scipy.interpolate.BPoly.from_derivatives，它替换了scipy 0.18中的’piecewise_polynomial’插值方法。

axis ： {0或’index’，1或’columns’，None}，默认为None；沿轴进行interpolate。

limit： int；要填充的连续NaN的最大数量。必须大于0。

inplace ： bool，默认为False；如果可以，更新现有数据。

limit_direction ： {‘forward’，‘backward’，‘both’}，默认为’forward’；如果指定了限制，则将沿该方向填充连续的NaN。

limit_area ： {None, ‘inside’, ‘outside’}, 默认为None；如果指定了限制，则连续的NaN将填充此限制。
None：无填充限制。
‘inside’：仅填充有效值包围的NaN。
‘outside’: 仅在有效值之外填充NaN。

0.23.0版中的新功能。

downcast ： 可选， ‘infer’ 或None，默认为None;如果可能，请向下转换dtype。

**kwargs

如果 method = ‘pad’ 或者 ‘ffill’, limit_direction 必须为 ‘forward’.

如果 method = ‘backfill’ 或者 ‘bfill’, limit_direction 必须为 ‘backwards’.

线性差值法中采用 method = ‘linear’，limit_direction = ‘both’

### 比较

```python
np.all(pd.Series([1., None, 3.]) ==   
           pd.Series([1., None, 3.]))  
False  
np.all(pd.Series([1, None, 3], dtype='Int64') ==   
           pd.Series([1, None, 3], dtype='Int64'))  
True  
np.all(pd.Series(['a', None, 'c']) ==   
           pd.Series(['a', None, 'c']))  
False
```

为了正确地比较nan，需要用数组中一定没有的元素替换nan。例如，使用-1或∞:

```python
>>> np.all(s1.fillna(np.inf) == s2.fillna(np.inf))   # works for all dtypes  
True
```

或者，更好的做法是使用NumPy或Pandas的标准比较函数:

```python
>>> s = pd.Series([1., None, 3.])  
>>> np.array_equal(s.values, s.values, equal_nan=True)  
True  
>>> len(s.compare(s)) == 0  
True
```

这里，compare函数返回一个差异列表(实际上是一个DataFrame)， array_equal则直接返回一个布尔值。

当比较混合类型的DataFrames时，NumPy比较失败(issue #19205)，而Pandas工作得很好。如下所示:

```python
>>> df = pd.DataFrame({'a': [1., None, 3.], 'b': ['x', None, 'z']})  
>>> np.array_equal(df.values, df.values, equal_nan=True)  
TypeError  
<...>  
>>> len(df.compare(df)) == 0  
True
```

由于序列中的每个元素都可以通过标签或位置索引访问，因此argmin (argmax)有一个姐妹函数idxmin (idxmax)

- std:样本标准差
- var，无偏方差
- sem，均值的无偏标准误差
- quantile分位数，样本分位数(s.quantile(0.5)≈s.median())
- oode是出现频率最高的值
- 默认为Nlargest和nsmallest，按出现顺序排列
- diff，第一个离散差分
- cumsum 和 cumprod、cumulative sum和product
- cummin和cummax，累积最小值和最大值

- pct_change，当前元素与前一个元素之间的变化百分比
- skew偏态，无偏态(三阶矩)
- kurt或kurtosis，无偏峰度(四阶矩)
- cov、corr和autocorr、协方差、相关和自相关
- rolling滚动窗口、加权窗口和指数加权窗口

s.is_monotonic_increasing()。它只对单调递减序列返回False

### DataFrame

对dataframes、series和它们的组合应用普通操作，如加、减、乘、除、求模、幂等。

所有的算术运算都是根据行标签和列标签对齐的:

每当你需要在dataframe和列式序列之间执行混合操作时,使用.add() .sub() .mul() .pow() .div() .floordiv() 取整 .mod() 取余

merge和join方法总结：

df.merge(df1,left_on=‘a’,right_on=’b’,suffixes=(‘_ed’,‘_ws’))

df.join(df1,on=‘a’,lsuffix=‘_ed’,rsuffix=‘_ws’)

join比concat更可配置:特别是，它有五种连接模式，而concat只有两种（0：垂直(默认)，1：水平）

- 合并非索引列上的连接，连接要求列被索引
- merge丢弃左DataFrame的索引，join保留它
- 默认情况下，merge执行内联结，join执行左外联结
- 合并不保持行顺序
- Join可以保留它们(有一些限制)
- join是合并的别名，left_index=True和/或right_index=True

分组：

当对单个列求和时，你将得到一个Series而不是DataFrame。如果出于某种原因，你想要一个DataFrame，你可以:

- 使用双括号:df.groupby('product')[['quantity']].sum()
- 显式转换:df.groupby('product')['quantity'].sum().to_frame()

切换到数值索引也会创建一个DataFrame:

- df.groupby('product', as_index=False)['quantity'].sum()
- df.groupby('product')['quantity'].sum().reset_index()

---

agg()对同一列进行多次使用聚合函数

df.groupby(‘product’).agg(quantity=(‘quantity’,’sum’),

min_price=(‘price’,’min’),max_price=(‘price’,’max’))

### 旋转和`反旋转`

当数据是“密集的”(当有很少的0元素)时，` short `格式更合适，而当数据是“稀疏的”(大多数元素为0，可以从表中省略)时，` long `格式更好

short:

```python
df=df.pivot(index='colient',columns='product',values='quantity',fil_value=0,[aggfunc='sum'])
```

该命令丢弃了与操作无关的任何信息(索引、价格)，并将来自三个请求列的信息转换为长格式，将客户名称放入结果的索引中，将产品名称放入列中，将销售数量放入DataFrame的` body `中。

pivot_table可以计算小计和合计

df=df.pivot_table(...,margins=True,margins_name=‘total’)

变long:

```python
#methods 1:列转行
df.stack().reset_index(name='quantity')
#methods 2:
df.reset_index().melt(id_vars='client',value_vars=['bananas','oranges'],value_name'quantity')
```

Pivot丢失了结果的` body `的名称信息，因此无论是stack还是melt，我们都必须提醒pandas ` quantity `列的名称。

a[3:10:2]等价于a[slice(3,10,2)]

### MultiIndex使用

#### 1、创建多层索引

方法一：隐式创建，即给DataFrame的index或columns参数传递两个或更多的数组。

```python
df1 = pd.DataFrame(np.random.randint(1.30, size=(8，4)),
index= ["小明','小花",'小丽','小玲','小军','小新', '小美', '小芳']，columns=[["男生'，‘男生'，'女生'，'女生"]，
['漂亮'，"不漂亮"，'漂亮'，"不漂亮']])
```

方法二：显示创建，推荐使用较简单的pd.MultiIndex.from_product方法。

MultiIndex表示多级索引，它是从Index继承过来的，其中多级标签用元组对象来表示。from_product()从多个集合的笛卡尔积创建MultiIndex对象。

```python
df = pd.DataFrame(np.random.randint(1,30，size=(8，4)),index=['小明','小花",'小丽",'小玲', '小军', '小新','小美','小芳']，columns=pd.MultiIndex.from_product([["男生'，'女生'],
["漂亮'，‘不漂亮']]))
```

2、更改多层索引的层级

所谓更改多层索引的层级

```python
df.columns.names = [ ' gender ' , 'isBeauty ' ]
#设置列索引名
#如果设置index行索引，则可以下面的方式
#df.index.names =['你的名字']
```

#### 2、获取多重索引的值

```python
import pandas as pd
from pandas import IndexSlice as idx 
df = pd.read_csv('dataset.csv',
    index_col=[0,1],
    header=[0,1]
)#行索引有City，Date。列索引有Day,Night,Weather,Wind,Max Temperature
df = df.sort_index()
df
```

首先来看一下“列”方向多重索引的层级，代码如下

```python
df.columns.levels
```

FrozenList([['Day', 'Night'], ['Max Temperature', 'Weather', 'Wind']])

获取第一层级上面的索引值，代码如下

```python
df.columns.get_level_values(0)
```

Index(['Day', 'Day', 'Day', 'Night', 'Night', 'Night'], dtype='object')

第二层级的索引值，只是把当中的0替换成1即可，代码如下

```python
df.columns.get_level_values(1)
```

Index(['Weather', 'Wind', 'Max Temperature', 'Weather', 'Wind',
    'Max Temperature'],
   dtype='object')

####  3、多重索引的数据获取

```python
df.loc['Cambridge', 'Day'].loc['2019-07-03']
```

在第一次调用`loc['Cambridge', 'Day']`的时候返回的是DataFrame数据集，然后再通过调用`loc()`方法来提取数据，当然这里还有更加快捷的方法，代码如下

```python
df.loc[('Cambridge', '2019-07-01'), 'Day']
```

我们需要传入元祖的形式的索引值来进行数据的提取。要是我们不只是想要获取单行或者是单列的数据，可以这么来操作

```python
df.loc[ 
    ('Cambridge' , ['2019-07-01','2019-07-02'] ) ,
    'Day'
] #这么来写是会报语法错误的
```

正确的方法应该是这么来做，

```python
df.loc[
    ('Cambridge','2019-07-01'):('London','2019-07-03'),
    'Day'
]
```

#### 4、`xs()`方法的调用

`xs()`方法来指定多重索引中的层级

例如想要获取伦敦在2019年7月4日的全天数据，代码如下

```python
df.xs(('London', '2019-07-04'), level=['City','Date'])
```

`axis`参数来指定是获取“列”方向还是“行”方向上的数据，例如我们想要获取“Weather”这一列的数据，代码如下

```python
df.xs('Weather', level=1, axis=1)
```

返回行索引City,Date列索引Day,Night

当中的`level`参数代表的是层级，

```python
df.xs('Day', level=0, axis=1)
```

返回行索引City,Date列索引Weather,Wind,Max Temperature

####  5、IndexSlice()方法的调用

同时`Pandas`内部也提供了`IndexSlice()`方法来方便我们更加快捷地提取出多重索引数据集中的数据，代码如下

```python
from pandas import IndexSlice as idx
df.loc[ 
    idx[: , '2019-07-04'], 
    'Day'
]
```

返回行索引City,Date列索引Weather,Wind,Max Temperature

指定行以及列方向上的索引来进行数据的提取，代码如下

```python
rows = idx[: , '2019-07-02']
cols = idx['Day' , ['Max Temperature','Weather']]
df.loc[rows, cols]
```

返回行索引City,Date列索引Day(Weather,Max Temperature

)