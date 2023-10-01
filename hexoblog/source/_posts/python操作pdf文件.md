---
title: python操作pdf文件
date: 2023-03-22 15:45:07
tags: python
categories: python
img: /images/python.jpg
---

## ChatPDF，PDF操作一键完成

ChatPDF 是一个基于 Python 的 PDF 处理工具，它可以用于创建、合并、拆分、加密和解密 PDF 文件，以及提取文本和图像等。ChatPDF 提供了简单易用的 API，使得用户可以方便地完成各种 PDF 处理任务。

### 安装

首先，我们需要安装 ChatPDF。你可以在终端中使用以下命令进行安装：

> pip install chatpdf

### 创建 PDF 文件

现在，让我们看一下如何使用 ChatPDF 创建 PDF 文件。以下是一个简单的示例：

```python
from chatpdf import ChatPDF

pdf = ChatPDF()
pdf.add_page()
pdf.set_font("Arial", size=12)
pdf.cell(200, 10, txt="Hello, World!")
pdf.output("hello.pdf")
```

### 合并 PDF 文件

现在，让我们看一下如何使用 ChatPDF 合并多个 PDF 文件。以下是一个简单的示例：

```python
from chatpdf import merge

pdf_files = ['file1.pdf', 'file2.pdf', 'file3.pdf']
output_file = 'output.pdf'

merge(pdf_files, output_file)
```

以上代码将会合并 file1.pdf、file2.pdf 和 file3.pdf 这三个文件，并将合并后的结果保存到 output.pdf 文件中

### 拆分 PDF 文件

将一个 PDF 文件拆分成多个 PDF 文件，可以使用 ChatPDF 库的 split() 方法。下面是一个示例代码：

```python
from chatpdf import split

pdf_file = 'input.pdf'
output_folder = 'output'

split(pdf_file, output_folder)
```

以上代码将会将 input.pdf 文件拆分成多个 PDF 文件，并保存到 output 文件夹中。

### 提取 PDF 页面

从一个 PDF 文件中提取某些页面，可以使用 ChatPDF 库的 extract_pages() 方法。下面是一个示例代码：

```
from chatpdf import extract_pages

pdf_file = 'input.pdf'
output_file = 'output.pdf'
pages = [1, 3, 5]

extract_pages(pdf_file, output_file, pages)
```

以上代码将会从 input.pdf 文件中提取第 1、3、5 页，并保存到 output.pdf 文件中。

### 旋转 PDF 页面

将 PDF 页面旋转一定角度，可以使用 ChatPDF 库的 rotate_pages() 方法。下面是一个示例代码：

```python
from chatpdf import rotate_pages
pdf_file = 'input.pdf'
output_file = 'output.pdf'
pages = [1, 3]
rotation_angle = 90
rotate_pages(pdf_file, output_file, pages, rotation_angle)
```

以上代码将会将 input.pdf 文件中的第 1、3 页旋转 90 度，并保存到 output.pdf 文件中

### 加密 PDF 文件

将 PDF 文件加密，可以使用 ChatPDF 库的 encrypt() 方法。下面是一个示例代码：

```python
from chatpdf import encrypt

pdf_file = 'input.pdf'
output_file = 'output.pdf'
password = 'mypassword'

encrypt(pdf_file, output_file, password)
```

以上代码将会将 input.pdf 文件加密，并使用密码 mypassword 进行保护。加密后的文件保存到 output.pdf 文件中。

### 解密 PDF 文件

将加密的 PDF 文件解密，可以使用 ChatPDF 库的 decrypt() 方法。下面是一个示例代码：

```python
from chatpdf import decrypt

pdf_file = 'input.pdf'
output_file = 'output.pdf'
password = 'mypassword'

decrypt(pdf_file, output_file, password)
```

以上代码将会将 input.pdf 文件解密，并使用密码 mypassword 进行解密

## ndarray的基本操作

### **1. 索引**

