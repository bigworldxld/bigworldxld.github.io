---
title: BrupSuite使用
categories: tools
img: /images/BrupSuite.jpg
tags:
  - BrupSuite
  - http
abbrlink: 38446
date: 2024-07-06 14:33:59
---

## Brup_Suite介绍

​Burp Suite 是用于攻击web 应用程序的集成平台。它包含了许多Burp工具，这些不同的burp工具通过协同工作，有效的分享信息，支持以某种工具中的信息为基础供另一种工具使用的方式发起攻击。它主要用来做安全性渗透测试，可以实现拦截请求、Burp Spider爬虫、漏洞扫描（付费）等类似Fiddler和Postman但比其更强大的功能

## 下载安装Brup_Suite

官网下载：(Burp Suite Release Notes)[https://portswigger.net/burp/releases]
浏览器设置，以firefox为例，进入firefox应用商店搜索switchy，下载如下插件
进入http://burp点击右上角下载证书，在浏览器设置中搜索证书，查看证书-证书颁发机构-导入，将其导入。
导入之后，会提示有是否信任的选项。全部选上，导入

## http请求包格式

请求方法 请求资源（目录结构/目录文件/传参的参数[GET]） HTTP版本

host: 主机名

User-Agent：客户端基本环境信息

Content-Type：传参的类型

Content-Length: 请求包长度

Referer: 上一步来源。

X-Forwarded-For：当前身份ip

Cookie：用户身份标识

## http应答包格式

HTTP版本 返回状态值 服务端自定变量

Date：日期

Server：服务端相关信息

X-Powered-By：当前编程语言环境

Content-Length: 返回包长度

当前响应回客户端（浏览器）的前端代码

## http状态值	

200 - 服务器成功返回网页

404 - 请求的网页不存在

503 - 服务器超时

1xx（临时响应）

表示临时响应并需要请求者继续执行操作的状态码。

100（继续）  请求者应当继续提出请求。服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。

101（切换协议）  请求者已要求服务器切换协议，服务器已确认并准备切换。

2xx （成功）表示成功处理了请求的状态码。

200（成功）  服务器已成功处理了请求。通常，这表示服务器提供了请求的网页。如果是对您的 robots.txt 文件显示此状态码，则表示 Googlebot 已成功检索到该文件。

201（已创建）     请求成功并且服务器创建了新的资源。

202（已接受）     服务器已接受请求，但尚未处理。

203（非授权信息）     服务器已成功处理了请求，但返回的信息可能来自另一来源。

204（无内容）     服务器成功处理了请求，但没有返回任何内容。

205（重置内容）  服务器成功处理了请求，但没有返回任何内容。与 204 响应不同，此响应要求请求者重置文档视图（例如，清除表单内容以输入新内容）。

206（部分内容）  服务器成功处理了部分 GET 请求。

3xx （重定向） 要完成请求，需要进一步操作。通常，这些状态码用来重定向。Google 建议您在每次请求中使用重定向不要超过 5 次。您可以使用网站管理员工具查看一下 Googlebot 在抓取重定向网页时是否遇到问题。诊断下的网络抓取页列出了由于重定向错误导致 Googlebot 无法抓取的网址。

300（多种选择）  针对请求，服务器可执行多种操作。服务器可根据请求者 (user agent) 选择一项操作，或提供操作列表供请求者选择。

301（永久移动）  请求的网页已永久移动到新位置。服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。您应使用此代码告诉 Googlebot 某个网页或网站已永久移动到新位置。

302（临时移动）  服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来响应以后的请求。此代码与响应 GET 和 HEAD 请求的 301 代码类似，会自动将请求者转到不同的位置，但您不应使用此代码来告诉 Googlebot 某个网页或网站已经移动，因为 Googlebot 会继续抓取原有位置并编制索引。

303（查看其他位置）  请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。对于除 HEAD 之外的所有请求，服务器会自动转到其他位置。

304（未修改）     自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。如果网页自请求者上次请求后再也没有更改过，您应将服务器配置为返回此响应（称为 If-Modified-Since HTTP 标头）。服务器可以告诉 Googlebot 自从上次抓取后网页没有变更，进而节省带宽和开销。

305（使用代理）  请求者只能使用代理访问请求的网页。如果服务器返回此响应，还表示请求者应使用代理。

307（临时重定向）     服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来响应以后的请求。此代码与响应 GET 和 HEAD 请求的 <a href=answer.py?answer=>301</a> 代码类似，会自动将请求者转到不同的位置，但您不应使用此代码来告诉 Googlebot 某个页面或网站已经移动，因为 Googlebot 会继续抓取原有位置并编制索引。

4xx（请求错误） 这些状态码表示请求可能出错，妨碍了服务器的处理。

400（错误请求）  服务器不理解请求的语法。

401（未授权）     请求要求身份验证。对于登录后请求的网页，服务器可能返回此响应。

403（禁止）  服务器拒绝请求。如果您在 Googlebot 尝试抓取您网站上的有效网页时看到此状态码（您可以在 Google 网站管理员工具诊断下的网络抓取页面上看到此信息），可能是您的服务器或主机拒绝了 Googlebot 访问。

404（未找到）    
服务器找不到请求的网页。例如，对于服务器上不存在的网页经常会返回此代码。

405（方法禁用）  禁用请求中指定的方法。

406（不接受）     无法使用请求的内容特性响应请求的网页。

407（需要代理授权）  此状态码与 <a href=answer.py?answer=35128>401（未授权）</a>类似，但指定请求者应当授权使用代理。如果服务器返回此响应，还表示请求者应当使用代理。

408（请求超时）  服务器等候请求时发生超时。

409（冲突）  服务器在完成请求时发生冲突。服务器必须在响应中包含有关冲突的信息。服务器在响应与前一个请求相冲突的 PUT 请求时可能会返回此代码，以及两个请求的差异列表。

410（已删除）     如果请求的资源已永久删除，服务器就会返回此响应。该代码与 404（未找到）代码类似，但在资源以前存在而现在不存在的情况下，有时会用来替代 404 代码。如果资源已永久移动，您应使用 301 指定资源的新位置。

411（需要有效长度）  服务器不接受不含有效内容长度标头字段的请求。

412（未满足前提条件）     服务器未满足请求者在请求中设置的其中一个前提条件。

413（请求实体过大）  服务器无法处理请求，因为请求实体过大，超出服务器的处理能力。

414（请求的 URI 过长）   请求的 URI（通常为网址）过长，服务器无法处理。

415（不支持的媒体类型）  请求的格式不受请求页面的支持。

416（请求范围不符合要求）     如果页面无法提供请求的范围，则服务器会返回此状态码。

417（未满足期望值）  服务器未满足"期望"请求标头字段的要求。

5xx（服务器错误）这些状态码表示服务器在处理请求时发生内部错误。这些错误可能是服务器本身的错误，而不是请求出错。

500（服务器内部错误）     服务器遇到错误，无法完成请求。

501（尚未实施）  服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码。

502（错误网关）  服务器作为网关或代理，从上游服务器收到无效响应。

503（服务不可用）     服务器目前无法使用（由于超载或停机维护）。通常，这只是暂时状态。

504（网关超时）  服务器作为网关或代理，但是没有及时从上游服务器收到请求。

505（HTTP 版本不受支持）      服务器不支持请求中所用的 HTTP 协议版本。

## burp功能详解

   ➢Dashboard仪表盘:仪表盘，扫描启动、暂停，用于显示任务、日志信息等

   ➢Target目标:设置工作的目标范围(URL)，以及报文过滤、报文展示等功能

   ➢Proxy代理:拦截HTTP/s请求的代理服务器，作为web浏览器与服务器的中间人，允许拦截、修改数据流

   ➢测试器:Intruder入侵功能，对web应用程序进行攻击，还可以漏洞利用、Web应用程序模糊测试、暴力破解等。

   ➢Repeater重发器:通过手动来触发单词HTTP请求，并分析应用程序的响应包

   ➢定序器:Sequencer会话模块，用于分析那些不可预知的应用程序会话令牌和重要数据的随机性的工具。

   ➢Decoder解码器:是一个进行手动执行或对应用程序数据者智能解码编码的工具.

   ➢Comparer对比器:对比模块，对数据进行差异化分析

   ➢Extender插件扩展:可以加载BP拓展模块和第三方代码

   ➢Project options,User options设置模块:可以设置项目、用户等信息  

### Target
Target模块组成和功能
1.Site map;站点地图，以站点(而不是某次请求)来组织burp获取的数据
2.Scope:目标，可以限制抓取内容，排除无关流量的干扰。
设置HTTP history中包含或者去除的流量
也可以在Site map中右击站点，将站点添加到目标域
3.lssue definitions:漏洞列表，列出burp可以扫描到的漏洞详情

### Proxy
Proxy代理模块作为BurpSuite的核心功能，是拦截HTTP(HTTPS)的代理服务器，作为一个在浏览器和目标应用程序之间的中间人，允许你拦截，查看，修改在两个方向上的原始数据流(请求和响应)。
intercept:
1.Forward:用于发送数据。当把所需要的HTTP请求编辑完成
后，手动发送数据。
2.Drop:将该请求包丢弃。
3.Intercept is off/on:拦截开关。当处于off状态下时,会自动转发所拦截的所有请求;当处于on状态下时，会将所有拦截所有符合规则的请求并将它显示出来等待编辑或其他操作。
4.Action:将请求发送到其它模块进行交互。

### intruder标签
用于自动对Web应用程序自定义的攻击。它可以用来自动执行测试过程中可能出现的所有类型的任务。例如目录爆破，注入，密码爆破(弱口令爆破)等
攻击类型
狙击手:最多可以一次性测试一个参数（1：A）

攻城锤：支持多个payload,只能使用同一个密码本（1：A；1：B）

音叉攻击：支持多个payload，每个payload独立密码本，多组密码一一对应（1：A；2：B）

集束炸弹：支持多个payload，每个payload独立密码本，两组密码排列组合，分别测试（1：A；1：B；2：A；2：C）

Payload
配置Positions设置的标记位的字典。
option
 Request Headers :
+ UPdate content-Length header是必选的，因为payload的长度是变化的，所以勾选上这个选项更新或添加每一次请求的content-Length请求头，否则就是错误的请求。
+ Set Connection:close为请求更新或添加请求头Connection值为close，这是为了避免和服务器进行长连接，在得到一次响应后就自动关闭连接，因为还有好多版本的请求等着呢，这样可以加快攻击速度。
burp中的重发器Repeater：重放，在这个模块里可以修改请求包再重新请求。通过重发器测试文件上传绕过

测试过程
proxy抓包->Intercept is on->右键发送到Repeater->使用验证码插件->右键发送到Intruder->positions选择攻击模式->修改url参数位置->选择payload set参数和Payload type->上传密码本->点击开始攻击->Results下看length不同之处

针对basic认证的登录爆破
1）抓包获取加密后的basic
2）定义payload
3）按照字符串分别定义密码本
4）定义编码

1.simple list(简单列表)
在内置字典里有:
Fuzzing-quick(模糊-快速)、Fuzzing-full(模糊-全)
Usernames(用户名)、Passwords(密码)
shortwords(短字符)、a-z(a-z的小写单一字符)、A-Z(A-Z的大写单一字符)、0-9(0-9单一字符)
Directories-short(短日录)、Directories-ong(长日录)
Filenames-short(短文件名)、Filenames-long(长文件名)
Extensions-short(短扩展名)、Extensions-short(长扩展名)
Format strings(字符串格式)、Form field names(表单字段名称)
Form field values(表单字段值)、Fuzzing-SQLinjection(模糊-SQL注入)
Fuzzing-XSS(模糊-XSS)、Fuzzing-path traversal(模糊-路径遍历)
N letter words(N个字母单词)、HTTP verbs(http动词)
CGl scripts(CGl脚本)、llS files and directories(lls文件和目录)、user agents(用户代理)

分类：游戏相关时间：2023-11-30 22:08浏览：10365
概述burpsuite汉化专业版破解(无cmd框版)注册机+密钥+JDK 附带详细安装教程内置jre环境，无需安装java即可使用！！！Burpsuite汉化专业版破解(无cmd框版) 图文安装教程：一、首次使用需要先使用英文版激活安装，首先打开【ENBurp(无CMD窗口).VBS】后，接着继续一直 下一步二、到了此处先打开【ENBurp(无CMD窗口).VBS】3、找到并且打开【BurpSuite】文件夹，右键打开【burpsuitlo内容
burpsuite汉化专业版破解(无cmd框版)注册机+密钥+JDK 附带详细安装教程


内置jre环境，无需安装java即可使用！！！

## 破解burp
**window**
Burpsuite汉化专业版破解(无cmd框版) 图文安装教程：
一、首次使用需要先使用英文版激活安装，首先打开【ENBurp(无CMD窗口).VBS】后，接着继续一直 下一步
二、到了此处先打开【ENBurp(无CMD窗口).VBS】
3、找到并且打开【BurpSuite】文件夹，右键打开【burpsuitloader.jar】文件，打开方式选择【OpenJDK Platform binary】
4、复制注册机里面的【License】提供的代码
5、粘贴到【ENBurp(无CMD窗口).VBS】的License里面后，点击【Next】进入下一步。
6、注意接着，到这里点击【Manual activation】
7、复制【Manual activation】里面的【response】代码
8、粘贴到注册机里面的【response】处，将生成的密钥粘贴到【Manual activation】处后，继续点击【Next】
10、点击【Finish】，到此安装burpsuite已经结束了
11、在弹出来的burpsuite软件，可以直接关闭掉了
12、这个时候就可以打开汉化版的了，找到【CNBurp(无CMD窗口).VBS】打开，即可跳到汉化版的程序了
 
下载地址： 
[Burpsuite汉化专业版破解(无cmd框版)V2023.10.34](https://pan.baidu.com/s/12BQAhaqdf8sVqlcF9QSrGQ?pwd=z1cy)
**mac**
一.安装burp
访问官网：
https://portswigger.net/burp/releases
点击下载最新版本(自动为mac的cpu类型是arm还是x86,默认就好)
二.打开burp（一定要打开一下再退出）
双击burp图标打开应用程序，出现以下界面，点击打开
出现以下界面command+q立即退出burp
三.下载破解
[破解:ku69](https://www.123pan.com/s/4v5A-dt0cv.html)
在应用程序文件夹找到burp，右击显示包的内容
找到并编辑vmoptions.txt
```txt
-XX:MaxRAMPercentage=50
-include-options user.vmoptions
-javaagent:BurpLoaderKeygen.jar
-javaagent:BurpSuiteChs.jar
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
--add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED
--add-opens=java.base/jdk.internal.org.objectweb.asm.Opcodes=ALL-UNNAMED
-noverify
```
进入app路径
/Applications/Burp Suite Professional.app/Contents/Resources/app
 将破解工具中的BurpLoaderKeygen.jar和BurpSuiteChs.jar放入app文件夹中
 注:
-javaagent:BurpSuiteChs.jar
该列为启用中文包，注释可取消中文
四.再次打开burp
去到burp破解包下的jdk-21.0.4.jdk/contents/home/bin/目录
终端输入
java -jar burp破解包下的BurpLoaderKeygen.jar &
最后，复制license即可，在help中查看时间

[最新mac版burp破解教程（亲测有效）](https://www.cnblogs.com/l1l1l1/p/17947167)
[mac的burp的手动破解安装](https://www.freebuf.com/articles/web/407303.html)



##  弱口令字典

https:/github.com/rootphantomer/Blasting_dictionary
https:/github.com/k8gege/Passwordbic
https://github.com/danielmiessler/secLists
https://192-168-1-1ip.mobi/defau1t-router-passwords-7ist/
https://github.com/danielmiessler/secLists/blob/master/Passwords/pefau1t-Credentials/default-
passwords.csv
https://github.com/Dormidera/wordList-compendium


参考链接：https://blog.csdn.net/qq_59344199/article/details/128022680
