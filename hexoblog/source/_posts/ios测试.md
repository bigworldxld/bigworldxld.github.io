---
title: ios测试
categories: test
img: /images/IOS测试.jpg
tags:
  - xcode
  - sream
  - ios
abbrlink: 28235
date: 2024-05-03 14:30:49
---

## ios测试

### 自动化

#### 一.环境搭建

应用场景
想要进行ios自动化测试，必须进行环境的搭建
**需要的环境**
**macOS** 系统电脑
10.13.6
**Xcode**	10.1
打开 app store 搜索 xcode 并下载

**待测试的ios项目**
。找开发人员
**appium** **Desktop**
1.10
https://github.com/appium/appium-desktop/releases/download/v1.10.0/Appium-1.10.0.dmg

**python**
3.6.1
https://wwwpython.org/ftp/python/3.6.1/python-3.6.1-macosx10.6.pkgpycharm
Professional 2018.2.2
ohttps://www.jetbrains.com/pycharm/download
**node.js**
 10.5.0

 https://npm.taobao.org/mirrors/node/v10.15.0/node-v10.15.0.pkg

**cnpm**

 5.2.0

npm install -g cnpm --registry=https://registry.npm.taobao .org

```
npm instal1 -g cnpm --registry=https://registry.npm.taobao.org
```

**ios**-**deploy** 依赖库
cnpm install -g ios-deploy
brew
 1.9.1

```bash
ruby <(cur1 -fsskL raw.github.com/mxc1/homebrew/go)
```

**libimobiledevice** 依赖库
brew install --HEAD libimobiledevice

可能出现的问题
报错: Requested busbmuxd >= 1.1.0' but version of libusbmuxd is 1.0.10
解决方法:

```bash
brew update
brew uninstall --ignore-dependencies ibimobiledevicebrew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
```

**carthage****依赖库**
brew install carthage
**ios 系统手机**
 iOS 12.1.2
**Apple ID**
https://appleid.apple.com/account#!&page=create
**WebDriverAgent**
https://github.com/facebook/WebDriverAgent

+ 点击 download zip
+ 配置WebDriverAgent 的内容(真机)

**adb**

adb 安装(brew cask install android-platform-tools)



#### 二.使用模拟器进行自动化测试

**2.1**运行iOS程序到模拟器
步骤
1.选择将要运行的 程序 和 模拟器设备
2.快捷键 command +r 运行
**2.2**查看iOS元素特征

type->find_element_by_class_name

name->find_element_bu_text

accessibility_by_id->find_element_by_id

滑动：driver.swipe(100,2000,100,1000)

driver.execute_script("mobile: scroll",{"direction": "down"})	向下滑动2次

driver.execute_script("mobile: swipe",{"direction": "up"})	向下滑动1次

**注意点**
1.官方已经明确表示 swipe 等 API 在模拟器7.x-8.x 中无法使用，真机并未提及,网上有人说真机并没有这个问题，但并没有表明版本。真机iOS 12.1.12 实测不行

本人曾经使用过 Appium 1.8 +模拟器和真机的ioS11 是可以使用的。工作中，如果碰到这种情况，就用 execute_script 的方法实现即可
在模拟器中可以连接电脑的键盘进行输入，但是如果连接，appium 将无法工作，所以需要消选Connect Hardware Keyboard

选中模拟器->左上角 Hardware - Keyboard->消选 Connect Hardware Keyboard

步骤
1.打开appium
2启动appium
3.左上角菜单栏选择appium-new session window

**2.3**编写和运行自动化脚本
**前置代码**

```python
from appium import webdriver
desired_caps =dict()
desired_caps['platformName']=ios
desired_caps['platformversion']=12.1
desired_caps['deviceName'] =iphone 8
desired_caps['app'] = "com.itcast.HMiosTest"
driver =webdriver.Remote('http://localhost:4723/wd/hub'，desired_caps)
```

