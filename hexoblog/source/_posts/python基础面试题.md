---
title: python基础面试题
date: 2022-12-12 21:43:09
tags: python
categories: python
img: /images/python基础面试题.jpg
---

## ***\*1、一行代码实现1--100之和\****

利用sum()函数求和



![640?wx_fmt=jpeg](https://img-blog.csdnimg.cn/img_convert/e9400bc1e18407f5167af94df6754f66.png)



***\*2、如何在一个函数内部修\*******\*改全局变量\****



函数内部global声明 修改全局变量



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d9d4d3843eb125b537ce6a879ce48b96.png)



***\*3、列出5个python标准库\****

os：提供了不少与操作系统相关联的函数

sys:  通常用于命令行参数

re:  正则匹配

math: 数学运算

datetime:处理日期时间



***\*4、字典如何删除键和合并两个字典\****

del和update方法



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/a876558709981c2f8b9df842d6650691.png)



***\*5、谈下python的GIL\****

GIL 是python的全局解释器锁，同一进程中假如有多个线程运行，一个线程在运行python程序的时候会霸占python解释器（加了一把锁即GIL），使该进程内的其他线程无法运行，等该线程运行完后其他线程才能运行。如果线程运行过程中遇到耗时操作，则解释器锁解开，使其他线程运行。所以在多线程中，线程的运行仍是有先后顺序的，并不是同时进行。

多进程中因为每个进程都能被系统分配资源，相当于每个进程有了一个python解释器，所以多进程可以实现多个进程的同时运行，缺点是进程系统资源开销大



***\*6、python实现列表去重的方法\****



先通过集合去重，在转列表



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/be0552738aa3b5bec9ffaeb652d3938d.png)



***\*7、fun(\*args,\*\*kwargs)中的\*args,\*\*kwargs什么意思？\****

![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/9150e69f5eb5e53551aa30fa4bd7f265.png)

![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/918948df20aec7cd09db85b1577f5f4b.png)



***\*8、python2和python3的range(100)的区别\****



python2返回列表，python3返回迭代器，节约内存.



***\*9、一句话解释什么样的语言能够用装饰器?\****



函数可以作为参数传递的语言，可以使用装饰器。



***\*10、python内建数据类型有哪些\****



整型--int

布尔型--bool

字符串--str

列表--list

元组--tuple

字典--dict



## ***\*11、简述面向对象中__new__和__init__区别\****



__init__是初始化方法，创建对象后，就立刻被默认调用了，可接收参数，如图



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/161e29f80c52f8035e2e455029680d7c.png)



1、__new__至少要有一个参数cls，代表当前类，此参数在实例化时由Python解释器自动识别。

2、__new__必须要有返回值，返回实例化出来的实例，这点在自己实现__new__时要特别注意，可以return父类（通过super(当前类名, cls)）__new__出来的实例，或者直接是object的__new__出来的实例。



3、__init__有一个参数self，就是这个__new__返回的实例，__init__在__new__的基础上可以完成一些其它初始化的动作，__init__不需要返回值。



4、如果__new__创建的是当前类的实例，会自动调用__init__函数，通过return语句里面调用的__new__函数的第一个参数是cls来保证是当前类实例，如果是其他类的类名，；那么实际创建返回的就是其他类的实例，其实就不会调用当前类的__init__函数，也不会调用其他类的__init__函数。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/94703017a6691a8fbf2ea2a6b207a0a8.png)



***\*12、简述with方法打开处理文件帮我我们做了什么？\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/7373673062ead266f953683c4a607419.png)



打开文件在进行读写的时候可能会出现一些异常状况，如果按照常规的f.open写法，我们需要try,except,finally，做异常判断，并且文件最终不管遇到什么情况，都要执行finally f.close()关闭文件，with方法帮我们实现了finally中f.close（当然还有其他自定义功能，有兴趣可以研究with方法源码）。



***\*13、列表[1,2,3,4,5]，请使用map()函数输出[1,4,9,16,25]，并使用列表推导式提取出大于10的数，最终输出[16,25]？\****



map()函数第一个参数是fun，第二个参数是一般是list，第三个参数可以写list，也可以不写，根据需求。



![640?wx_fmt=jpeg](https://img-blog.csdnimg.cn/img_convert/52717e67344878c689cb3526ed3a279d.png)



***\*14、python中生成随机整数、随机小数、0--1之间小数方法\****



随机整数：random.randint(a,b),生成区间内的整数。

随机小数：习惯用numpy库，利用np.random.randn(5)生成5个随机小数。

0-1随机小数：random.random(),括号中不传参。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/72584ffddb0f5a9551f70cadc636e7de.png)



***\*15、避免转义给字符串加哪个字母表示原始字符串？\****



r , 表示需要原始字符串，不转义特殊字符。



***\*16、<div class="nam">中国</div>，用正则匹配出标签里面的内容（“中国”），其中class的类名是不确定的。\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/112c30c3a7ff037bb79a7468fa620ee4.png)



***\*17、python中断言方法举例\****



assert（）方法，断言成功，则程序继续执行，断言失败，则程序报错。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/7587c20e72c3c9fc5554b81caf10c214.png)



***\*18、数据表student有id,name,score,city字段，其中name中的名字可有重复，需要消除重复行,请写sql语句\****



select  distinct  name  from  student



***\*19、10个Linux常用命令\****



ls  pwd  cd  touch  rm  mkdir  tree  cp  mv  cat  more  grep  echo 



***\*20、python2和python3区别？列举5个\****



1、Python3 使用 print 必须要以小括号包裹打印内容，比如 print('hi')

Python2 既可以使用带小括号的方式，也可以使用一个空格来分隔打印内容，比如 print 'hi'

2、python2 range(1,10)返回列表，python3中返回迭代器，节约内存