```python
n = np.zeros((6,6), dtype=int)
n
# 输出
# array([[0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0]])

# 修改1行
n[0] = 1
n
# 输出
# array([[1, 1, 1, 1, 1, 1],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0]])

# 修改多行
n[[0,3,-1]] = 2
n
# 输出
# array([[2, 2, 2, 2, 2, 2],
#       [0, 0, 0, 0, 0, 0],
#       [0, 0, 0, 0, 0, 0],
#       [2, 2, 2, 2, 2, 2],
#       [0, 0, 0, 0, 0, 0],
#       [2, 2, 2, 2, 2, 2]])
```

### **2. 切片**

```python
# 多维数组
n = np.random.randint(0,100, size=(5,6))
n
# 输出
# array([[94, 45, 60, 71, 70, 88],
#       [43, 16, 39, 70, 14,  4],
#       [59, 12, 84, 38, 96, 88],
#       [80,  9, 72, 95, 69, 91],
#       [44, 84,  5, 47, 92, 31]])

#  行 翻转
n[::-1]

# 列 翻转 (第二个维度)
n[:, ::-1]
```

### 3. 变形

使用reshape函数, 注意参数是一个tuple

```python
n = np.arange(1, 21)
n
# array([ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20])

n.shape
# (20,)

# 变成2维
n2 = np.reshape(n, (4,5))
n2
# array([[ 1,  2,  3,  4,  5],
#       [ 6,  7,  8,  9, 10],
#       [11, 12, 13, 14, 15],
#       [16, 17, 18, 19, 20]])
n2.shape
# (4, 5)
n2.reshape((20,))  # 变成一维
n2.reshape((-1,))  # 变成一维,-1 表示行会自动分配
```

### 4. 级联

concatenate

```python
n1 = np.random.randint(0,100, size=(4,5))
n2 = np.random.randint(0,100, size=(4,5))
display(n1, n2)
# array([[48, 89, 82, 88, 55],
#       [63, 80, 77, 27, 51],
#       [11, 77, 90, 23, 71],
#       [ 4, 11, 19, 84, 57]])
# 
#array([[86, 26, 71, 62, 46],
#       [75, 43, 84, 87, 99],
#       [34, 33, 58, 56, 29],
#       [56, 32, 53, 43,  5]])

# 级联,合并
# 上下合并:垂直级联
np.concatenate((n1, n2))
np.concatenate((n1, n2), axis=0)  # axis=0表示行，第一个维度
# array([[48, 89, 82, 88, 55],
#       [63, 80, 77, 27, 51],
#       [11, 77, 90, 23, 71],
#       [ 4, 11, 19, 84, 57],
#       [86, 26, 71, 62, 46],
#       [75, 43, 84, 87, 99],
#       [34, 33, 58, 56, 29],
#       [56, 32, 53, 43,  5]])

# 左右合并:水平级联
np.concatenate((n1, n2), axis=1)  # axis=1表示列，第二个维度
# array([[48, 89, 82, 88, 55, 86, 26, 71, 62, 46],
#       [63, 80, 77, 27, 51, 75, 43, 84, 87, 99],
#       [11, 77, 90, 23, 71, 34, 33, 58, 56, 29],
#       [ 4, 11, 19, 84, 57, 56, 32, 53, 43,  5]])
# 左右合并:水平级联
np.hstack((n1, n2)) 
# array([[48, 89, 82, 88, 55, 86, 26, 71, 62, 46],
#       [63, 80, 77, 27, 51, 75, 43, 84, 87, 99],
#       [11, 77, 90, 23, 71, 34, 33, 58, 56, 29],
#       [ 4, 11, 19, 84, 57, 56, 32, 53, 43,  5]])

# 上下合并:垂直级联
np.vstack((n1, n2)) 
# array([[48, 89, 82, 88, 55],
#       [63, 80, 77, 27, 51],
#       [11, 77, 90, 23, 71],
#       [ 4, 11, 19, 84, 57],
#       [86, 26, 71, 62, 46],
#       [75, 43, 84, 87, 99],
#       [34, 33, 58, 56, 29],
#       [56, 32, 53, 43,  5]])
```

### 5. 拆分

