---
title: AirtestIDE使用
categories: tools
img: /images/AirtestIDE.jpg
tags:
  - airtest
  - poco
  - tidevice
abbrlink: 35131
date: 2024-03-13 23:17:32
---

## AirtestIDE使用

### Airtest和poco（绝对坐标与相对坐标）

**Airtest的坐标系**
airtest的touch接口，实际上点击的是一个 (x, y)绝对坐标，在截图语句中，airtest会先根据传入的图片找到该图片在当前画面上的位置坐标，然后进行点击(查看log日记可以看到)。

而airtest的swipe接口，滑动起点和滑动终点使用的也是(x, y)绝对坐标。

根据设备分辨率，计算合适的坐标进行操作。

```python
width, height = device().get_current_resolution()
start_pt=(width*0.9,height/2)
end_pt=(width*0.1,height/2)
for i in range(5):
    swipe(start_pt, end_pt)
    sleep(1)
```

使用屏幕坐标的功能查看坐标

打开IDE的选项>--设置，可以看到在Device设置部分，有实时坐标显示和相对坐标显示俩个选项

勾选实时坐标选项，可以实时在手机屏幕画面上显示出鼠标位置的坐标，方便大家获取绝对坐标。此时 点击鼠标右键，还可以自动将当前坐标信息复制到剪贴板中

在勾选了实时显示坐标的情况下，再勾选相对坐标选项，将会以(0, 0) 到 (1, 1) 为范围显示出相对坐标。使用相对坐标可以避免跨分辨率的操作点超出屏幕的问题，使坐标操作兼容性更好。

**poco的坐标系**

使用局部坐标系的click接口

首先我们需要知道点击默认是点在 anchorPoint 上的，每个UI都会有一个 anchorPoint ，也就是检视器(Inspector)中UI包围盒的那个红点，大部分情况下 anchorPoint 都在UI包围盒的正中央。

引入局部坐标系来表示**相对于某UI的坐标**。局部坐标系以UI包围盒左上角为原点，向右为x轴，向下为y轴，包围盒宽和高均为单位一。局部坐标系可以更灵活地定位UI内或外的位置，例如(0.5, 0.5)就代表UI的正中央，也就相当于我们上文中默认的anchorPoint；超过1或小于0的坐标值则表示UI的外面

想点击 anchorPoint 以外的其他指定位置时，可以传一个参数到 click 方法中，这个参数是一个用list或tuple表示的2维向量，其 [x, y] 值分别表示相对于包围盒左上角的偏移量，左上角为 [0, 0] ，右下角为 [1, 1] 。

想改变以anchorPoint为起点，也可以使用focus方法：

poco("com.miui.home:id/workspace").offspring("天气").offspring("com.miui.home:id/icon_icon").focus([0,0]).click()

**使用归一化坐标系的swipe接口**

归一化坐标系就是将屏幕宽和高按照单位一来算，这样UI在poco中的宽和高其实就是相对于屏幕的百分比大小了，好处就是不同分辨率设备之间，同一个UI的归一化坐标系下的位置和尺寸是一样的

```python
joystick = poco('movetouch_panel').child('point_img')
joystick.swipe('up')
joystick.swipe([0.2, -0.2])  #45度向右上方
joystick.swipe([0.2, -0.2], duration=0.5)
```

### Airtest封装的ADB操作