3、python2中使用ascii编码，python中使用utf-8编码

4、python2中unicode表示字符串序列，str表示字节序列

   python3中str表示字符串序列，byte表示字节序列

5、python2中为正常显示中文，引入coding声明，python3中不需要

6、python2中是raw_input()函数，python3中是input()函数



***\*21、列出python中可变数据类型和不可变数据类型，并简述原理\****



不可变数据类型：数值型、字符串型string和元组tuple不允许变量的值发生变化，如果改变了变量的值，相当于是新建了一个对象，而对于相同的值的对象，在内存中则只有一个对象（一个地址），如下图用id()方法可以打印对象的id。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/182afb27d4f2f520e096da9d28c90a53.png)



可变数据类型：列表list和字典dict；允许变量的值发生变化，即如果对变量进行append、+=等这种操作后，只是改变了变量的值，而不会新建一个对象，变量引用的对象的地址也不会变化，不过对于相同的值的不同对象，在内存中则会存在不同的对象，即每个对象都有自己的地址，相当于内存中对于同值的对象保存了多份，这里不存在引用计数，是实实在在的对象。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/a047a4121dee265f46d38cdb261ecaea.png)



## ***\*22、s = "ajldjlajfdljfddd"，去重并从小到大排序输出"adfjl"\****



set去重，去重转成list,利用sort方法排序，reeverse=False是从小到大排

list是不 变数据类型，s.sort时候没有返回值，所以注释的代码写法不正确。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/b50009544d1f17457181f0b1a7153ec4.png)



***\*23、用lambda函数实现两个数相乘\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/395dc1a53195c1a75f5077f9e2503cb2.png)



***\*24、字典根据键从小到大排序\****



dic={"name":"zs","age":18,"city":"深圳","tel":"1362626627"}



![640?wx_fmt=jpeg](https://img-blog.csdnimg.cn/img_convert/f47fd9b6d401f19765ff29d2a3809c0f.png)



***\*25、利用collections库的Counter方法统计字符串每个单词出现的次数"kjalfj;ldsjafl;hdsllfdhg;lahfbl;hl;ahlf;h"\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/7802dc3445f9fa5c4daacda11ed08aca.png)



***\*26、字符串a = "not 404 found 张三 99 深圳"，每个词中间是空格，用正则过滤掉英文和数字，最终输出"张三  深圳"\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/fee01919ff8c3aa02ad3095bdcb22497.png)



顺便贴上匹配小数的代码，虽然能匹配，但是健壮性有待进一步确认。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/f4cfe8f9720e2149a211dba2a86fa796.png)



***\*27、filter方法求出列表所有奇数并构造新列表，a =  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]\****



filter() 函数用于过滤序列，过滤掉不符合条件的元素，返回由符合条件元素组成的新列表。该接收两个参数，第一个为函数，第二个为序列，序列的每个元素作为参数传递给函数进行判，然后返回 True 或 False，最后将返回 True 的元素放到新列表。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/8205b542940e1ff77ef0c837ca2df6b5.png)



***\*28、列表推导式求列表所有奇数并构造新列表，a =  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/13b68252364be34b3dc441b0b0d9e886.png)



***\*29、正则re.complie作用\****



re.compile是将正则表达式编译成一个对象，加快速度，并重复使用。



30、a=（1，）b=(1)，c=("1") 分别是什么类型的数据？



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/256913c48d0a1ade6f5c2ef96923f363.png)



***\*31、两个列表[1,5,7,9]和[2,2,6,8]合并为[1,2,2,3,6,7,8,9]\****



extend可以将另一个集合中的元素逐一添加到列表中，区别于append整体添加。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/e9b14585c068e11e3f9bc34f5c993bac.png)



***\*32、用python删除文件和用linux命令删除文件方法\****



python：os.remove(文件名)

linux:    rm  文件名



## ***\*33、log日志中，我们需要用时间戳记录error,warning等的发生时间，请用datetime模块打印当前时间戳 “2018-04-01 11:38:54”，顺便把星期的代码也贴上。\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/16d87fb048efe5e1eb6f5638bf582193.png)



***\*34、数据库优化查询方法\****



外键、索引、联合查询、选择特定字段等等



***\*35、请列出你会的任意一种统计图（条形图、折线图等）绘制的开源库，第三方也行\****



pychart、matplotlib



***\*36、写一段自定义异常代码\****



自定义异常用raise抛出异常。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/9d39f7488cf2eb0d2a472282d8eb08c6.png)



***\*37、正则表达式匹配中，（.\*）和（.\*?）匹配区别？\****



（.*）是贪婪匹配，会把满足正则的尽可能多的往后匹配

（.*?）是非贪婪匹配，会把满足正则的尽可能少匹配



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/4833ebcfe511746b78c054c5c81c2dd9.png)



***\*38、简述Django的orm\****



ORM，全拼Object-Relation Mapping，意为对象-关系映射。实现了数据模型与数据库的解耦，通过简单的配置就可以轻松更换数据库，而不需要修改代码只需要面向对象编程,orm操作本质上会根据对接的数据库引擎，翻译成对应的sql语句,所有使用Django开发的项目无需关心程序底层使用的是MySQL、Oracle、sqlite....，如果数据库迁移，只需要更换Django的数据库引擎即可。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/aeddc2271be018c3434ca5fa6944fe2b.png)



***\*39、[[1,2],[3,4],[5,6]]一行代码展开该列表，得出[1,2,3,4,5,6]\****



列表推导式的骚操作![2_02.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_02.png)![2_02.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_02.png)![2_02.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_02.png)

运行过程：for i in a ,每个i是【1,2】，【3,4】，【5,6】，for j in i，每个j就是1,2,3,4,5,6,合并后就是结果。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d4623346fc8c8a687138f7d5d904b6e4.png)