app包名在项目根目录：HMiOSTest(项目名)->General->TARGETS下的HMiOSTest->Bandle Identifier对应的值
BundleID 概念：一个应用的唯一标识
开发证书， 开发证书用于真机调试
发布证书， 发布证书用于提交到 appStore
如果想用 Xcode 运行 App 到真机上，要满足几个条件：
1：成为 Apple Developer 计划的成员
2：需要设置认证书和应用 ID（开发证书和发布证书）

Xcode模拟器命令
操作模拟器命令：xcrun simctl
操作真机命令：idevice<xxx>
# 查看已安装的模拟器
xcrun simctl list devices
 
# 查看已经开机的模拟器
xcrun simctl list devices |grep Booted
 
# 查看已连接的真机设备的 udid 信息
idevice_id -l


# 查看 模拟器列表
xcrun simctl list devices
# 启动模拟器
xcrun simctl boot 模拟器id


# 模拟器安装应用
## 单设备
xcrun simctl install booted demo.app
## 多设备
xcrun simctl install <device> demo.app
 
# 真机安装应用
ideviceinstaller --install </path/to/file/xxx.app>
ideviceinstaller -i </path/to/file/xxx.app>


# 模拟器卸载应用
xcrun simctl uninstall <device> <bundleID>
 
# 真机卸载应用
ideviceinstaller --uninstall <appid>
ideviceinstaller -U <appid>

 查看应用的 bundleid
模拟器查看应用 bundleid

找到 app 的安装包
右键点击
显示包内容
找到 Info.plist 文件
双击打开，就可以找到对应的 Bundle identifier 项
真机查看应用的 bundleid : ideviceinstaller -l                        

**需求**
1点击按进入列表
2.获取所有列表的文字内容并输入
3.动一次
4.点击20
5.清空文本框内容6.在文本框中输入“hello"7.单击返回按钮

核心代码

```python
time.sleep(1)
driver.find_element_by_id("进入列表").click()
time.sleep(1)
eles = driver.find_elements_by_class_name("XCUIEementTypeStaticText")for i in eles:
print(i.text)
time.sleep(1)
# 无法使用
# driver.swipe(100，2000，100，1000)
driver.execute_script("mobile: swipe"，{"direction": "up"})
time.sleep(1)
driver.find_element_by_xpath("//*[@name='20]")click()
time.sleep(1)
text_view = driver.find_element_by_class_name("XCUIEementTypeTextField")text_view.clear(
time.sleep(1)
text_view.send_keys("he11o!!!")
time.sleep(1)
driver.find_element_by_xpath("//XCUIElementTypeButton[@name='Back']").click()
time.sleep(1)
```

#### 三.使用真机进行自动化测试

**3.1**运行ios程序到真机
步骤
1.在Xcode 中登录自己的Apple ID

Xcode->Preferences->Accounts->点击+号，输入自己apple id,确认

2.配开发者信息

HMiOSTest(项目名)->General->TARGETS下的HMiOSTest->Signing下的team选择自己账号，等待signing certificate加载完成

3.选择将要运行的 程序和设备

xcode->顶栏的项目名->Device下的真机

4.快捷键command+r 运行

5.弹出不信任窗口：在手机中进入:设置->通用->描述文件和设备管理->开发者应用->信任自己Apple ID程序，最后弹窗点ok
6.重新command +r 运行
**3.2**配置WebDriverAgent
步骤
1.进入到下载的 WebDriverAgent 项目下

2.输入命令./scripts/bootstrap.sh

3.进入WebDriverAgent-master下的启动 WebDriverAgentxcodeproj
4.配置WebDriverAgentLib 的开发者信息

HMiOSTest(项目名)->General->TARGETS下的WebDriverAgentLib->Signing下的team选择自己账号

5.配置WebDriverAgentRunner 的开发者信息

HMiOSTest(项目名)->General->TARGETS下的WebDriverAgentRunner->Signing下的team选择自己账号

