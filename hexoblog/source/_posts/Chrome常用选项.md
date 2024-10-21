---
title: Chrome常用选项
categories: tools
img: /images/chrome.jpg
tags: chrome
abbrlink: 19662
date: 2024-03-29 22:36:40
---

## Chrome Network 选项

### 1、界面总览

{% asset_img 1.png This is an example image %}

记录请求

> 开启按钮时记录所有请求
>
> 关闭按钮时不会记录

清除请求

> 清除按钮，清除所有记录的请求

过滤请求

> 过滤器，能够自定义筛选条件，也可以选择预定义的过滤方式

自定义条件

+ 通过关键字搜索

预定义的过滤

+ ALL：显示所有请求
+ XHR：显示AJAX异步请求
+ JS：显示js文件
+ CSS：显示css文件
+ Img：显示图片
+ Media：显示媒体文件，音频、视频等
+ Font：显示Web字体
+ Doc：显示html
+ WS:显示websocket请求
+ Other:显示其他请求

保留请求记录

{% asset_img 2.png This is an example image %}

是否进行缓存

> 当打开开发者工具时生效，打开这个开关，则页面资源不会存入缓存，可以从Status栏的状态码看文件请求状态。

弱网设置

> 设置模拟限速，如下图所示

{% asset_img 3.png This is an example image %}

### 2、网络设置

> caching：缓存设置
>
> Network throttling ：弱网设置
>
> User agent：属于http请求头一部分。表示所用浏览器类型及版本、操作系统及版本、浏览器内核、等信息的标识。
>
> Accepted Content-Encodings：服务端压缩格式

{% asset_img 4.png This is an example image %}

右键桌面

{% asset_img 5.png This is an example image %}

Open in new tab:在新的标签中打开链接 

Clear browser cache：清空浏览器缓存 

Clear browser cookies：清空浏览器cookies 

Copy：复制 

+ Copy Link Address：复制资源url到系统剪贴板
  Copy Response：复制HTTP响应
  Copy stack trace：复制堆栈信息
  Copy as PowerShell：复制请求PwoerShell代码
  Copy as fetch：复制请求fetch代码
  Copy as Node.js fetch：复制请求Node.js fetch代码
  Copy as cUrl（cmd）：复制请求cUrl 命令代码
  Copy as cUrl（bash）：复制请求cUrl 命令代码
  Copy all as PowerShell：复制所有请求PwoerShell代码
  Copy all as fetch：复制所有请求fetch代码
  Copy all as Node.js fetch：复制所有请求Node.js fetch代码
  Copy all as cUrl（cmd）：复制所有请求cUrl 命令代码
  Copy all as cUrl（bash）：复制所有请求cUrl 命令代码
  Copy All as HAR：复制所有请求HAR文件

Block request URL：拦截当前请求url 

Block request domian：拦截当前域名下所有请求 

Replay XHR:重新请求AJAX 

Sort By：排序请求 

Header Options：显示请求头选项 

Save all as HAR with content：保存所有请求为.har文件

### 3、Requests Table

{% asset_img resize.png This is an example image %}

Name 资源名称，点击名称可以查看资源的详情情况，包括Headers、Preview、Response、Cookies、Timing
Status HTTP状态码
Type 请求的资源MIME类型
Initiator 标记请求是由哪个对象或进程发起的(请求源)

+ Parser:请求由Chrome的HTML解析器时发起的。
+ Redirect:请求是由HTTP页面重定向发起的，
+ Script:请求是由Script脚本发起的。
+ Other:请求是由其他进程发起的，比如用户点击一个链接跳转到另一个页面或者在地址栏输入URL地址

Size 从服务器下载的文件和请求的资源大小。如果是从缓存中取得的资源则该列会显示(from cache)Time 请求或下载的时间，从发起Request到获取到Response所用的总时间

Time 请求或下载的时间，从发起Request到获取到Response所用的总时间。

Timeline 显示所有网络请求的可视化瀑布流(时间状态轴)，点击时间轴，可以查看该请求的详细信息，点击歹头则可以根据指定的字段可以排序。

**捕获屏幕**
Controls窗格包括的功能有网络日志录制、日志清理、捕获屏幕、过滤器，视图切换、保留日志开关、Cache开关、网络连接开关、网速阀值。