还有更骚的方法，将列表转成numpy矩阵，通过numpy的flatten（）方法，代码永远是只有更骚，没有最骚![2_06.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_06.png)![2_06.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_06.png)![2_06.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_06.png)



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/46a7674e45005fb0310d969c24576039.png)



40、x="abc",y="def",z=["d","e","f"],分别求出x.join(y)和x.join(z)返回的结果



join()括号里面的是可迭代对象，x插入可迭代对象中间，形成字符串，结果一致，有没有突然感觉字符串的常见操作都不会玩了![2_12.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_12.png)![2_12.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_12.png)![2_12.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/emoji_wx/2_12.png)



顺便建议大家学下os.path.join()方法，拼接路径经常用到，也用到了join,和字符串操作中的join有什么区别，该问题大家可以查阅相关文档。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/7ec3ddcbc8db6ff3ecef14978a27b959.png)



***\*41、举例说明异常模块中try except else finally的相关意义\****



try..except..else没有捕获到异常，执行else语句。

try..except..finally不管是否捕获到异常，都执行finally语句。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/2b0948dc60ed9f28e81f5c4729d564a4.png)



***\*42、python中交换两个数值\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/248ff6518f829d711774b7267c31d8ca.png)



***\*43、举例说明zip()函数用法\****



zip()函数在运算时，会以一个或多个序列（可迭代对象）做为参数，返回一个元组的列表。同时将这些序列中并排的元素配对。



zip()参数可以接受任何类型的序列，同时也可以有两个以上的参数;当传入参数的长度不同时，zip能自动以最短序列长度为准进行截取，获得元组。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/218e014daff1cd0c6d2766ea4896a3da.png)



## ***\*44、a="张明 98分"，用re.sub，将98替换为100\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/835100b1876321c28715111d36be6e13.png)



***\*45、写5条常用sql语句\****



show databases;

show tables;

desc 表名;

select * from 表名;

delete from 表名 where id=5;

update students set gender=0,hometown="北京" where id=5



***\*46、a="hello"和b="你好"编码成bytes类型\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/81c4a51d19cfd991a53aab2225b3b8b8.png)



***\*47、[1,2,3]+[4,5,6]的结果是多少？\****



两个列表相加，等价于extend。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/371774453c4202b3ce044d62151cfcf1.png)



***\*48、提高python运行效率的方法\****



1、使用生成器，因为可以节约大量内存

2、循环代码优化，避免过多重复代码的执行

3、核心模块用Cython  PyPy等，提高效率

4、多进程、多线程、协程

5、多个if elif条件判断，可以把最有可能先发生的条件放到前面写，这样可以减少程序判断的次数，提高效率



***\*49、简述mysql和redis区别\****



redis： 内存型非关系数据库，数据保存在内存中，速度快

mysql：关系型数据库，数据保存在磁盘中，检索的话，会有一定的Io操作，访问速度相对慢



***\*50、遇到bug如何处理\****



1、细节上的错误，通过print（）打印，能执行到print（）说明一般上面的代码没有问题，分段检测程序是否有问题，如果是js的话可以alert或console.log。

2、如果涉及一些第三方框架，会去查官方文档或者一些技术博客。

3、对于bug的管理与归类总结，一般测试将测试出的bug用teambin等bug管理工具进行记录，然后我们会一条一条进行修改，修改的过程也是理解业务逻辑和提高自己编程逻辑缜密性的方法，我也都会收藏做一些笔记记录。

4、导包问题、城市定位多音字造成的显示错误问题。



***\*51、正则匹配，匹配日期2018-03-20\****



url='https://sycm.taobao.com/bda/tradinganaly/overview/get_summary.json?dateRange=2018-03-20%7C2018-03-20&dateType=recent1&device=1&token=ff25b109b&_=1521595613462'

仍有同学问正则，其实匹配并不难，提取一段特征语句，用（.*?）匹配即可。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/b6b4c8d22be1bc1b6cf29a11b778362a.png)



***\*52、list=[2,3,5,4,9,6]，从小到大排序，不许用sort，输出[2,3,4,5,6,9]\****



利用min()方法求出最小值，原列表删除最小值，新列表加入最小值，递归调用获取最小值的函数，反复操作。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/5aa2ab3bcf1851019455bad8b1036bb4.png)



***\*53、写一个单列模式\****



因为创建对象时__new__方法执行，并且必须return 返回实例化出来的对象所cls.__instance是否存在，不存在的话就创建对象，存在的话就返回该对象，来保证只有一个实例对象存在（单列），打印ID，值一样，说明对象同一个。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/dadf57886f6063cf8a9b51f5202753df.png)



***\*54、保留两位小数\****



题目本身只有a="%.03f"%1.3335,让计算a的结果，为了扩充保留小数的思路，提供round方法（数值，保留位数）。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d18772156875b71f51c18305747b1741.png)



## ***\*55、求三个方法打印结果\****



fn("one",1）直接将键值对传给字典；

fn("two",2)因为字典在内存中是可变数据类型，所以指向同一个地址，传了新的额参数后，会相当于给字典增加键值对；

fn("three",3,{})因为传了一个新字典，所以不再是原先默认参数的字典。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d699d255a47da84e0f483531e3f27a60.png)



***\*56、列出常见的状态码和意义\****



200 OK 

请求正常处理完毕

204 No Content 

请求成功处理，没有实体的主体返回

206 Partial Content 

GET范围请求已成功处理

301 Moved Permanently 

永久重定向，资源已永久分配新URI

302 Found 

临时重定向，资源已临时分配新URI

303 See Other 

临时重定向，期望使用GET定向获取

304 Not Modified 

发送的附带条件请求未满足

307 Temporary Redirect 