```python
n = np.random.randint(0, 100, size=(6,4))
n
# array([[ 3, 90, 62, 89],
#       [75,  7, 10, 76],
#       [77, 94, 88, 59],
#       [78, 66, 81, 83],
#       [18, 88, 40, 81],
#       [ 2, 38, 26, 21]])

# 垂直方向，平均切成3份
np.vsplit(n, 3)
# [array([[ 3, 90, 62, 89],
#        [75,  7, 10, 76]]),
# array([[77, 94, 88, 59],
#        [78, 66, 81, 83]]),
# array([[18, 88, 40, 81],
#        [ 2, 38, 26, 21]])]

# 如果是数组
np.vsplit(n, (1,2,4))
# [array([[ 3, 90, 62, 89]]),
#  array([[75,  7, 10, 76]]),
#  array([[77, 94, 88, 59],
#        [78, 66, 81, 83]]),
#  array([[18, 88, 40, 81],
#        [ 2, 38, 26, 21]])]

# 水平方向
# 通过axis来按照指定维度拆分
np.split(n, 2, axis=1)
np.hsplit(n, 2)
# [array([[97, 86],
#        [16, 70],
#        [26, 95],
#        [ 6, 83],
#        [97, 43],
#        [96, 57]]),
# array([[88, 69],
#        [60,  7],
#        [32, 82],
#        [24, 86],
#        [62, 23],
#        [43, 19]])]
```

### 6、copy

```python
# 赋值: 不使用copy
n1 = np.arange(10)
n2 = n1
n1[0] = 100
display(n1, n2)
# array([100,   1,   2,   3,   4,   5,   6,   7,   8,   9])
# array([100,   1,   2,   3,   4,   5,   6,   7,   8,   9])

# 拷贝: copy
n1 = np.arange(10)
n2 = n1.copy()
n1[0] = 100
display(n1, n2)
# array([100,   1,   2,   3,   4,   5,   6,   7,   8,   9])
# array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
```

### 7. 转置

```python
# 转置
n = np.random.randint(0, 10, size=(3, 4)) 
n.T  
# transpose改变数组维度
n = np.random.randint(0, 10, size=(3, 4, 5)) # shape(3, 4, 5)
np.transpose(n, axes=(2,0,1))
```

### `argpartition()`

借助于 `argpartition()`，`Numpy` 可以找出 N 个最大数值的索引，也会将找到的这些索引输出。然后我们根据需要对数值进行排序。

```python
x = np.array([12, 10, 12, 0, 6, 8, 9, 1, 16, 4, 6, 0])
index_val = np.argpartition(x, -4)[-4:] #负号大到小列举，取四个
index_val
```

output

```
array([1, 8, 2, 0], dtype=int64)np.sort(x[index_val])
array([10, 12, 12, 16])
```

### `allclose()`

`allclose()` 用于匹配两个数组，并得到布尔值表示的输出。如果在一个公差范围内（within a tolerance）两个数组不等同，则 `allclose()` 返回 False。该函数对于检查两个数组是否相似非常有用。

```python
array1 = np.array([0.12,0.17,0.24,0.29])
array2 = np.array([0.13,0.19,0.26,0.31])# with a tolerance of 0.1, it should return False:
np.allclose(array1,array2,0.1)
```

output

```
False
```

### `Clip()`

`Clip()` 使得一个数组中的数值保持在一个区间内。有时，我们需要保证数值在上下限范围内。为此，我们可以借助 `Numpy` 的 `clip()` 函数实现该目的。给定一个区间，则区间外的数值被剪切至区间上下限（interval edge）

```python
x = np.array([3, 17, 14, 23, 2, 2, 6, 8, 1, 2, 16, 0])
np.clip(x,2,5)
```

output

```
array([3, 5, 5, 5, 2, 2, 5, 5, 2, 2, 5, 2])
```

### `extract()`

顾名思义，`extract()` 是在特定条件下从一个数组中提取特定元素。借助于 `extract()`，我们还可以使用 `and` 和 `or` 等条件。

```python
# Random integers
array = np.random.randint(20, size=12)
array
```

output

