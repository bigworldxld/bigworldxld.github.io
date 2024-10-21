---
title: python字典处理技巧
tags: python
categories: python
img: /images/python.jpg
abbrlink: 5870
date: 2023-02-19 09:55:25
---

## Python 字典处理技巧

### 1.使用联合运算符合并字典

写 for 循环来组合不同字典的元素。但是从 Python 3.9 开始，你再也不需要手动完成了。

使用联合操作是合并字典的最简单方法。

```python
cities_us = {'New York City': 'US', 'Los Angeles': 'US'}
cities_uk = {'London': 'UK', 'Birmingham': 'UK'}

cities = cities_us|cities_uk
print(cities)
# {'New York City': 'US', 'Los Angeles': 'US', 'London': 'UK', 'Birmingham': 'UK'}
```

您还可以使用 :|= 进行就地更新

```python
cities_us = {'New York City': 'US', 'Los Angeles': 'US'}
cities_uk = {'London': 'UK', 'Birmingham': 'UK'}

cities_us |= cities_uk
print(cities_us)
# {'New York City': 'US', 'Los Angeles': 'US', 'London': 'UK', 'Birmingham': 'UK'}
```

### 2.带星号的字典解包

由于它的简单性，如果可能的话，我将始终使用联合运算符。

可以改用字典解包技巧：

```python
cities_1 = {'New York City': 'US', 'Los Angeles': 'US'}
cities_2 = {'London': 'UK', 'Birmingham': 'UK'}

cities = {**cities_1, **cities_2}
print(cities)
# {'New York City': 'US', 'Los Angeles': 'US', 'London': 'UK', 'Birmingham': 'UK'}
```

### 3.使用字典推导式来创建词典

与 Python 中的列表推导式一样，字典推导式是一种创建字典的绝妙方式，它使我们可以灵活地过滤数据，因为它可以包含语句。dict理解的模板如下：if

D = {key: value for key,value in iterable (if condition)}

下面的示例利用了 dict 推导的强大功能，在一行代码中从两个列表生成一个 dict：

```python
cities = ['London', 'New York', 'Tokyo', 'Cambridge', 'Oxford']
countries = ['UK', 'US', 'Japan', 'UK', 'UK']
uk_cities = {city:country for city, country in zip(cities, countries) if country == 'UK'}
print(uk_cities)
# {'London': 'UK', 'Cambridge': 'UK', 'Oxford': 'UK'}
```

### 4.反转字典的键和值

有许多单行方法可以反转字典的键和值。以下是我最喜欢的三种方法：

```python
cities = {'London': 'UK', 'Tokyo': 'Japan', 'New York': 'US'}
# Method 1
reversed_cities = {v: k for k, v in cities.items()}
print(reversed_cities)
# {'UK': 'London', 'Japan': 'Tokyo', 'US': 'New York'}

# Method 2
reversed_cities = dict(zip(cities.values(), cities.keys()))
print(reversed_cities)

# Method 3
reversed_cities = dict(map(reversed, cities.items()))
print(reversed_cities)
```

### 5.将列表转换为字典

列表也是一种常用的数据结构。在某些情况下，我们需要将列表转换为字典。

如果列表包含“键”和“值”：

```python
cities = [('London', 'UK'), ('New York', 'US'), ('Tokyo', 'Japan')]
d_cities = dict(cities)
print(d_cities)
# {'London': 'UK', 'New York': 'US', 'Tokyo': 'Japan'}
```

如果不：

```python
cities = ['London', 'Leeds', 'Birmingham']
d_cities = dict.fromkeys(cities,'UK') # set the default value to 'UK' 
print(d_cities)
# {'London': 'UK', 'Leeds': 'UK', 'Birmingham': 'UK'}
```

### 6. 字典排序

只需要一行代码就可以根据需要对列表进行排序：

```python
cities = {'London': '2', 'Tokyo': '3', 'New York': '1'}
print(sorted(cities.items(),key=lambda d:d[1]))
# [('New York', '1'), ('London', '2'), ('Tokyo', '3')]
```

### 7. 使用默认字典

当您通过不存在的键获取字典的值时，将引发异常：

```python
>>> city = {'UK':'London','Japan':'Tokyo'}
>>> print(city['Italy'])
# KeyError: 'Italy'
```

为避免意外问题，一个好的解决方案是使用 :defaultdict

```python
from collections import defaultdict
city = defaultdict(str)
city['UK'] = 'London'
print(city['Italy'])
```

如上所示，我们可以避免异常，即使需要一个不存在的 key.defaultdict()

### 8. 使用计数器

如果每个字母在一个字符串中出现了多少次，最直观的方法可能是写一个 for 循环来遍历所有字母并计算数字。

但是如果你知道，上面的任务将像下面的代码一样简单：Counter

```python
from collections import Counter

author = "Yang Zhou"
chars = Counter(author)
print(chars)
# Counter({'Y': 1, 'a': 1, 'n': 1, 'g': 1, ' ': 1, 'Z': 1, 'h': 1, 'o': 1, 'u': 1})
```