以捕获屏幕为例，点击摄像机按钮（捕获屏幕），重新加载页面即可捕获屏幕。

双击其中的截屏可以放大显示，在放大的图下方可以点击跳转到上一帧或者下一帧。

单击则可以查看该帧被捕获时的网络请求信息，并且在Overview上会有一条黄色竖线以标记该帧被捕获的具体时间点

{% asset_img resize1.png This is an example image %}

**查看DOMContentLoaded和load事件信息**
DOMContentLoaded和load这两个事件会高亮显示。

DOMContentLoaded事件会在页面上DOM完全加载并解析完毕之后触发，不会等待CSS、图片、子框架加载完成。
load事件会在页面上所有DOM、CSS、JS、图片完全加载完毕之后触发。

DOMContentLoaded事件在Overview上用一条蓝色竖线标记，并且在Summary以蓝色文字显示确切的时间。

load事件同样会在Overview和Requests Table上用一条红色竖线标记，在Summary也会以红色文字显示确切的时间

{% asset_img resize2.png This is an example image %}

#### **查看具体资源的详情**

通过点击某个资源的Name可以查看该资源的详细信息，根据选择的资源类型显示的信息也不太一样，可能包括如下Tab信息：

Headers 该资源的HTTP头信息。
Preview 根据你所选择的资源类型（JSON、图片、文本）显示相应的预览。
Response 显示HTTP的Response信息。
Cookies 显示资源HTTP的Request和Response过程中的Cookies信息。
Timing 显示资源在整个请求生命周期过程中各部分花费的时间

针对上面4个Tab进行详细功能：
① 查看资源HTTP头信息

在Headers标签里面可以看到HTTP Request URL、HTTP Method、Status Code、Remote Address等基本信息和详细的Response Headers
、Request Headers以及Query String Parameters或者Form Data等信息

{% asset_img resize3.png This is an example image %}

② 查看资源预览信息

在Preview标签里面可根据选择的资源类型（JSON、图片、文本、JS、CSS）显示相应的预览信息。下图显示的是当选择的资源是JSON格式时的预览信息


③ 查看资源HTTP的Response信息

在Response标签里面可根据选择的资源类型（JSON、图片、文本、JS、CSS）显示相应资源的Response响应内容。下图显示的是当选择的资源是CSS格式时的响应内容。

④ 查看资源Cookies信息

如果选择的资源在Request和Response过程中存在Cookies信息，则Cookies标签会自动显示出来，在里面可以查看所有的Cookies信息。


⑤ 分析资源在请求的生命周期内各部分时间花费信息

在Timing标签中可以显示资源在整个请求生命周期过程中各部分时间花费信息，可能会涉及：

Queuing 排队的时间花费。可能由于该请求被渲染引擎认为是优先级比较低的资源（图片）、服务器不可用、超过浏览器的并发请求的最大连接数（Chrome的最大并发连接数为6）.
Stalled 从HTTP连接建立到请求能够被发出送出去(真正传输数据)之间的时间花费。包含用于处理代理的时间，如果有已经建立好的连接，这个时间还包括等待已建立连接被复用的时间。
Proxy Negotiation 与代理服务器连接的时间花费。
DNS Lookup 执行DNS查询的时间。网页上每一个新的域名都要经过一个DNS查询。第二次访问浏览器有缓存的话，则这个时间为0。
Initial Connection / Connecting 建立连接的时间花费，包含了TCP握手及重试时间。
SSL 完成SSL握手的时间花费。
Request sent 发起请求的时间。
Waiting (Time to first byte (TTFB)) 是最初的网络请求被发起到从服务器接收到第一个字节这段时间，它包含了TCP连接时间，发送HTTP请求时间和获得响应消息第一个字节的时间。
Content Download 获取Response响应数据的时间花费

{% asset_img resize4.png This is an example image %}



TTFB这个部分的时间花费如果超过200ms，则应该考虑对网络进行性能优化了

**查看资源发送者（请求源）和依赖项**

通过按住Shift并且把光标移到资源名称上，可以查看该资源是由哪个对象或进程发起的（请求源）和对该资源的请求过程中引发了哪些资源（依赖资源）

{% asset_img resize5.png This is an example image %}