```
array([ 0,  1,  8, 19, 16, 18, 10, 11,  2, 13, 14,  3])#  Divide by 2 and check if remainder is 1
```

```python
cond = np.mod(array, 2)==1
cond
```

output

```
array([False,  True, False,  True, False, False, False,  True, False, True, False,  True])# Use extract to get the values
```

又例如

```python
np.extract(cond, array)
```

output

```
array([ 1, 19, 11, 13,  3])# Apply condition on extract directly
```

```python
np.extract(((array < 3) | (array > 15)), array)
```

output

```
array([ 0,  1, 19, 16, 18,  2])
```

### `where()`

`Where()` 用于从一个数组中返回满足特定条件的元素。比如，它会返回满足特定条件的数值的索引位置。`Where()` 与 `SQL` 中使用的 `where condition` 类似，如以下示例所示：

```python
y = np.array([1,5,6,8,1,7,3,6,9])# Where y is greater than 5, returns index position
np.where(y>5)
```

output

```
array([2, 3, 5, 7, 8], dtype=int64),)# First will replace the values that match the condition, 
```

### `percentile()`

`Percentile()` 用于计算特定轴方向上数组元素的第 n 个百分位数。

```python
a = np.array([1,5,6,8,1,7,3,6,9])
print("50th Percentile of a, axis = 0 : ",  np.percentile(a, 50, axis =0))
```

output

```
50th Percentile of a, axis = 0 :  6.0
```

又例如

```python
b = np.array([[10, 7, 4], [3, 2, 1]])
print("30th Percentile of b, axis = 0 : ",  np.percentile(b, 30, axis =0))
```

output

```
30th Percentile of b, axis = 0 :  [5.1 3.5 1.9]
```

## Pandas 必知必会的使用技巧

### **1.计算变量缺失率**

```python
df=pd.read_csv('titanic_train.csv')
def missing_cal(df):
    """
    df :数据集
    return：每个变量(列字段)的缺失率
    """
    missing_series = df.isnull().sum()/df.shape[0]
    missing_df = pd.DataFrame(missing_series).reset_index()
    missing_df = missing_df.rename(columns={'index':'col',0:'missing_pct'})
    missing_df = missing_df.sort_values('missing_pct',ascending=False).reset_index(drop=True)
    return missing_df
missing_cal(df)
```

如果需要计算样本的缺失率分布，只要加上参数axis=1

### **2.获取分组里最大值所在的行方法**

分为分组中有重复值和无重复值两种。无重复值的情况。

```python
df = pd.DataFrame({'Sp':['a','b','c','d','e','f'], 
                   'Mt':['s1', 's1', 's2','s2','s2','s3'],
                   'Value':[1,2,3,4,5,6], 
                   'Count':[3,2,5,10,10,6]})
df.iloc[df.groupby(['Mt']).apply(lambda x: x['Count'].idxmax())]
```

先按Mt列进行分组，然后对分组之后的数据框使用idxmax函数取出Count最大值所在的列，再用iloc位置索引将行取出。有重复值的情况

```python
df["rank"] = df.groupby("ID")["score"].rank(
       method="min",
      ascending=False).astype(np.int64)
df[df["rank"] == 1][["ID", "class"]]
```

对ID进行分组之后再对分数应用rank函数，分数相同的情况会赋予相同的排名，然后取出排名为1的数据

### **3.多列合并为一行**

```python
df = pd.DataFrame({'id_part':['a','b','c','d'], 'pred':[0.1,0.2,0.3,0.4], 'pred_class':['women','man','cat','dog'], 
'v_id':['d1','d2','d3','d1']})
df.groupby(['v_id']).agg({'pred_class': [', '.join],'pred': lambda x: list(x),
'id_part': 'first'}).reset_index()
```

### **4.删除包含特定字符串所在的行**

```python
df = pd.DataFrame({'a':[1,2,3,4], 
'b':['s1', 'exp_s2', 's3','exps4'], 
'c':[5,6,7,8], 'd':[3,2,5,10]})
df[~df['b'].str.contains('exp')]
```

### 5.组内排序

