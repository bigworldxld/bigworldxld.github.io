---
title: go语言基础
categories: OPL
tags: GoLang
img: /images/go2.jpg
abbrlink: 25470
date: 2022-04-21 22:15:20
---

## 背景

go的官方文档：https://studygolang.com/pkgdoc

搭建Go语言开发环境，Go语言的开发工具包下载地址：https://golang.org/dl/，(墙内下载地址http://www.golangtc.com/download)

go sdk路径添加到环境变量GO_PATH里

## 基础

### helloworld

go语言的helloWorld如下

```go
package main  // 必须打main包
import "fmt"
func main()  {
    fmt.Println("helloWorld")
}
```

main函数必须在main目录下，包名则必须和上级目录名一致(main)；一个项目必须有且只有一个main目录(或main包)

go程序既可以直接运行，亦可以先编译，再运行

### 变量

三种声明方式

```go
package main
import "fmt"
func main() {
    var i int  = 10 	// var 变量名 类型 = 值
    var j = 1.2 		// var 变量名 = 值
    name := "szc" 		// 变量名 := 值，自动推导类型
    fmt.Println("i = ", i, ", j = " , j , ", name = ", name)
}
```

一次声明多个变量，变量名和值一一对应

```go
var a, sex, b = 1, "male", 7
a, sex, b := 2, "male", 4
```

函数外声明全局变量

```go
var (
    n1 = 1
    n2 = 2
    n3 = 3
)
var n4 = "n4"
func main() {
    fmt.Println("n1 = ", n1, ", n2 = ", n2, "n3 = ", n3, ", n4 = ", n4)
}
```

变量声明后必须使用，而且不能隐式改变类型(int转float)

### 常量

常量必须赋初值，而且不可更改

```go
    const tax int = 1
    const name_ = "szc"
    const b = 4 / 2
//    const b_ = getVal() 	// 编译期值不确定
//    num := 1
//    const b_ = num / 2  	// 编译期值不确定
//    const tax_0 int 	// 必须赋初值
//    tax = 2	 // 不可修改
```

常量只能修饰布尔、数值、字符串类型

也可以这么声明常量，可以在函数里面声明

**iota是go语言的常量计数器**，只能在常量的表达式中使用。

 使用iota时只需要记住以下两点

1. iota在const关键字出现时将被重置为0。

2. const中每新增一行常量声明将使iota计数一次(iota可理解为const语句块中的行索引)。

 使用iota能简化定义，在定义枚举时很有用。

```go
const (
    a = iota  //0
    b = iota  //1
    c = iota  //2
    d,e = iota,iota  //3 3
)
fmt.Println(a, b, c, d, e) // 0 1 2 3 3 依次加1
```

上面b和c可以不写= iota，但是a必须写

### 标识符

#### **标识符概念**

1. Golang对各种变量、方法、函数等命名时使用的字符序列称为标识符
2. 凡是自己可以起名字的地方都叫标识符

#### **标识符的命名规则**

1. 由26个英文字母大小写, 0-9, _组成
2. 数字不可以开头。

1. Golan中严格区分大小写
2. 标识符不能包含空格
3. 下划线”_”本身在Go中是一个特殊的标识符,称为空标识符。可以代表任何其它的标识符,但是它对应的值会被忽略(比如:忽略某个返回值)。所以仅能被作为占位符使用,不能作为标识符使用
4. 不能以系统保留关键字作为标识符,比如 break，if 等等

#### 关键字

下面列出25个Go语言的关键字或保留字：

| break    | default     | func   | interface | select |
| :------- | :---------- | :----- | :-------- | :----- |
| case     | defer       | go     | map       | struct |
| chan     | else        | goto   | package   | switch |
| const    | fallthrough | if     | range     | type   |
| continue | for         | import | return    | var    |

除了以上介绍的这些关键字，Go 语言还有 36 个预定义标识符，其中包含了基本类型的名称和一些基本的内置函数，见下表：

| append | bool    | byte    | cap     | close  | complex | complex64 | complex128 | uint16  |
| :----- | :------ | :------ | :------ | :----- | :------ | :-------- | :--------- | :------ |
| copy   | false   | float32 | float64 | imag   | int     | int8      | int16      | uint32  |
| int32  | int64   | iota    | len     | make   | new     | nil       | panic      | uint64  |
| print  | println | real    | recover | string | true    | uint      | uint8      | uintptr |

#### 标识符的命名规范

1. 包名：保持 package的名字和目录保持一致，包名统一使用单数形式，尽量采取有意义的包名，简短，有意义，不要和标准库冲突
2. 变量名、函数名、常量名：采用驼峰法
3. 如果变量名、函数名、常量名**首字母大写,则可以被其他的包访问**；如果**首字母小写则只能在本包中使用**(注：可以简单的理解成,首字母大写是公有的,首字母是私有的)，在 golan没有public, private等关键字
4. 文件命名：一律采用小写，不用驼峰式，尽量见名思义，看见文件名就可以知道这个文件下的大概内容。
   其中测试文件以_test.go结尾，除测试文件外，命名不出现。

### 数据类型

{% asset_img 数据类型.png This is an example image %}

- 数值型中的int32又称为rune，可保存一个unicode码点。int和uint的大小和操作系统位数一样，32位OS(系统)则为4字节，64位OS则为8字节。浮点数默认64位，整数默认int。
- float32 精确度为小数点后6位，float64为14位；

#### 值类型与引用类型

值类型：基本数据类型、数组、结构体。变量直接存储值，通常存储于栈中，函数传参时使用值传递

引用类型：指针、切片、映射、管道、接口等。变量存储的是值的地址，通常存储于堆中，会发生GC，函数传参时使用引用传递。

**函数参数的两种传递方式**：

1. 值传递
2. 引用传递

不管是值传递还是引用传递，传递给函数的都是变量的副本，不同的是，值传递的是值的拷贝，引用传递的是地址的拷贝，一般来说，地址拷贝效率高，因为数据量小，而值拷贝决定拷贝的数据大小，数据越大，效率越低。

#### 查看变量类型

查看变量类型：

```go
a, sex:= 2, "male"
 
fmt.Printf("a的类型:%T，sex的类型:%T\n", a, sex)
```

查看变量占用内存大小时，先导入unsafe和fmt包

```go
import (
    "fmt"
    "unsafe"
)
```

再调用unsafe.Sizeof函数就行

```go
fmt.Printf("a占用内存大小：%d, sex占用内存大小：%d", unsafe.Sizeof(a), unsafe.Sizeof(sex))
```

#### 字符与字符串

输出字符时，需要格式化输出，否则会输出的它的ascii值

```go
c1 := 's'
c2 := '0'
fmt.Println("c1 = ", c1, ", c2 = ", c2)
fmt.Printf("c1 = %c, c2 = %c\n", c1, c2)
```

输出汉字和对应unicode码值

```go
c3 := '宋'
fmt.Printf("c3 = %c, 对应unicode码值: %d\n", c3, c3)
```

跨行字符串，用`包住

```go
var s = `
一个人真正炫耀的东西，
是善良，是教养，是包容，
是见过世面的涵养，
向阳而生，做一个温暖的人，
不卑不亢清澈善良。
`
```

多行拼接字符串，要在+后面换行，而不是字符串后面

```go
s1 := "abc" +
 " def" + "hij"
```

#### 类型转换

不同数据类型之间必须显式类型转换

```go
a1 := 1.2
a2 := int(a1)
fmt.Println("a2 = ", a2)
```

如果范围大转换成范围小的，可能会发生精度损失，以下是例子：

```go
var i1 int32 = 12
var i2 int8
var i3 int8
i2 = int8(i1) + 127 // 运行时溢出，得不到想要结果
i3 = int(i1) + 128 // 直接溢出，编译错误
fmt.Println("i2 = ", i2)
```

基本数据类型转string:

```go
var s0 = fmt.Sprintf("%d", n1)
fmt.Printf("s type:%T, s = %v\n", s0, s0)
s0 = fmt.Sprintf("%t", b)
fmt.Printf("s type:%T, s = %v\n", s0, s0)
```

**%v 默认格式输出，%t 布尔值输出**

也可以用strconv包中的函数进行转换。用之前先导入strconv包

```go
import (
    "fmt"
    "strconv"
)
```

然后调用函数进行转换

```go
var num int = 12345	
str = strconv.Itoa(num)	//int 转string

s0 = strconv.FormatInt(int64(n1), 10) // 10表示十进制
   fmt.Printf("s type:%T, s = %v\n", s0, s0)

   s0 = strconv.FormatFloat(a1, 'f', 10, 64) // 'f'表示浮点数类型、10表示精度10位，64表示float64
   fmt.Printf("s type:%T, s = %v\n", s0, s0)

   s0 = strconv.FormatBool(b)
   fmt.Printf("s type:%T, s = %v\n", s0, s0)
```

string转基本类型：

也是用strconv包中的Parse方法，但Parse方法会返回两个值：转换的值，以及转换错误

```go
var b2, _ = strconv.ParseBool(s0) // go中可以有多个返回值，_表示接收但忽略
fmt.Printf("b2 type:%T, b2 = %v\n", b2, b2)

var i0, _ = strconv.ParseInt("1233", 10, 64) // 后两个参数分别表示进制和转换成int的位数
fmt.Printf("i0 type:%T, i0 = %v\n", i0, i0)
 
var f0, _ = strconv.ParseFloat("21.291", 64) //后面的参数表示转换成float的位数
fmt.Printf("f0 type:%T, f0 = %v\n", f0, f0)
```

如果待转换的string不合法，就会转换成对应类型的默认值(0)

#### 指针

1. 基本数据类型，变量存的就是值,也叫值类型

2. 获取变量的地址，用&，比如:var num int,获取num的地址: &num

3. 指针类型，指针变量存的是一个地址，这个地址指向的空间存的才是值 如:

   ```go
   var ptr *int = &num   // ptr是指针变量  ptr的类型为 *int   值为&num
   ```

4. 获取指针类型所指向的值，使用:，比如: var ptr int，使用ptr获取p指向的值

和C里面的指针类似

```go
i := 1
ptr0 := &i
 
fmt.Printf("%x, %d, %x", ptr0, *ptr0, &ptr0)  //*ptr0表示取出ptr0指向地址对应的值
```

**%x表示十六进制**

同样，通过指针改变变量的值也是一样

```go
(*ptr0) = (*ptr0) * 10
fmt.Printf("%v\n", i)
```

### 运算符

#### 算数运算符

下表列出了所有Go语言的算术运算符。假定 A 值为 10，B 值为 20。

| 运算符 | 描述 | 实例               |
| :----- | :--- | :----------------- |
| +      | 相加 | A + B 输出结果 30  |
| -      | 相减 | A - B 输出结果 -10 |
| *      | 相乘 | A * B 输出结果 200 |
| /      | 相除 | B / A 输出结果 2   |
| %      | 求余 | B % A 输出结果 0   |
| ++     | 自增 | A++ 输出结果 11    |
| –      | 自减 | A– 输出结果 9      |

**注：**

1. / 运算符需要保留小数位时需要有浮点型参与运算
2. % 结果计算公式为： **a % b = a - a / b \* b**
3. Golang的自增自诚只能当做一个独立语言使用时,不能这样使 b := a++或者b := a–

1. Golang的++ 和 – 只能写在变量的g后面,不能写在变量的前面,即:只有a++，a–没有++a，–a

1. Golang的设计者去掉c/java中的自增自诚的容易混淆的写法,让 Galang更加简洁,统一 (强制性的)

#### 关系运算符

下表列出了所有Go语言的关系运算符。假定 A 值为 10，B 值为 20。

| 运算符 | 描述                                                         | 实例              |
| :----- | :----------------------------------------------------------- | :---------------- |
| ==     | 检查两个值是否相等，如果相等返回 True 否则返回 False。       | (A == B) 为 False |
| !=     | 检查两个值是否不相等，如果不相等返回 True 否则返回 False。   | (A != B) 为 True  |
| >      | 检查左边值是否大于右边值，如果是返回 True 否则返回 False。   | (A > B) 为 False  |
| <      | 检查左边值是否小于右边值，如果是返回 True 否则返回 False。   | (A < B) 为 True   |
| >=     | 检查左边值是否大于等于右边值，如果是返回 True 否则返回 False。 | (A >= B) 为 False |
| <=     | 检查左边值是否小于等于右边值，如果是返回 True 否则返回 False。 | (A <= B) 为 True  |

#### 逻辑运算符

下表列出了所有Go语言的逻辑运算符。假定 A 值为 True，B 值为 False。

| 运算符 | 描述                                                         | 实例               |
| :----- | :----------------------------------------------------------- | :----------------- |
| &&     | 逻辑 AND 运算符。 如果两边的操作数都是 True，则条件 True，否则为 False。 | (A && B) 为 False  |
| \|\|   | 逻辑 OR 运算符。 如果两边的操作数有一个 True，则条件 True，否则为 False。 | (A \|\| B) 为 True |
| !      | 逻辑 NOT 运算符。 如果条件为 True，则逻辑 NOT 条件 False，否则为 True。 | !(A && B) 为 True  |

1. &&也叫短路与：如果第一个亲件为 False,则第二个条件不会判断,最终结果为 false
2. ||叫短路或：如果第一个亲件为true,则第二个条件不会判断,最终结果为true

#### 位运算符

位运算符对整数在内存中的二进制位进行操作。假定 A 为60，B 为13

| 运算符 | 描述                                                         | 实例                                   |
| :----- | :----------------------------------------------------------- | :------------------------------------- |
| &      | 按位与运算符”&”是双目运算符。 其功能是参与运算的两数各对应的二进位相与。 | (A & B) 结果为 12, 二进制为 0000 1100  |
| \|     | 按位或运算符”\|”是双目运算符。 其功能是参与运算的两数各对应的二进位相或 | (A \| B) 结果为 61, 二进制为 0011 1101 |
| ^      | 按位异或运算符”^”是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1。 | (A ^ B) 结果为 49, 二进制为 0011 0001  |
| <<     | 左移运算符”<<”是双目运算符。左移n位就是乘以2的n次方。 其功能把”<<”左边的运算数的各二进位全部左移若干位，由”<<”右边的数指定移动的位数，高位丢弃，低位补0。 | A << 2 结果为 240 ，二进制为 1111 0000 |
| >>     | 右移运算符”>>”是双目运算符。右移n位就是除以2的n次方。 其功能是把”>>”左边的运算数的各二进位全部右移若干位，”>>”右边的数指定移动的位数。 | A >> 2 结果为 15 ，二进制为 0000 1111  |

演示示例：

```go
package main