在该资源的上方第一个标记为绿色的资源就是该资源的发起者（请求源），有可能会有第二个标记为绿色的资源是该资源的发起者的发起者，以此类推

{% asset_img resize6.png This is an example image %}

## Elements面板

双击DOM树视图里面的节点，可以实时编辑标签属性，修改的效果会立刻反应在浏览器里面

点击右侧Style面板，可以实时修改CSS的属性值，这里面的所有样式Name和Value都是可以编辑的，在每个属性后面单击可以添加新的样

点击右侧Computed面板，可以编辑左侧选中的盒子模型参数，所有的值都是可以修改的:点击不同的位置(top、bottom、left、right)就可以修改元素的padding、border、margin属性值。

查看网页的本地修改历史

点击Styles面板中修改过属性的文件名，会跳转到Source面板

在文件位置右击选择Local modifications,可以查看本地的所有修改记录

点击指定的时间点可以看到粉红背景的删除内容和绿色背景的添加内容

## Sources面板

调试JS代码
点击JS代码块前面的数字外来设置断点，如果当前代码是经过压缩的话，可以点击下方的花括号{}来增强可读性，所有的断点都会列出右侧的断点区

{% asset_img format.png This is an example image %}



## 技巧之console

### console.log([data][, ...args])

在控制台打印自定义字符串 第一个参数是一个字符串，包含零个或多个占位符。 每个占位符会被对应参数转换后的值所替换，支持printf风格输出 只支持字符（%s）、整数（%d或%i）、浮点数（%f）和对象（%o）四种 支持格式化输出

console.log 第一个参数如果有%c，会把%c后面的文案按照你提供的样式输出，此时第二个参数应为css属性的字符串

换下一行输出：shift+enter

```shell
console.log(666)

//插值输出
console.log("%s年%d月", 2019, 3) //%s 字符串 %d 整数
console.log("%c我是%c自定义样式", "color:red", "color:blue;font-size:25px")

//格式化输出
console.log("%c3D Text"," text-shadow: 0 1px 0 #ccc,0 2px 0 #c9c9c9,0 3px 0 #bbb,0 4px 0 #b9b9b9,0 5px 0 #aaa,0 6px 1px rgba(0,0,0,.1),0 0 5px rgba(0,0,0,.1),0 1px 3px rgba(0,0,0,.3),0 3px 5px rgba(0,0,0,.2),0 5px 10px rgba(0,0,0,.25),0 10px 10px rgba(0,0,0,.2),0 20px 20px rgba(0,0,0,.15);font-size:5em");
```

### console.error

在控制台打印自定义错误信息

### console.warn

打印自定义警告信息



### console.dir

dir方法用来对一个对象进行检查（inspect），并以易于阅读和打印的格式显示 该方法对于输出DOM对象非常有用，因为会显示DOM对象的所有属性

```shell
console.dir($('.text-light'))
console.dirxml($('.text-light'))
console.dir(document.body)
xonsole.log(document,body)
```

console.dir输出的结果更详细

### console.table

对于某些复合类型的数据，console.table方法可以将其转为表格显示 参数被转为数组的前提条件是必须有主键值，数组的主键是其index，对象的主键是它的最外层键 如果参数不是复合类型的数据，console.table的表现同console.log

```shell
var text1 = [
	{name:"Bob",age:13,hobby:"playing"},
	{name:"Lucy",age:14,hobby:"reading"},
	{name:"Jane",age:11,hobby:"shopping"}
];
console.table(text1);
 
var test2 = {
  csharp: { name: "C#", paradigm: "object-oriented" },
  fsharp: { name: "F#", paradigm: "functional" }
}
console.table(test2)
 
var test3 = 8888
console.table(test3)
 
console.table('%c我是格式化文字', 'font-size:20px;color:red')
```

### console.group

成组输出，分在一组的信息，可以用鼠标折叠/展开 console.group接受一个字符串作为参数，并把字符串作为这组数据的组名输出到控制台 必须配合console.groupEnd使用，console.groupEnd如果接受参数，则会寻找跟它同名的console.group,如果没有参数，则会采用就近原则配对