6.配置IntegrationApp 的开发者信息

HMiOSTest(项目名)->General->TARGETS下的IntegrationApp ->Signing下的team选择自己账号

7.修改 WebDriverAgentRunner 的 Product Bundle ldentifier

HMiOSTest(项目名)->General->TARGETS下的WebDriverAgentRunner->Build Settings->搜索id,找Product Bundle ldentifier改值（**改为一个唯一的值**）为：com.idcast.WebDriverAgentRunner

8.修改 IntegrationApp 的 Product Bundle ldentifier

HMiOSTest(项目名)->General->TARGETS下的IntegrationApp ->Build Settings->搜索id,找Product Bundle ldentifier改值为：com.idcast.WebDriverAgentRunner

9.数据线连接真机
10.选择将要运行的 WebDriverAgentRunner 和真机设备

Xcode->顶栏的WebDriverAgentRunner->Device下的真机

11.使用command +u 运行

运行前，清理缓存，在product下的clean build folder

稍等之后会在log中出来一个 url 地址

**设置代理**

在命令行终端通过Homebrew安装iproxy

brew install libimobileddevice

iproxy  <ios真机端口> <mac本地端口>

iproxy 8100 8100

在浏览器中打开这个地址（192.168.31.189:8100/status或192.168.31.189:8100/inspector），如果**显示一串json 数据**即为正确连接手机并且，真机会多一个程序



12.将配置好的 WebDriverAgent 项目替换到 appium 的 WebDriverAgent 项目

+ 打开finder
+ 快捷键 command +shift +g
+ 输入路径/Applications/Appium.app/Contents/Resources/app/node_modules/appium-xcuitest-driver
+ 回车
+ 将旧项目换个名字(WebDriverAgent-back)，当做备份
+ 将配置好的项目(WebDriverAgent-master改名为WebDriverAgent)放在这个目录下

**3.3**运行自动化脚本
步骤
1.修改对应的 platformVersion、deviceName
2.查看udid并增加为启动参数

Xcode->Windows->Devices and Simulators->找到Identifier对应的值

3.运行即可

### ios抓包Sream

直接在app store下载

**抓包请求**
在主界面点击开始抓包按钮，界面顶部显示VPN 标志表示开始抓包，然后启动需要抓包的应用进行操作即可。操作完成之后回到 stream 然后点击停止抓包即可。

根据抓包时间点击抓包记录，即可进入抓包操作的全部请求。全部请求中可以选择按域名或者按进程来分类。

**构建请求**
Stream 除了可以自动抓包之外，还可以进行手动构建请求，类似 Postman 工具的作用。在主界面点击构建请求即可进入到构建请求界面。

**HTTPS 抓包**
如果想要使用 stream对 HTTPS 进行抓包，需要安装 CA证书。点击安装 CA证书，然后直接安装证书。

安装好证书之后，在设置-通用-关于本机-证书信任设置，信任证书即可

**Hosts 设置**
如果想对 Host 设置可以点击主界面 Hosts设置菜单,然后点击添加绑定对应的域名和 IP地址。

**收藏请求**
如果想收藏某个单个请求，可以在请求详情界面点击收藏按钮

**抓包模式**
点击界面中的设置抓包模式，则进入到设置界面

**黑名单**

若设置了具体的黑名单，抓取的请求则是除了具体黑名单外的接口数据。例如上图中我们设置黑名单域名为*.baidu.com表示会忽略百度相关的网络请求。*表示通配符，点击立即生效会开始生效规则。
**白名单**
白名单表示只抓取设置的域名请求，我们设置的域名是*.sougou.com表示抓取搜狗相关的网络请求

**常用工具**
常用工具里面包含一些网络调试用的小工具，主要如下
URL 编码解码
Base64 加密解密
MD5
时间戳转化
RSA 加密解密

### Sandbox 数据存储安全

https://developer.apple.com/app-sandboxing/ 

