---
title: UI自动化测试之Airtest
tags: Airtest
categories: tools
img: /images/Airtest.jpg
abbrlink: 30584
date: 2023-12-10 15:58:20
---

## UI自动化测试之Airtest

### Airtest简介

Airtest由网易团队出品，是一个基于图像识别原理的跨平台UI自动化测试框架，适用于游戏和应用程序。官方文档：https://airtest.readthedocs.io/zh-cn/latest/index.html

特点：

（1）跨平台：Airtest几乎可以在所有平台上执行游戏和APP自动化

（2）易操作：使用图像识别技术来定位UI元素，无需嵌入任何代码即可对游戏和应用进行自动化测试。

（3）可扩展性：通过使用Airtest提供的命令行和Python API接口，可以轻松地在大规模设备集群上运行脚本

（4）GUI工具：AirtestIDE是一个强大的GUI工具，可以帮助你录制和调试脚本

Airtest优点：

1）框架基于图像识别（SURFMatching，TemplateMatching，BRISKMatching），UI和控件识别，操作简单，功能简洁。

2）对代码能力要求不高，容易上手。结合工具本身的脚本录制功能，开发脚本速度快，适合版本快速迭代的要求。

3）可引入Python第三方库，支持Python进行个性化脚本编程。

4）可一键生成测试报告，报告美观，清晰明了。

Airtest缺点：

1）最大的缺点是图像、控件定位不够准确，如果不同设备的尺寸、分辨率不同，或者图像的背景色变化，控件图案修改的话。

2) 因为是基于图像识别的框架，所以代码执行速度慢，容易造成图像识别不到。

总结：优点大于缺点，且图像识别准确度的问题有很多办法可以改善、提高。

3、Airtest库

Airtest有图像识别、Poco、selenium三大类库

4、Airtest环境搭建

（1）Python

输入cmd打开命令行窗口，执行命令：pip install -U airtest

说明：安装Python的Airtest库，通过Python代码直接调用Airtest库的API方法。

提示：此方法需要有一定的Python基础。AirtestIDE内置了Python3.6.5，亲测Python3.6.5版本可以安装airtest

### Airtest与安卓模拟器进行连接

常用的安卓模拟器：网易的MUMU、夜神、雷电等。我们这里使用网易MUMU，直接下载安装到C:\Program Files\MuMu

1、准备工作

（1）打开开发者模式

一般安卓手机：进入设置—>系统（或关于手机）—>找到版本号，多点击几次，就可以开启开发者模式。

（2）打开USB调试模式

先打开开发者模式，进入开发人员选项，可开启USB调试

**注意：一定要选择USB配置：MIDI（打开文件传输）！！！**

（3）连接设置

a.启动安卓模拟器

b.在Airtest窗口点击【刷新ADB】或【远程设备连接】

.使用备用连接参数，设置兼容模式

真机或手动给模拟器装上Airtets专用输入法Yosemite.apk，因为很多款模拟器都不能自动装上这个应用，所以我们提前手动装下更加稳妥

AirtestIDE提供了3个备用的连接参数：

① 第一个 Use javacap ，是给部分无法正常看到手机画面、minicap初始化失败 的手机或设备用的，所以模拟器看到黑屏、部分特殊的平板等设备可以考虑勾选这个选项。

② 第二个 Use ADB orientation 是 屏幕旋转 的，如果在安卓手机屏幕旋转方向检测有问题、或者是部分特殊的平板无法显示正确的屏幕方向时可以勾选。

③ 第三个 Use ADB touch 是 发送adb指令来点击屏幕 ，效果很差，速度也很慢，不建议勾选，只有在部分无法点击屏幕的特殊安卓设备上才需要使用（例如智能后视镜、特殊型号的平板等设备上） 正常情况下，手机都可以点击，如果无法被点击（比如小米设备），一般都是因为手机设置有选项漏了打开，特别是小米设备要注意 开启允许模拟点击 的设置。

（4）修改设备地址及端口号（因为真机或模拟器都分不同的厂商）

Airtest远程连接，默认展示的是网易MUMU的端口号。如果使用其他厂商的模拟器，需要修改端口号。

### Airtest图像库（Touch、脚本运行、测试报告）

Touch方法

作用：触摸/点击动作

常用参数：

v : 点击对象的图像或坐标

times: 点击次数，默认是1

duration: 点击时间，默认是0.01秒

right_click:右键点击（仅限Windows模式）

运行脚本

点击三角形的【运行】按钮，或者使用快捷键F5

停止运行：Shift + F5

运行单行代码：选中代码行，右键，选中并单击“只运行选中代码”

d、查看报告

方法1：点击菜单栏【运行】——>打开报告目录

方法2：右键脚本文件名称Tab——>打开报告文件目录

这种方法，可以将相对路径的图片资源和静态资源整个打包，后续发送给其他查看。

方法3：使用快捷键：Ctrl + L

方法4：cmd打开命令行窗口，进入脚本所在路径，执行如下命名：

airtest report D:\zxt\AirtestIDE\xiaokang.air\xiaokang2_auto_script.py --log_root D:\zxt\AirtestIDE\xiaokang.air\log --outfile D:\zxt\AirtestIDE\xiaokang.air\log\xiaokang2_auto_script.log\log.html --static_root D:\zxt\AirtestIDE\airtest\report --lang zh --export D:/zxt/AirtestIDE/xiaokang.air/log

方法5：

generate html report

from airtest.report.report import simple_report