import "fmt"
func main() {
   var a int = 60     //二进制是：111100  
   var b int = 13     //二进制是：001101

   fmt.Printf("%b\n%d\n",a&b,a&b)    //二进制是：1100,对应的十进制是12。说明&进行的是上下对应位的与操作
   fmt.Printf("%b\n%d\n",a|b,a|b)    //二进制是：111101,对应的十进制是61。说明&进行的是上下对应位的或操作
   fmt.Printf("%b\n%d\n",a^b,a^b)    //二进制是：110001,对应的十进制是49。^位运算符是上下对应位不同时，值为1
}
```

左移右移运算符示例（实现计算器存储单位）：

```go
package main
import "fmt"
const (   
    KB float64 = 1<<(10*iota)      //iota是 const 结构里面，定义常量行数的索引器，每个 const 里面，iota 都从 0 开始
    MB                             //下面是一个省略调用，继承了上面的表达式
    GB
    TB
    PB
)
func main() {
   fmt.Printf("1MB = %vKB\n",MB) 
   fmt.Printf("1GB = %vKB\n",GB)
   fmt.Printf("1TB = %vKB\n",TB)
   fmt.Printf("1PB = %vKB\n",PB)
}
```

运行结果：
1MB = 1024KB
1GB = 1.048576e+06KB
1TB = 1.073741824e+09KB
1PB = 1.099511627776e+12KB

#### 赋值运算符

下表列出了所有Go语言的赋值运算符。假定 A 为21

| 运算符 | 描述                                           | 实例                                       |
| :----- | :--------------------------------------------- | :----------------------------------------- |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A 将 A 赋值给 C，结果：21              |
| +=     | 相加后再赋值                                   | C += A 等于 C = C + A，结果：42            |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A，结果：21            |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A，结果：441           |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A，结果：21            |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A，结果：0//不记入计算 |
| <<=    | 左移后赋值                                     | C <<= 2 等于 C = C << 2，结果：84          |
| >>=    | 右移后赋值                                     | C >>= 2 等于 C = C >> 2，结果：21          |
| &=     | 按位与后赋值                                   | C &= 2 等于 C = C & 2，结果：0             |
| ^=     | 按位异或后赋值                                 | C ^= 2 等于 C = C ^ 2，结果：2             |
| \|=    | 按位或后赋值                                   | C \|= 2 等于 C = C \| 2，结果：2           |

1. 赋值运算符的运算顺序为：从右往左
2. 运算符的左边只能是变量，右边可以是变量、表达式、常量

#### 其他运算符

| 运算符 | 描述             | 实例                       |
| :----- | :--------------- | :------------------------- |
| &      | 返回变量存储地址 | &a; 将给出变量的实际地址。 |
| *      | 指针变量。       | *a; 是一个指针变量         |

内存地址和指针的示例：打印变量类型用%T

```go
package main
import "fmt"

func main() {
   var a int = 4
   var b int32
   var c float32
   var ptr *int
   fmt.Printf("a 变量类型为 = %T\n", a )     //输出变量类型%T
   fmt.Printf("b 变量类型为 = %T\n", b )
   fmt.Printf("c 变量类型为 = %T\n", c )
   ptr = &a    
   fmt.Printf("a 的内存地址为 = %p",ptr)     //go里面的内存块地址通常都是用十六进制表示的，因此输出：0x10414020a
   fmt.Printf("*ptr 为 %d\n", *ptr)        //这是个指向a的内存地址的指针，因此输出：4
}
```

#### 运算符优先级

有些运算符拥有较高的优先级，二元运算符的运算方向均是从左至右。下表列出了所有运算符以及它们的优先级，由上至下代表优先级由高到低：

```go
优先级     运算符
 7      ^ !
 6      * / % << >> & &^
 5      + - | ^
 4      == != < <= >= >
 3      <-
 2      &&
 1      ||
```

1. 只有单目运算符、赋值运算符是从右向左运算的。其余都是从左向右
2. 可以通过使用括号来临时提升某个表达式的整体运算优先级。

### 打包

- 包的本质是创建不同的文件夹，来存放程序文件
- 包名通常和文件夹名保持一致。
- 文件中变量、函数名首字母大写，则为public，小写则为包私有
- 在import包时，路径从$GOPATH的src下开始，不用带src，编译器会从src下开始引入
- 然后就可以引用model包下首字母大写的变量或函数了
- 如果包名过长，可以给包起别名。一旦起别名，原来的名就不能再使用了
- 在同一个包下不能有相同的函数名和变量名

```go
fmt.Printf(model.Name)   // 包名.函数/变量
```

model包下的test_model.go内容如下所示，文件不用引用。引用目录就行

```go
package model   //打包
var Name = "Jason"
var age = 23
```

**编译go项目：**

1. 进入到项目的$GOPATH目录下
2. 运行 go build -o bin/name.exe 路径 // -o 为给生成的可执行文件指定位置和命名， 路径为src下开始到main包 例： /go_code/helloword/main

### 读取控制台数据

调用fmt.Scan等方法即可

```go
var j string
fmt.Scanln(&j) // Scanln读取一行
fmt.Println("j = ", j)
```

或者指定输入格式

```go
var j string
var m float32
var n bool
fmt.Scanf("%f%s%t", &m, &j, &n)
fmt.Println("j = ", j, "m = ", m, "n = ", n)
```

输入时按空格或回车区分即可

### 流程控制

#### if-else 流程控制

基本语法

> if 表达式1 { // 表达式可以写()，官方不推荐写
>
>  //代码块
>
> } else if 表达式2{ // else if 可省略，不能换行写
>
>  //代码块
>
> } else { // else 可省略，不能换行写
>
>  //代码块
>
> }

#### switch分支结构

基本语法

> switch 表达式 {
>
>  case 表达式1，表达式2，…… ：
>
>  // 语句块1
>
>  case 表达式3，表达式4，…… ：
>
>  // 语句块2
>
>  // 多个case，结构同上
>
>  default ：
>
>  // 语句块3
>
> }

**switch 的执行的流程**：

1. 先执行表达式，得到值，然后和 case 的表达式进行比较，如果相等，就匹配到，然后执行对应的 case 的语句块，然后**退出** switch 控制。
2. 如果 switch的表达式的值没有和任何的 case 的表达式匹配成功，则执行 default的语句块。
3. 多个表达式使用**逗号**间隔

#### for循环

基本语法

> for 循环变量初始化 ；循环条件 ；循环变量迭代 {
>
>  //循环操作
>
> }

例：

```go
for  i := 0 ; i < 10 ; i++  {
	//代码块
}
```

**注：Go中没有while，do…while循环，Go语言支持goto跳转，但不支持使用**

#### break

break用于终止某个语句块的执行，用于中断当前for或跳出switch语句

break 出现在多层嵌套循环中可以使用标签（label）表明要终止哪个循环

```go
for i := 0 ; i < 4 ; i++ {
	for j := 0 ; j < 10 ; j++ {
		if j==2 {
			break  // break  默认会跳出最近的for循环
		}
		fmt.Println("j=",j)
	}
}
```

输出结果：

> j= 0
> j= 1
> j= 0
> j= 1
> j= 0
> j= 1
> j= 0
> j= 1

使用标签

```go
label1:
	for i := 0 ; i < 4 ; i++ {
		for j := 0 ; j < 10 ; j++ {
			if j==2 {
				break  label1	// 一旦触发直接终止i的循环
			}
			fmt.Println("j=",j)
		}
		fmt.Println("i=",i)
	}
```

输出结果：

> j= 0
> j= 1

#### continue

continue用于结束本次循环，在多层嵌套时可用标签跳转，用法同上（break是终止循环，continue是跳过本次循环）

#### for-range遍历

这是一种同时获取索引和值或键值的遍历方式

```go
str := "任何信手拈来的从容，都是厚积薄发的沉淀"
for index, s := range str {
	fmt.Printf("%d --- %c ",index, s)
}
```

### 生成随机数

导入math/random和time包

```go
import (
    "fmt"
    "math/rand"
    "time"
)
```

设置种子，生成随机数

```go
rand.Seed(time.Now().UnixNano())  //time.Now().UnixNano() 返回当前系统的纳秒数
n := rand.Intn(100) + 1
fmt.Println("n=",n)
```

## 函数

#### 普通函数

函数定义:

> func 函数名 (参数列表) （返回值列表） { //返回值只有一个时可以不写（）
>
> //函数体
>
> return 返回值列表
>
> }

```go
func generateRandom(time int64, _range int) int {
    rand.Seed(time)
    return rand.Intn(_range)
}
```

调用如下：

```go
fmt.Println(generateRandom(time.Now().Unix(), 100))
```

调用函数，如果函数有多个返回值时，在接收时，希望忽略某个返回值，则使用 _ 符号表示占位忽略

Go函数支持可变参数(可变参数要放在形参列表的最后一位)

```go
func sum(args...int) int{  //支持0-多个参数
	//......
}
```

#### init函数

每一个源文件中都有一个init函数，最大的作用是用来初始化源文件，该函数会在main函数执行前被调用

```go
func init()  {
    fmt.Println("init variable_advanced..")
}
```

源文件执行流程：全局变量定义 -> init -> main，如果此文件还引入了别的文件，就先执行被引用文件的变量定义和init

#### 匿名函数

匿名函数，没有名字的函数，如下

```go
res := func (n1 int, n2 int) int  {
    return n1 * n2
}(2, 8)
fmt.Println("res = ", res)}
```

后面的(2, 8)表示调用并传参

也可以把匿名函数赋给一个变量

```go
a := func(n1 int, n2 int) (int, int) {
    return n2, n1
}
 
n1 := 10
n2 := 29
n1, n2 = a(n1, n2)
```

然后就可以对a进行多次调用了

也可以把匿名函数定义成全局变量

```go
var (
    fun1 = func(n1 int, n2 int) int {
        return n1 * n2
    }
)
func main() {
    fmt.Println(fun1(42, 44))
}
```

#### 闭包

一个函数和其相关的引用环境组合的整体叫做闭包，例如

```go
func AddUpper() func (int) int {
    var n int = 10
    return func(x int) int {
        n = n + x
        return n
    }
}
```

AddUpper()返回的匿名函数，引用了匿名函数外的n，所以AddUpper内部形成了闭包。AddUpper的调用如下：

```go
f := AddUpper()
fmt.Println(f(1)) // 11
fmt.Println(f(3)) // 14
fmt.Println(f(3)) // 17
```

由于形成了匿名函数+外部引用的形式，所以每次调用AddUpper()时，n都会继承上一次调用的值。

就当n是AddUpper()的属性，一个对象只会初始化一次。

```go
f := AddUpper()
fmt.Println(f(1)) // 11
fmt.Println(f(3)) // 14
fmt.Println(f(3)) // 17
 