沙盒也叫沙箱,其原理是通过重定向技术，把程序生成和修改的文件定向到自身文件夹中。在沙盒机制下，每个程序之间的文件夹不能互相访问。
当应用程序需要向外部请求或接收数据时，都需要经过权限认证，否则，无法获取到数据。应用程序中所有的非代码文件都保存在沙盒中,比如图片、音频、属性列表(Plist),sqlite数据库和文本文件等

**Sandbox 文件存储结构**
因为应用的沙盒机制，应用只能在指定的几个目录下读写文件。默认情况下，每个沙盒含有
3个文件夹:Documents,Library 和 tmp

AppName.app:存储 App 执行文件和静态资源文件，该目录包含了应用程序本身的数据包括资源文件和可执行文件等。程序启动以后，会根据需要从该目录中动态
加载代码或资源到内存。

Documents:App 的配置文件等，我们可以将应用程序的数据文件保存在该目录下。不过这些数据类型仅限于不可再生的数据，可再生的数据文件应该存放在 Library/cache 目录下

Library:用来存放默认设置或其它状态信息

Library/Preference:此目录保存应用程序的所有偏好设置,如.plist 文件，iTunes 同步设备时会备份该目录

Library/Cache:缓存数据(cache file and cookie file),该文件夹内的内容不会被同步

tmp：临时文件

**获取沙盒文件**
需要将测试设备绑定开发者证书才可以查看。
xcode
1.打开导航栏中window->device and simulators

2.显示设备下可以查看的 APP

3.选中目标 APP, 点击齿轮图标，然后点击 Download container...保存到指定目录

4.选择下载的文件点击右键弹出菜单,然后选择显示包内容

5.打开之后就可以查看到沙盒文件

**Sandbox验证点**
Sandbox 中存储的文件，主要有Plist files,sqlite、cookie 三种类型，这三种类型的文件
安全验证点分别如下
1.Plist fles(查看具:Xcode(Mac),plistEditor(windows)，)

文件中是否存储敏感信息，敏感信息是否加密
文件是否会被备份，备份泄露是否有风险
文件能否被用于跨客户端的逻辑校验(如某个存储文件的内容是客户端用于判断用户是否登陆，测试将该文件导出，拷贝至其他设备，查看能否越过登陆校验)