simple_report(__file__,logpath=True,output='D:\zxt\AirtestIDE\xiaokang.air\report\log.html')

### Airtest图像API-wait，swipe

wait()方法

作用：在等待界面元素出现，默认0.5s找一次，最多找20s。如果找到则返回返回图片中心点坐标；否则，raise TargetNotFoundError

常用参数：

v：图片

timeout：等待超时时间，默认是20s

interval：每次匹配的时间间隔

wait方法解决什么问题？

解决界面元素存在，但加载需要时间的问题。

swipe方法基本使用：

作用：从屏幕一个位置滑动到另一个位置

常用参数：

v1: 图片 或 坐标（x,y）

v2: 图片 或 坐标（x,y），从v1滑动到v2

vector: [x,y]录制时自动生成，记录了屏幕中的滑动比例，向右为x轴正方向，向下为y轴正方向。

### Airtest图像API（text、snapshot、sleep、keyevent）

1、text方法

作用：输入文本操作

常用参数：

text: 要输入的文本

（注：输入的位置为当前页面光标焦点所在的位置，一般与touch方法一起使用）

enter: 完成输入后自动执行Enter操作，默认为True

2、keyevent方法

作用：模拟键盘按钮输入，支持键码，如3为home键,4为back键

常用参数：

keyname: 固定键名或键码，参考：https://www.cnblogs.com/vip136510786/p/14705567.html

3、snapshot方法

作用：截取当前屏幕图片，可以在测试报告中显示。

常用参数：

filename: 保存截屏为指定文件。

msg: 描述测试点，可在html报告中呈现。

4、sleep方法

作用：暂停时间

常用参数：

secs: 暂停时间，单位秒，默认1.0s

### Airtest图像API-断言方法

1、assert_exists方法

作用：断言页面存在某元素，结果是布尔类型值

常用参数：

v: 图片

msg：描述测试点

return：找到图片，则返回图片中心点坐标；否则，将raise AssertionError

2、assert_not_exists方法

作用：断言页面不存在某元素，结果是布尔类型值

常用参数：

v: 图片（注：判断当前页面中不存在指定图片，不存在则通过，存在则不通过）

msg：描述测试点

3、assert_equal方法

作用：判断第一个值和第二个值相等

常用参数：

first：第一个值

second：第二个值

msg：描述此断言语句对应的测试点内容。

4、assert_not_equal方法

作用：判断第一个值和第二个值不相等

常用参数：

first：第一个值

second：第二个值

msg：描述此断言语句对应的测试点内容

拓展：

如何解决无法输入账号的问题：MUMU模拟器设置——>语言和输入法——>将输入法改为nemu-vinput

切换代码模式：在代码编辑区域，选中代码行，右键选择并点击“图片/代码模式切换”。

### Airtest-实战iOS真机（环境搭建）

1、环境搭建需要

（1）硬件

一台苹果电脑（运行xcode）

一部iphone手机（运行APP）

（2）软件

iOS-Tagent(WebDriver服务器)

xcode(iOS集成开发工具，运行iOS-Tagent)

iproxy(代理工作，做端口映射)

AirtestIDE(图像识别自动化测试工具)

2、软件功能

（1）iOS-Tagent

作用：在手机上创建一个WebDriver服务器，可用于远程控制iOS设备，定位UI元素。

下载：https://github.com/facebook/archive/WebDriverAgent

运行依赖：xcode

（2）xcode

作用：iOS集成开发工具，主要作用为运行WebDriverAgent文件到手机中

下载：appStore搜索xcode

运行依赖：开发者账号

xcode设置：

前提：将真机使用数据线连接上mac电脑

测试运行WebDriverAgentRunner到手机

如果失败，排查思路：

①在xcode中点击Test后，第一次将WebdriverRunner时，手机需要信任该项目(设置->通用->

设备管理)

②在手机中启用UI自动化(设置->开发者->EnableUIAutomation)

③如果有其他异常，根据异常提示信息自行参考百度或访问

https://github.com/appium/appium/blob/master/docs/en/drivers/ios-xcuitest-real-devices.md

查阅相关解决方案

xcode需要的操作：

a.添加开发者账号，普通apple ID即可

b.配置WebDriverAgent（Team、Product Bundle Identifier）

c.测试运行WebDriverAgentRunner到手机

### Airtest-实战iOS真机（连接设备）

1、连接真机注意事项

（1）在xcode中点击Test前， 检查项目默认终端是否选择真机设备。

（2）点击在xcode中Test之后， 要查看控制信息， 如果控制台没任何信息输出，可以等待或者多Test几次， 直到控制台输出启动相关信息

（3）xcode配置iOS-Tagent只需第一次配置，之后使用无需在单独配置， 切莫乱修改参数；

（4）真机设备中， 要开启自动化测试和信任iOS-Tagent项目

2、连接真机步骤

（1）将真机使用数据线连接电脑

（2）启动xcode并打开配置好的iOS-T agent项目(菜单-Product->Test启动自动化服务程序)

（3）打开终端运行：iproxy 8100      8100(启动端口映射服务程序)

（4）启动AirTestIDE工具(选择【连接ios设备】->点击【connect】)

Airtest-实战iOS真机（钉钉登录、退出）

案例总结：

1、API使用方法和安卓没有区别

2、真机速度快，输入用户名和密码时需要暂停一会

3、ISO和安卓区别在于环境搭建

AirTest参考API：[官网](https://airtest.doc.io.netease.com/)





