```shell
var stu = [{name:"Bob",age:13,hobby:"playing"},{name:"Lucy",age:14,hobby:"reading"},{name:"Jane",age:11,hobby:"shopping"}];
console.group('group1')
console.log(stu[0])
console.log(stu[1])
console.log(stu[2])
 
console.group('group2')
console.log(stu[2].name)
console.log(stu[2].age)
console.log(stu[2].hobby)
console.groupEnd()
 
console.groupEnd()
```

### console.count

计数，每次执行到count的位置都输出所在函数的执行次数

```js
function fib(n){ //输出前n个斐波那契数列值
  if(n == 0) return;
  console.count("调用次数");
  //console.trace();//显示函数调用轨迹(访问调用栈）
  var a = arguments[1] || 1;
  var b = arguments[2] || 1;
  console.log("fib=" + a);
  [a, b] = [b, a + b];
  fib(--n, a, b);
}

fib(6);
```

### console.trace

访问调用栈 打印当前执行位置到console.trace()的路径信息

```

```

### console.time

启动一个计时器（timer）来跟踪某一个操作的占用时长 每一个计时器必须拥有唯一的名字，页面中最多能同时运行10,000个计时器。当以此计时器名字为参数调用 console.timeEnd() 时，浏览器将以毫秒为单位，输出对应计时器所经过的时间 该方法在使用时不会将输出的时间返回到js,它只能用于控制台调试

```js
var oldList = new Array(1000);
var oldList1 = new Array(1000);
for(let i = 0;i<1000;i++){
	oldList[i] = Math.floor(Math.random()*1000);
	oldList1[i] = Math.floor(Math.random()*1000);
}

// 冒泡排序
function bubbleSort(arr) {
    var len = arr.length;
    for (var i = 0; i < len; i++) {
        for (var j = 0; j < len - 1 - i; j++) {
            if (arr[j] > arr[j+1]) {        //相邻元素两两对比
                var temp = arr[j+1];        //元素交换
                arr[j+1] = arr[j];
                arr[j] = temp;
            }
        }
    }
    return arr;
}
console.time('bubbleSort');
bubbleSort(oldList);
console.timeEnd('bubbleSort');
console.time('sort');
oldList1.sort();
console.timeEnd('sort');
```

### console.assert

如果断言为false，则将一个错误消息写入控制台。如果断言是true，没有任何反应 在浏览器中当console.assert()方法接受到一个值为假断言（assertion）的时候，会向控制台输出传入的内容，但是并不会中断代码的执行，而在Node.js中一个值为假的断言将会导致一个AssertionError被抛出，使得代码执行被打断 显示样式同console.warn

```js
var testList = [{name:'aa',age:19},{name:'bb',age:25},{name:'cc',age:22}];
testList.forEach((item, index) => {
	console.assert(true , '666');
	console.assert(false, 'false message')
	console.log(index)
})
```

## console定位selector，XPath和CSS

### **Selector**

$(“selector”)

+ 会列出与selector匹配的所有元素，

$$(“selector”)

+ 把这些匹配到的元素组成了数组。（点击返回的每个元素，则会定位到页面中的img元素及html中的具体位置）

document.querySelector()会返回DOM中匹配的第一个元素（只返回一个元素）；
document.querySelectorAll()等同于$$(selector)

### **Xapth**

$x(“your_xpath_selector”)

```shell
<input type="submit" value="百度一下" id="su" class="btn self-btn bg s_btn">

#定位不含id=su的元素
$x("//input[not(@id='su')]")

#定位子孙元素为input的元素
$x("//input")

#定位input标签，且其id属性值为su的元素
$x("//input[@id='su']")
```

### **CSS**

$("css表达式")

```shell
<input type="submit" value="百度一下" id="su" class="btn self-btn bg s_btn">

#根据id属性去定位，按钮的id值为：id="su" 
$("#su")

#根据class属性去定位，按钮的class属性值为class="btn self-btn bg s_btn"，取其中一段即可
$(".btn")
```

## Application面板

主要是记录网站加载的所有资源信息，包括存储数据（Local Storage、Session Storage、IndexedDB、Web SQL、Cookies）、缓存数据、字体、图片、脚本、样式表等

{% asset_img format1.png This is an example image %}

### **Application**

Manifest

用来告诉浏览器如何在用户的桌面上"安装"这个app，及安装后该展现的信息。主要是展现其信息，不具备操作性质

Service Workers