2.sqlite(查看工具:sqlite manager
文件中是否存储敏感信息，敏感信息是否加密
文件是否会被备份，备份是否有泄露风险
3.Cookie
查看 cookie 有效期(有效期不能短于登录 cookie 的有效期 )

敏感信息重点关注“登陆信息、用户身份信息、服务器 sq注入链接、管理员登陆账号密码”一类信息。

### Keychain 简介

ios 设备中的 Keychain 是一个安全的存储容器,可以用来为不同应用保存敏感信息比如用户名，密码，网络密码，认证令牌。苹果自己用 keychain 来保存 wi-Fi 网络密码，VPN凭证等等。它是一个在所有 app 之外的 sqlite数据库。

**获取 Keychain 数据**
ios 越狱
需要获取 keychain 数据文件必须要越狱，i0s 越狱教程请根据自己的系统版本来选择:爱思越狱教程 https://www.i4.cn/news_4.html越狱之前切集备份重要资料 ,最好不要使用自己日常使用的设备越狱。
OpenSSH
越狱成功之后需要安装 openssH 工具，i0s 设备其实就是一个小型的 unix 系统，由于苹果的封闭性，在不越狱的手机上，我们能操作的东西很少。如果想在 ios 设备上，通过 pc 直接执行 shell 命令，可以在 ios设备(已越狱)上安装 openssh 服务器，通过pc的ssh 连接过
打开 cydia直接搜索 openssH 安装。如果搜索不到可以在软件源菜单中添加源，如雷锋源:
http://apt.abcydia.com

**KeyChain 数据库**
所有存储在 Keychain 中的数据，实际上是保存在一个 keychain-2.db 的数据库中。 该数据
库位于/private/var/Keychains/目录下

**更改权限**
默认情况下，我们是不能都读取 keychain-2.db 数据库的，所以需要先赋予其可读权限,给
keychain-2.db 数据库可读权限

**查看 KeyChain 数据**
将 keychain-2.db拷贝到桌面，然后使用 DB Browser for sQLite 即可查看数据内容

### htpps安全

但是即使使用了 HTTPS，也有可能因为没有校验服务器证书的原因导致被中间人劫持。

就是打开 APP 登录帐号，使用抓包工具如 charles 去看是否有请求获取敏感信息，比如获取资源包或者文件脚本。安全加固实施建议:
1.App 内要对 HTTPS 证书做校验。
2.避免使用有漏洞的第三网网络库(如 AFNetworking<2.5.3 版本)
3.关键数据(如登录密码、卡号、交易密码等)单独加密

### 防止网络请求被抓包

App 安全越来越受重视，要分析一个App，抓包是必不可少的，那么如何防止像 charles 之类(中间人攻击类型)的抓包软件的抓包呢?主要有以下几个思路
1.检测是否使用了代理，检测到使用了代理就关闭网络请求。

2.使用自签名证书的应用和双向验证的应用。
3.通过 HTTP/1.1 及以上版本的 CONNECT 请求方式
4.对返回的数据进行加密(RSAltoken|AES128 等等)

### 绕过代理发送请求

现在 ios 上的网络请求基本分为三类
**NSURLConnec tion**
**NSURLSession**
**CFNetWork**

通过 HTTP/1.1 及以上版本的 CONNECT 请求方
式
什么是 CONNECT 请求 ?
平时工作中，GET 跟 POST 是我们用的比较多的请求方式，而CONNECT 是在 HTTP/1.1 协议中HTTP/1.0 定义了三种请求方法:GET，POST和 HEAD方法，HTTP/1.1 新增了五种请求方法OPTIONS、PUT、DELETE、TRACE和CONNECT 方法。

它主要是把服务器作为跳板，先验证用户名和密码等信息，再让服务器代替用户去访问其它网页，之后把数据返回给用户,之所以说采用 CONNECT 请求当跳板，可以防止 charles 抓包所以就能达到防抓包的目的了。是因为 charles抓CONNECT 的请求，会识别为 unknown 

### 脱壳背景

开发提交给 ppstore 发布的 app 都经过官方保护加密。经过 Appstore 加密的应用，我们无法通进行反编译静态分析,在逆向分析过程中需要对加密的二进制文件进行解密才可以进行静态分析，这一过程就是所谓的脱壳(砸壳)。
ios 脱壳工具目前主要有一下3种:
**Clutch**
**dumpdecrypted**
**frida-ios-dump**
由于 clutch 脱壳不太稳定，frida-ios-dump环境配置比较复杂，所以本文以 dumpdecrypted这个工具做为脱壳工具

### 安全扫描平台

目前有很多 App 安全扫描平台如 360 漏洞扫描，腾讯金刚审计系统等等。这些安全扫描平
台的功能如下:

静态分析：静态分析应用源代码中存在的安全风险，检测包含 Android组件安全，应用程序安全，数据安全

动态检测：检测包含客户端自身安全，Android组件增强检测，应用通信安全，数据安全

模拟人机交互：模拟用户和手机交互行为，检测交互过程中应用存在的通信安全风险

服务器指纹检测：探测后端服务器指纹，进行安全分析，检测后端服务器系统，开发框架web 服务器，数据库等服务组件安全

服务器后端API检测：抓取应用通信过程中的资源地址，检测应用与服务器通信接口是否存在SQL 注入，xss 跨站，中间人攻击等安全问题

### MobSF

一种开源自动化的移动移动应用程序安全测试框架，能执行静态，动态和恶意软件分析

它可用于 Android/i0s 和 windows 移动应用程序的有效和快速安全分析，并支持二进制文件(APK，IPA和 APPX)分析

**静态分析**
扫描内容
**Android**
APK基本信息:文件名、文件大小、MD5、SHA-1、SHA-256
APP 信息:包名、Main Activity、版本号等
组件:Activity、Service、Broadcast Receiver、Content Provider
证书信息(siger certificate)
权限信息
Android API信息
Androidmanifest 分析(标志位、组件配置等
代码分析、文件分析
url、email、string等

**ios**
IPA基本信息
自定义网址方案
权限许可
应用传输安全性(ATS)
Plist 文件分析
文件分析
请求网站分析
防火墙数据库
邮件，源文件

**Show/Stop Screen**
1.点击 show Screen 可以实时同步设备屏幕，方便测试执行査看。在 Dynamic Analyzer 菜单可以查看实时动态分析日志，Errors菜单可以查看错误日志。

**Install/Remove MobSF RootCA**
Insta11/Remove MobSF RootcA用来安装卸载MobSF CA证书,方便对样本中 HTTPS 流量进行截
获。
**Start Exported Activity Tester**
遍历获取 AndroidManifest.x1文件中的所有 Exported Activity 测试流程如下
1.依次启动activity,adb-s IP:PoRT shell am start -n PACKAGE/ACTIVITY
2.获取当前 activity运行时的屏幕截图，并保存截屏
强制关闭应用:adb-sIP:PORT shell am force-stop PACKAGE

**Start Activity Tester**
遍历 AndroidManifest.xm1文件中的所有 Activity，而不单单是 Exported。
处理流程与 Exported Activity 一致。
**Take a Screenshot**
截屏并保存到本地

### 指标计算公式

1.**吞吐量计算公式**
指单位时间内系统处理用户的请求数
从业务角度看，吞吐量可以用:请求数/秒、页面数/秒、人数/天或处理业务数/小时等单位来衡量,
从网络角度看，吞吐是可以用:字节/秒 来衡量对于交互式应用来说，吞吐量指标反映的是服务器承受的压力，他能够说明系统的负载能力

F=N*R/T

这里，F表示吞吐量;N表示并发虚拟用户个数(Concunency Virtual User，并发虚拟用户)，R表示每个VU发出的请求数量，T表示性能测试所用的时间。但如果遇到了性能瓶颈，此时吞吐量和 VU 数量之间就不再符合给出公式的关系

2.**并发用户计算公式**
指在客户端的一批用户同时执行一个操作的数量

案例 : 一个 BBS 有 3000 个用户,平均每天大约有 400 个用户要访问该网站，一天之内，用户从登录到退出的平均时间是4小时,用户只在一天8小时内使用该系统,计算平均并发用户数和并发用户的峰
值为多少 ?

(1):C=nL/T

在公式(1)中;C是平均的并发用户数:n是login session 的数量;L是login session 的平均长度:T指考察的时间段长度

(2)
$$
C^u=C+3\sqrt{C}
$$
公式(2)则给出了并发用户数峰值的计算公式，其中，C^u指并发用户数的峰值，C 就是公式(1)中得到的平均的并发用户数。该公式的得出是假设用户的login session 产生符合泊松分布而估算得到的

C=400*4/8=200 

C^u=200+3V200=242根据公式求得因此平均并发用户数为 200 人，峰值大约为 242 人。

说明:并发用户数量的统计的方法目前还没有非常准确的公式，因为不同系统会有不同的并发特点。例如 OA 系统统计并发用户数量的经验公式为:使用系统用户数量"(5%~20%)。对于这个公式是没有必要拘泥于计算的结果，因为为了保证系统的扩展空间，测试时的并发用户数量要稍微大一些,除非是要测试系统能承载的最大并发用户数量

### **附录参考资料**


https://testerhome.com/topics/6962

https://testerhome.com/topics/14911