## 展开嵌套列表

模拟数据

```python
data = [[1,2,3],[4],[5,6,7],[8,9],[10]]  # 模拟数据
data
[[1, 2, 3], [4], [5, 6, 7], [8, 9], [10]]
```

这份模拟的数据有2个特点：

- 嵌套列表只有两层

- 里面的元素也全部是列表类型

  方式1：for循环

```python
# 导入Iterable 模块
from collections import Iterable 
sum_data = []

for i in data:
    if isinstance(i,Iterable):  # 如果可迭代（比如列表形式）
        for j in i:  # 再次循环追加元素
            sum_data.append(j)  
    else:
        sum_data.append(i)  # 否则直接追加
       
sum_data
```

[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

方式2：列表推导式

for循环能够实现，那么列表推导式肯定也可以：

```python
sum_data = [i for j in data  if isinstance(j,Iterable) for i in j]  # 循环一定是从大到小
sum_data
```

[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

方式3：使用itertools库

借助第三方的库itertools：

```python
import itertools
# 通过chain方法从可迭代对象中生成；最后展开成列表
sum_data = list(itertools.chain.from_iterable(data))
sum_data
```

方式4：使用sum函数

```python
sum_data = sum(data, [])
sum_data
```

方式5：Python自加

```python
sum_data = []

for i in data:
    sum_data += i  # 实现自加
sum_data
```

方式6：extend函数

如何快速理解python的extend函数，给个案例

```python
# 如何理解python的extend函数

list1 = [1,2,3,4]
list1.extend([5,6])  # 追加功能extend；就地修改

list1  # list1已经发生了变化
```

[1, 2, 3, 4, 5, 6]

```python
sum_data = [] 

for i in data:
    sum_data.extend(i)   # 对空列表逐步追加
    
sum_data
```

[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

方式7：使用递归函数

data
[[1, 2, 3], [4], [5, 6, 7], [8, 9], [10]]

```python
def flatten(data):  # 定义递归函数
    sum_data = []
    for i in data:
        if isinstance(i, Iterable):  # 如果i是可迭代的对象（列表等），调用函数本身；直到执行else语句
            sum_data.extend(flatten(i))
        else:
            sum_data.append(i)
    
    return sum_data
sum_data = flatten(data)  # 调用递归函数
sum_data
```



## Python新模块

### Pathlib

pathlib 与旧的 os.path 相比具有许多优点 - 虽然 os 模块以原始字符串格式表示路径，但 pathlib 使用面向对象的样式，这使得它更具可读性和编写自然：

```python
from pathlib import Path
import os.path

# 老方式
two_dirs_up = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
# 新方式，可读性强
two_dirs_up = Path(__file__).resolve().parent.parent
```

路径被视为对象而不是字符串这一事实也使得可以创建一次对象，然后查找其属性或对其进行操作：

```python
readme = Path("README.md").resolve()
print(f"Absolute path: {readme.absolute()}")
# Absolute path: /home/martin/some/path/README.md
print(f"File name: {readme.name}")
# File name: README.md
print(f"Path root: {readme.root}")
# Path root: /
print(f"Parent directory: {readme.parent}")
# Parent directory: /home/martin/some/path
print(f"File extension: {readme.suffix}")
# File extension: .md
print(f"Is it absolute: {readme.is_absolute()}")
# Is it absolute: True
```

我最喜欢 pathlib 的一个特性是可以使用 /（“除法”）运算符来连接路径：

```python
# Operators:
etc = Path('/etc')

joined = etc / "cron.d" / "anacron"
print(f"Exists? - {joined.exists()}")
# Exists? - True
```

重要的是要注意 pathlib 只是替代 os.path 而不是整个 os 模块， 它还包括 glob 模块的功能，因此如果你习惯于将 os.path 与 glob.glob 结合使用，那么你可以完全用pathlib替代它们。

在linux下面，一般如果你自己使用系统的时候，是可以用~来代表＂/home/你的名字/＂这个路径的．但是python是不认识~这个符号的，如果你写路径的时候直接写＂~/balabala＂，程序是跑不动的．所以如果你要用~，你就应该用这个os.path.expanduser把~展开

在上面的片段中，我们展示了一些方便的路径操作和对象属性，但 pathlib 还包括你习惯于 os.path 的所有方法，例如：

```python
print(f"Working directory: {Path.cwd()}")  # same as os.getcwd()
# Working directory: /home/martin/some/path
Path.mkdir(Path.cwd() / "new_dir", exist_ok=True)  # same as os.makedirs()
print(Path("README.md").resolve())  # same as os.path.abspath()
# /home/martin/some/path/README.md
print(Path.home())  # same as os.path.expanduser()
# /home/martin
```

### Secrets

说到 os 模块，你应该停止使用的另一部分是 os.urandom。相反，你应该使用自 Python 3.6 以来可用的新秘密模块：

```python
# 老方式:
import os
length = 64
value = os.urandom(length)
print(f"Bytes: {value}")
# Bytes: b'\xfa\xf3...\xf2\x1b\xf5\xb6'
print(f"Hex: {value.hex()}")
# Hex: faf3cc656370e31a938e7...33d9b023c3c24f1bf5

# 新方式:
import secrets
value = secrets.token_bytes(length)
print(f"Bytes: {value}")
# Bytes: b'U\xe9n\x87...\x85>\x04j:\xb0'
value = secrets.token_hex(length)
print(f"Hex: {value}")
# Hex: fb5dd85e7d73f7a08b8e3...4fd9f95beb08d77391
```

os.urandom(n)函数在python官方文档中做出了这样的解释

函数定位：The returned data should be **unpredictable** enough for cryptographic applications。
意思就是，返回一个有n个byte那么长的一个string，然后很适合用于加密。

引入secrets模块的原因是因为人们使用随机模块来生成密码等，即使随机模块不产生密码安全令牌。

根据文档，随机模块不应用于安全目的， 你应该使用 secrets 或 os.urandom，但 secrets 模块绝对更可取，因为它比较新，并且包含一些用于十六进制令牌的实用程序/便利方法以及 URL 安全令牌。

### Zoneinfo

在 Python 3.9 之前，没有用于时区操作的内置库，所以每个人都在使用 pytz，但现在我们在标准库中有 zoneinfo，所以是时候切换了。

```python
from datetime import datetime
import pytz  # pip install pytz

dt = datetime(2022, 6, 4)
nyc = pytz.timezone("America/New_York")

localized = nyc.localize(dt)
print(f"Datetime: {localized}, Timezone: {localized.tzname()}, TZ Info: {localized.tzinfo}")

# 新方式:
from zoneinfo import ZoneInfo

nyc = ZoneInfo("America/New_York")
localized = datetime(2022, 6, 4, tzinfo=nyc)
print(f"Datetime: {localized}, Timezone: {localized.tzname()}, TZ Info: {localized.tzinfo}")
# Datetime: 2022-06-04 00:00:00-04:00, Timezone: EDT, TZ Info: America/New_York
```

datetime 模块将所有时区操作委托给抽象基类 datetime.tzinfo， 这个抽象基类需要一个具体的实现——在引入这个很可能来自 pytz 的模块之前。

**tuple类型数据的获取：**

元组里面的数据获取只能通过下标的方式去获取，

比如 ：

```cobol
a = ('username', 'age', 'phone')
```

要获取username的话 ，就需要用a[0]的方式去获取.

namedtuple是继承自tuple的子类。namedtuple创建一个和tuple类似的对象，而且对象拥有可访问的属性。

```python
from collections import namedtuple
 
# 定义一个namedtuple类型User，并包含name，sex和age属性。
User = namedtuple('User', ['name', 'sex', 'age'])
 
# 创建一个User对象
user = User(name='kongxx', sex='male', age=21)
 
# 也可以通过一个list来创建一个User对象，这里注意需要使用"_make"方法
user = User._make(['kongxx', 'male', 21])
 
print user
# User(name='user1', sex='male', age=21)
 
# 获取用户的属性
print user.name
print user.sex
print user.age
 
# 修改对象属性，注意要使用"_replace"方法
user = user._replace(age=22)
print user
# User(name='user1', sex='male', age=21)
 
# 将User对象转换成字典，注意要使用"_asdict"
print user._asdict()
# OrderedDict([('name', 'kongxx'), ('sex', 'male'), ('age', 22)])
```

### Dataclasses

Python 3.7 的一个重要补充是 dataclasses 包，它是 namedtuple 的替代品。

你可能想知道为什么需要替换 namedtuple？以下是你应该考虑切换到数据类的一些原因：

- 它可以是可变的

- 默认提供 **repr**、**eq**、**init**、**hash** 魔术方法，

- 允许指定默认值，

- 支持继承。

  此外，数据类还支持 **frozen** 和 **slots**（从 3.10 开始）属性以提供与命名元组的特征奇偶校验。

```python
# 老方式:
# from collections import namedtuple
from typing import NamedTuple
import sys

User = NamedTuple("User", [("name", str), ("surname", str), ("password", bytes)])

u = User("John", "Doe", b'tfeL+uD...\xd2')
print(f"Size: {sys.getsizeof(u)}")
# Size: 64

# 新方式:
from dataclasses import dataclass

@dataclass()
class User:
   name: str
   surname: str
   password: bytes

u = User("John", "Doe", b'tfeL+uD...\xd2')

print(u)
# User(name='John', surname='Doe', password=b'tfeL+uD...\xd2')

print(f"Size: {sys.getsizeof(u)}, {sys.getsizeof(u) + sys.getsizeof(vars(u))}")
# Size: 48, 152
```

在上面的代码中，我们还包含了大小比较，因为这是 namedtuple 和数据类之间的较大差异之一

如果你不想切换并且出于某种原因真的想使用命名元组，那么你至少应该使用键入模块而不是collections中的 NamedTuple：

```python
# 不好方式的:
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])

# 更好的方式:
from typing import NamedTuple
class Point(NamedTuple):
    x: float
    y: float
```











