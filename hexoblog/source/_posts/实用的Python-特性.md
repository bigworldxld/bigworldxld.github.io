---
title: 实用的Python 特性
date: 2022-12-07 16:33:19
tags: python
categories: python
img: /images/Python特性.jpg
---

## 1 一行 For 循环

for 循环是一个多行语句，但是在 Python 中，我们可以使用列表推导式方法在一行中编写 for 循环。以过滤小于**250**的值为例

```python
#For循环在一行
mylist = [200, 300, 400, 500]
#正常方式
result = [] 
for x in mylist: 
    if x > 250: 
        result.append(x) 
print(result) # [300, 400, 500]
#一行代码方式
result = [x for x in mylist if x > 250] 
print(result) # [300, 400, 500]
```

## 2 一行 While 循环

```python
#方法 1 Single Statement 
while True: print(1) #infinite 1
#方法 2 多语句
x = 0 
while x < 5: print(x); x= x + 1 # 0 1 2 3 4 5
```

## 3 一行 IF Else 语句

好吧，要在一行中编写 IF Else 语句，我们将使用三元运算符。三元的语法是**“[on true] if [expression] else [on false]”**。

运算符。

```python
#if Else 在一行中
#Example 1 if else
print("Yes") if 8 > 9 else print("No") # No
#Example 2 if elif else 
E = 2 
print("High") if E == 5 else print("数据STUDIO") if E == 2 else print("Low") # 数据STUDIO 
 
#Example 3 only if
if 3 > 2: print("Exactly") # Exactly
```

## 4 一行合并字典

```python
# 在一行中合并字典
d1 = { 'A': 1, 'B': 2 } 
d2 = { 'C': 3, 'D': 4 }
#方法 1 
d1.update(d2) 
print(d1) # {'A': 1, 'B': 2, 'C': 3, 'D': 4}
#方法 2 
d3 = {**d1, **d2} 
print(d3) # {'A': 1, 'B': 2, 'C': 3, 'D': 4}
```

## 5 一行函数

第一种方法中，我们将使用与三元运算符或单行循环方法相同的函数定义。

第二种方法是用 lambda 定义函数

```python
#函数在一行中
#方法一
def fun(x): return True if x % 2 == 0 else False 
print(fun(2)) # True
#方法2
fun = lambda x : x % 2 == 0 
print(fun(2)) # True 
print(fun(3)) # False
```

## 6 一行递归

```python
# 单行递归
#Fibonaci 单行递归示例
def Fib(x): return 1 if x in {0, 1} else Fib(x-1) + Fib(x-2)
print(Fib(5)) # 8
print(Fib(15)) # 987
```

## 7 一行数组过滤

```python
# 一行中的数组过滤
mylist = [2, 3, 5, 8, 9, 12, 13, 15]
#正常方式
result = [] 
for x in mylist: 
    if x % 2 == 0: 
        result.append(x)
print(result) # [2, 8, 12]
# 单线方式
result = [x for x in mylist if x % 2 == 0] 
print(result) # [2, 8, 12]
```

## 8 一行异常处理

一行中编写这个 Try except 语句吗？通过使用 **exec()** 语句，我们可以做到这一点。

```python
# 一行异常处理
#原始方式
try:
    print(x)
except:
    print("Error")
#单行方式
exec('try:print(x) \nexcept:print("Error")') # 错误
```

## 9 一行列表转字典

使用 Python **enumerate()** 函数将 List 转换为一行字典。在**enumerate()** 中传递列表并使用**dict()** 将最终输出转换为字典格式

```python
# 字典在一行
mydict = ["John", "Peter", "Mathew", "Tom"]
mydict = dict(enumerate(mydict))
print(mydict) # {0: 'John', 1: 'Peter', 2: 'Mathew', 3: 'Tom'}
```

## 10 一行多变量

一行中进行多个变量赋值。下面的示例代码将向你展示如何做到这一点。