问题：遇到ADB版本冲突的报错：
raise AdbError(stdout, stderr)
airtest.core.error.AdbError: stdout[] stderr[adb server version (36) doesn't match this client (40); killing...
could not read ok from ADB Server
而我们不清楚电脑里面那么多ADB，究竟哪些是40版本，哪些是36版本时，我们就可以使用 adb version 查看当前的ADB版本，然后将电脑里面所有的ADB替换成同一个版本，从而解决这个版本冲突的问题

[adb教程](https://github.com/mzlogin/awesome-adb)

```python
__author__=AirtestProject
from airtest.core.api import *
from airtest.core.android.android import *
auto_setup(__file__,devices=["Android:///"],compress=90)
android = Android()
#打印本地设备的序列号
print(android.get_default_device())
#打印设备上安装的第三方应用的包名列表
#等同于:adb -5 《设备序列号> shell pm list packages -3
print(android.list_app(third_only=True))
```

1）返回应用的完整路径：path_app()
android = Android()
android.path_app("com.netease.cloudmusic")
2）检查应用是否存在于当前设备上：check_app()
android = Android()
android.check_app("com.netease.cloudmusic")
3）停止应用运行：stop_app()
stop_app("com.netease.cloudmusic")

启动应用:start_app()

start_app("com.netease.cloudmusic")

清除应用数据：clear_app()

clear_app("com.netease.cloudmusic")
4）安装应用：install_app()
install(r"D:\demo\tutorial-blackjack-release-signed.apk")

卸载应用：uninstall_app()

uninstall("org.cocos2dx.javascript")

5）关键词操作：keyevent()
keyevent("HOME")
keyevent("POWER")
6）唤醒设备：wake()
wake()
7）返回HOME：home()
home()
8）文本输入：text()
text("123")
9）检查屏幕是否打开：is_screenon()
android = Android()
android.is_screenon()
10）检查设备是否锁定：is_locked()
android = Android()
android.is_locked()
11）获取当前设备的分辨率：get_current_resolution()
android = Android()
android.get_current_resolution()
12）其它adb shell命令：shell()
shell("ls")
shell("pm list packages -3")

### 截图脚本

Airtest会尝试用 SURFMatching 、TemplateMatching 和 BRISKMatching 这三种算法来进行图像识别

**阙值 和 可信度** ，他们的取值范围都是[0,1]。在每一条图像识别的脚本中，都会有1个用于结果筛选的阙值，默认值为0.7。
当上述三种算法在执行过程中识别到初始结果时，就会计算出来这个初始结果的可信度，当 可信度＞阙值 的时候，程序会认为 找到了最佳的匹配结果 ；而当 可信度＜阙值 的时候，程序则会认为 没有找到最佳的匹配结果 。

在执行截图脚本的时候，查看log窗口，观察算法识别结果的可信度：

① 可信度>阙值，程序判定找到匹配结果

② 可信度<阙值，程序判定未找到匹配结果，循环用三种算法继续查找直到超时

+ 截取图标时尽量不要截入过多的背景内容

+ 打开应用尽量使用start_app而不是截图脚本，

  start_app("com.netease.cloudmusic")  #打开网易云音乐

+ 用image editor查看截图识别结果的可信度

录制/编写好1条截图脚本之后，无需运行，可以直接双击截图，进入图片编辑器，点击左上角的 `snapshot+recognition` 按钮，即可查看截图在当前页面的识别情况，包含识别出来的位置以及识别结果的可信度

+ 巧用target_pos点击截图的不同位置

截图脚本都是点击截图的中心位置，即 `target_pos=5` 。对于一张截图来说，总共有9个 `target_pos` ，当我们把截图的 `target_pos` 设置成不同的值时，脚本会点击在截图不同的位置上

1	2	3

4	5	6

7	8	9

打开图片编辑器，右侧可以修改 `target_pos` 的值

修改完成之后，把截图脚本切换成代码模式，我们就可以看到此时的截图脚本里面多了 `target_pos` 这个参数：

当精准截图（仅截取某个按钮/图标）不能满足唯一定位时，我们可以考虑加大截图范围，增加更多的特征点，确保截图定位的准确性

+ 巧用坐标进行点击/滑动,如首页的轮播图
+ 巧用keyevent("BACK")替代返回的截图脚本
+ 录制功能要注意截图的兼容性
+ 画面切换的时候，可以多使用wait或者sleep，再进行点击操作
+ 合理调整阙值

双击截图打开图片编辑器，在右侧修改截图的阙值

设置全局的 threshold ：
from airtest.core.setting import Settings as ST
ST.THRESHOLD = 0.7  # 其他语句的默认阈值

截图语句的阙值设置：

from airtest.core.setting import Settings as ST
ST.THRESHOLD_STRICT = 0.7

+ 用自定义语句（例如截图列表）

对应截图纳入搜索列表。代码脚本如下：

```python
picList = [pic1,pic2,pic3]  # 截图的图片对象列表
for pic in picList:
     pos = exists(pic)
     if pos:
         touch(pos)
         break
```

注意：如果for循环中没有break语句，会导致次逻辑运行时将所有的图片都找一遍(找到后执行touch)，而非找到合适结果立即返回

+ 用poco语句代替截图脚本

  取前3行遍历节点的脚本

```python
select=list(poco("android.widget.LinearLayout").offspring("com.netease.cloudmusic:id/dragSortplayListlist").child("android.view.view"))
for s in select[:3]:
  s.offspring("com.netease.cloudmusic:id/checkedImage").click()
```

### tidevice-不依赖 xcode 启动 WebDriverAgent 

 **不依赖 xcode 启动 WebDriverAgent** 完成设备连接

**1）安装tidevice库**

在本地python环境中，使用 `pip install tidevice` 命令安装 tidevice 库。

另外需要注意的是，目前 tidevice 库仅支持安装在 **python3.7及以上版本** 中。

**2）常用的tidevice命令**

查看已连接设备：

```shell
tidevice list
```

查看设备上的第三方应用包名：

```shell
tidevice applist
```

指定设备安装：

```
# $UDID可以使用tidevice list命令查看
tidevice -u $UDID  install D:/test.ipa
# 或者
tidevice -u $UDID install https://example.org/example.ipa
```

更多详细的功能可以查看 tidevice 的github文档：https://github.com/alibaba/taobao-iphone-device 

**3）确保手机上已经安装上WebDriverAgent**

对于未跑过自动化的iOS设备，我们需要先检查设备上是否安装好了WebDriverAgent这个APP，如未安装，则可以通过以下2种方式安装：

① 将iOS设备与一台Mac连接，然后使 **用xcode编译源码安装** ，成功安装WebDriverAgent即可脱离Mac；

② 使用tidevice的安装命令，将 **开发者证书重签名** 的 WebDriverAgent.ipa 安装到iOS设备上

**在IDE连接tidevice启动的iOS**

1）用数据线将iOS设备与Windows电脑连接
2）查看设备里WebDriverAgent的BundleID
tidevice applist

**3）指定BundleID启动** 

```shell
tidevice xctest -B com.gameappium.WebDriverAgentRunner.xctrunner
```

WebDriverAgent 启动成功后，后台挂着该命令行窗口即可。
4）在IDE的设备连接窗口连接iOS设备
最后打开 最新版 的IDE（1.2.8版本），在连接iOS设备框中输入：

DeviceIdentifier可以在启动的信息中查看

http+usbmux://DeviceIdentifier
最后点击连接即可

**5）补充另一种启动方式**

```
tidevice wdaproxy -B com.gameappium.WebDriverAgentRunner.xctrunner --port 8200
```

与步骤3）中的xctest启动方式不同的是，使用wdaproxy启动之后，我们可以在浏览器中使用http://localhost:8200/status来访问到这个iOS

### 用脚本连接windows窗口

**窗口句柄**是每个Windows窗口对象拥有的独一无二的32位无符号整数，而且每次打开窗口，旬柄的数值都会变化。
使用句柄连接窗口的脚本我们可以这么写auto_setup( file_,devices=["Windows:///133194"])

大多数情况下，**搜索窗口**的title比较不容易变化，所以我们可以写一个正则表达式去匹配到待测窗口的title。

例如匹配“吹梦到西洲”后面跟着换行符以外的任意字符的窗口titleauto_setup( file_,devices=["Windows:///?title_re=吹梦到西洲.*"])

如果不需要指定某个窗口应用的话，我们还可以使用如下代码直接连接整个桌面来做自动化:
auto setup( file_,devices=["Windows:///"])

### poco定位器和操作

①基本选择器
利用元素的一些基本属性来进行定位，比如name、text等等poco(name="网易云音乐")
②相对选择器
利用控件之间的父子关系、爷孙关系和兄弟关系等来定位控件，例如parent()、child（）、offspring（）等poco("LinearLayoutCompat").child("搜索”)
③空间顺序选择器
常用于UI树中多个相同名称的节点定位，根据节点在U1树的索引顺序进行定位，索引坐标从0开始
poco("mainActivityTab").offspring("aActionBar$Tab")[0]

①点击操作:
poco("star_single").click()

poco('star_single').long_click()
②)读取和设置控件的属性
poco("star_single").get_name()
poco("star_single").attr('name' )

poco("star_single").get_text()

poco("pos_input").set_text("123”)

poco("pos_input").setattr('text',"456")
③判断元素是否存在
poco(XXX).exists()
④)拖动与滑动
poco("star").drag_to(poco("shell"))

⑤内部偏移和外部偏移:
poco("pearl").focus([0.1,0.1]).long_click()

poco("pearl").focus([0.9,0.9]).long_click()

poco("pearl").focus([0.5,-3]).long_click ()
⑥遍历元素:
for star in poco("playDragAndDrop").child("star"):star.drag_to(poco("shell"))
⑦等待事件:
poco("bomb").wait_for_appearance()poco("bomb").wait_for _disappearance ()
yellow = poco ("yel low")blue = poco ("blue")
black = poco ("black")
poco.wait for all([yellow,blue,black])poco.wait for any([bomb, yeilow, blue])

输入backspace键

driver.find element by id("kw").send keys(Keys.BACK_SPACE)

### unity游戏接入poco-sdk

**Poco的支持情况**
Android和iOS原生直接支持，无需嵌入任何

代码引擎渲染的项目都需要事先接入Poco-SDK才能查看UI树

目前poco支持umity、Coc0s2dx-lua、Cocos2dx-js、Cocos-creator、UE4、Egret......

**在unitv项目中接入Poco-SDK**

下载Poco-SDK包

把Unity3D文件夹放到项目脚本中

根据UI类型选择，只留下1中gui文件夹

在unity载入脚本

成功后可使用IDE单独连接Game窗口查看控件树信息

**成功接入后打成Android包**

文件-build settings-选择安卓-切换平台

进入玩家设置-修改包名-生成Android包

在待测设备安装上包，并启动项目，IDE选择对应模式查看控件树

**接入Poco-SDK的常见问题**

如何判断项目是否需要接入sdk:引擎渲染项目都需要(游戏应用)

接入Poco-SDK的前提条件:有项目源码

为什么要在根节点或者主camera载入脚本:因为这些节点不会被销毁

### 其他技巧

连接上1台安卓设置，然后点击右上角的工具按钮，再点击 `显示Android助手` 选项，可以看到，弹出窗口的左下角，显示了设备当前所有应用的包名（该助手 **仅适用于安卓设备**）

除了使用使用poco辅助窗的录制功能，还可以通过 **双击控件树上的某个节点** ，来快速生成该节点的定位脚本，在加其他操作

**获取设备连接的字符串**

用IDE连接待测设备，随便开个脚本点击运行，1、2秒后终止运行，拉到log查看窗的最上方，看到完整的运行命令。含有设备连接字符串，直接复制

**在命令行连接设备**
①连接安卓真机
airtest run D:/test/moniqi_test.air 	--device Android://127.0.0.1:5037/5PZTQWQ0GES8RWUG
②)连接安卓模拟器
airtest run D:/test/monigi test.air	--device Android://127.0.0.1:5037/emulator-5554?cap_method=JAVACAP&&ori_method=ADBORI
③注意事项
在IDE需要勾选备选参数才能够连接上的设备，命令行的设备参数中也要加上这些备选参数;&&符号在Windows和MacOS下都需要转义，否则命令会出现被截断的情况

**在脚本中行连接设备**
①使用auto_setup接口:
auto_setup( file_,devices=["Android://127.0.0.1:5037/5PZTQWQ0GES8RWUG","Android://127.0.0.1:5037/emulator-5554"])
②)使用connect device接口:
dev =connect_device("Android://127.0.0.1:5037/5PZTQWQ0GES8RWUG")

dev = connect device("Android://127.0.0.1:5037/emulator-5554")
③)使用init_device接口:

init_device(platform="Ándroid", uuid="5PZTQWQOGES8RWUG")

**开启录屏**

adb=ADB(serialno=”PFT4PBLF75GQHYBM”)

recorder = Recorder(adb)

recorder.start_recording()
**结束录屏**

recorder.stop_recording(output=r”D:\airtest\music.mp4”)

**生成报告**

simple_report(__file__,logpath=r“D:\airtest\mylog”,output=“D:\airtest\log.html”)



[学习教程](https://airtest.doc.io.netease.com/summary/summary/)

testerhome: https://testerhome.com/

掘金: https:/uejin.cn/

CSDN:https://www.csdn.net

博客园: https:/www.cnblogs.com/

开源中国:https://www.oschina.net
《软件测试》
《软件测试的艺术》
《Google软件测试之道》
《mysql必知必会》
《移动app测试实战》
《腾讯Android自动化测试实战》
《鸟哥的Linux私房菜》
《Web安全测试》