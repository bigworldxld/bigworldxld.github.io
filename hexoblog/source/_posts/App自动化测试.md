---
title: App自动化测试
img: /images/app.jpg
categories: test
tags:
  - android
  - appium
  - adb
abbrlink: 52919
date: 2023-09-19 20:00:06
---

### 一、App环境搭建

1、安装jdk，配置jdk环境变量
2、Android SDK环境安装
3、Appium server安装
4、模拟器的安装(夜神模拟器)
5、安装appium-python-client	Python第三方库

主流App自动化方法:
SDK原生 : java + android
UIAutomator2:python+android
w.Appium:各种编程语言 + 各种平台* (android 、ios、 windows)

### 二、App自动化测试原理

如何通过代码操作不同操作系统 (ios/Android)不同版本的手机终端?

自动化脚本——》发送http指令——》AppiumServer——》转发指令——》Android SDK环增(软件工具包/区架)——》操作手机终端——》被测手机终端(模拟器/真机)

Android SDK环境:不同系统不同版本差异比较大

不同系统不同版本可能用到不同软件包/框架?

所以需要自动化脚本需要指定操作终端的设置参数

### 三、Desired Capabilities-Appium 自动化配置项

设置参数:操作系统 platformName

版本 platformVersion

设备名称 deviceName

包名 (应用程序) appPackage

入口启动页面 appActivity

不重置 noreset

https://appium.io/docs/en/2.1/Desired 

### 四、常用ADB命令

1、连接模拟器

adb connect 127.0.0.1:62001

其他模拟器:雷神 5555	夜神 62001	mumu 7555 逍遥

2、查看连接的设备

adb devices

如果出现如下提示：

adb server version (31) doesn't match this client (36); killing...
原因： adb版本不对 ,Androd SDK的版本和模拟器的adb版本不一致

解决方案：将Android SDK的 adb替换掉模拟器的adb即可。模拟器adb路径 ：{安装Path}\Nox\bin

3、查看被测app的包名及入口启动页面

查看被测app所有信息

aapt dump badging apk的路径

查看指定字符串包名

aapt dump badging apk的路径 | findstr 包名	

查到信息中包含

package:name=“包名	”

launchable-activity:name=“启动项包名”

常见命令：https://blog.csdn.net/thundersoft230/article/details/126158186



**查看qq日志导出文件到主机下**

adb logcat | findstr "qq" >> ./Desktop/qq.log

adb shell dumpsys battery set status 2  充电模式

adb shell dumpsys battery set level 50 电量

adb shell pm list package -3	用于列出第三方应用

```python
from appium import webdriver
# 1、设置终端参数项
desired_caps={
"platformName":"Android",
 "platformVersion":"5.1.1",
 "deviceName":"xiaomi",
 "appPackage": "com.tal.kaoyan",
 "appActivity":"com.tal.kaoyan.ui.activity.SplashActivity",
 "noReset":True		#不记住用户信息（包括登录等）
}
# 2、appium server进行启动
#还需要做哪些前期的准备工作
#1、appium server进行启动
#模拟器/真机必须能够被电脑识别 ----》如何操作? adb命今	adb devices
#3、发送指令给到appiumserver
driver=webdriver.Remote("http://127.0.0.1:4723/wd/hub",desired_caps)
```

页面管理(am activity manager)操作: 

手机应用中的每个页面就是一个activity启动应用，需要通过应用的启动activity来完成调用。

adb shell am start -W -S[包名]/[启动activity名] 启动对应的应用

adb完成自动化操作:

1、先获取包名
adb shell pm list package -3

2、根据包名获取应用的启动activity
adb shell monkey -p [被测包名]-v -v -v 1

3、根据获取到的activity名字，启动应用
adb shell am start -W -S com.tencent.mobileqq/.activity.SplashActivity

4、按顺序执行 input操作，完成对手机的控制。

5、写成一个test.bat脚本执行就好

```bash
rem 启动应用
adb shell am start -W -S com,tencent.mobileqg/.activity.SplashActivity
rem 等待
ping 127.0.0.1 -n 3
rem 点击用户名框
adb shell input tap 471 425
rem 点击删除的x
adb shell input tap 704 437
ping 127.0.0.1 -n 1
rem 输入用户名
adb shell input text 2798145476
ping 127.0.0.1 -n 1
rem 输入密码
adb shell input tap 446 572
ping 127.0.0.1 -n 1
adb shell input text roy123456
rem 点击登录按钮
ping 127.0.0.1 -n 1
adb shell input tap 446 812
```

Appium日志的查看

python自动化脚本如何操作手机终端过程:

1、自动化脚本发送http请求，请求参数:终端设置参数
2、创建会话

3、确认终端设备是否连接，并且确认安卓版本，确认设置参数是否与终端的设置是否一致

4、appium会推送一个包“AppiumBootstrap.jar”到模拟器上 ,相当于api包:appium server指令进行接收，操控手机端

5、相应http请求
6、下一个http请求，循环上述过程

### 五、Appium元素定位工具

1.1 **UIAutomatorView**

Android SDK自带的定位工具
位置: Android SDK/tools/uiautomatorviewer.bat

元素常见几个属性
text	文本
resoure-id:
class: 元素标签
content-desc（或者为description）: 元素功能描述 语音播报