```python
#多行变量
#正常方式
x = 5 
y = 7 
z = 10 
print(x , y, z) # 5 7 10
#单行方式
a, b, c = 5, 7, 10 
print(a, b, c) # 5 7 10
```

## 11 一行交换值

```python
#换成一行
#正常方式
v1 = 100
v2 = 200
temp = v1
v1 = v2 
v2 = temp
print(v1, v2) # 200 100
# 单行交换
v1, v2 = v2, v1 
print(v1, v2) # 200 100
```

## 12 一行排序

```python
# 在一行中排序
mylist = [32, 22, 11, 4, 6, 8, 12] 
# 方法 1
mylist.sort() 
print(mylist) # # [4, 6, 8, 11, 12, 22, 32]
print(sorted(mylist)) # [4, 6, 8, 11, 12, 22, 32]
```

## 13 一行读取文件

不使用语句或正常读取方法，也可以正确读取一行文件。

```python
#一行读取文件
#正常方式
with open("data.txt", "r") as file: 
    data = file.readline() 
    print(data) # Hello world
#单行方式
data = [line.strip() for line in open("data.txt","r")] 
print(data) # ['hello world', 'Hello Python']
```

## 14 一行类

```python
# 一行中的类
#普通方式
class Emp: 
    def __init__(self, name, age): 
        self.name = name 
        self.age = age
        emp1 = Emp("云朵君", 22) 
print(emp1.name, emp1.age) # 云朵君 22
#单行方式
#方法 1 带有动态 Artibutes 的 Lambda

Emp = lambda:None; Emp.name = "云朵君"; Emp.age = 22
print(Emp.name, Emp.age) # 云朵君 22

#方法 2 
from collections import namedtuple
Emp = namedtuple('Emp', ["name", "age"]) ("云朵君", 22) 
print(Emp.name, Emp.age) # 云朵君 22
```

## 15 一行分号

分号将向你展示如何使用分号在一行中编写多行代码。

```python
# 一行分号
# 例 1 
a = "Python"; b = "编程"; c = "语言"; print(a, b, c)
# 输出
# Python 编程语言
```

## 16 一行打印

```python
# 一行打印
#正常方式
for x in range(1, 5):
    print(x) # 1 2 3 4
#单行方式
print(*range(1, 5)) # 1 2 3 4
print(*range(1, 6)) # 1 2 3 4 5
```

## 17 一行map函数

```python
#在一行中map
print(list(map(lambda a: a + 2, [5, 6, 7, 8, 9, 10])))
# 输出
# [7, 8, 9, 10, 11, 12]
```

## 18 删除列表第一行中的 Mul 元素

使用 **del** 方法在一行代码中删除 List 中的多个元素，而无需进行任何修改**。**

```python
# 删除一行中的Mul元素
mylist = [100, 200, 300, 400, 500]
del mylist[1::2]
print(mylist) # [100, 300, 500]
```

## 19 一行打印图案

```
# 在一行中打印图案# 
# 正常方式
for x in range(3):
    print('😀')
# 输出
# 😀 😀 😀
#单行方式
print('😀' * 3) # 😀 😀 😀 
print('😀' * 2) # 😀 😀 
print('😀' * 1) # 😀
```

## 20 一行查找质数

单行代码来查找范围内的素数。

```python
# 查找质数
print(list(filter(lambda a: all(a % b != 0 for b in range(2, a)),
                  range(2,20))))
# 输出
# [2, 3, 5, 7, 11, 13, 17, 19]
```

## 21 find() 方法

`find() `方法是一个很棒的功能，可以查找字符串中任何字符的任何起始索引号：

```
# 查找方法
x = "Python"
y = "Hello From Python"
print(x.find("Python")) # 0
print(y.find("From"))  # 6
print(y.find("From Python")) #6
```

## 22 iter()函数