g := AddUpper()
fmt.Println(g(4)) // 14
```

#### defer

defer用来表示一条语句在函数结束后再执行（主要用于释放资源），**defer语句会把语句和相应数值的拷贝进行压栈，先入后出**。以如下代码为例，这是一个defer + 闭包的例子，makeSuffix的入参为suffix，而返回值是一个函数，此函数入参类型为string，返回值类型也是string。

```go
func makeSuffix(suffix string) func(string) string {
    var n = 1
    defer fmt.Println("suffix = ", suffix, ", n = ", n)
    defer fmt.Println("...")
    n = n + 1
 
    fmt.Println("makeSuffix..")
    return func(file_name string) string {
        if (! strings.HasSuffix(file_name, suffix)) {
            return file_name + suffix
        }
                return file_name
    }
}
func main() {
    f := makeSuffix(".txt")
    fmt.Println(f("szc.txt"))
    fmt.Println(f("szc"))
}
```

可见虽然匿名函数执行了两次，但闭包函数makeSuffix里的语句只执行了一次，而且defer语句先定义的后输出，且都在函数体执行完之后。

#### 字符串常用函数

1. 统计字符串的长度,按字节 len(str)
2. 字符串遍历，同时处理有中文的问题 r := []rune(str)
3. 字符串转整数： n , err := strconv.Atoi(“12”)
4. 整数转字符串: str = strconv.Itoa(12345)
5. 字符串转[]byte: var bytes= []byte(“hello go”)
6. []byte转字符串: str = string([]byte{97,98,99})
7. 10进制转2,8,16进制: str = strconv.FormatInt(123,2) // 2->8,16S
8. 查找子串是否在指定的字符串中: strings.Contains(“seafood”, “foo”) //true
9. 统计一个字符串有几个指定的子串:strings.Count(“ceheese”, “e”) //4
10. 不区分大小写的字符串比较(== 是区分字母大小写的): fmt.Println(strings.EqualFold("abc", "Abc")) // true
11. 返回子串在字符串第一次出现的index值，如果没有返回-1 : strings.Index(“NLT_abc”,”abc”) //4
12. 返回子串在字符串最后一次出现的index，如没有返回-1 : strings.LastIndex(“go golang” , “go”)
13. 将指定的子串替换成 另外一个子串: strings.Replace(“go go hello” , “go”, “go语言”, n) n 可以指定你希望替换几个,如果n = -1表示全部替换
14. 按照指定的某个字符，为分割标识，将一个字符串拆分成字符串数组:
    strings.Split(“hello,wrold,ok” , “,”)
15. 将字符串的字母进行大小写的转换: strings.ToLower(“Go”) // go strings.ToUpper(“Go”) //GO
16. 将字符串左右两边的空格去掉 : strings.TrimSpace(“ tn a lone gopher ntrn “)
17. 将字符串左右两边指定的字符去掉: strings.Trim(“! hello! “, “ !”) // [“hello”]//将左右两边!和””去掉
18. 将字符串左边指定的字符去掉: strings.TrimLeft(“! hello! “,” !”) // [“hello”]//将左边!和”“去掉
19. 将字符串右边指定的字符去掉: strings.TrimRight(“! hello! “,” !”) // [“hello”]//将右边!和””去掉
20. 判断字符串是否以指定的字符串开头: strings.HasPrefix(“[ftp://192.168.10.1"](ftp://192.168.10.1"/) ,”ftp”) // true
21. 判断字符串是否以指定的字符串结束: strings.HasSuffix(“‘NLT_abc.jpg”,”abc”) //false

#### 内置函数

1. len : 用来求长度，比如string、array、slice、map、channel
2. new : 用来分配内存，主要用来分配值类型，比如int、float32、struct…返回的是指针
3. make:用来分配内存，主要用来分配引用类型，比如chan、map、slice。

值类型的用new，返回的是一个指针

```go
p := new(int)
fmt.Println("*p = ", *p, ", p = ", p)
*p = 29
fmt.Println("*p = ", *p)
```

引用类型的用make

### 异常捕获

panic、 defer、recover 用于异常处理

defer、recover捕获异常，相当于try-catch

**`nil`是中预先的标识符**

我们可以直接使用nil，而不用声明它。

`nil`**可以代表很多类型的零值**

在go语言中，nil可以代表下面这些类型的零值：

- 指针类型（包括unsafe中的）
- map类型
- slice类型
- function类型
- channel类型
- interface类型

```go
func test() {
    defer func() {
        err := recover() // 捕获异常
        if err != nil {
            fmt.Println("err:", err) // 输出异常
        }
    }()
    n1 := 1
    n2 := 0
    fmt.Println("res:", n1 / n2)
}
func main() {
    test()
}
```

当我们需要自定义错误时，使用errors.New(“错误说明”)。遇到错误终止程序，使用panic()函数（输出错误信息，并退出程序），示例如下

```go
func testError(name string) (err error) {
    if name == "szc" {
        return nil
    } else {
        return errors.New("Something wrong with " + name + "...") // 定义新的错误信息
    }
}
func test2() {
    err := testError("sss")
    if err != nil {
        panic(err) // 终止程序
    }
 
    fmt.Println("...")
}
func main() {
    test2()
}
```

要先导入errors包

```go
import (
    "fmt"
    "errors"
)
```

## 数组

定义和使用如下所示

```go
var hens [6]float64
total := 0.0

rand.Seed(time.Now().Unix())
for i:= 0; i < len(hens); i++ {
    hens[i] = rand.Float64() * 20 + 5
    fmt.Println("第", (i + 1), " 个数是", hens[i])
    total += hens[i]
}
fmt.Println("均值为", (total / float64(len(hens))))
```

数组初始化：元素值默认为0，也可以用下面的方式初始化

```go
var nums [4]int = [4]int{1, 2, 3, 4}
 
var nums1 = [4]int{1, 2, 3, 4}
 
var nums3 = [...]int{1, 2, 3, 4} // 自行判断长度，中括号里...一个不能少
 
var num4 = [...]int{1:3, 0:4, 2:5} // 指定索引和值
```

由于函数调用时数组形参的值传递，我们可以使用数组指针来实现数组内容在函数里的实际改变，如下所示

```go
func modify(array *[6]float64) {
    array[0] += 5
}
func main() {
    var hens [6]float64
    total := 0.0
    
    rand.Seed(time.Now().Unix())
    for i:= 0; i < len(hens); i++ {
        hens[i] = rand.Float64() * 20 + 5
        fmt.Println("第", (i + 1), " 个数是", hens[i])
        total += hens[i]
    }
 
    fmt.Println("均值为", (total / float64(len(hens))))
 
    modify(&hens)
 
    total = 0
    for i:= 0; i < len(hens); i++ {
        fmt.Println("第", (i + 1), " 个数是", hens[i])
        total += hens[i]
    }
 
    fmt.Println("均值为", (total / float64(len(hens))))
}
```

## 切片(slice)

切片就是动态数组，是数组的一个引用，遍历，访问切片元素，获取切片长度和数组一样。

切片内存结构相当于一个结构体，由三部分构成：引用数组部分的首地址(ptr*)、切片长度(len)和切片容量(cap)

由于是引用，所以改变切片的值，也会改变原数组的对应值

切片定义的基本语法： var 变量名 [] 类型 // 例如： var a [] int

引用切片的三种方式：

1. 定义一个切片，用切片去引用一个已经创建好的数组
2. 通过make来创建切片 ： var 切片名 [] type =make([],len,[cap]) //cap可选，要求cap>= len // slice0 := make([]int, 4, 10)
3. 定义一个切片，直接指定具体数组，使用原理类似make的方式

```go
//1.
array0 := [...]int{1, 2, 3, 4, 5, 6}
slice := array0[1: 4] // 切片  array0[1: 4] 表示slice引用array0数组起始下标为1，最后下标为4(不包含4)

   slice[0] = 7
   fmt.Println(array0[1]) // 7
//3.
slice2 := []int{1, 2, 4}
```

make方法创建切片时，会在底层创建一个数组，只是这个数组是我们不可见的

slice可以通过append的方式来进行动态追加，append时底层会构建一个新的数组，把所有要装进去的元素装进去，然后返回。

```go
slice0 = append(slice0, 4, 5, 7, 1, 0, 4, 8) // slice0后面的参数都是要追加的元素
slice0 = append(slice0, slice...) // 把slice的值追加到slice0后面
```

切片的拷贝可以通过copy函数实现

```go
slice1 := make([]int, 10) // 长度为10
copy(slice1, slice) // 参数列表：dest、src
fmt.Println(slice1)
slice1[len(slice1) - 1] = -1
fmt.Println(slice) // 原切片不变
```

copy时，dest切片的长度并不重要

切片操作字符串：

```go
str := "hello@world!"
slece := str[:]
fmt.Println("slece=",slece)
// string 是不可变的，不能通过str[0] ='z' 方式来修改字符串
// 修改字符串，可以将 string -> []byte  或者 []rune  -> 修改  ->重写转成string
upstr:= []byte(str)

// 可以处理数字和英文还有符号，不能处理中文。原因：一个汉字占3个字节
upstr[5] = '-'
str = string(upstr)
fmt.Println("str=",str)

// 处理中文 转成 []rune 即可
chstr := []rune(str)
chstr[5] = '一'
chstr = string(chstr)
fmt.Println("chstr=",chstr)
```

**练习**

```go
func main() {
	var array = [...]int{24,69,80,57,13,34,1,9,56,90,44,11,16,78,99,4}
	fmt.Println("排序前 array=",array)
	BubbleSort(array[:])
	fmt.Println("冒泡排序后 array=",array)
	in := 5
	findResult := BinaryFind(in, array[:], 0, len(array))
	fmt.Println("二分查找 in=",in," 数组array=",array)
	fmt.Println("二分查找结果下标为： index=",findResult)
}
// BubbleSort 冒泡排序
func BubbleSort(arr []int)  {
	for j := 0; j< len(arr)-1 ; j++ {
		for i := 0; i< len(arr) -1 - j ; i++ {
			if arr[i] > arr[i+1] {
				temp := arr[i]
				arr[i] = arr[i+1]
				arr[i+1] =temp
			}
		}
	}
}
// BinaryFind 对有序数组进行二分查找
func BinaryFind(in int,arr []int,leftIndex int,rightIndex int ) int {
	middle := (leftIndex+rightIndex)/2
	if leftIndex > rightIndex {
		return -1
	}
	if arr[middle] > in {
		//查找范围  leftIndex  -  middle-1
		return BinaryFind(in,arr,leftIndex,middle-1)
	}else if arr[middle] < in {
		//查找范围  middle+1   - rightIndex
		return BinaryFind(in,arr,middle+1,rightIndex)
	}else {
		//找到了
		return middle
	}
	return -1
}
```

## 映射(map)

基本语法：var 变量名 map[keytype] valuetype

key的类型可以为：bool，数字，string，指针，channel，接口，结构体，数组 （slice和function不可以）

map声明后无法直接使用，要先make申请内存，再使用

```go
    m1 := make(map[string] string) // map[键类型] 值类型
    m1["name"] = "Jason" // 键值对赋值
    m1["age"] = "23"
    fmt.Println(m1)
//按键取值
    fmt.Println(m1["name"]) // songzeceng
    fmt.Println(m1["gender"]) // 空字符串
    fmt.Println(m1["gender"] == "") // true
//删除某值
delete(m1, "age") // 如果不存在age键，则也不会报错
//如果需要清空映射，直接分配新的内存就行
```

map的使用：

```go
//1.先声明再make
	var a map[string]string
	a = make(map[string]string,10)
//2.直接make
	b := make(map[string]string)
//3.声明时赋值
c := map[string]string{
    "key1":"宋江",
    "key2":"无用", // ,不能省略
}
```

遍历映射，使用for-range

```go
for k, v := range m1 {
    fmt.Println(k, "--", v)
}
```

map切片可以动态变化map的个数

```go
var slice_map []map[string] string
slice_map = make([]map[string] string, 0)
 
slice_map = append(slice_map, m1, m2)
 
fmt.Println(slice_map)
```

注：

1. map的key是无序的
2. map在函数传参时是引用传递
3. map达到容量后，再添加元素，会自动扩容

**判断key是否存在**

```go
func main() {
    dict := map[string]int{"key1": 1, "key2": 2}
    value, ok := dict["key1"]
    if ok {
        fmt.Println(value)
    } else {
        fmt.Println("key1 不存在")
    }
}

func main() {
    dict := map[string]int{"key1": 1, "key2": 2}
    if value, ok := dict["key1"]; ok {
        fmt.Printf(value)
    } else {
        fmt.Println("key1 不存在")
    }
}
```

**new 和make的区别：**

1. new和make都可以用来初始化内存
2. new多用于基本数据类型的初始化（bool，string，int…）返回的是指针
3. make用于初始化slice、map、channel，返回的是对应类型

## 面向对象

### 结构体

结构体是go面向对象的实现方式，没有this指针、没有方法覆写、没有extends关键字等

其声明和使用如下所示

```go
type Person struct {
    Name string
    Age int
    Hometown string
}
 
func main()  {
    person0 := Person{"Jason", 23, "Washington"}
    fmt.Println(person0)
}
```

结构体是值类型，因此函数传参是值传递，而且拷贝也是浅拷贝

```go
person1 := person0
person1.Age = 21;
fmt.Println(person0) // person0的age依旧是23
```

结构体指针声明和使用如下：

结构体指针访问字段的标准方式为：(*结构体指针).字段名 go底层做了优化支持 ： 结构体指针.字段名 的访问方式

```go
person2 := new (Person) // 指针声明方式1
(*person2).Name = "Jason"
(*person2).Age = 24
 