浏览器在后台独立于网页运行的脚本，它们已包括如推送通知和后台同步等功能，支持离线应用，每24小时都会更新本地的离线脚本

Clear storage

清除`Service Worker`、`Storage`、数据库和缓存

### **Local Storage** 

本地存储空间，表示访问页面储存的变量信息，这些数据只有在同一个会话中的页面才能访问并且当会话结束后数据也随之销毁（浏览器关闭）。

- 双击键或值能够修改相应的值。
- 双击空白单元格能够添加新条目。
- 点击对应的条目 ，而后按 `Delete` 按钮能够删除该该条目。
- 点击 refresh 按钮能够查看您的更改

**Session** **Storage** 

会话存储空间，用于本地存储一个会话（session）的数据，session Storage不是一种持久化的本地存储，仅仅是会话级别的存储，与localStorage一样

### **IndexedDB**

持久存储数据，一种在浏览器中持久存储数据的方法，允许我们不考虑网络可用性，创建具有丰富查询能力的可离线web应用程序。indexedDB对于存储大量数据的应用程序和不需要持久internet链接的应用程序（例如邮件客户端、待办事项列或记事本）很有用

IndexedDB 的储存空间比 LocalStorage 大得多，一般来说不少于 250MB，甚至没有上限

## **Cookies**

浏览器缓存，一种保存在电脑上的一种文件，当我们使用电脑进行浏览网页的时候，服务器就会生成一个证书，并且返回给我们的电脑，这个证书就是cookie，一般情况下，cookie是服务器写入客户端的文件，我们也可以叫浏览器缓存

Name：是Cookie的名称，Cookie一旦创建，名称便不可更改，一般名称不区分大小写。