注意：在截屏的时候，必须当前截屏的终端没有其他的进程在占用，包括appium server

或者使用命令重启

adb kill-server

adb start-server

1.2 **Appium Desktop Inspector**

appium server自带的定位工具，需要单独下载

1.3 **Weditor**
Uiautomator2	python第三方库 appUi自动化测试框架

安装:
pip install Uiautomator2

python -m uisutomator2 init

pip install weditor 

确认安装：

weditor --help

启动

weditor 

### 六、Appium界面元素定位方法

(id/ClassName/accessibility/xpath)

1、通过resourceid属性定位 find_element_by_id 返回是WebElement对象
2、通过文本定位 find_element_by_android_uiautomator(“new UiSelector().方法1.()方法2())	组合定位
通过调佣系统自带框架(Uiautomator1/Uiautomator2)实现元素定位，基于java代码编写UiSelector实现元素定位提供很多方法，通过多个属性实现元素定位

3、通过这个content-desc/description属性实现元素定位find_element_by_accessibility_id("content-desc/description属性值”)

4、通过xpath定位 一般不建议采用这个方法来进行对应的定位

元素定位:找出 能定位到元素的 表达式-> 找出表达式
appium继承selenium的定位策略，外加自己定制的策略

driver.find_element(By.ID，"XXXXXX") # selenium的定位策略
driver.find_element(AppiumBy.ID，"XXXXXX") # Appium的定位袋略

1.通用定位策略
ACCESSIBILITY_ID # 无障碍访问ID

	- Android :content-desc
	- IOS: accessibility-id

ID

 - Android: resource-id
 - IOS: name

Class

XPath	脆弱、性能

CUSTON	使用自己窗户就的的定位策略

IMAGE	定位图片的

2.专用的定位策略

为不同平台专用
ANDROID_UIAUTOMATOR	常用UIAUTOMATOR2
ANDROID_VIEWTAG
ANDROID DATA MATCHER

### 七、APP元素的操作

1、APP四大常用元素操作: 点击click() 	send_keys() 	get_attibute() 	text()
2、滑屏、多点触控、长按 ....
3滑屏操作: 左滑 右滑

get_window_size()	返回字典包含width,height

swipe(self: T, start x: int, start y: int, end x: int, end y: int, duration: int = 0):

参数说明:
start_x:开始位置的x坐标
start_y:开始位置的y坐标
endx:结束位置的坐标
end_y:结束位置的y坐标
duration: 延时多少毫秒
左滑:开始位置的x坐标>结束位置的x坐标

开始位置的y坐标= 结束位置的y坐标

el.screenshot(“1.png”)	#元素截图
driver.get_screenshot_as_file("a.png")	#屏幕截图

send_keys(Keys.RETURN)

**定位一闪而过的元素**

1.元素是 android.widget.Toast

2.出现和消失时间不确定

wait = WebDriverWait(driver，5) # 显示等待5秒
num=wait.until(lambda _:driver.find_element(AppiumBy.XPATH,'//android.widget.Toast').text)

print(“获取到的元素为：”,num)

**POM及POM设计原理**
POM page object model 页面对象模型，主要应用于Ui自动化测试框架的搭建，主流设计模式之-页面对象模型:结合面向对象编程思路:把项目的每个页面当做一个对象来进行编程

python基础:什么对象?Python怎么描述一个对象=属性+行为

通过类定义=具有相同属性+相同行为对象集合

POM一般分为四层: 项目=n个页面=base层+pageobject层 (页面1，页面2，页面3，....页面n)

第一层：base层描述每个页面相同的属性及行为

第二层：pageobject层	每个的独有特征及独有的行为
第三层：testcases层	用例层，描述项目业务流程
第四层：testdata	数据层

pom模式设计意义: 层次更加清晰，方便管理以及提供代码的复用性及扩展性
核心技能: 搭建自动化框架(代码复用性、维护性、扩展性)

### 八、查看多开模拟器端口

查看模拟器端口号

**如果不知道模拟器的端口怎么办，例如多开模拟器，怎么查看模拟器端口号是多少？**
首先打开夜神模拟器安装位置的\bin\BignoxVMS目录，多开模拟器会在这个路径创建一个新的目录

进入目录使用记事本打开以下文件，找到guestport=“5555”，而hostport="62025"就是该模拟器的端口

**使用adb连接多开夜神模拟器**

```javascript
adb connect 127.0.0.1:62025
```

进入adb shell后有两种状态显示：#代表有root权限，$代表没有root权限

> ```bash
> root@android:/ #
> shell@mx4:/ $
> ```

```bash
adb install | -r <apkName>  -r 覆盖原安装文件 -s 可以指定设备
adb uninstall  | -k  <apkName>  卸载软件 -k 加 -k 参数,为卸载软件但是保留配置和缓存文件
```

**文件读取写入**

将文件从PC写入到设备

adb push <本地路径> <设备路径>

将文件从设备读取到PC

adb pull <remote>  <local>

注意：由于权限问题，不能直接pull到电脑磁盘根目录，否则会报错

屏幕截图

adb shell screencap /sdcard/screen.png
adb pull /sdcard/screen.png  C:\Users\Shuqing\Desktop

Tips：如果5037端口被占用可以使用如下命令释放端口

```bash
netstat -ano | findstr "5037"
taskkill -f -pid XXX
```