fmt.Println(*person2) // 没有赋值的字段默认为0值
 
person3 := &Person{"Mike", 20, "London"} // 指针声明方式2
fmt.Println(*person3)
```

如果结构体有切片、映射等属性，也要先分配内存再使用

结构体地址为首字段地址，且内部字段在内存中的地址连续分配。举例如下

```go
type Point struct {
    x, y int
}
type Rect struct {
    leftUp, rightDown Point
}
```

则以下代码

```go
rect0 := Rect {Point{1, 2}, Point{3, 4}}
fmt.Printf("%p ", &rect0)
fmt.Println(&rect0.leftUp.x, &rect0.leftUp.y, &rect0.rightDown.x, &rect0.rightDown.y)
```

的输出如下

> 0xc00000e460 0xc00000e460 0xc00000e468 0xc00000e470 0xc00000e478

当然，结构体内变量值不一定连续分配，看以下示例

```go
type Rect_ struct {
    leftUp, rightDown *Point
}
```

则以下代码

```go
fmt.Printf("%p\t%p\n", rect1.leftUp, rect1.rightDown)
   fmt.Println(&rect1.leftUp.x, &rect1.leftUp.y, &rect1.rightDown.x, &rect1.rightDown.y)
```

的输出如下

> %!p(main.Point={1 2})	%!p(main.Point={3 4}) 0xc0000122c0 0xc0000122c8 0xc0000122d0 0xc0000122d8 

结构体和其他类型进行转换时需要有完全相同的字段（名字、个数和类型）

给结构体取别名，相当于定义新的数据类型，两者的变量赋值时，必须强转。

给结构体属性取标签，可以方便转json时转换大小写

```go
import (
    "fmt"
    "encoding/json"
)
 
type Person struct {
    Name string `json:"name"` // 标签
    Age int `json:"age"`
    Hometown string `json:"homeTown"`
}
 
func main()  {
    person0 := Person{"szc", 23, "Washington"}
    jsonStr, _ := json.Marshal(person0)
    //Marshal  结集; 收集
    fmt.Println(string(jsonStr))
}
```

输出如下

> {“name”:”szc”,”age”:23,”homeTown”:”Washington”}

### 方法

go中方法是作用在指定的数据类型上的（即：和指定的数据类型绑定），因此自定义类型，都可以有方法，而不仅仅是struct

go方法的声明（定义）

```go
func (t type) methodName (参数列表) (返回值列表){
	方法体
	return 返回值
}
// t type 表示这个方法和type这个类型进行绑定 t为type的一个实例
```

go中的方法定义如下

```go
type Person struct {
    Name string `json:"name"` // 标签
    Age int `json:"age"`
    Hometown string `json:"homeTown"`
}

func (p Person) test() { 
    fmt.Println("name:", p.Name, "\tage:", p.Age, "\thometown:", p.Hometown)
}
```

调用方法如下

```go
person0 := Person{"林黛玉", 23, "西方灵河岸绛珠仙草"}
 
person2 := new (Person)
(*person2).Name = "贾宝玉"
(*person2).Age = 22
(*person1_2).Hometown="赤霞宫神瑛侍者"
 
person0.test()
(*person2).test()
```

输出如下

> name: 林黛玉 	age: 23 	hometown: 西方灵河岸绛珠仙草 
>
> name: 贾宝玉 	age: 22 	hometown: 赤霞宫神瑛侍者

**绑定方法时的p是实际调用者的副本，方法调用时会发生值拷贝**。所以当结构体有引用型成员变量时，在方法里发生的修改会同步到方法外面

```go
type Person struct {
    Name string
    Age int 
    Hometown string 
    score map[string]int
}
func (p Person) test() {
    p.Age += 1
    p.score["China"] += 1
}
 
func main()  {
    m0 := make(map[string]int)
    m0["China"] = 80
    person0 := Person{"szc", 23, "Henan Anyang", m0}
 
    person2 := new (Person)
    (*person2).Name = "Jason"
    (*person2).Age = 24
    m2 := make(map[string]int)
    m2["Math"] = 90
    (*person2).score = m2
 
    person0.test()
    fmt.Println(person0)
    (*person2).test()
    fmt.Println(*person2)
}
```

会得到以下输出，age没有变，但映射属性却发生了改变

> {szc 23 Henan Anyang map[China:81]}
> {Jason 24 map[China:1 Math:90]}

对应的map变量的值也会发生变化

```go
fmt.Println(m0, "\n", m2)
```

输出如下

> map[China:81]
> map[China:1 Math:90]

不过，为了能使方法里的修改更高效地同步到外面，声明方法时一般会绑定结构体指针，如下

```go
func (p *Person) test_1(n int) string {
    (*p).score["China"] += n
    (*p).Age -= n
 
    return "succeed"
}
```

调用时，还是可以直接使用变量名调用方法，而不必取址

```go
(&person0).test_1(5)
fmt.Println(person0)
person0.test_1(5)
fmt.Println(person0)
```

输出如下

> {Jason 18 Washington map[China:86]}
> {Jason 13 Washington map[China:91]}

所以，方法里对结构体变量的成员进行的修改能不能同步到外面，关键要看方法绑定时绑定的是不是指针，而不是调用时用什么调用的。

以上的方法定义也适用于系统自带类型，定义方法如下

```go
type integer int // 要先定义别名
func (i *integer) test(n int) {
    *i += integer(n) // int和integer虽然只是别名关系，但依旧不是同一个类型
}
```

调用过程如下

```go
var num integer
num = 8
num.test(6)
fmt.Println(num)
```

会得到输出14

如果要实现类似java里的toString，我们可以对指定数据类型绑定String()方法，返回string

```go
func (p Person) String() string {
    return fmt.Sprintf("name:%v\tage:%v\thometown:%v\tscore:%v", p.Name, p.Age, p.Hometown, p.score)
}
```

然后使用fmt输出Person变量

```go
m0 := make(map[string]int)
m0["China"] = 80
person0 := Person{"Mike", 23, "Manchester", m0}
 
person2 := new (Person)
(*person2).Name = "Jason"
(*person2).Age = 24
m2 := make(map[string]int)
m2["Math"] = 90
(*person2).score = m2
 
fmt.Println(person0)
fmt.Println(*person2)
```

> name:Mike age:23 hometown:Manchester score:map[China:80]
> name:Jason age:24 hometown: score:map[Math:90]

如果方法实现了String()这个方法，那么fmt.Println默认会调用这个变量的String()方法进行输出

**如果String()方法绑定的是结构体指针，那么输出时要传入地址，否则会按照原来的方式输出**

```go
func (p *Person) String() string {
    return fmt.Sprintf("name:%v\tage:%v\thometown:%v\tscore:%v", p.Name, p.Age, p.Hometown, p.score)
}
 
func main()  {
    m0 := make(map[string]int)
    m0["China"] = 80
    person0 := Person{"Mike", 23, "Manchester", m0}
 
    person1 := person0
    person1.Age = 21;
    
    person2 := new (Person)
    (*person2).Name = "Jason"
    (*person2).Age = 24
    m2 := make(map[string]int)
    m2["Math"] = 90
    (*person2).score = m2
 
    fmt.Println(&person0)
    fmt.Println(person0)
    fmt.Println(person2)
    fmt.Println(*person2)
}
```

会得下面的输出

> name:Mike age:23 hometown:Manchester score:map[China:80]
> {Mike 23 Manchester map[China:80]}
> name:Jason age:24 hometown: score:map[Math:90]
> {Jason 24 map[Math:90]}

**方法和函数的区别:**

1. 调用方式不一样. 函数： 函数名(实参列表) ;方法： 变量名.方法名(实参列表)
2. 对于普通函数，接收者为值类型时，不能将指针类型的数据直接传递，反之亦然
3. 对于方法（如struct的方法），接收者为值类型时，可以直接用指针类型的变量调用方法，反之亦然

### 工厂模式

当我们的结构体首字母小写时，我们可以采取对外暴露一个函数，返回结构体变量指针，来进行结构体变量的构造与访问

```go
package model
type student struct { // 结构体名首字母小写，则仅能包内访问
    Name string
    Age int
}
func CreateStudent(name string, age int) *student {
    // 暴露函数名首字母大写的函数，重当构造方法
    return &student {name, age}
}
```

然后在main包里进行如下调用

```go
package main
 
import (
    "fmt"
    "go_code/project01/model" // 导入model包
)
func main()  {
    student0 := model.CreateStudent("Jason", 23) // 调用公有方法，获得指针对象
    fmt.Println(*student0)
}
```

会得到以下输出

> {Jason 23}

访问包私有属性也是同样的方法，暴露公有的方法，返回私有的属性

```go
package model
 
type student struct {
    Name string
    age int
}
 
func CreateStudent(name string, age int) *student {
    return &student {name, age}
}
 
func (student *student) GetAge() int {
    return student.age
}
```

外部进行如下调用

```go
fmt.Println(student0.GetAge())
```

输出为23

这就是go语言里的工厂模式

### 继承

继承可以通过嵌套匿名结构体来实现，如下

```go
package model
type Student struct {
    Name string
    Age int
}
type Graduate struct {
    Student // 匿名结构体
    Major string
}
func (student *Student) GetAge() int {
    return student.Age
}
 
func (graduate *Graduate) GetMajor() string {
    return graduate.Major
}
```

外部调用如下

```go
package main
 
import (
    "fmt"
    "go_code/project01/model"
)
 
func main()  {
    graduate0 := &model.Graduate{}
    graduate0.Name = "szc" 
    graduate0.Age = 23
    graduate0.Major = "software"
    
    fmt.Println(graduate0.GetAge())
    fmt.Println(graduate0.GetMajor())
}
```

结构体可以使用嵌套匿名结构体的所有的字段和方法，即：首字母大小写的字段和方法都可以

**graduate0.Name是graduate0.Student.Name的简写**，但由于Student在Graduate里是匿名结构体，所以可以省略。

如果结构体和匿名结构体有相同的方法和字段时，编译器采用**就近访问原则**，如果要访问匿名结构体中的字段或方法，就要显式调用

如果结构体嵌入了两个或者两个以上的匿名结构体。两个匿名结构体有相同的字段或者方法，在访问时需要明确指定匿名结构体的名字

结构体内部嵌入有名结构体，访问有名结构体的属性需要带上名字访问，两者的关系为**组合**

```go
package main
import (
    "fmt"
    "go_code/project01/model"
)
type A struct {
    n int
}
type B struct {
    A
    n int
}
func (a *A) test() {
    fmt.Println("A...")
}
func (b *B) test() {
    fmt.Println("B...")
}
func main()  {
    var b B
    b.n = 10
    b.A.n = 21
 
    fmt.Println(b.n)
    fmt.Println(b.A.n) // 显式调用
    fmt.Println(b)
 
    b.test()
    b.A.test()
}
```

当结构体嵌入了多个匿名结构体，并且这些匿名结构体拥有同名字段或方法时，访问时就必须显式调用了。

如果把匿名结构体改成有名结构体，那么这个有名结构体就相当于外层结构体的属性，访问其属性或方法就必须显式调用。

```go
package main
import (
    "fmt"
    "go_code/project01/model"
)
type C struct {
    n int
}
type A struct {
    n int
}
type B struct {
    A // 匿名结构体，父类
    n int
    c C // 有名结构体，成员变量
}
 
func (a *A) test() {
    fmt.Println("A...")
}
func (b *B) test() {
    fmt.Println("B...")
}
func (c *C) test() {
    fmt.Println("C...")
}
func main()  {
    var b B
    b.n = 10
    b.A.n = 21
    b.c.n = 31 // 显式访问成员变量的属性
 
    fmt.Println(b.n)
    fmt.Println(b.A.n)
    fmt.Println(b)
    fmt.Println(b.c.n)
 
    b.test()
    b.A.test()
    b.c.test() // 显式调用成员变量的方法
}
```

### 接口

interface类型可以定义一组方法，不需要实现。并且interface不能包含任何变量

基本语法：

> type 接口名 interface{
>
>  method1(参数列表) 返回值列表
>
>  method2(参数列表) 返回值列表
>
> }

go中的接口定义如下

```go
type ICalculate interface { // type 接口名 interface
    add()
    sub()
}
```

**实现接口含义为：实现了这个接口的所有方法**

然后定义两个结构体，来实现ICalculate

```go
type B struct {
}
type D struct {
}
 
func (b B) add() {
    fmt.Println("B..add")
}
func (b B) sub() {
    fmt.Println("B..sub")
}
func (d D) add() {
    fmt.Println("D..add")
}
func (d D) sub() {
    fmt.Println("D..sub")
}
```

再定义一个结构体，为其绑定一个方法，传入接口对象

```go
type E struct {
}
func (e *E) add(ic ICalculate) { // 接口是引用类型，所以这里传递的是变量的引用
    ic.add()
}
func (e *E) sub(ic ICalculate) {
    ic.sub()
}
```

最后，调用E中的方法

```go
b0 := B{}
d0 := D{}
e0 := E{}
 