临时重定向，POST不会变成GET

400 Bad Request 

请求报文语法错误或参数错误

401 Unauthorized 

需要通过HTTP认证，或认证失败

403 Forbidden 

请求资源被拒绝

404 Not Found 

无法找到请求资源（服务器无理由拒绝）

500 Internal Server Error 

服务器故障或Web应用故障

503 Service Unavailable 

服务器超负载或停机维护



***\*57、分别从前端、后端、数据库阐述web项目的性能优化
\****



该题目网上有很多方法，我不想截图网上的长串文字，看的头疼，按我自己的理解说几点。



前端优化：

1、减少http请求、例如制作精灵图。

2、html和CSS放在页面上部，javascript放在页面下面，因为js加载比HTML和Css加载慢，所以要优先加载html和css,以防页面显示不全，性能差，也影响用户体验差。



后端优化：

1、缓存存储读写次数高，变化少的数据，比如网站首页的信息、商品的信息等。应用程序读取数据时，一般是先从缓存中读取，如果读取不到或数据已失效，再访问磁盘数据库，并将数据再次写入缓存。

2、异步方式，如果有耗时操作，可以采用异步，比如celery。

3、代码优化，避免循环和判断次数太多，如果多个if else判断，优先判断最有可能先发生的情况。



数据库优化：

1、如有条件，数据可以存放于redis，读取速度快。

2、建立索引、外键等。



***\*58、使用pop和del删除字典中的"name"字段，dic={"name":"zs","age":18}\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/b9df948006223d127965075b3cb90a58.png)



***\*59、列出常见MYSQL数据存储引擎\****



InnoDB：支持事务处理，支持外键，支持崩溃修复能力和并发控制。如果需要对事务的完整性要求比较高（比如银行），要求实现并发控制（比如售票），那选择InnoDB有很大的优势。如果需要频繁的更新、删除操作的数据库，也可以选择InnoDB，因为支持事务的提交（commit）和回滚（rollback）。 



MyISAM：插入数据快，空间和内存使用比较低。如果表主要是用于插入新记录和读出记录，那么选择MyISAM能实现处理高效率。如果应用的完整性、并发性要求比 较低，也可以使用。



MEMORY：所有的数据都在内存中，数据的处理速度快，但是安全性不高。如果需要很快的读写速度，对数据的安全性要求较低，可以选择MEMOEY。它对表的大小有要求，不能建立太大的表。所以，这类数据库只使用在相对较小的数据库表。



***\*60、计算代码运行结果，zip函数历史文章已经说了，得出[("a",1),("b",2)，("c",3), ("d",4), ("e",5)]\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/e291c36078ca1d792a074aae41288aae.png)



dict()创建字典新方法



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/f1f764bc28e354985f05774c43e3e33e.png)



***\*61、简述同源策略\****



 同源策略需要同时满足以下三点要求： 

1）协议相同 

 2）域名相同 

3）端口相同 

 http:www.test.com与https:www.test.com 不同源——协议不同 

 http:www.test.com与http:www.admin.com 不同源——域名不同 

 http:www.test.com与http:www.test.com:8081 不同源——端口不同

 只要不满足其中任意一个要求，就不符合同源策略，就会出现“跨域”。



***\*62、简述cookie和session的区别\****



1、session 在服务器端，cookie 在客户端（浏览器）。

2、session 的运行依赖 session id，而 session id 是存在 cookie 中的，也就是说，如果浏览器禁用了 cookie ，同时 session 也会失效，存储Session时，键与Cookie中的sessionid相同，值是开发人员设置的键值对信息，进行了base64编码，过期时间由开发人员设置。

3、cookie安全性比session差。



***\*63、简述多线程、多进程\****



进程：

1、操作系统进行资源分配和调度的基本单位，多个进程之间相互独立

2、稳定性好，如果一个进程崩溃，不影响其他进程，但是进程消耗资源大，开启的进程数量有限制



线程：

1、CPU进行资源分配和调度的基本单位，线程是进程的一部分，是比进程更小的能独立运行的基本单位，一个进程下的多个线程可以共享该进程的所有资源。

2、如果IO操作密集，则可以多线程运行效率高，缺点是如果一个线程崩溃，都会造成进程的崩溃。



应用：

IO密集的用多线程，在用户输入，sleep 时候，可以切换到其他线程执行，减少等待的时间

CPU密集的用多进程，因为假如IO操作少，用多线程的话，因为线程共享一个全局解释器锁，当前运行的线程会霸占GIL，其他线程没有GIL，就不能充分利用多核CPU的优势



***\*64、简述any()和all()方法\****



any():只要迭代器中有一个元素为真就为真

all():迭代器中所有的判断项返回都是真，结果才为真

python中什么元素为假？

答案：（0，空字符串，空列表、空字典、空元组、None, False）



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/ccc9e828034984c7efa392e42e1ba924.png)



测试all()和any()方法



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/fd8062050d9872b59ab9383c684589f1.png)



***\*65、IOError、AttributeError、ImportError、IndentationError、IndexError、KeyError、SyntaxError、NameError分别代表什么异常\****



IOError：输入输出异常

AttributeError：试图访问一个对象没有的属性

ImportError：无法引入模块或包，基本是路径问题

IndentationError：语法错误，代码没有正确的对齐

IndexError：下标索引超出序列边界

KeyError：试图访问你字典里不存在的键

SyntaxError：Python代码逻辑语法出错，不能执行

NameError：使用一个还未赋予对象的变量



## ***\*66、python中copy和deepcopy区别\****



1、复制不可变数据类型，不管copy还是deepcopy,都是同一个地址当浅复制的值是不可变对象（数值，字符串，元组）时和=“赋值”的情况一样，对象的id值与浅复制原来的值相同。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/7217ce8002ce640a04db8f25b76f267e.png)