Value：是cookie名称Name对应的Cookie的值，如果值为[Unicode](https://so.csdn.net/so/search?q=Unicode&spm=1001.2101.3001.7020)字符，需要为字符编码。如果值为二进制数据，则需要使用BASE64编码。	


domain: 指定Cookie的有效域, 决定在向该域发送请求时是否携带此Cookie

expires / max-age: 相对/绝对过期时间，由访问的apache服务器设置，A—相对；M—绝对；

httponly: 通过js脚本将无法读取到cookie信息，无法用document.cookie打出cookie的内容。这样能有效的防止XSS（cross siteScript—跨站脚本）攻击，窃取cookie内容，这样就增加了cookie的安全性；

secure: 这个cookie只能用https和ssl等安全协议发送给服务器，在不安全的http协议中不传输此cookie;

samesite: 用来限制第三方Cookie。

+ Strict—完全禁止第三方Cookie
   Lax—导航到目标网址的 Get 请求发送Cookie
   None: 显式关闭SameSite属性

priority: chrome的提案，定义了三种优先级，Low/Medium/High，当cookie数量超出时，低优先级的cookie会被优先清除

## Trust Tokens

信任令牌，为了在不需要明确知道用户的情况下验证用户身份，相比Cookie，trust tokens不能跨网站追踪用户，可以让网站向广告商证明访问的是真正的用户，而不是机器人

## Cache

### Cache Storage - 缓存空间

用来存储Response（请求）对象，作用于需要使用请求缓存的方法

### Back-forward Cache - 往返缓存

是浏览器为了在用户页面间执行前进后退操作时拥有更加流畅体验的一种策略。浏览器原本就允许用户按下Back或Forward来回到上一个访问的网站，或是移至下一个网站，但Back-Forward Cache是个内存缓存功能，于内存中存储了完整的页面快照，以让用户在切换前后所访问的网页时更快速，换句话说就是back和fordword操作读取浏览器缓存，而不请求服务器

## Profiles的使用

性能

Record JavaScript CPU Profile 用于分析网页上的JavaScript函数在执行过程中的CPU消耗信息。
Take Heap Snapshot 创建堆快照用来显示网页上的JS对象和相关的DOM节点的内存分布情况。
Record Allocation Timeline 从整个Heap角度记录内存的分配信息的时间轴信息，利用这个可以实现隔离内存泄漏问题。

Record Allocation Profile 从JS函数角度记录内存的分配信息

有三种不同的视图可供选择：

+ Chart 按时间先后顺序显示的火焰图

+ Heavy(Bottom Up) （自底向上）根据对性能的消耗影响列出所有的函数，并可以查看该函数的调用路径
+ Tree(Top Down) (自顶向下) 从调用栈的顶端（最初调用的位置）开始，显示调用结构的总体的树状图情况

## Background Services

后台服务

使用方式： 点击启动录制事件按钮，开始录制。
保留时间： 3天
属性说明：

Timestamp	Event	Origin	Service Worker Scope	Instance Id
时间戳	事件	来源	服务作用范围	实例名
 Origin：对应domain
 Service Worker Scope：对应path

Background Fetch - 后台提取
容许开发人员离开当前的页面，进行加载和操做大文件或文件列表。这样能够加大上传和下载的成功率，并容许sevice worker 缓存结果。Background Fetch在后台进行，在你点击录制按钮后，即便关闭标签，甚至关闭浏览器，这两类事件仍然可以被录制到。

点击 Background Fetch 或 Background Sync 选项，然后点击 Record 开始记录

Background Sync- 后台同步
Background Sync 可延迟用户行为，直到用户网络链接稳定。经常使用语聊天消息，电子邮件，文档更新等功能中。提高性能和体验。

Notifications - 通知
Notifications是在Web页面以外，以弹出桌面对话框的形式通知用户发生了某事件。一旦开始录制，页面上的推送消息也会保留3天。

Payment Handler - 付款处理程序
触发付款处理事件后，点击事件行以了解有关该事件的更多信息。如果 Payment Handler 事件发生在其他域，可以启用 Show events from other domains 选项。

Periodic Background Sync - 定期后台同步
跟后台同步类似，额外增加一个定时任务

Push Messaging - 推送消息
Push Messaging使应用或扩展程序可以接收经过 Google 云消息服务发送的消息数据。

## Frames

该面板显示了该网站所有内容资源

**top**是一个主文档，在top下面是主文档的*Fonts*、*Images*、*Scripts*、*Stylesheets*等资源。最后一个就是主文件自身

在资源上右击后在弹出菜单选择**Reveal in Network Panel**，就会跳转到**Network**面板并定位到该资源的位置

{% asset_img format2.png This is an example image %}

## Security面板

通过该面板你可以去调试当前网页的安全和认证等问题并确保您已经在你的网站上正确地实现HTTPS

HTTPS和HTTP的区别主要为以下四点：
 ① https协议需要到CA申请证书，一般免费证书很少，需要交费。
 ② http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。
 ③ http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
 ④ http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全

如果网页是安全的，则会显示这样一条消息：*This page is secure (valid HTTPS)*

通过点击**View certificate**可以查看*main origin*的服务器证书信息。
点击左侧可以查看指定源的连接和证书详情

该面板可以区分两种类型的不安全的页面：

- 如果被请求的页面通过HTTP提供服务，那么这个主源就会被标记为不安全。
- 如果被请求的页面是通过HTTPS获取的，但这个页面接着通过HTTP继续从其他来源检索内容，那么这个页面仍然被标记为不安全。这就是所谓的**混合内容**页面,混合内容页面只是部分受到保护,因为HTTP内容(非加密的内容)可以被嗅探者入侵,容易受到**中间人攻击**

> **中间人攻击**(Man-in-the-Middle Attack,"MITM攻击")是一种“间接”的入侵攻击，这种攻击模式是通过各种技术手段将受入侵者控制的一台计算机虚拟放置在网络连接中的两台通信计算机之间，这台计算机就称为“中间人”。

## Audits面板

对当前网页进行网络利用情况、网页性能方面的诊断，并给出一些优化建议。比如列出所有没有用到的CSS文件等

{% asset_img format3.png This is an example image %}

选中**Network Utilization**、**Web Page Performance**，点击**Run**按钮，将会对当前页面进行网络利用率和页面的性能优化作出诊断，并给出相应的优化建议。

如果页面很多，文件也很多，静态资源也很多，那么得一个一个去下载么？

谷歌的Save All Resources插件，一键下载Sources源码

参考：

[链接](https://blog.csdn.net/qq_25628891/article/details/121656225)

[链接1](https://www.jianshu.com/p/43ee00808c1d)

[链接2](http://testingpai.com/article/1624970574654)

[链接3](https://www.jianshu.com/p/dd5bc5460636)

[链接4](https://blog.51cto.com/u_12890843/5347153)