e0.add(b0)
e0.add(d0)
e0.sub(b0)
e0.sub(d0)
```

会得到如下输出

> B..add
> D..add
> B..sub
> D..sub

go中接口变量可以指向接口实现结构体的变量，如下所示

```go
var ic ICalculate
ic = b0
ic.add()
```

不止结构体，自定义类型都可以实现接口

```go
type integer0 int
func (i integer0) add() {
    fmt.Println("integer..add")
}
```

调用方法也是一样的

```go
i0 := integer0(1)
ic = i0
ic.add()
```

使用接口数组，是实现多态的一种方式

```go
var cals []ICalculate
cals = append(cals, b0)
cals = append(cals, d0)
 
fmt.Println(cals)
```

**接口应用实例：结构体切片排序，要实现sort包下Interface接口中Len()、Less()、Swap()三个接口**

```go
// Student 1、声明结构体
type Student struct {
	Name string
	Sex string
	Score int
}
// StuSlice 2、声明一个结构体切片
type StuSlice []Student

// 3、实现Len()、Less()、Swap()三个接口
func (stu StuSlice) Len() int{
	return len(stu)
}

func (stu StuSlice) Less(i,j int) bool{
	// 按照学生成绩排序 从大到小排序，从小到大用 <
	return stu[i].Score > stu[j].Score
}

func (stu StuSlice) Swap(i,j int)   {
	//temp := stu[i]
	//stu[i] = stu[j]
	//stu[j] =temp
	// 上面三行等价于
	stu[i] , stu[j] = stu[j] , stu[i]
}
func main() {
	var stuSlice StuSlice
	for i := 0; i < 5; i++ {
		stu := Student{fmt.Sprintf("学生%d", i),"男",rand.Intn(100)}
		stuSlice = append(stuSlice, stu)
	}
	fmt.Printf("排序前数据：%v \n",stuSlice)
	sort.Sort(stuSlice)
	fmt.Printf("排序后数据：%v \n",stuSlice)
}
```

输出如下

> 排序前数据：[{学生0 男 81} {学生1 男 87} {学生2 男 47} {学生3 男 59} {学生4 男 81}]
> 排序后数据：[{学生1 男 87} {学生0 男 81} {学生4 男 81} {学生3 男 59} {学生2 男 47}]

**接口注意事项：**

1. 接口本身不能实例化，但是可以指向实现了该接口的自定义类型的变量
2. 只要是自定义类型都可以实现接口，而不仅仅是结构体类型
3. 一个接口（例如A）可以继承多个别的接口（B、C）,这时如果要实现A接口，也必须实现B、C全部接口方法
4. interface类型默认是一个指针（引用类型），如果没有对interface初始化就使用会输出 nil
5. 空接口里没有任何方法，所以任何类型都实现了空接口
6. 接口体现了多态的特性

### 类型断言

由于接口是一般类型，不知道具体类型，如果要转成具体类型就需要使用类型断言

如果类型不匹配会报错

```go
  b1, succeed := cals[0].(B) // 使用方法：待断言变量.(断言类型) 
  if succeed {
      fmt.Println("convert success")
  } else {
      fmt.Println("convert fail")
  }

c1, succeed = cals[1].(B)
  if succeed {
      fmt.Println("convert success")
  } else {
      fmt.Println("convert fail")
  }
```

也可以使用switch语句

```go
switch cals[0].(type) {
    case B: fmt.Println("type b")
    case D: fmt.Println("type d")
    default: fmt.Println("type unkown")
}
```

## 文件操作

### 打开与关闭

文件在go中是一个结构体，它的定义和相关函数在os包中，所以要先导包

```go
import (
    "os"
)
```

打开文件和关闭文件的方法如下

```go
file, err := os.Open("D:/output.txt")
if err != nil {
    fmt.Println("open file error = ", err)
    return
}
fmt.Println("file = ", *file)
err = file.Close()
if err != nil {
    fmt.Println("close file error = ", err)
}
```

其中file的输出如下，可以看到file结构体里存放着一个指针

> file = {0xc000110780}

如果指定文件不存在，那么打开文件时会返回如下的错误

> open file error = open D:/output00.txt: The system cannot find the file specified.

### 读取文件

文件读取方法如下所示

```go
reader := bufio.NewReader(file) // 默认缓冲4096
 
for {
    str, err := reader.ReadString('\n') // 一次读取一行
    if err == nil {
        fmt.Print(str) // reader会把分隔符\n读进去，所以不用Println
    } else if err == io.EOF { // 读到文件尾会返回一个EOF异常
        fmt.Println("文件读取完毕")
        break
    } else {
        fmt.Println("read error: " , err)
    }
}
```

要先导包

```go
import (
    "fmt"
    "os"
    "bufio"
    "io"
)
```

如果文件不大，就可以使用io/ioutil包下的ReadFile函数一次性读取

```go
bytes, err1 := ioutil.ReadFile("D:/output.txt")
if err1 != nil {
    fmt.Println("open file error = ", err1)
    return
}
 
fmt.Println(string(bytes))
```

导包如下

```go
import (
    "fmt"
    "io/ioutil"
)
```

### 创建与写入

创建文件并写入内容的方法如下

```go
    file_path := "D:/out_go.txt"
// os.OpenFile(name string, flag int, perm FileMode)  参数1：文件路径，参数2：打开模式，参数3：权限控制（windows无效） 
    file, err := os.OpenFile(file_path, os.O_WRONLY | os.O_CREATE, 0777) // 最后的777在windows下没有用
    
    if err != nil {
        fmt.Println("Open file error: " , err)
        return
    }
    defer file.Close()
    writer := bufio.NewWriter(file)
    for i := 0; i < 5; i++ {
        writer.WriteString("New content" + fmt.Sprintf("%d", i) + "\n") // 写入一行数据
    }
    writer.Flush() // 把缓存数据刷入文件中
```

打开模式：

```none
const (
    O_RDONLY int = syscall.O_RDONLY // 只读模式打开文件
    O_WRONLY int = syscall.O_WRONLY // 只写模式打开文件
    O_RDWR   int = syscall.O_RDWR   // 读写模式打开文件
    O_APPEND int = syscall.O_APPEND // 写操作时将数据附加到文件尾部
    O_CREATE int = syscall.O_CREAT  // 如果不存在将创建一个新文件
    O_EXCL   int = syscall.O_EXCL   // 和O_CREATE配合使用，文件必须不存在
    O_SYNC   int = syscall.O_SYNC   // 打开文件用于同步I/O
    O_TRUNC  int = syscall.O_TRUNC  // 如果可能，打开时清空文件
)
```

如果打开已存在的文件，覆写新内容，就要把模式换成os.O_TRUNC

判断文件或文件夹是否存在使用os.Stat()返回错误值进行判断：

1. 如果返回错误为nil说明文件或文件夹存在
2. 如果返回的错误类型使用os.IsNotExist()判断为true，说明不存在
3. 如果返回的错误为其它类，则不确定是否存在

## 命令行参数

命令行参数保存在os.Args里，是一个字符串切片

先导入os包

```go
import (
    "fmt"
    "os"
)
```

然后用for-range遍历os.Args即可

```go
for index, arg := range os.Args {
    fmt.Println("第", (index + 1), "个参数是", arg)
}
```

会得到如下输出

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go run .\cmd_args.go 123 json
> 第 1 个参数是 D:\deveop\temp\go-build629970270\b001\exe\cmd_args.exe
> 第 2 个参数是 123
> 第 3 个参数是 json

也可以用**flag包进行参数解析**

```go
var s0 string
var s1 string
var i0 int
 
flag.StringVar(&s0, "u", "", "字符串参数1") // 接收字符串参数，参数列表：参数接收地址，参数名，默认值， 参数说明
flag.StringVar(&s1, "p", "", "字符串参数2")
flag.IntVar(&i0, "i", 0, "整型参数1")
flag.Parse() // 开始解析必须调用
fmt.Println("s0 = ", s0, ", s1 = ", s1, ", i0 = ", i0)
```

导包如下

```go
import (
	"flag"
	"fmt"
)
```

测试输出如下

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go run .\cmd_args.go -u s -p c -i 98
> s0 = s , s1 = c , i0 = 98

## 序列化反序列化

### 序列化

把结构体序列化成json的方法

```go
func main()  {
    p0 := Person_ser{Name: "szc", Age: 23,}
 
    json_bytes, error_ := json.Marshal(&p0)
    if error_ != nil {
        fmt.Println("Json error:", error_)
        return
    }
 
    fmt.Println(string(json_bytes))
     // {"Name":"szc","Age":23}
}
```

导包

```go
import (
    "fmt"
    "encoding/json"
)
```

一定要记得结构体的属性如果要序列化成json，就必须首字母大写；包私有的属性不能被json包序列化成json

如果要把首字母大写的属性名序列化首字母小写的json键，就需要使用tag

```go
type Person_ser struct {
    Name string `json:"name"` // 标签
    Age int `json:"age"`
}
```

json.Marshal函数也可以对映射进行序列化

```go
var a map[string] interface{} // 值是空接口，表示能接收任意类型
a = make(map[string] interface{})
 
a["name"] = "szc"
a["age"] = 23
 
bytes, error_ := json.Marshal(&a)
if error_ != nil {
    fmt.Println("Json error:", error_)
    return
}
 
fmt.Println(string(bytes))
// {"age":23,"name":"szc"}
```

对切片序列化也是可以的

```go
var slice []map[string] interface{}
slice = make([]map[string] interface{}, 0)
 
for i := 0; i < 5; i++ {
    var a map[string] interface{}
    a = make(map[string] interface{})
 
    a["name"] = "szc" + fmt.Sprintf("%d", i)
    a["age"] = 23
 
    slice = append(slice, a)
}
 
bytes, error_ := json.Marshal(&slice)
if error_ != nil {
    fmt.Println("Json error:", error_)
    return
}
 
fmt.Println(string(bytes))
// [{"age":23,"name":"szc0"},{"age":23,"name":"szc1"},{"age":23,"name":"szc2"},{"age":23,"name":"szc3"},{"age":23,"name":"szc4"}]
```

甚至对普通数据类型也能序列化，只是只有值，没有键

```go
i := 1
bytes, error_ := json.Marshal(&i)
if error_ != nil {
    fmt.Println("Json error:", error_)
    return
}
fmt.Println(string(bytes))
// 1
```

### 反序列化

反序列化时，要调用Unmarshal函数，传入待解析字符串的bytes，以及接收结果的对象指针

```go
str := "{\"Name\":\"szc\",\"Age\":23}"
var p0 Person_ser
err := json.Unmarshal([]byte(str), &p0)
if err != nil {
    fmt.Println("Json error:", err)
    return
}
fmt.Println(p0)
// {szc 23}
```

同样可以解析成映射、切片

```go
str := "{\"Name\":\"szc\",\"Age\":23}"
var m0 map[string]interface{}
 
err := json.Unmarshal([]byte(str), &m0)
 
if err != nil {
    fmt.Println("Json error:", err)
    return
}
 
slice := make([]map[string] interface{}, 0)
err = json.Unmarshal([]byte("[{\"age\":23,\"name\":\"szc0\"},{\"age\":23,\"name\":\"szc1\"},{\"age\":23,\"name\":\"szc2\"}]"), &slice)
if err != nil {
    fmt.Println("Json error:", err)
    return
}
 
fmt.Println(m0) // map[Age:23 Name:szc]
fmt.Println(slice) // [map[age:23 name:szc0] map[age:23 name:szc1] map[age:23 name:szc2]]
```

## 单元测试

单元测试**(testing**)用来检测代码错误、逻辑错误和性能高低

首先有待测试文件**first.go**，内有函数addUpper()

```go
package main
 
func addUpper(n int) int {
    ret := 0
    for i := 0; i < n; i++ {
        ret += i
    }
    return ret
}
```

然后添加**first_test.go**文件，导入testing包，编写**TestAddUpper()**函数

```go
package main
 
import (
    "testing" // 引入testing框架
)
 
func TestAddUpper(t *testing.T) {
    res := addUpper(10) // 调用目标函数
    if res != 45 {
        t.Fatalf("AddUpper(10)执行错误, 期望值%d, 实际值%d\n", 55, res) // 打出错误日志
    }
 
    t.Logf("AddUpper(10)执行正确") // 打出正常日志
}
```

然后在命令行执行**go test -v**，就会看到结果

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go test -v
> init variable_advanced..
> === RUN TestAddUpper
> TestAddUpper: first_test.go:14: AddUpper(10)执行正确
> — PASS: TestAddUpper (0.00s)
> PASS
> ok go_code/project01/main 0.323s

**go test -v命令会执行这个目录内所有的测试用例**，例如再在这个目录下添加测试文件map_test.go文件和TestSub()函数

```go
package main
 