2、复制的值是可变对象（列表和字典）



浅拷贝copy有两种情况：

第一种情况：复制的 对象中无 复杂 子对象，原来值的改变并不会影响浅复制的值，同时浅复制的值改变也并不会影响原来的值。原来值的id值与浅复制原来的值不同。

第二种情况：复制的对象中有 复杂 子对象 （例如列表中的一个子元素是一个列表）， 改变原来的值 中的复杂子对象的值 ，会影响浅复制的值。

深拷贝deepcopy：完全复制独立，包括内层列表和字典。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/6a2fd48f5f35848caaccfd45eaeab909.png)



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/c06a296694ee95396f91f2450e153eef.png)



***\*67、列出几种魔法方法并简要介绍用途\****



__init__:对象初始化方法

__new__:创建对象时候执行的方法，单列模式会用到

__str__:当使用print输出对象的时候，只要自己定义了__str__(self)方法，那么就会打印从在这个方法中return的数据

__del__:删除对象执行的方法



68、C:\Users\ry-wu.junya\Desktop>python 1.py 22 33命令行启动程序并传参，print(sys.argv)会输出什么数据？



文件名和参数构成的列表



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/bd256f3076fc91048bb89f95aeaa096c.png)



***\*69、请将[i for i in range(3)]改成生成器\****



生成器是特殊的迭代器：

1、列表表达式的【】改为（）即可变成生成器。

2、函数在返回值得时候出现yield就变成生成器，而不是函数了，

中括号换成小括号即可，有没有惊呆了![smiley_14.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/smiley/smiley_14.png)![smiley_14.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/smiley/smiley_14.png)![smiley_14.png](https://res.wx.qq.com/mpres/htmledition/images/icon/common/emotion_panel/smiley/smiley_14.png)



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/c99346e1e9c399eff8f0de3ec740642d.png)



***\*70、a = "  hehheh  ",去除收尾空格\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/5011f439df7dc84f438ea197dbb28c24.png)



***\*71、举例sort和sorted对列表排序，list=[0,-1,3,-10,5,9]\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/0d8d82080d857b98c0299be50c742e28.png)



***\*72、对list排序foo = [-5,8,0,4,9,-4,-20,-2,8,2,-4],使用lambda函数从小到大排序\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/b7752bf2f11e4c417c9c8f1b494f134a.png)



***\*73、使用lambda函数对list排序foo = [-5,8,0,4,9,-4,-20,-2,8,2,-4]，输出结果为[0,2,4,8,8,9,-2,-4,-4,-5,-20]，正数从小到大，负数从大到小】（传两个条件，x<0和abs(x)）\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/888616a7e2a6999aea28b38a8e6900f2.png)



***\*74、列表嵌套字典的排序，分别根据年龄和姓名排序\****



foo = [{"name":"zs","age":19},{"name":"ll","age":54},

​    {"name":"wa","age":17},{"name":"df","age":23}]



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/ce1687986d774ac400815c50d2b360ea.png)



***\*75、列表嵌套元组，分别按字母和数字排序\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/9c763ef7f721e6ecde87b3b656c53cb4.png)



***\*76、列表嵌套列表排序，年龄数字相同怎么办？\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/98c5028af6d13a630f6c03d8fcfe41e1.png)



## ***\*77、根据键对字典排序（方法一，zip函数）\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/183c0f7d8740a0ce5ec42e47ec03ee1b.png)



***\*78、根据键对字典排序（方法二，不用zip)\****



有没有发现dic.items和zip(dic.keys(),dic.values())都是为了构造列表嵌套字典的结构，方便后面用sorted()构造排序规则。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/9f4f6190fe42589669093ac5eb172f08.png)



***\*79、列表推导式、字典推导式、生成器\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/fe03bf4242501edb2edbd2ecf8548d49.png)



***\*80、最后出一道检验题目，根据字符串长度排序，看排序是否灵活运用\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/8964d699c472d8bed9cc52bc61af55e5.png)



***\*81、举例说明SQL注入和解决办法\****



当以字符串格式化书写方式的时候，如果用户输入的有;+SQL语句，后面的SQL语句会执行，比如例子中的SQL注入会删除数据库demo。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/34b702878753b29667de65d8e576c353.png)



解决方式：通过传参数方式解决SQL注入



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/944ff6a5158d26319d2fa5317b47733d.png)

***\*82、s="info:xiaoZhang 33 shandong",用正则切分字符串输出['info', 'xiaoZhang', '33', 'shandong']\****



|表示或，根据冒号或者空格切分



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d1ba63707cfba35b3ab62275d4825be7.png)



***\*83、正则匹配以163.com结尾的邮箱\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/046efc006cef6973fda779e404bdd22d.png)



***\*84、递归求和\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/ac4ee347ef9a487d4cb6ce321a05a1c2.png)



***\*85、python字典和json字符串相互转化方法\****



json.dumps()字典转json字符串，json.loads()json转字典



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/3ecb60f3c3d5645fa76b7b74f9168ce0.png)



***\*86、MyISAM 与 InnoDB 区别\****



1、InnoDB 支持事务，MyISAM 不支持，这一点是非常之重要。事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而 MyISAM就不可以了；

2、MyISAM 适合查询以及插入为主的应用，InnoDB 适合频繁修改以及涉及安全性较高的应用；

3、InnoDB 支持外键，MyISAM 不支持；

4、对于自增长的字段，InnoDB 中必须包含只有该字段的索引，但是在 MyISAM表中可以和其他字段一起建立联合索引；

5、清空整个表时，InnoDB 是一行一行的删除，效率非常慢。MyISAM 则会重建表。



***\*87、统计字符串中某字符出现次数\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d5834294cbcc07fe5ba1c6ba8e136742.png)