```python
df = pd.DataFrame([['A',1],['A',3],['A',2],['B',5],['B',9]],
                  columns = ['name','score'])
```

介绍两种高效地组内排序的方法。

```python
df.sort_values(['name','score'], ascending = [True,False])
df.groupby('name').apply(lambda x: x.sort_values('score', ascending=False)).reset_index(drop=True)
```

### 6.选择特定类型的列

```python
drinks = pd.read_csv('data/drinks.csv')
# 选择所有数值型的列
drinks.select_dtypes(include=['number']).head()
# 选择所有字符型的列
drinks.select_dtypes(include=['object']).head()
drinks.select_dtypes(include=['number','object','category','datetime']).head()
# 用 exclude 关键字排除指定的数据类型
drinks.select_dtypes(exclude=['number']).head()
```

### 7.字符串转换为数值

```python
df = pd.DataFrame({'列1':['1.1','2.2','3.3'],
                  '列2':['4.4','5.5','6.6'],
                  '列3':['7.7','8.8','-']})
df
df.astype({'列1':'float','列2':'float'}).dtypes
```

用这种方式转换第三列会出错，因为这列里包含一个代表 0 的下划线，pandas 无法自动判断这个下划线。为了解决这个问题，可以使用 to_numeric() 函数来处理第三列，让 pandas 把任意无效输入转为 NaN。

```python
df = df.apply(pd.to_numeric, errors='coerce').fillna(0)
```

### 8.优化 DataFrame 对内存的占用

方法一：只读取切实所需的列，使用usecols参数

```python
cols = ['beer_servings','continent']
small_drinks = pd.read_csv('data/drinks.csv', 
                           usecols=cols)
```

方法二：把包含类别型数据的 object 列转换为 Category 数据类型，通过指定 dtype 参数实现。

```python
dtypes ={'continent':'category'}
smaller_drinks = pd.read_csv('data/drinks.csv',
                             usecols=cols, 
                             dtype=dtypes)
```

### 9.根据最大的类别筛选 DataFrame

```python
movies = pd.read_csv('data/imdb_1000.csv')
counts = movies.genre.value_counts()
movies[movies.genre.isin(counts.nlargest(3).index)].head()
```

### 10.把字符串分割为多列

```python
df = pd.DataFrame({'姓名':['张 三','李 四','王 五'],
                   '所在地':['北京-东城区',
                          '上海-黄浦区',
                          '广州-白云区']})
df1=df.姓名.str.split(' ', expand=True)
df1
```

### 11.把 Series 里的列表转换为 DataFrame

```python
df = pd.DataFrame({'列1':['a','b','c'],'列2':[[10,20], [20,30], [30,40]]})
df

df_new = df.列2.apply(pd.Series)
df1=pd.concat([df,df_new], axis='columns') #水平拼接
df1
```

### 12.用多个函数聚合

```python
orders = pd.read_csv('data/chipotle.csv', sep='\t')
orders.groupby('order_id').item_price.agg(['sum','count']).head()
```

### **13.分组聚合**

```python
import pandas as pd
df = pd.DataFrame({'key1':['a', 'a', 'b', 'b', 'a'],
    'key2':['one', 'two', 'one', 'two', 'one'],
    'data1':np.random.randn(5),
     'data2':np.random.randn(5)})
df
for name, group in df.groupby('key1'):
    print(name)
    print(group)

dict(list(df.groupby('key1')))
```

a
  key1 key2     data1     data2
0    a  one  0.186906 -0.568693
1    a  two -0.710720  1.791654
4    a  one -2.093426 -0.050511
b
  key1 key2     data1     data2
2    b  one -0.525039 -0.861005
3    b  two  0.026497  0.560620

通过字典或Series进行分组

```python
people = pd.DataFrame(np.random.randn(5, 5),
     columns=['a', 'b', 'c', 'd', 'e'],
     index=['Joe', 'Steve', 'Wes', 'Jim', 'Travis'])
mapping = {'a':'red', 'b':'red', 'c':'blue',
     'd':'blue', 'e':'red', 'f':'orange'}
by_column = people.groupby(mapping, axis=1)
by_column.sum()
```