import "testing"
 
func TestSub(t *testing.T) {
    ret := sub(9, 3)
    if ret != 6 {
        t.Fatalf("sub 执行错误, 预期值%d, 实际值%d\n", 6, ret)
    }
 
    t.Logf("sub执行正确")
}
```

待检测函数sub如下

```go
func sub(n1 int, n2 int) int {
    return n1 - n2
}
```

运行测试用例

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go test -v
> init variable_advanced..
> === RUN TestAddUpper
> TestAddUpper: first_test.go:14: AddUpper(10)执行正确
> — PASS: TestAddUpper (0.00s)
> === RUN TestSub
> TestSub: map_test.go:11: sub执行正确
> — PASS: TestSub (0.00s)
> PASS
> ok go_code/project01/main 0.322s

会发现测试累计用时比测试那两个函数用时的和要大，因为加载testing框架也要消耗时间

如果要测试单个文件，则要执行命令go test -v xx_test.go xx.go

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go test -v .\map_test.go .\map.go
> === RUN TestSub
> TestSub: map_test.go:11: sub执行正确
> — PASS: TestSub (0.00s)
> PASS
> ok command-line-arguments 0.382s

如果测试单个函数的话，使用-test.run TestXxxx选项即可

>  go test -v -test.run TestAddUpper
> init variable_advanced..
> === RUN TestAddUpper
> TestAddUpper: first_test.go:14: AddUpper(10)执行正确
> — PASS: TestAddUpper (0.00s)
> PASS
> ok go_code/project01/main 0.410s

如果要在测试前统一进行一些操作，可以覆写TestMain函数

```go
func TestMain(m *testing.M) {
    fmt.Println("testing start")
 
    m.Run() // 执行测试
}
```

如果要在测试时执行另一个测试函数，可以执行t.Run()函数

```go
type Sale struct {
	Widget_id int
	Qty       int
	Street    string
	City      string
	State     string
	Zip       int
	Sale_date string
}
func (s *Sale) AddSale() {
	fmt.Println("添加了一个销售信息")
}
func TestAddSale(t *testing.T) {
    sale := &Sale{Widget_id: 9, Qty: 80, Street: "Huanghe South Road", City: "Anyang Henan", State: "China", Zip: 455000, Sale_date: "2020-03-24"}
    sale.AddSale()
    t.Run("fun", fun_test)
}
 
func fun_test(t *testing.T) {
    fmt.Println("fun_test")
}
```

测试输出如下

> D:\GoProject\src\test\main10>go test -v
> testing start
> === RUN   TestAddUpper
>     first_test.go:30: AddUpper(10)执行正确
> --- PASS: TestAddUpper (0.00s)
> === RUN   TestSub
>     first_test.go:39: sub执行正确
> --- PASS: TestSub (0.00s)
> === RUN   TestAddSale
> 添加了一个销售信息
> === RUN   TestAddSale/fun
> fun_test
> --- PASS: TestAddSale (0.00s)
>     --- PASS: TestAddSale/fun (0.00s)
> PASS
> ok      test/main10     0.259s

## 并发编程

### 协程goroutine

协程从主线程开启是轻量级线程，是逻辑态的，特点：

- 有独立的栈空间
- 共享程序堆空间
- 调度由用户控制

go中开启协程执行函数的方法如下

```go
func test_r() {
    for i := 0; i < 10; i++ {
        fmt.Println("test_r test.......", strconv.Itoa(i + 1))
        time.Sleep(2 * time.Second) // 休眠2秒
    }
}
//strconv.Itoa函数的参数是一个整型数字，它可以将数字转换成对应的字符串类型的数字。
func main()  {
    go test_r()  // 开启一个协程
    for i := 0; i < 10; i++ {
        fmt.Println("main test.......", strconv.Itoa(i + 1))
        time.Sleep(time.Second)
    }
}
```

导入strconv和time包

```go
import (
    "fmt"
    "strconv"
    "time"
)
```

输出如下

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go run .\go_routine.go
> main test……. 1
> test_r test……. 1
> main test……. 2
> main test……. 3
> test_r test……. 2
> main test……. 4
> test_r test……. 3
> main test……. 5
> main test……. 6
> test_r test……. 4
> main test……. 7
> main test……. 8
> test_r test……. 5
> main test……. 9
> main test……. 10
> test_r test……. 6

主线程退出，不管协程的任务有没有执行完，协程也会立即退出

等待协程执行完再往下执行时使用WaitGroup

```go
package main
import (
	"fmt"
	"sync"
)
func Show(i int)  {
	fmt.Println(i)
	defer wp.Done() //执行完任务协程-1
}

var wp sync.WaitGroup  //使用WaitGroup 等待各个协程执行完

func main() {
	for i := 0; i < 10; i++ {
		go Show(i) // 开启10个 go协程执行任务
		wp.Add(1) //协程+1
	}
	wp.Wait()  //阻塞 直到协程组内协程数为0时往下执行
}
```

获取并设置使用cpu数量

```go
num := runtime.NumCPU() // 获取cpu核数
fmt.Println("CPU count:", num)
runtime.GOMAXPROCS(num - 1) // 设置最大并发数
```

事先导入runtime包

```go
import (
    "fmt"
    "runtime"
)
```

输出如下

> CPU count: 12

### MPG模式

MPG模式：

M：操作系统主线程（物理线程）

P：协程执行所需的上下文

G：协程

三者关系如下图所示

![img](https://img2.baidu.com/it/u=3053497369,948986322&fm=15&fmt=auto)

假设主线程M1的G1协程阻塞，如果协程队列里有别的协程，那么就会新启一个M2主线程，把协程队列里的其他协程挂在到M2上执行，这就是MPG模式

### 全局互斥锁

当涉及多个协程对同一个引用类型的对象进行读写操作时，就需要全局锁来帮助同步。

案例一：

先导包

```go
import (
    "math"
    "fmt"
    "strconv"
    "time"
    "runtime"
    "sync" //　同步包
)
```

然后声明锁变量

```go
var (
    result []int = make([]int, 0)　// 素数切片
    lock sync.Mutex // 全局锁
)
```

编写is_prime函数，在append素数切片时，进行锁的请求和释放

```go
func exits(slice []int, n int) bool {
    for _, value := range slice {
        if value == n {
            return true
        }
    }
    return false
}
func is_prime(n int) {
    is_prime := true
    for i := 2; i < int(math.Sqrt(float64(n))) + 1; i++ {
        if n % i == 0 {
            is_prime = false
            break
        }
    }
    if is_prime && !exits(result, n) {
        lock.Lock() // 请求锁
        result = append(result, n)
        lock.Unlock() // 释放锁
    }
}
```

主线程开启2000个协程，进行素数判断，等待10秒后，读取素数切片内容。

由于读取的是全局变量，所以读的时候也要加锁和释放锁

```go
func main()  {
    num := runtime.NumCPU()
    runtime.GOMAXPROCS(num - 1)
        for i := 2; i < 2000; i++ {
        // 开启将近2000个协程，判断素数
        go is_prime(i)
    }
    time.Sleep(10 * time.Second) // 主线程等待10秒
    lock.Lock() // 遍历的时候依旧要利用锁进行同步控制
    for _, value := range result {
        fmt.Println(value)
    }
    lock.Unlock()
}
```

最后的输出如下

> 2
> 3
> 5
> 7
> 11
> 13
> 17
> 19
> 23
> …
> 1987
> 1993
> 1997
> 1999

案例二：

计算 1-200 的各个数的阶乘，并且把各个数的阶乘放入到 map 中。最后显示出来。要求使用 goroutine 完成

```go
package main import (
    "fmt"
    "time"
)
var (
    result []int = make([]int, 10)　// 素数切片
    lock sync.Mutex // 全局锁
)
// test 函数就是计算 n!, 让将这个结果放入到 myMap 
func test(n int) {
    res := 1
    for i := 1; i <= n; i++ {
        res *= i
    }

    //这里我们将 res 放入到 
    lock.Lock()
    myMap myMap[n] = res 
    lock.Unlock()
}

func main() {
    // 我们这里开启多个协程完成这个任务[200 个] 
    for i := 1; i <= 200; i++ {
        go test(i)
    }
    //休眠 10 秒钟【第二个问题 】
    time.Sleep(time.Second * 10)

    //这里我们输出结果,变量这个结果
    lock.Lock()
    for i, v := range myMap {
        fmt.Printf("map[%d]=%d\n", i, v)
    }
    lock.Unlock()
}
```

### 管道

管道(channel)为引用类型，必须先初始化才能使用；本质是一个队列，有类型，而且线程安全，

管道的定义： var 变量名 chan 数据类型

1. ch <- v  // 发送值v到Channel ch中 
2. v := <-ch // 从Channel ch中接收数据，并将数据赋值给v 

以下为实例：一个写协程，一个读协程，主线程等待两者完成后退出

先构造两个协程，一个存储数据，一个表示是否读写完成

```go
var (
    write_chan chan int = make(chan int, 50) // 数据管道，整型管道，容量50(不可扩容)
    exit_chan chan bool = make(chan bool, 1) // 状态管道，布尔型管道，容量1(不可扩容)
)
```

然后，构造读写函数。写函数往数据管道里写入50个数据，并关闭数据管道；读函数负责从数据管道里读取数据，如果读完，则往状态管道里置入true，表示读取完成

```go
func write_data() {
    for i := 0; i < 50; i++ {
        write_chan<- i // 往管道里写数据
        fmt.Println("write data: ", i)
    }
 
    close(write_chan)
    // 关闭管道不影响读，只影响写
}
 
func read_data() {
    for {
        v, ok := <-write_chan // 从管道里读数据，返回具体数据和成功与否。如果管道为空，就会阻塞
        if !ok { // 如果管道为空，则ok为false
            break
        }
        fmt.Println("read data: ", v)
    }
    exit_chan<- true
    close(exit_chan)
}
//主线程负责开启两个协程，并监视状态管道
func main()  {
    go write_data()
    go read_data()
 
    for {
        _, ok := <-exit_chan
        if !ok {
            break
        }
    }
 
}
```

最后输出如下

> write data: 0
> write data: 1
> write data: 2
>
> ……

- 往管道里写数据时，如果超出了管道容量，就会阻塞；但是读写频率不一致，则不会发生阻塞问题。
- 不使用协程时，从空管道里读数据会发生死锁错误；
- 普通for-range遍历没有关闭的管道时，也发生死锁错误。
- 管道默认是双向的（可读可写），也可以声明为只读或者只写管道
- 使用select可以解决从管道取数据的阻塞问题
- goroutine中使用recover，解决协程中出现panic，导致程序崩溃的问题

channel版的寻找素数。

```go
//原始数据管道、素数结果管道和协程状态管道
var (
    int_chan chan int = make(chan int, 80000) // 待判断的数为2-80001
    prime_chan chan int = make(chan int, 2000) // 素数管道
    over_chan chan bool = make(chan bool, 4) // 状态管道
)

//写入数据
func put_num() {
    for i := 2; i < 80002; i++ {
        int_chan<- i
    }
    close(int_chan)
}
//用来判断数据是否是素数
func check_prime() {
    for {
        is_prime := true
        num, ok := <- int_chan
        if !ok {
            break
        }
        for i := 2; i < int(math.Sqrt(float64(num))) + 1; i++ {
            if num % i == 0 {
                is_prime = false
                break
            }
        }
 
        if is_prime {
            prime_chan<- num
        }
    }
 
    fmt.Println("One routine has exit for the lack of data..")
 
    over_chan<- true
}
//由于有多个管道处理素数判断，所以这里最后不关闭over_chan和prime_chan
//匿名函数，判断是否所有判断素数的协程都已完成
func() {
        over_num := 0
        for {
            if over_num == 4 {
                break
            }
            status, ok := <-over_chan
            if !ok {
                break
            }
    
            if status {
                over_num += 1
            }
        }
 
        close(prime_chan) // 此时所有判断协程已经结束，关闭prime_chan，主线程遍历prime_chan处唤醒阻塞
    }
//最后在main函数里，启动一个输入协程、四个判断携程，最后自己负责从素数管道里拿素数，顺便计时
    go put_num()
 
    start := time.Now().Unix()
 
    for i := 0; i < 4; i++ {
        go check_prime()
    }
 
    go func() {
        over_num := 0
        for {
            if over_num == 4 {
                break
            }
    
            status, ok := <-over_chan
            if !ok {
                break
            }
    
            if status {
                over_num += 1
            }
        }
        close(prime_chan)
    }()

    for {
        num, ok:= <-prime_chan
        if !ok {
            break
        }
        fmt.Println(num)
    }
 
    fmt.Println("Time used:", strconv.Itoa(int(time.Now().Unix()) - int(start)))
    close(over_chan)