## ***\*88、字符串转化大小写\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/11e2927484e665f097bb061c0155d799.png)



***\*89、用两种方法去空格\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/fe186662222f9da995ffad127667be78.png)



***\*90、正则匹配不是以4和7结尾的手机号\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/a60ef4091ffdd5994a19336e644184f6.png)



***\*91、简述python引用计数机制\****



python垃圾回收主要以引用计数为主，标记-清除和分代清除为辅的机制，其中标记-清除和分代回收主要是为了处理循环引用的难题。



引用计数算法

当有1个变量保存了对象的引用时，此对象的引用计数就会加1；

当使用del删除变量指向的对象时，如果对象的引用计数不为1，比如3，那么此时只会让这个引用计数减1，即变为2，当再次调用del时，变为1，如果再调用1次del，此时会真的把对象进行删除。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/868a0f2be31d79e971481f57b305d238.png)



***\*92、int("1.4"),int(1.4)输出结果？\****



int("1.4")报错，int(1.4)输出1



***\*93、列举3条以上PEP8编码规范\****



1、顶级定义之间空两行，比如函数或者类定义。

2、方法定义、类定义与第一个方法之间，都应该空一行。

3、三引号进行注释。

4、使用Pycharm、Eclipse一般使用4个空格来缩进代码。



***\*94、正则表达式匹配第一个URL\****



findall结果无需加group(),search需要加group()提取



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/c96f9f38041bb48b9a1c9f010b52834d.png)



***\*95、正则匹配中文\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/a2c95bfa46606da5f394a1192ba5484b.png)



***\*96、简述乐观锁和悲观锁\****



悲观锁, 就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。



乐观锁，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制，乐观锁适用于多读的应用类型，这样可以提高吞吐量



***\*97、r、r+、rb、rb+文件打开模式区别\****



模式较多，比较下背背记记即可



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/d1ac7e5a2ce6120f2af27e45e147ff63.png)



***\*98、Linux命令重定向 > 和 >>\****



Linux允许将命令执行结果重定向到一个文件，将本应显示在终端上的内容 输出／追加 到指定文件中。



\> 表示输出，会覆盖文件原有的内容

\>> 表示追加，会将内容追加到已有文件的末尾



用法示例：

```

```

将 echo 输出的信息保存到 1.txt 里echo Hello Python > 1.txt
将 tree 输出的信息追加到 1.txt 文件的末尾tree >> 1.txt



## ***\*99、正则表达式匹配出<html><h1>www.itcast.cn</h1></html>\****



前面的<>和后面的<>是对应的，可以用此方法



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/fab86ed0971a49c843531e52056bde79.png)



***\*100、python传参数是传值还是传址？\****



Python中函数参数是引用传递（注意不是值传递）。对于不可变类型（数值型、字符串、元组），因变量不能修改，所以运算不会影响到变量自身；而对于可变类型（列表字典）来说，函数体运算可能会更改传入的参数变量。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/2489abe99561555900167a966971fd9d.png)



## ***\*101、求两个列表的交集、差集、并集\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/23ea4035fb43d6ed27e7ae974d5bc303.png)



***\*102、生成0-100的随机数\****



random.random()生成0-1之间的随机小数，所以乘以100



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/875abaa61d42e3c763291870cc441696.png)



***\*103、lambda匿名函数好处\****



精简代码，lambda省去了定义函数，map省去了写for循环过程



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/70e07d075c535a4cea20b50433a1554f.png)



***\*104、常见的网络传输协议\****



UDP、TCP、FTP、HTTP、SMTP等等



***\*105、单引号、双引号、三引号用法\****



1、单引号和双引号没有什么区别，不过单引号不用按shift，打字稍微快一点。表示字符串的时候，单引号里面可以用双引号，而不用转义字符,反之亦然。

'She said:"Yes." ' or "She said: 'Yes.' "



2、但是如果直接用单引号扩住单引号，则需要转义，像这样：

 ' She said:\'Yes.\' '



3、三引号可以直接书写多行，通常用于大段，大篇幅的字符串

"""

hello

world

"""



***\*106、python垃圾回收机制\****



python垃圾回收主要以引用计数为主，标记-清除和分代清除为辅的机制，其中标记-清除和分代回收主要是为了处理循环引用的难题。



引用计数算法

当有1个变量保存了对象的引用时，此对象的引用计数就会加1；

当使用del删除变量指向的对象时，如果对象的引用计数不为1，比如3，那么此时只会让这个引用计数减1，即变为2，当再次调用del时，变为1，如果再调用1次del，此时会真的把对象进行删除。



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/ab802b051ba7f59207422808f5de4743.png)



***\*107、HTTP请求中get和post区别\****



1、GET请求是通过URL直接请求数据，数据信息可以在URL中直接看到，比如浏览器访问；而POST请求是放在请求头中的，我们是无法直接看到的。



2、GET提交有数据大小的限制，一般是不超过1024个字节，而这种说法也不完全准确，HTTP协议并没有设定URL字节长度的上限，而是浏览器做了些处理，所以长度依据浏览器的不同有所不同；POST请求在HTTP协议中也没有做说明，一般来说是没有设置限制的，但是实际上浏览器也有默认值。总体来说，少量的数据使用GET，大量的数据使用POST。



3、GET请求因为数据参数是暴露在URL中的，所以安全性比较低，比如密码是不能暴露的，就不能使用GET请求；POST请求中，请求参数信息是放在请求头的，所以安全性较高，可以使用。在实际中，涉及到登录操作的时候，尽量使用HTTPS请求，安全性更好。



***\*108、python中读取Excel文件的方法\****



应用数据分析库pandas



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/16efab4f74aa6dab3f4527326e84b445.png)