`iter() `方法对于没有任何循环帮助的迭代列表很有用。这是一个内置方法，因此你不需要任何模块：

```
# Iter() 
values = [1, 3, 4, 6]
values = iter(values) 
print(next(values)) # 1 
print(next(values)) # 3 
print(next(values)) # 4 
print(next(values)) # 6
```

## 23. Python 中的文档测试

Doctest 功能将让你测试你的功能并显示你的测试报告。如果你检查下面的示例，你需要在三引号中编写一个测试参数

```python
# Doctest 
from doctest import testmod 
def Mul(x, y) -> int: 
   """
   这个函数返回 x 和 y 参数的 mul
   调用函数，然后是预期的输出：
   >>> Mul(4, 5) 
   20 
   >> > Mul(19, 20) 
   39 
   """ 
   return x * y 
# 调用 testmod 函数
testmod(name='Mul', verbose=True)
```

#### **输出：**

```python
Trying:
    Mul(4, 5)
Expecting:
    20
ok
Trying:
    Mul(19, 20)
Expecting:
    39
**********************************************************************
File "__main__", line 10, in Mul.Mul
Failed example:
    Mul(19, 20)
Expected:
    39
Got:
    380
1 items had no tests:
Mul
**********************************************************************
1 items had failures:
   1 of   2 in Mul.Mul
2 tests in 2 items.
1 passed and 1 failed.
***Test Failed*** 1 failures.
```

## 24. Yield声明

`Yield` 语句是 Python 的另一个令人惊叹的特性，它的工作方式类似于 return 语句，但它不是终止函数并返回，而是返回到它返回给调用者的点：

```python
# Yield声明
def func(): 
    print(1) 
    yield 1 
    print(2) 
    yield 2 
    print(3) 
    yield 3 
    
for call in func(): 
    pass# 输出
# 1 
# 2 
# 3
```

## 25.处理字典缺失键

有时我们正在访问的键不存在于字典中，这会导致键错误。为了处理丢失的键，我们可以使用**get() 方法**而不是**括号方法：**

```python
# 处理字典中的缺失值
dict_1 = {1："x"，2："y"}
# 不要使用get
print(dict_1[3]) # key error
# 使用get  
print(dict_1.get(3)) #  None
```

## 26. For/Else 和 While/Else

当你的循环完成其迭代而没有任何中断时，将执行此 else 语句。

下面的 For 循环示例 else 将执行，但在 While 循环中，我添加了一个不会触发 else 语句的 break 语句：

```python
# 为/否则  
for x in range(5): 
    print(x) 
else: 
    print("Loop Completed") # 执行# While / Else 
i = 0 
while i < 5: 
    break 
else: 
    print("Loop Completed") # 未执行
```

## 27.命名字符串格式化

```python
# 命名格式化字符串
a = "Python" 
b = "工作"# Way 1 
string = "I looking for a {} Programming {}".format(a, b) 
print(string) # I looking for a Python Programming Job

#Way 2 
string = f"I looking for a {a} Programming {b}" 
print(string) # I looking for a Python Programming Job
```

## 28.设置递归限制

设置 Python 程序的递归限制。请查看以下代码示例以更好地理解：

```python
# 设置递归限制
import sys
sys.setrecursionlimit = 2000
print(sys.getrecursionlimit) # 2000
```

## 29 参数拆包

你可以解压缩函数中的任何可迭代数据参数。看看下面的代码示例：

```python
# 参数解包  
def func(x, y): 
    print(x, y) 
list_1 = [100, 200] 
dict_1 = {'x':300, 'y':400} 
func(*list_1) 
func(**dict_1)  
# 输出
# 100 200 
# 300 400
```

## 30. 多行字符串

写不带三引号的多行字符串。看看下面的代码示例：

```python
# 多行字符串
str1= "你是否正在寻找免费的Python学习材料" \ 
      "欢迎来到 " \
      "公众号 数据STUDIO"
```