```

最后输出如下

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go run .\go_routine.go
> CPU count: 12
> 2
> 3
> 5
> 7
> ….
> 79979
> 79987
> 79997
> 79999
> One routine has exit for the lack of data..
> One routine has exit for the lack of data..
> One routine has exit for the lack of data..
> One routine has exit for the lack of data..
> Time used: 2

把管道声明为只读或只写

```go
int_chan1 chan <- int = make(chan int, 8) // 只写
int_chan2 <- chan int  = make(chan int, 8) // 只读
```

传统方法遍历管道时，如果管道不关闭，就会发生死锁。如果我们不确定何时关闭管道，就可以使用select，如下所示

```go
label:
for {
    select {
        case v := <- int_chan_3:
            // 如果管道一直不关闭，也不会死锁，而会向下匹配
            fmt.Println("data from int chan: ", v)
        case v := <- string_chan:
            fmt.Println("data from string chan: ", v)
        default:
            break label
    }
}
```

向int_chan_3和string_chan中赋值的代码如下

```go
int_chan_3 := make(chan int, 10)
for i := 0; i < 10; i++ {
    int_chan_3 <- i
}
string_chan := make(chan string, 5)
for i := 0; i < 5; i++ {
    string_chan <- "string " + strconv.Itoa(i)
}
```

最后输出如下

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go run .\go_routine.go
> data from string chan: string 0
> data from int chan: 0
> data from int chan: 1
> data from string chan: string 1
> data from string chan: string 2
> data from string chan: string 3
> data from string chan: string 4
> data from int chan: 2
> data from int chan: 3
> data from int chan: 4
> data from int chan: 5
> data from int chan: 6
> data from int chan: 7
> data from int chan: 8
> data from int chan: 9

#### 定时器（Timer）

Timer顾名思义，就是定时器的意思，可以实现一些定时操作，内部也是通过channel来实现的。Timer只执行一次

定时器实质是单向通道，time.Timer结构体类型中有一个time.Time类型的单向chan

Timer只有两个成员：

- C: 管道，上层应用根据此管道接收事件；
- r: runtime定时器，该定时器即系统管理的定时器，对上层应用不可见；

```go
package main
import (
	"fmt"
	"time"
)
func main() {
	timer1 := time.NewTimer(time.Second * 2)
	t1 := time.Now()
	fmt.Printf("t1:%v\n", t1)

	t2 := <-timer1.C
	fmt.Printf("t2:%v\n", t2)

	//如果只是想单纯的等待的话，可以使用 time.Sleep 来实现
	timer2 := time.NewTimer(time.Second * 2)
	<-timer2.C
	fmt.Println("2s后")

	time.Sleep(time.Second * 2)
	fmt.Println("再一次2s后")

	<-time.After(time.Second * 2) //time.After函数的返回值是chan Time
	fmt.Println("再再一次2s后")

	timer3 := time.NewTimer(time.Second)
	go func() {
		<-timer3.C
		fmt.Println("Timer 3 expired")
	}()

	stop := timer3.Stop() //停止定时器
	////阻止timer事件发生，当该函数执行后，timer计时器停止，相应的事件不再执行
	if stop {
		fmt.Println("Timer 3 stopped")
	}

	fmt.Println("before")
	timer4 := time.NewTimer(time.Second * 5) //原来设置5s
	timer4.Reset(time.Second * 1)            //重新设置时间,即修改NewTimer的时间
	<-timer4.C
	fmt.Println("after")
}
```

#### Ticker

Ticker是周期性定时器，即周期性的触发一个事件，通过Ticker本身提供的管道将事件传递出去。

Ticker的数据结构与Timer完全一致。

Timer只执行一次，Ticker可以周期的执行。

```go
import (
	"fmt"
	"time"
)
func main() {
	ticker := time.NewTicker(time.Second)
	counter := 1
	for _ = range ticker.C {
		fmt.Println("ticker 1") //每秒执行一次
		counter++
		if counter >= 5 {
			break
		}
	}
	ticker.Stop()  //停止
}
```

### 原子操作

 sync/atomic包提供了原子操作的能力，直接有底层CPU硬件支持，因而一般要比基于操作系统API的锁方式效率高些。

atomic 提供的原子操作能够确保任一时刻只有一个goroutine对变量进行操作，善用 atomic 能够避免程序中出现大量的锁操作。

atomic常见操作有：

- 增减
- 载入 read
- 比较并交换 cas
- 交换
- 存储 write

下面将分别介绍这些操作。

#### 增减操作

atomic 包中提供了如下以Add为前缀的增减操作:

```go
- func AddInt32(addr *int32, delta int32) (new int32)
- func AddInt64(addr *int64, delta int64) (new int64)
- func AddUint32(addr *uint32, delta uint32) (new uint32)
- func AddUint64(addr *uint64, delta uint64) (new uint64)
- func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)
```

#### 载入操作

unsafe 是类型安全的操作，指向不同类型数据的指针，是无法直接相互转换的，必须借助unsafe.Pointer(类似于C的 void指针)代理一下再转换。pointer是任意类型的指针，可以指向任意类型数据。

1. 任何类型的指针都可以被转化为Pointer
2. Pointer可以被转化为任何类型的指针
3. uintptr可以被转化为Pointer
4. Pointer可以被转化为uintptr

atomic 包中提供了如下以Load为前缀的增减操作:

```go
- func LoadInt32(addr *int32) (val int32)
- func LoadInt64(addr *int64) (val int64)
- func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
- func LoadUint32(addr *uint32) (val uint32)
- func LoadUint64(addr *uint64) (val uint64)
- func LoadUintptr(addr *uintptr) (val uintptr)
```

> 载入操作能够保证原子的读变量的值，当读取的时候，任何其他CPU操作都无法对该变量进行读写，其实现机制受到底层硬件的支持。

#### 比较并交换

该操作简称 CAS(Compare And Swap)。 这类操作的前缀为 `CompareAndSwap` :

```go
- func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
- func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
- func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)
- func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
- func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)
- func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)
```

> 该操作在进行交换前首先确保变量的值未被更改，即仍然保持参数 `old` 所记录的值，满足此前提下才进行交换操作。CAS的做法类似操作数据库时常见的乐观锁机制。

#### 交换

此类操作的前缀为 `Swap`：

```go
- func SwapInt32(addr *int32, new int32) (old int32)
- func SwapInt64(addr *int64, new int64) (old int64)
- func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)
- func SwapUint32(addr *uint32, new uint32) (old uint32)
- func SwapUint64(addr *uint64, new uint64) (old uint64)
- func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)
```

> 相对于CAS，明显此类操作更为暴力直接，并不管变量的旧值是否被改变，直接赋予新值然后返回背替换的值。

#### 存储

此类操作的前缀为 `Store`：

```go
- func StoreInt32(addr *int32, val int32)
- func StoreInt64(addr *int64, val int64)
- func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
- func StoreUint32(addr *uint32, val uint32)
- func StoreUint64(addr *uint64, val uint64)
- func StoreUintptr(addr *uintptr, val uintptr)
```

> 此类操作确保了写变量的原子性，避免其他操作读到了修改变量过程中的脏数据。

### 异常捕获

当我们需要在某个协程函数里捕获异常时，使用以前的defer-recover即可

```go
func test_r_0() {
    defer func() {
        if err := recover(); err != nil {
            fmt.Println("test_r_0 发生错误:", err)
        }
    }()
    var map0 map[int]string
    map0[0] = "szc"
}
```

同时写一个正常方法

```go
func test_r() {
    for i := 0; i < 10; i++ {
        fmt.Println("test_r test.......", strconv.Itoa(i + 1))
        time.Sleep(500 * time.Millisecond)
    }
}
```

最后在main函数里测试

```go
func main()  {
    go test_r()
    go test_r_0()
    time.Sleep(time.Second * 10)
}
```

输出如下

> PS D:\develop\Microsoft VS Code\workspace\src\go_code\project01\main> go run .\go_routine.go
> test_r test……. 1
> test_r_0 发生错误: assignment to entry in nil map
> test_r test……. 2
> test_r test……. 3
> test_r test……. 4
> test_r test……. 5
> test_r test……. 6
> test_r test……. 7
> test_r test……. 8
> test_r test……. 9
> test_r test……. 10

## 反射

- 反射可以动态获取变量的类型、结构体的属性和方法，以及设置属性、执行方法等信息
- 通过反射可以修改变量的值，可以调用关联的方法
- 使用反射，需要导入 import (“reflect”)

### 反射基本数据类型

下面是对基本数据类型进行反射的方法

```go
import (
    "reflect"
)
func reflect_base(n int) {
    rTyp := reflect.TypeOf(n) // 获取反射类型
    fmt.Println("rType = ", rTyp) // int
    fmt.Println("rType`s name = ", rTyp.Name()) //int
    
    rVal := reflect.ValueOf(n) // 获取反射值
    fmt.Printf("rValue = %v, rValue`s type = %T\n", rVal, rVal) // 100, reflect.Value
 
    n1 := 2 + rVal.Int()　// 获取反射值持有的整型值
    fmt.Println("n1 = ", n1)
 
    iV := rVal.Interface() // 反射值转换成空接口
    num := iV.(int) // 类型断言
    fmt.Println("num = ", num)
}
```

注意反射值必须转换成空接口，然后进行类型断言，才能获取真正的值，因为反射是运行时进行的。

### 反射结构体

以下是对结构体进行反射的方法

```go
func reflect_struct(n interface{}) {
    rType := reflect.TypeOf(n)
    rValue := reflect.ValueOf(n)
 
    iv := rValue.Interface()
 
    fmt.Println("rType = ", rType, ", iv = ", iv)
    // rType =  main.Student_rf , iv =  {szc 23}
    fmt.Printf("Type of iv = %T\n", iv)
    // Type of iv = main.Student_rf
 
    // 类型断言
    switch iv.(type) {
        case Student_rf:
            student := iv.(Student_rf)
            fmt.Println(student.Name, ", ", student.Age)
        case Student_rf_0:
            student := iv.(Student_rf_0)
            fmt.Println(student.Name, ", ", student.Age)
    }
}
```

获取反射种类：

```go
fmt.Println("rKind = ", rValue.Kind(), ", rKind = ", rType.Kind())
    // rKind =  struct , rKind =  struct
```

故而反射类型就是变量的类型，反射种类则更宽泛一些。例如对于基本数据类型，反射类型=反射种类；对于结构体，反射类型则是包名.结构体名，反射种类则是struct

变量 <———-> interface{} <———-> reflect.Value 三者可以互相转换

### 反射改基本数据类型变量的值

如果要通过反射改变基本数据类型变量的值，那么要调用反射值的**Elem()**方法，再调用setXXX()方法，而且反射的对象应该是指针

```go
func reflect_base(n interface{}) {
    rVal := reflect.ValueOf(n) // 这里不要传入空接口的指针
    fmt.Printf("rValue = %v, rValue`s type = %T\n", rVal, rVal) // 100, reflect.Value
 
    rVal.Elem().SetInt(10)
}
```

主函数调用时要传入指针

```go
func main() {
    n := 100
    reflect_base(&n) // 传入指针
    fmt.Println(n) // 10
}
```

### 获取结构体所有属性和标签

获取结构体所有属性和json标签的方法如下

```go
func reflect_struct(n interface{}) {
    rType := reflect.TypeOf(n)
    rValue := reflect.ValueOf(n)
 
    if rValue.Kind() != reflect.Struct {
        // 如果不是Struct类别，直接结束
        return
    }
 
    num := rValue.NumField() // 获取字段数量
    for i := 0; i < num; i++ {
        fmt.Printf("Field %d value = %v\n", i, rValue.Field(i)) // 获取字段值
        tagVal := rType.Field(i).Tag.Get("json") // 获取字段的json标签值
        if tagVal != "" {
            fmt.Printf("Field %d tag = %v\n", i, tagVal)
        }
    }
}
```

修改结构体，为其添加标签

```go
type Student_rf struct {
    Name string `json:"name"`
    Age int `json:"age"`
}
```

main函数调用测试

```go
func main() {
    reflect_struct(Student_rf{
        Name: "szc",
        Age: 23,
    })
}
```

输出如下

> Field 0 value = szc
> Field 0 tag = name
> Field 1 value = 23
> Field 1 tag = age

### 调用结构体方法

调用结构体方法的过程如下

```go
num = rValue.NumMethod() // 获取方法数量
for i := 0; i < num; i++ {
    method := rValue.Method(i)
    fmt.Println(method) // 打印方法地址
}
 
var params []reflect.Value
params = append(params, reflect.ValueOf("szc"))
params = append(params, reflect.ValueOf(24))
rValue.Method(1).Call(params) // 调用方法，传入参数
 
fmt.Println("...")
 
res := rValue.Method(0).Call(nil) // 调用方法，接收返回值
fmt.Println(res[0].Int())
```

对应的方法如下

```go
func (s Student_rf) Show(name string, age int ) {
    fmt.Println(name, " -- ", age)
}
 