***\*109、简述多线程、多进程\****



进程：

1、操作系统进行资源分配和调度的基本单位，多个进程之间相互独立。

2、稳定性好，如果一个进程崩溃，不影响其他进程，但是进程消耗资源大，开启的进程数量有限制。



线程：

1、CPU进行资源分配和调度的基本单位，线程是进程的一部分，是比进程更小的能独立运行的基本单位，一个进程下的多个线程可以共享该进程的所有资源。

2、如果IO操作密集，则可以多线程运行效率高，缺点是如果一个线程崩溃，都会造成进程的崩溃。



应用：

IO密集的用多线程，在用户输入，sleep 时候，可以切换到其他线程执行，减少等待的时间。

CPU密集的用多进程，因为假如IO操作少，用多线程的话，因为线程共享一个全局解释器锁，当前运行的线程会霸占GIL，其他线程没有GIL，就不能充分利用多核CPU的优势。



***\*110、python正则中search和match\****



![640?wx_fmt=png](https://img-blog.csdnimg.cn/img_convert/487902921e1ad5cbbffc5f196d7e5de8.png)

102、在Python中是如何管理内存的？

一对象的引用计数机制,二垃圾回收机制,三内存池机制

Python有一个私有堆空间来保存所有的对象和数据结构。作为开发者，我们无法访问它，是解释器在管理它。但是有了核心API后，我们可以访问一些工具。Python内存管理器控制内存分配。

另外，内置垃圾回收器会回收使用所有的未使用内存，所以使其适用于堆空间。

103、解释Python中的help()和dir()函数
Help()函数是一个内置函数，用于查看函数或模块用途的详细说明：

>>> import copy
>>> help(copy.copy)
>>> 运行结果为：

Help on function copy in module copy:


copy(x)

Shallow copy operation on arbitrary Python objects.

See the module’s __doc__ string for more info.

Dir()函数也是Python内置函数，dir() 函数不带参数时，返回当前范围内的变量、方法和定义的类型列表；带参数时，返回参数的属性、方法列表。

以下实例展示了 dir 的使用方法：

>>> dir(copy.copy)
>>> 1
>>> 运行结果为：

[‘__annotations__’, ‘__call__’, ‘__class__’, ‘__closure__’, ‘__code__’, ‘__defaults__’, ‘__delattr__’, ‘__dict__’, ‘__dir__’, ‘__doc__’, ‘__eq__’, ‘__format__’, ‘__ge__’, ‘__get__’, ‘__getattribute__’, ‘__globals__’, ‘__gt__’, ‘__hash__’, ‘__init__’, ‘__init_subclass__’, ‘__kwdefaults__’, ‘__le__’, ‘__lt__’, ‘__module__’, ‘__name__’, ‘__ne__’, ‘__new__’, ‘__qualname__’, ‘__reduce__’, ‘__reduce_ex__’, ‘__repr__’, ‘__setattr__’, ‘__sizeof__’, ‘__str__’, ‘__subclasshook__’]

104、 当退出Python时，是否释放全部内存？
答案是No。循环引用其它对象或引用自全局命名空间的对象的模块，在Python退出时并非完全释放。

另外，也不会释放C库保留的内存部分。

105、 什么是猴子补丁？
在运行期间动态修改一个类或模块。

```python
class A:
    def func(self):
        print("Hi")
        def monkey(self):
            print "Hi, monkey"
m.A.func = monkey
a = m.A()
a.func()
```

Hi, Monkey
106、解释Python中的join()和split()函数
Join()能让我们将指定字符添加至字符串中。

>>> ','.join('12345')
>>> 1
>>> 运行结果：

‘1,2,3,4,5’
1
Split()能让我们用指定字符分割字符串。

>>> '1,2,3,4,5'.split(',')
>>> 1
>>> 运行结果：

[‘1’, ‘2’, ‘3’, ‘4’, ‘5’]
107、元组的解封装是什么？
首先我们来看解封装：

>>> mytuple=3,4,5
>>> mytuple
>>> (3, 4, 5)
>>> 1
>>> 2
>>> 3
>>> 这将 3，4，5 封装到元组 mytuple 中。

现在我们将这些值解封装到变量 x，y，z 中：

>>> x,y,z=mytuple
>>> x+y+z

108、Python代码实现删除一个list里面的重复元素
答：

1,使用set函数，set(list)

2，使用字典函数，

>>>a=[1,2,4,2,4,5,6,5,7,8,9,0]

>>> b={}

>>>b=b.fromkeys(a)

>>>c=list(b.keys())

109、Python里面match()和search()的区别？
答：re模块中match(pattern,string[,flags]),检查string的开头是否与pattern匹配。

re模块中research(pattern,string[,flags]),在string搜索pattern的第一个匹配值。

>>>print(re.match(‘super’, ‘superstition’).span())

(0, 5)

>>>print(re.match(‘super’, ‘insuperable’))

None

>>>print(re.search(‘super’, ‘superstition’).span())

(0, 5)

>>>print(re.search(‘super’, ‘insuperable’).span())

(2, 7)

110、有没有一个工具可以帮助查找python的bug和进行静态的代码分析？

答：PyChecker是一个python代码的静态分析工具，它可以帮助查找python代码的bug, 会对代码的复杂度和格式提出警告

Pylint是另外一个工具可以进行codingstandard检查

## 111、单引号，双引号，三引号的区别

答：单引号和双引号是等效的，如果要换行，需要符号(\),三引号则可以直接换行，并且可以包含注释

如果要表示Let’s go 这个字符串

单引号：s4 = ‘Let\’s go’

双引号：s5 = “Let’s go”

s6 = ‘I realy like“python”!’

112、如何用Python来发送邮件？

 可以使用smtplib标准库。

以下代码可以在支持SMTP监听器的服务器上执行。

```python
import sys, smtplib
fromaddr =raw_input(“From: “)
toaddrs = raw_input(“To: “).split(‘,’)
print "Enter message, end with ^D:"
msg = ''
while 1:
	line = sys.stdin.readline()
	if not line:
		break
	msg = msg + line
```

发送邮件部分

```python
server = smtplib.SMTP(‘localhost’)
server.sendmail(fromaddr, toaddrs, msg)
server.quit()
```

113、python代码得到列表list的交集与差集
交集

```python
b1=[1,2,3]

b2=[2,3,4]

b3 = [val for val in b1if val in b2]
print b3
```

差集

```python
b1=[1,2,3]
b2=[2,3,4]
b3 = [val for val in b1 if val not in b2]
print b3
```

##  114、写一个简单的python socket编程

python 编写server的步骤：

1第一步是创建socket对象。调用socket构造函数。如：

socket = socket.socket(family, type )

family参数代表地址家族，可为AF_INET或AF_UNIX。AF_INET家族包括Internet地址，AF_UNIX家族用于同一台机器上的进程间通信。

type参数代表套接字类型，可为SOCK_STREAM(流套接字)和SOCK_DGRAM(数据报套接字)。

2.第二步是将socket绑定到指定地址。这是通过socket对象的bind方法来实现的：

socket.bind( address )由AF_INET所创建的套接字，address地址必须是一个双元素元组，格式是(host,port)。host代表主机，port代表端口号。如果端口号正在使用、主机名不正确或端口已被保留，bind方法将引发socket.error异常。

3.第三步是使用socket套接字的listen方法接收连接请求。

socket.listen( backlog )

backlog指定最多允许多少个客户连接到服务器。它的值至少为1。收到连接请求后，这些请求需要排队，如果队列满，就拒绝请求。

4.第四步是服务器套接字通过socket的accept方法等待客户请求一个连接。

connection, address =socket.accept()

调用accept方法时，socket会时入“waiting”状态。客户请求连接时，方法建立连接并返回服务器。accept方法返回一个含有两个元素的元组(connection,address)。第一个元素connection是新的socket对象，服务器必须通过它与客户通信；第二个元素address是客户的Internet地址。

5. 第五步是处理阶段，服务器和客户端通过send和recv方法通信(传输数据)。服务器调用send，并采用字符串形式向客户发送信息。send方法返回已发送的字符个数。服务器使用recv方法从客户接收信息。调用recv 时，服务器必须指定一个整数，它对应于可通过本次方法调用来接收的最大数据量。recv方法在接收数据时会进入“blocked”状态，最后返回一个字符串，用它表示收到的数据。如果发送的数据量超过了recv所允许的，数据会被截短。多余的数据将缓冲于接收端。以后调用recv时，多余的数据会从缓冲区删除(以及自上次调用recv以来，客户可能发送的其它任何数据)。

6. 传输结束，服务器调用socket的close方法关闭连接。

python编写client的步骤：

1. 创建一个socket以连接服务器：socket= socket.socket( family, type )

2.使用socket的connect方法连接服务器。对于AF_INET家族,连接格式如下：

socket.connect((host,port) )

host代表服务器主机名或IP，port代表服务器进程所绑定的端口号。如连接成功，客户就可通过套接字与服务器通信，如果连接失败，会引发socket.error异常。

3. 处理阶段，客户和服务器将通过send方法和recv方法通信。
4. 传输结束，客户通过调用socket的close方法关闭连接。

```python
import socket
sock=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
sock.bind(('localhost',8001))
sock.listen(5)
while True:
    connection,address =sock.accept()
    try:
        connection.settimeout(5)
        buf =connection.recv(1024)
        if buf == '1':
            connection.send('welcometo server!')
        else:
            connection.send('pleasego out!')
    except socket.timeout:
        print ('time out')
	connection.close()
```

```python
import socket
sock =socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.connect(('localhost',8001))
import time
time.sleep(2)
sock.send('1')
print sock.recv(1024)
sock.close()
```

115、如何用Python删除一个文件？
使用os.remove(filename)或者os.unlink(filename);

## 找出字符串中第一个不重复的字符

```python
def first_char(str):
    dic = {}
    for i in range(len(str)):
        #累计字符的出现次数
        if str[i] in dic:
            dic[str[i]] += 1
        #只出现一次，key对应的value就记1次
        else:
            dic[str[i]] = 1
    for i in range(len(str)):
        if dic[str[i]] == 1:
            return str[i], i+1
#方法二：
from collections import Counter
def deal_str(s):
    counter = Counter(s)
    singles = [_c for _c, _i in counter.items() if _i == 1]
    idxs = [s.index(_c) for _c in singles]
    idx = min(idxs)+1 if idxs else -1
    print(f"字符串（{s}）中的第一个不重复字符的位置是（{idx}）")
if __name__ == '__main__':
    str1 = input('请输入字符:')
    print(first_char(str1))
```

## 字符串转换为数字实例

不使用int()函数的情况下把字符串转换为数字，如把字符串"12345"转换为数字12345。

**方法一：利用str函数**

既然不能用int函数，那我们就反其道而行，用str函数找出每一位字符表示的数字大写。

```python
def atoi(s):
 s = s[::-1]
 num = 0
 for i, v in enumerate(s):
  for j in range(0, 10):
   if v == str(j):
   num += j * (10 ** i)
 return num
```

**方法二：利用ord函数**

利用ord求出每一位字符的ASCII码再减去字符0的ASCII码求出每位表示的数字大写。

```python
def atoi(s):
 s = s[::-1]
 num = 0
 for i, v in enumerate(s):
  offset = ord(v) - ord('0')
  num += offset * (10 ** i)
 return num
```