func (s Student_rf) GetAge() int {
    return s.Age
}
```

**反射中方法的排序按照方法名的ascii码排序**，所以GetAge()在前，Show()在后

main函数中调用测试

```go
func main() {
    s := Student_rf{
        Name: "szc",
        Age: 23,
    }
 
    reflect_struct(s)
}
```

输出如下

> szc – 24
> …
> 23

### 修改结构体字段值

修改结构体字段的值，就要和修改普通类型变量的值一样，获取地址的引用

```go
s := Student_rf{
    Name: "szc",
    Age: 23,
}
rValue := reflect.ValueOf(&s)
rValue.Elem().Field(0).SetString("szc")
rValue.Elem().Field(1).SetInt(24)
 
fmt.Println(s)
```

输出如下

> {szc 24}

### 反射构造结构体变量

利用反射构造结构体变量并赋予属性值

```go
var (
    ptr *Student_rf
    rType reflect.Type
    rValue reflect.Value
)
ptr = &Student_rf{} // 结构体指针
rType = reflect.TypeOf(ptr).Elem() // 结构体反射类型
rValue = reflect.New(rType) // 由结构体反射类型，获取新结构体指针反射值
ptr = rValue.Interface().(*Student_rf) // 把指针反射值转成空接口，并进行类型断言
rValue = rValue.Elem() // 由结构体指针反射值获取结构体反射值
rValue.FieldByName("Name").SetString("szc") // 根据属性名，对结构体反射值设置值
rValue.FieldByName("Age").SetInt(22)
fmt.Println(*ptr) // 输出结果
```

结果如下

> {szc 22}

综上，我们可以发现：如果要通过反射改变变量的值，就要先获取指针的反射，再通过Elem()方法获取变量的反射值，然后进行设置；如果只是查看变量的值，就用变量的反射即可

## 网络编程

以tcp为例，服务端建立监听套接字，然后阻塞等待客户端连接。客户端连接后，开启协程处理客户端。

```go
package main
import (
    "fmt"
    "net"
)
func process_client(conn net.Conn) {
    for {
        var bytes []byte = make([]byte, 1024)
        n, err := conn.Read(bytes)
         // 从客户端读取数据，阻塞。返回读取的字节数
        if err != nil {
            fmt.Println("Read from client error:", err)
            fmt.Println("Connection with ", conn.RemoteAddr().String(), " down")
            break
        }
 
        fmt.Println(string(bytes[:n])) // 字节切片转string
    }
}
func main() {
    fmt.Println("Server on..")
    listen, err := net.Listen("tcp", "0.0.0.0:9999")
    // 建立tcp的监听套接字，监听本地9999号端口
    if (err != nil) {
        fmt.Println("Server listen error..")
        return
    }
    defer listen.Close()
    for {
        fmt.Println("Waiting for client to connect..")
        conn, err := listen.Accept() // 等待客户端连接
 
        if err != nil {
            fmt.Println("Client connect error..")
            continue
        }
        defer conn.Close()
 
        fmt.Println("Connection established with ip:", conn.RemoteAddr().String()) // 获取远程地址
        go process_client(conn)
    }
}
```

客户端方面，直接连接服务端，然后通过连接套接字发送信息即可

```go
package main
 
import (
    "fmt"
    "net"
    "bufio"
    "os"
    "strings"
)
 
func main() {
    conn, err := net.Dial("tcp", "localhost:9999") // 和本地9999端口建立tcp连接
    if err != nil {
        fmt.Println("Connect to server failure..")
        return
    }
 
    fmt.Println("Connected to server whose ip is ", conn.RemoteAddr().String())
 
    reader := bufio.NewReader(os.Stdin) // 建立控制台的reader
 
    for {
        line, err := reader.ReadString('\n') // 读取控制台一行信息
        if err != nil {
            fmt.Println("Read String error :", err)
        }
    
        line = strings.Trim(line, "\r\n")
 
        if line == "quit" {
            break
        }
    
        _, err = conn.Write([]byte(line)) // 向服务端发送信息，返回发送的字节数和错误
        if err != nil {
            fmt.Println("Write to server error:", err)
        }
    }
}
```

## go连接redis

首先安装所需第三方库redisgo

> go get github.com/garyburd/redigo/redis

1）、然后导包，并建立和服务器的连接

```go
import (
    "fmt"
    "github.com/garyburd/redigo/redis"
)

func main() {
    conn, err := redis.Dial("tcp", "localhost:6379")
    if err != nil {
        fmt.Println("redis connection failed..")
        return
    }
 
    defer conn.Close()
}
```

2）、往redis里写入数据

```go
_, err  = conn.Do("Set", "name", "songzeceng") // 参数列表：指令、键、值
if err != nil {
    fmt.Println("redis set failed..")
    return
}
```

在redis-cli中查看

```shell
get name
```

3）、从redis读取数据并转为字符串

```go
r, err := redis.String(conn.Do("Get", "name"))
fmt.Println("result = ", r)
```

4）、哈希的插入和读取

```go
_, err = conn.Do("HSet", "userhash01", "name", "szc") // 操作、哈希名、键、值
_, err = conn.Do("HSet", "userhash01", "age", 23)
 
r, err := redis.String(conn.Do("HGet", "userhash01", "name")) // 从哈希userhash01中读取name
fmt.Println("hash name = ", r)
age, err := redis.Int(conn.Do("HGet", "userhash01", "age")) // 读取int
fmt.Println("hash age = ", age)
```

在redis-cli中查看

```shell
hget userhash01 name
hget userhash01 age
```

5）、一次写入或读取多个值

```go
_, err = conn.Do("MSet", "name", "songzeceng", "home", "Henan,Anyang")
multi_r, err := redis.Strings(conn.Do("MGet", "name", "home")) // 注意String多了个s，而且multi_r是[]string
```

6）、为了提高效率，可以使用**redis连接池**来获取连接

```go
var pool *redis.Pool // 全局连接池指针
pool = &redis.Pool{
    MaxIdle: 8, // 最大空闲连接数
    MaxActive: 0, // 最大连接数,0表示不限
    IdleTimeout: 100, // 最大空闲时间
    Dial: func() (redis.Conn, error) { // 产生连接的函数
        return redis.Dial("tcp", "localhost:6379")
    },
}
conn := pool.Get() // 获取连接
defer pool.Close() // 连接池关闭
```

go连接redis也可使用： https://gopkg.in/redis.v8

1、获取第三方库

```shell
go get gopkg.in/redis.v8
```

2、引入

```she'l'l
import "gopkg.in/redis.v8"
```

## go连接mysql

首先下载github上的mysql驱动 https://github.com/go-sql-driver/mysql，放入GO_PATH环境变量下

然后导包

```go
import (
    "fmt"
    "database/sql" // 操作数据库的方法、结构体等
    _ "github.com/go-sql-driver/mysql" // 导入驱动，不用使用
    "os"
)
```

1）、连接数据库

```go
var (
    Db *sql.DB
    err error
)
 
func init() {
    Db, err = sql.Open("mysql", "root:root@tcp(localhost:3306)/test")
    if err != nil {
        fmt.Println("Open mysql error:", err)
        os.Exit(-1)
    }
}
```

sql.Open()函数的参数列表：数据库类型(mysql)，数据库url(用户名:密码@tcp(url)/数据库名)

2）、插入数据

先定义结构体(最好）

```go
type Sale struct {
    Widget_id int
    Qty int
    Street string
    City string
    State string
    Zip int
    Sale_date string
}
```

然后使用占位符+预编译的方式进行插入数据

```go
func (sale *Sale) AddSale() (err_ error) {
    sql_str := "insert into sales(widget_id, qty, street, city, state, zip, sale_date) values(?, ?, ?, ?, ?, ?, ?)"
 
    inStmt, err_ := Db.Prepare(sql_str) // 预编译
 
    _, err_ = inStmt.Exec(sale.Widget_id, sale.Qty, sale.Street, sale.City, sale.State, sale.Zip, sale.Sale_date) // 执行预编译语句，传入参数
 
    return err_
}

func main() {
    sale := &Sale{Widget_id: 9, Qty: 80, Street: "Huanghe South Road", City: "Anyang Henan", State: "China", Zip: 455000, Sale_date: "2020-03-24"}
 
    err_ := sale.AddSale()
    if err_ != nil {
        fmt.Println("sql execute err:", err_)
    }
}
```

或者使用单元测试，新建first_test.go文件，写入以下内容

```go
package main
 
import (
    "testing"
)
func TestAddSale(t *testing.T) {
    sale := &Sale{Widget_id: 9, Qty: 80, Street: "Huanghe South Road", City: "Anyang Henan", State: "China", Zip: 455000, Sale_date: "2020-03-24"}
 
    sale.AddSale()
}
```

然后在此目录下运行命令

> PS D:\develop\Go\workspace\src\go_code\go_web\src\main> go test
>
> PASS
> ok go_code/go_web/src/main 0.799s

3）、查询单条数据

```go
func (sale *Sale) GetRecordById() (ret *Sale, err_ error) {
    sql_str := "select * from sales where widget_id = ?"
    in_stmt, _ := Db.Prepare(sql_str)
 
    row := in_stmt.QueryRow(sale.Widget_id)
 
    if row == nil {
        fmt.Println("No such record with id = ", sale.Widget_id)
        return nil, errors.New("No such record with id = " + fmt.Sprintf("%d", sale.Widget_id))
    }
 
    ret = &Sale{}
 
    err_ = row.Scan(&ret.Widget_id, &ret.Qty, &ret.Street, &ret.City, &ret.State, &ret.Zip, &ret.Sale_date)
 
    return ret, err_
}
```

QueryRow()最多只接收一行查询结果，main函数中测试如下

```go
func main() {
    sale := &Sale{Widget_id: 9, Qty: 80, Street: "Huanghe South Road", City: "Anyang Henan", State: "China", Zip: 455000, Sale_date: "2020-03-24"}
 
    ret, _ := sale.GetRecordById()
     // {9 80 Huanghe South Road Anyang Henan China 455000 2020-03-24}
    if ret != nil {
        fmt.Println(*ret)
    }
}
```

4）、查询所有数据

```go
func (sale *Sale) GetAllRecord() (ret []*Sale, err_ error) {
    sql_str := "select * from sales"
    in_stmt, _ := Db.Prepare(sql_str)
 
    rows, err_ := in_stmt.Query()
 
    if err_ != nil {
        fmt.Println("Error get all: ", err_)
        return nil, err_
    }
 
    ret = make([]*Sale, 0)
 
    for rows.Next() {
        record := &Sale{}
 
        err_ = rows.Scan(&record.Widget_id, &record.Qty, &record.Street, &record.City, &record.State, &record.Zip, &record.Sale_date)
 
        if err_ != nil {
            fmt.Println("Error get record: ", err_)
            continue
        }
 
        ret = append(ret, record)
    }
 
    return ret, nil
}
```

Query()接收多行查询结果，main函数中测试如下

```go
func main() {
    sale := &Sale{}
 
    ret2, _ := sale.GetAllRecord()
    if ret2 != nil {
        for _, record := range ret2 {
            fmt.Println(*record)
        }
    }
 
/*    {1 20 Huasha Road Anyang Henan China 455000 2019-11-03}
...
{8 28 Dongfeng Road Anyang Henan China 455000 2019-11-10}
{9 80 Huanghe South Road Anyang Henan China 455000 2020-03-24}
*/
}
```

## 常用包

### 时间

```go
var goTime = "2022-04-02 15:04:05"
func main() {
	//字符串转 time
	str := "2022-04-22 15:00:00"
	res1, _ := time.ParseInLocation(goTime, str, time.Local)
	res2, _ := time.Parse(goTime, str)
	fmt.Printf("res1 = %v\n",res1)
	fmt.Printf("res2 = %v\n",res2)
	//time转字符串
	format1 := res1.Format(goTime)
	format2 := res2.Format(goTime)

	fmt.Printf("format1 = %v\n",format1)
	fmt.Printf("format2 = %v\n",format2)
}


func StrToTime(str string) *time.Time {
	t, _ := time.ParseInLocation("2022-04-1 15:04:05", str, time.Local)
	return &t
}

func TimeToStr(t *time.Time) string {
	return t.Format("2022-04-22 15:04:05")
}
```

### strconv

```go
   //将常规的数据类型转换为string
   str := fmt.Sprintf("%d%c",123,`a`)

// int转字符串
intSize := strconv.Itoa(123)
fmt.Printf("%T  \n",intSize)
atoi, _ := strconv.Atoi(intSize)
// 字符串转int
fmt.Printf("%T  \n",atoi)
// 将int64转字符串
formatInt := strconv.FormatInt(int64(123), 10)
fmt.Printf("%T  \n",formatInt)
// 将字符串转换成 10进制64位int 即int64
parseInt, _ := strconv.ParseInt("123", 10, 64)
fmt.Printf("%T  \n",parseInt)
```

## 结语

最后，列一下go语言的特点：

1、继承了C的指针

2、每个文件都属于一个包

3、垃圾回收

4、天然并发，goroutine，基于CPS并发模型实现

5、管道通信，解决goroutine之间的通信

6、函数返回多个值(Python)

7、切片、延迟执行defer等

