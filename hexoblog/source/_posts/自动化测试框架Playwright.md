---
title: 自动化测试框架Playwright
categories: test
tags: playwright
img: /images/playwright.jpg
abbrlink: 2116
date: 2024-04-19 16:34:13
---

## 自动化测试框架Playwright

### playwright简介

跨浏览器。 Playwright 支持所有现代渲染引擎，包括：Chromium、WebKit 和 Firefox。

跨平台。 适用于 Windows、Linux、macOS、本地运行、 CI、headless 和 headed。

跨语言。 在 TypeScript, JavaScript, Python, .NET, Java 中使用 Playwright API

测试移动端 Web 。 对 Android 和 Mobile Safari 的 Google Chrome 原生移动端模拟。与你的移动端和云端应用采用相同的渲染引擎

自动等待（auto-waits）。 Playwright 能够自动等待元素达到可操作的状态，外加一系列丰富的内置事件。不用再人工定义超时（timeouts） —— 这是测试容易失效的主要原因。

Web 优先的断言。 Playwright 断言专门为动态加载的 web 应用设计。能够在满足需要的条件前自动重试。

可追踪。 通过配置重试策略，采用捕捉执行轨迹、视频、截图来解决测试容易失效的问题。

浏览器在不同进程中运行属于不同来源的 Web 内容。Playwright 与现代浏览器架构保持一致，并在进程外运行测试。这使 Playwright 摆脱了典型的进程内测试运行程序限制。

一切并行。 跨越多个 tabs, 多个 origins 和多个 users 的测试场景。在一个测试中能够为不同的用户创建具有不同上下文的场景，并能在你的服务器上运行。

可信事件。 元素悬停（hover）、动态控件的交互、生产可信事件。Playwright 使用与真实用户一致的输入方式（pipeline）。

测试 frames，穿透 Shadow DOM。 Playwright 的选择器能够穿透 shadowDOM 和允许无缝输入 frame。

浏览器上下文。 Playwright 为每个测试创建一个浏览器上下文。浏览器上下文相当于一个全新的浏览器配置文件。这提供了零开销的完整测试隔离。创建一个新的浏览器上下文只需要几毫秒。

一次登录。 保存上下文的身份验证状态并在所有测试中重用它。这绕过了每个测试中的重复登录操作，但提供了独立测试的完全隔离。

Codegen 。 通过记录您的操作来生成测试。将它们保存为各种语言。

Playwright inspector 。 检查页面，生成选择器，逐步执行测试，查看点击点，浏览执行日志。

Trace Viewer 。 捕获所有的信息来调查失败了的测试，Playwright 追踪包含测试运行截屏视频、实时 DOM 快照、动作浏览器、测试源等信息。

### playwright安装与使用

安装(python3.7+)

```shell
pip install playwright
playwright install-deps  # 安装playwright支持的浏览器的所有依赖
playwright install-deps chromium  # 单独安装chromium驱动的依赖
```


Playwright 安装驱动，相比起selenium就简单多了

```shell
python -m playwright install
playwright install webkit  # 安装webkit
```

Playwright 录制脚本

```shell
python -m playwright codegen
```

winsdows下scripts的cmd中输入：playwright.exe codegen

python -m playwright codegen --help

-o:指定脚本录制输出位置；

--target：指定脚本语言，告诉他用什么语言写，后面一大堆，还有个python-pytest!，默认用python

-b：指定浏览器驱动，默认用的谷歌

例子：

```shell
python -m playwright codegen -o "D:\PythonProject\playwright_project\playwright_code.py"
```

### **终端命令使用**

**保存认证状态**（将会保存所有的cookie和本地存储到文件）
playwright codegen --save-storage=auth.json  # 存储到本地
playwright open --load-storage=auth.json my.web.app  # 打开存储
playwright codegen --load-storage=auth.json my.web.app # 使用存储运行生成代码（保持认证状态）

打开网页，playwright内置了一个跨平台的浏览器webkit内核。

```shell
playwright open www.baidu.com  # 使用默认的chromium打开百度
playwright wk www.baidu.com  # 使用webkit打开百度
```

模拟设备打开

```shell
# iPhone 11.
playwright open --device="iPhone 11" www.baidu.com
# 屏幕大小和颜色主题
playwright open --viewport-size=800,600 --color-scheme=dark www.baidu.com
# 模拟地理位置、语言和时区
playwright open --timezone="Europe/Rome" --geolocation="41.890221,12.492348" --lang="it-IT" maps.google.com
```

**Inspect selectors**，在浏览器调试工具使用命令。下面是一些例子。
playwright.$(selector)  # 定位到匹配的第一个元素
playwright.$$(selector)  # 定位到所有匹配的元素
playwright.selector(element)  # 给指定的元素生成selector
playwright.inspect(selector)  # 在 Elements 面板中显示元素（如果相应浏览器的 DevTools 支持它
playwright.locator(selector)  # 使用playwright内置的查询引擎来查询匹配的节点
playwright.highlight(selector)  # 高亮显示第一个匹配的元素
playwright.clear()  #取消现存的所有的高亮

生成PDF（只有在无头模式下才能运行）

```shell
# 生成PDF
playwright pdf https://en.wikipedia.org/wiki/PDF wiki.pdf
```

### 导航周期

在页面中显示新文档的过程分为导航和加载。导航从更改页面 URL 或与页面交互（例如，单击链接）开始。 导航意图可能会被取消，例如，在点击未解析的 DNS 地址或转换为文件下载时。当响应头被解析并且会话历史被更新时，导航被提交。 只有在导航成功（提交）后，页面才开始加载文档。加载包括通过网络获取剩余的响应体、解析、执行脚本和触发加载事件

```python
#等待
#默认情况下，playwright会自动等待直到触发load事件。
# 同步模式
page.goto("http://www.baidu.com")
# 异步模式
await page.goto("http://www.baidu.com")
#可以重载等待的事件，比如等待直到触发networkidle
page.goto("http://www.baidu.com", wait_until="networkidle")
#也可以为特定的元素添加等待
page.goto("http://www.baidu.com")
page.locator("title").wait_for()   # 等待直到title元素被加载完全
# 会自动等待按钮加载好再执行点击
page.locator("button", has_text="sign up").click() 
#导航栏的功能
page.goto(url, **kwargs) # 定向
page.reload(**kwargs) # 刷新
page.go_back(**kwargs)  # 后退
page.go_forward(**kwargs)  # 前进
```

### 网络

HTTP认证

```python
context = browser.new_context(
	http_credenttials={
		"username": "Mike",
		"password": "123456"
	}
)
page = context.new_page()
```

HTTP全局代理

```python
# 指定代理服务器，和代理所使用的用户和密码
browser = chromium.launch(
	"server": "http://myproxy.com:port",
	"username": "Mike",
	"password": "123456"
)
```

局部代理，为每一个上下文都创建一个代理

```python
browser = chromium.launch(proxy={"server": "per-context"})
context = browser.new_context(proxy={"server": "http://myproxy.com:port"})
```





### 脚本模式

playwright代码的编写，有两种模式：同步模式和异步模式

同步模式就如同selenium

```python
from playwright.sync_api import Playwright, sync_playwright, expect
def run(playwright: Playwright) -> None:
    browser = playwright.chromium.launch(headless=False)
    context = browser.new_context()
    page = context.new_page()
    page.goto("https://www.baidu.com/")
    page.locator("#kw").click()
    page.locator("#kw").fill("csdn")
    page.get_by_role("button", name="百度一下").click()
    context.close()
    browser.close()
with sync_playwright() as playwright:
    run(playwright)
```

异步模式：

```python
import asyncio
from playwright.async_api import Playwright,async_playwright
async def main():
    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=False)
        context = await browser.new_context()
        page = await context.new_page()
        await page.goto("https://www.baidu.com/")
        print(await page.title())
        await context.close()
        await browser.close()
asyncio.run(main())
```

它俩很相似，异步模式主要用了async_playwright的api，主要采用async/await 关键字

### 脚本编写

1、Playwright类
主要用于定义设备的信息，为page服务或者context服务

2、生成浏览器驱动：
browser = playwright.chromium.launch(headless=False)
可支持的浏览器有'chromium', 'webkit' or 'firefox'

launch函数的使用

```python
def launch(
    self,
    *,
    executable_path: typing.Optional[typing.Union[str, pathlib.Path]] = None,
    # 传入一个浏览器可执行的文件必须绑定支持的浏览器
    channel: typing.Optional[str] = None,
    #传入浏览器的版本，有"chrome", "chrome-beta", "chrome-dev", "chrome-canary",
    # "msedge", "msedge-beta", "msedge-dev", "msedge-canary"，msedge是Microsoft edge，还有其他版本可以查看官网
    args: typing.Optional[typing.List[str]] = None,
    #传递给浏览器的其他参数，以列表形式传入
    ignore_default_args: typing.Optional[
        typing.Union[bool, typing.List[str]]
    ] = None,
    # 如果为'true'，Playwright不会传递自己的配置参数，而只使用来自'args'的配置参数。如果数组是
    # 给定，然后过滤掉给定的默认参数。危险选项；小心使用。默认为“False”。
    handle_sigint: typing.Optional[bool] = None,
    # 通过Ctrl+c关闭浏览器的进程，默认开启
    handle_sigterm: typing.Optional[bool] = None,
    # 收到SIGTERM信号正常关闭浏览器进程，默认开启
    handle_sighup: typing.Optional[bool] = None,
    # 收到SIGHUP信号终止浏览器进程，默认开启
    timeout: typing.Optional[float] = None,
    # 等待浏览器实例启动的最长时间（以毫秒为单位）。默认为“30000”（30秒）。传递“0”以禁用超时。
    env: typing.Optional[typing.Dict[str, typing.Union[str, float, bool]]] = None,
    # 指定浏览器可见的环境变量。默认为process. env
    headless: typing.Optional[bool] = None,
    # 是否在没有窗口下运行浏览器，默认开启，开启后你就看不到程序打开浏览器，一般传入False
    devtools: typing.Optional[bool] = None,
    # **仅Chromium**是否为每个选项卡自动打开开发人员工具面板。如果此选项为“真”，则“无头”选项将设置为“false”。
    proxy: typing.Optional[ProxySettings] = None,
    # 网络代理设置
    downloads_path: typing.Optional[typing.Union[str, pathlib.Path]] = None,
    # 如果指定，接受的下载将下载到此目录中。否则，将创建临时目录并当浏览器关闭时被删除。在任何一种情况下，下载都会在它们所在的浏览器上下文关闭时被删除。
    slow_mo: typing.Optional[float] = None,
    # 将Playwright操作减慢指定的毫秒数。这个很有用，以便您可以看到正在发生的事情。
    traces_dir: typing.Optional[typing.Union[str, pathlib.Path]] = None,
    # 如果指定，跟踪将保存到此目录中。
    chromium_sandbox: typing.Optional[bool] = None,
    # 启用Chromium沙盒。默认为False
    firefox_user_prefs: typing.Optional[
        typing.Dict[str, typing.Union[str, float, bool]]
    ] = None
    # 一些火狐偏好设置
) -> "Browser":
```

3、生成浏览器上下文（context）对象
浏览器上下文对象是浏览器实例中一个独立隐身会话，资源、cookie缓存这些不与其他上下文共享，对于自动化多用户、多场景测试非常有意义，相当于打开两个浏览器，两个进程，可以每个测试用例单独开一个上下文，需要注意的是，浏览器上下文最好显式的关闭，也就是

context.close()

```python
# 同步模拟UA
context = browser.new_context(
  user_agent='My user agent'
)
# 异步模拟UA
context = await browser.new_context(
  user_agent='My user agent'
)
# 创建上下文
context = browser.new_context(
  viewport={ 'width': 1280, 'height': 1024 }
)
# 调整page的视图大小
await page.set_viewport_size({"width": 1600, "height": 1200})

```





**context常用操作**

context.pages :获取context所有page对象

context.new_page():生成一个新的page对象

context.close()：关闭context

context.add_cookies()：将cookie添加到此浏览器上下文中。此上下文中的所有页面都将安装这些cookie。

  只能传入列表

  List[{name: str, value: str, url: Union[str, None], domain: Union[str, None], path: Union[str, None], expires: Union[float, None]

context.clear_cookies()：清除context的cookie

context.grant_permissions()：授予浏览器上下文的指定权限，具体见api

context.clear_permissions()：清除授权

4、生成浏览器页面page

```python
page = context.new_page()
```

page就相当于浏览器的选项卡，一般在浏览器context上创建，后续打开网页 ，业绩元素定位，页面操作都是基于page

page的常用参数

```python
	viewport: ViewportSize = None,#为每个页面设置一致的视口。默认为1280x720视口
	screen: ViewportSize = None, # 通过“window.screen”模拟网页内可用的一致窗口屏幕大小 ,只能在viewport设置之后使用  
	noViewport: bool = None,# 不强制固定视口，允许在标题模式下调整窗口大小
    userAgent: str = None, # 设置代理用于上下文
    locale: str = None, #指定用户区域，设置将影响Accept-Language'请求标头值以及数字和日期格式规则
    isMobile: bool = None, #设备相关，不用管
    colorScheme: ColorScheme = None, #Union["dark", "light", "no-preference", "null", None]
    #设置颜色主题
```

page常用操作

page.goto(url,timeout,waitUntil,referer):打开一个新的网页

url传入一个字符串，导航到该地址

timeout 最大等待时间，传入毫秒，默认30s，为0则禁用，不等待

waitUntil:定义触发事件，当定义事件触发将被认为操作成功

  可选["commit", "domcontentloaded", "load", "networkidle", None]

  domcontentloaded：出现【文件加载中】就被认为操作成功

  load：出现【加载】事件就被认为操作成功

  networkidle：当发生【没有网络（网络空闲）】被认为操作成功，超过500ms没有网络就触发了【不建议使用】

  commit：当网络响应被接受，文件开始加载，就认为操作成功了

  referer：可以传入str或者none，引用头部，如果设置就有限用这个作为header

  page.locator():playwright的核心功能之一，元素定位器，常用的元素定位方法在后面单独拎出来讲

元素定位的常用操作
page.locator().click()：点击元素

 page.locator().dblclick()：双击元素，double click

 page.locator().fill()：输入str

click和fill操作只可以直接在页面操作

page.fill('css=[name="username"]',"017575863")
 page.fill('css=[name="password"]',"06523244")

 page.locator().count():返回匹配元素的个数

page.locator().clear()：清空input的内容

page.locator().drag_to()：把一个元素拖到另一个元素位置然后释放

page.locator().frame_locator()：使用iframe时，您可以创建一个帧定位器，该定位器将进入iframe并允许选择iframe中元素。

示例：

```python
locator = page.frame_locator("#my-iframe").get_by_text("Submit")
await locator.click()

locator = page.frame_locator("#my-iframe").get_by_text("Submit")
locator.click()
```

   page.locator().all_text_contents():返回对应元素所有文本内容，基于代码不会格式化，返回的是列表

page.locator().all_inner_texts()：返回对应元素所有文本内容，基于页面，格式化，返回列表

page.locator().get_attribute()：返回匹配元素的属性值

page.locator().wait_for()：等待元素加载，没必要，playwright会自动等待直到触发load事件。

page.locator().filter()：此方法根据选项缩小现有定位器，例如按文本过滤。它可以多次过滤。

```python
await row_locator.filter(has_text="text in column 1").filter(
    has=page.get_by_role("button", name="column 2 button")
).screenshot((path="screenshot.png", full_page=True) #是否开启全截图
```

**`page.frame()`****：**返回指定的frame，必须传入name 或者url**

`page.frame_locator()`：**定位一个frame，并可以切换进去定位元素

```python
locator = page.frame_locator("#my-iframe").get_by_text("Submit")
await locator.click()
```

page.content()：返回页面内容

page.set_default_navigation_timeout()：设置超时默认时间

page.keyboard.press()：键盘操作（直接在page页面），可以输入的键有以下

F1 - F12, Digit0- Digit9, KeyA- KeyZ, Backquote, Minus, Equal, Backslash, Backspace, Tab,Delete, Escape, ArrowDown, End, Enter, Home, Insert, PageDown, PageUp, ArrowRight, ArrowUp,etc.

**`page.screenshot()`**：截屏

**`page.press(selector,key)`**：找到元素，然后调用键盘的按下、放开某个键的操作

```python
page.keyboard.press("Shift+O")
page.press("body", "Shift+O")
```

**`page.close()`**:关闭页面

**`page.wait_for_load_state(state)`**：当达到所需的加载状态时返回。state有["domcontentloaded", "load", "networkidle"]

当页面达到所需的加载状态时，默认为“加载”。

打开新的选项卡

```python
# 可以预期的
with context.expect_page() as new_page_info:
    page.locator('a[target="_blank"]').click() # 打开新的选项卡
new_page = new_page_info.value

new_page.wait_for_load_state()
print(new_page.title())

# 不可以预期的
def handle_page(page):
	page.wait_for_load_state()
	print(page.title())
context.on('page', handle_page)
```





**`page.route()`**：网络劫持，路由提供了修改页面发出的网络请求的能力

```python
page = await browser.new_page()
await page.route(re.compile(r"(\.png$)|(\.jpg$)"), lambda route: route.abort())
await page.goto("https://example.com")
await browser.close()
```

劫持响应结果

我们要让百度详情结果为我们的定义的页面：custom_response.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>hack test</title>
</head>
<body>
    <h1>hack test</h1>
</body>
</html>
```

```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    page = browser.new_page()
    page.route('**/*',lambda route: route.fulfill(path='./custom_response.html'))
    page.goto("https://www.baidu.com")
    browser.close()
```

page.expect_download()：下载文件

```python
with page.expect_download() as download_info:
#点击下载的按钮
    page.locator("button#delayed-download").click()
download = download_info.value
#等待下载
print(download.path())
#保存文件
download.save_as("/path/to/save/download/at.txt")
```

**执行javascript**

```python
page.evaluate(expression, **kwargs)
# expression: javascript脚本
# kwargs: 参数（可序列化对象，JSHandle实例，ElementHandle实例）
#不带参数
href = page.evaluate('() => document.location.href')  # 同步
href = await page.evaluate('() => document.location.href')  # 异步
# 一个值
page.evaluate('num => num', 32)
# 一个数组
page.evaluate('array => array.length', [1,2,3])
# 一个对象
page.evaluate('object => object.foo', {'foo': 'bar'})
# jsHandler
button = page.evaluate('window.button')
page.evaluate('button => button.textContent', button)

# 使用elementhandler.evaluate的方法替代上面的写法
button.evaluate('(button, from) => button.textContent.substring(from)', 5)

# 对象与多个jshandler结合
button1 = page.evaluate('window.button1')
button2 = page.evaluate('.button2')
page.evaluate("""o => o.button1.textContent + o.button2.textContent""",
    { 'button1': button1, 'button2': button2 })

# 使用析构的对象，注意属性和变量名字必须匹配。
page.evaluate("""
    ({ button1, button2 }) => button1.textContent + button2.textContent""",
    { 'button1': button1, 'button2': button2 })

# 数组析构也可以使用，注意使用的括号
page.evaluate("""
    ([b1, b2]) => b1.textContent + b2.textContent""",
    [button1, button2])

# 可序列化对象的混合使用
page.evaluate("""
    x => x.button1.textContent + x.list[0].textContent + String(x.foo)""",
    { 'button1': button1, 'list': [button2], 'foo': None })
```

写断言（expect）
Playwright Test 使用 expect 库进行测试断言。它提供了很多匹配器， toEqual, toContain, toMatch, toMatchSnapshot 等等。

将 expect 与各种 Playwright 方法相结合，来实现测试目标：

page.isVisible(selector[, options])#
page.waitForSelector(selector[, options])#
page.textContent(selector[, options])#
page.getAttribute(selector, name[, options])#
page.screenshot([options])#

[官网断言](https://playwright.dev/docs/test-assertions)



### **定位方法**

playwright定位主要通过slector选择器定位支持文本定位、css定位、xpath定位、组合定位

文本选择器

```python
page.click('text=登录') # 没有加引号(单引号或者双引号)，模糊匹配，对大小写不敏感
 page.click('text="登录"')# 有引号，精确匹配，对大小写敏感
```


has_text()查找子代或后代所有包含对应文本的

相反的也有has_not_text（）

text（）查找第一个文本等于...的元素

比如：<article><div >Playwright</div></article>

```python
page.locator(':has_text("Playwright")').click()
# 也可以这样写，指定标签
page.locator('article:has_text("Playwright")').click()
# 也可以这样
page.locator(":text('Playwright')").click()
```

css选择器

如果 `元素2` 是 `元素1` 的 直接子元素， CSS Selector 选择子元素的语法是这样的

元素1 > 元素2

如果 元素2 是 元素1 的 后代元素， CSS Selector 选择后代元素的语法是这样的


元素1   元素2

css 选择器支持通过任何属性来选择元素，语法是用一个方括号 `[]`

比如要选择上面的a元素，就可以使用 `[href="http://www.miitbeian.gov.cn"]`

比如， 要选择a节点，里面的href属性包含了 miitbeian 字符串，就可以这样写

a[href*="miitbeian"]

选择 属性值 以某个字符串 `开头` 的元素

比如， 要选择a节点，里面的href属性以 http 开头 ，就可以这样写

a[href^="http"]

 选择 属性值 以某个字符串 `结尾` 的元素

比如， 要选择a节点，里面的href属性以 gov.cn 结尾 ，就可以这样写

a[href$="gov.cn"]

如果一个元素具有多个属性

<div class="misc" ctype="gun">沙漠之鹰</div>

CSS 选择器 可以指定 选择的元素要 同时具有多个属性的限制，像这样 `div[class=misc][ctype=gun]`

选择所有 `id 为 t1 里面的 span` 和 `所有的 p 元素`

不能这样写：#t1 > span,p

只能这样写

```css
#t1 > span , #t1 > p
```

选择的是 第2个子元素，并且是span类型

所以这样可以这样写 `span:nth-child(2)`

就是选择第倒数第1个子元素，并且是p元素

```css
p:nth-last-child(1)
```

选择的是 `第1个span类型` 的子元素

所以也可以这样写 `span:nth-of-type(1)`

如果要选择的是父元素的 `偶数节点`，使用 `nth-child(even)`

比如

```css
p:nth-child(even)
```

如果要选择的是父元素的 `奇数节点`，使用 `nth-child(odd)`

```css
p:nth-child(odd)
```

 h3 `后面紧跟着的兄弟节点` span。

这就是一种 相邻兄弟 关系，可以这样写 `h3 + span`

如果要选择是 选择 h3 `后面所有的兄弟节点` span，可以这样写 `h3 ~ span`

```python
page.locator('button').click() # 根据标签
page.locator('#nav-bar .contact-us-item').click() # 通过id +class
page.locator('[data-test=login-button]').click() # 属性定位
page.locator("[aria-label='Sign in']").click()
#css定位前面可以加个css=，但是不加也可以程序也会自动匹配
```

```python
<span class="chinese student">张三</span>
page.locator('.chinese')
#或者
page.locator('.student')
#如果要表示同时具有两个class 属性，可以这样写
page.locator('.chinese.student')
```



Xpath 定位

```python
page.locator("xpath=//button").click()
```

组合定位：text、css、xpath三者可以两组合定位

css+text组合定位

```python
page.locator("article:has-text('Playwright')").click()
page.locator("#nav-bar :text('Contact us')").click()
```

css+css组合定位

```python
page.locator(".item-description:has(.item-promo-banner)").click()
```

Xpath + css 组合定位

```python
page.fill('//div[@class="SignFlow-account"] >>css=[name="username"]',"0863")
```

xpath+xpath组合定位

```python
page.fill('//div[@class="SignFlowInput"] >> //input[@name="password"]',"ma160065")
```

nth选择

`nth` 方法，获取指定次序的元素，参数0表达第一个， 1 表示第2个

```python
lct = page.locator('.plant')
print(lct.nth(1).inner_text())
# Click first button
page.locator("button >> nth=0").click()
# Click last button
page.locator("button >> nth=-1").click()
```

playwright推荐的内置定位

page.get_by_text()通过文本内容定位。

page.get_by_label()通过关联标签的文本定位表单控件。page.get_by_placeholder()按占位符定位输入。

page.get_by_title()通过标题属性定位元素。

page.get_by_role()通过显式和隐式可访问性属性进行定位。page.get_by_alt_text()通过替代文本定位元素，通常是图像。page.get_by_test_id()根据data-testid属性定位元素(可以配置其他属性)

### `ARIA `

`（Accessible Rich Internet Applications）`。

ARIA 根据web界面元素的用途，为这些元素定义了一套 [角色（ Role ）](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles) 信息，添加到页面中

```html
<progress value="75" max="100">75 %</progress>
就等于隐含了如下信息
<progress value="75" max="100"
  role="progressbar"
  aria-valuenow="75"
  aria-valuemax="100">75 %</div>
```

```python
# 根据 role 定位
lc = page.get_by_role('progressbar')
# 打印元素属性 value 的值
print(lc.get_attribute('value'))
```

```html
<h1>白月黑羽标题1</h1>
<h2>白月黑羽标题2</h2>
```

`h1` 隐含了 `role="heading" aria-level="1"` 属性

`h2` 隐含了 `role="heading" aria-level="2"` 属性

h2 元素，隐含了 `role="heading" aria-level="2"`， 所以可以用下面代码定位

```python
lc = page.get_by_role('heading',level=2)
print(lc.inner_text())
```

html 元素标准属性 `name` 是浏览器内部的，用户看不到，比如

```html
<a name='link2byhy' href="https://www.byhy.net">教程</a>
```

`Accessible Name` 不一样，它是元素界面可见的文本名，

比如上面的元素，暗含的 `Accessible Name` 值就是 `教程` ， 当然也暗含了 `ARIA role` 值为 `link`

所以，可以这样定位

```python
lc = page.get_by_role('link',name='教程')
print(lc.click())
```

Accessible Name 和 参数name 的内容完全一致，可以指定 `exact=True`

name值还可以通过[正则表达式](https://www.byhy.net/py/lang/extra/regex/)，进行较复杂的匹配规则，比如

```python
lc = page.get_by_role('link',name=re.compile("^教"))
```

**常见的**：

<a> <td> <button> Accessible Name 值 就是其内部的文本内容。

`<textarea> <input>` 这些输入框，它们的 Accessible Name 值 是和他们关联的 的文本

一些元素 比如 `<img>` ，它的 Accessible Name 是其html 属性 `alt` 的值

比如

```html
<img src="grape.jpg" alt="banana"/>
```

它的 Accessible Name 值为 `banana` ，role 为 `img`

### 视觉定位

```html
<input type="text" placeholder="captcha" />
```

就可以这样定位

```python
page.get_by_placeholder('captcha',exact=True).fill('白月黑羽')
```

参数 `exact` 值为 `True` ，表示完全匹配，且区分大小写。如果值为False，就只需包含参数字符串即可，且不区分大小写

```html
  <input aria-label="Username">
  <label for="password-input">Password:</label>
  <input id="password-input">
```

就可以这样定位

```python
page.get_by_label("Username").fill("john")
page.get_by_label("Password").fill("secret")
```

```html
<img 
  src="https://doc.qt.io/qtforpython/_images/windows-pushbutton.png" 
  alt="qt-button">
```

就可以这样定位

```python
href = page.get_by_alt_text("qt-button").get_attribute('src')
print(href)
```

```html
  <a href="https://www.byhy.net" title="byhy首页">教程</a>
```

就可以这样定位

```python
page.get_by_title("byhy首页").click()
```

Playwright不需要我们额外设置，本来元素操作时，如果根据定位规则找不到元素，就会等待最多 30秒

想修改 `缺省` 等待时间， 可以使用 `BrowserContext` 对象， 如下

```python
from playwright.sync_api import sync_playwright
p = sync_playwright().start()
browser = p.chromium.launch(headless=False, slow_mo=50)
context = browser.new_context()
context.set_default_timeout(10) #修改缺省等待时间为10毫秒
page = context.new_page() # 通过context 创建Page对象
page.goto("https://www.byhy.net/_files/stock1.html")
page.locator('#kw').fill('通讯\n')
page.locator('#go').click()
```

**Handles（句柄）**

Playwright 可以为页面 DOM 元素或页面内的任何其他对象创建句柄。 

JSHandle：该浏览页中，任意的javascript对象的引用。
ElementHandle: 该浏览页中，任意DOM元素的引用。
实际上所有的DOM元素也是javascript对象，所有ElementHandle也是JSHandle

```python
# 获取JSHandle
js_handle = page.evaluate_handle('window')
# 获取ElementHandle
element_handle = page.wait_for_selector('#box')
# 可以使用assert方法来检测元素的属性
bounding_box = element_handle.bounding_box
assert bounding_box.width == 100
```

locator和elementHandle的区别

locator保存的是元素捕获的逻辑。
elementHandle指向特定的元素。
如果一个elementHandle所指向的元素的文本或者属性发生改变，那么elementHandle仍然指向原来那个没有改变的元素，而locator每一个都会根据捕获的逻辑去获取最新的那个元素，也就是改变后的元素

### 界面操作

获取元素内部的整个html文本， 可以使用 Locator对象的 [inner_html](https://playwright.dev/python/docs/api/class-locator#locator-inner-html) 方法

如果要 `双击` ，可以使用 [dblclick](https://playwright.dev/python/docs/api/class-locator#locator-dblclick) 方法

让光标悬停在某个元素上方，可以使用 Locator对象的 [hover](https://playwright.dev/python/docs/api/class-locator#locator-hover) 方法

需要根据当前界面中，`是否存在某些内容` ，来决定下一步操作。

使用 Locator对象的 [is_visible](https://playwright.dev/python/docs/api/class-locator#locator-is-visible) 方法

```python
page.locator("#source").is_visible()
```

该方法不会等待元素出现，而是立即返回 True 或 False 

```html
<input type="file" multiple="multiple">
```

要设置选中的文件，可以使用 Locator 对象的 [set_input_files](https://playwright.dev/python/docs/api/class-locator#locator-set-input-files) 方法

如果要获取输入框 `<input>` ， `<textarea>` 对应的用户输入文本内容，不能用 `inner_text()` 方法。而是应该用 [input_value](https://playwright.dev/python/docs/api/class-locator#locator-input-value) 方法

```python
# 先定位
lc = page.locator('input[type=file]')
# 单选一个文件
lc.set_input_files('d:/1.png')
# 或者 多选 文件
lc.set_input_files(['d:/1.png', 'd:/2.jpg'])
```



常见的选择框包括： radio框、checkbox框、select框

 radio框、checkbox框

 可以使用 Locator对象的 [check](https://playwright.dev/python/docs/api/class-locator#locator-check) 方法

 可以使用 Locator对象的 [uncheck](https://playwright.dev/python/docs/api/class-locator#locator-uncheck) 方法

可以使用 Locator对象的 [is_checked](https://playwright.dev/python/docs/api/class-locator#locator-is-checked) 方法

可以使用 `select` 元素对应的 Locator对象的 [select_option](https://playwright.dev/python/docs/api/class-locator#locator-select-option) 方法

```python
# 根据 索引 选择， 从0 开始， 但是为0的时候，好像有bug
page.locator('#ss_single').select_option(index=1)
# 根据 value属性 选择
page.locator('#ss_single').select_option(value='老师')
# 根据 选项文本 选择
page.locator('#ss_single').select_option(label='老师')
# 清空所有选择
page.locator('#ss_single').select_option([])
```

通过css selector 表达式 `:checked` 伪选择 获取所有选中的 select选项

比如：

```python
page.locator('#ss_multi').select_option(['小江老师','小雷老师'])
lcs = page.locator('#ss_multi option:checked').all_inner_texts()
print(lcs)
```

要 `打开网址/刷新/前进/后退` ， 可以分别调用 Page 对象的 `goto/reload/go_back/go_forward` 方法

`BrowserContext` 对象有个 `pages` 属性，这是一个列表，里面依次为所有窗口对应Page对象

如果当前界面有很多窗口，要把某个窗口作为当前活动窗口显示出来，可以调用该窗口对应的Page对象的 [bring_to_front](https://playwright.dev/python/docs/api/class-page#page-bring-to-front) 方法

**冻结窗口**

在 开发者工具栏 console 里面执行如下js代码

```bash
setTimeout(function(){debugger}, 5000)
```

对下面这段 html

```html
<span id="t1">t1</span>
<span id="t2">t2</span>
<form >
  <div id="captcha">
  </div>
  <input type="text" placeholder="captcha" />
  <button type="submit">Submit</button>
</form>
```

要选中 `span#t1` 文本内容，并且拖拽到 输入框 `[placeholder="captcha"]` 里面去，可以使用如下代码：

```python
# 选中  `span#t1`  文本内容
page.locator('#t1').select_text()
# 拖拽到 输入框  `[placeholder="captcha"]` 里面去
page.drag_and_drop('#t1', '[placeholder="captcha"]')
```

如果被拖动元素的Locator对象已经产生，可以直接调用其 [drag_to](https://playwright.dev/python/docs/api/class-locator#locator-drag-to) 方法 进行拖动

上例中，代码也可以这样写

```python
# 选中  `span#t1`  文本内容
lc = page.locator('#t1')
lc.select_text()
# 拖拽到 输入框  `[placeholder="captcha"]` 里面去
lc.drag_to(page.locator('[placeholder="captcha"]'))
```

注意， drag_to 的参数是 目标元素的 Locator 对象 ， 而不是 selector 表达式字符串

弹出的对话框有三种类型，分别是 `alert（警告信息）` 、 `confirm（确认信息）` 和 `prompt（提示输入）`

`accept` 方法作用等同于点击确定按钮

`dismiss` 方法作用等同于点击取消按钮

`message` 属性就是对话框界面的提示信息字符串

```python
from playwright.sync_api import sync_playwright
pw = sync_playwright().start()
browser = pw.chromium.launch(headless=False)
page = browser.new_page()
page.goto("https://cdn2.byhy.net/files/selenium/test4.html")

# 处理 弹出对话框 的 回调函数
def handleDlg(dialog):
    # 等待1秒
    page.wait_for_timeout(1000)
    # 点击确定
    dialog.accept()
    # 打印 弹出框 提示信息
    print(dialog.message) 
# 设置弹出对话框事件回调函数
page.on("dialog", handleDlg )
# 点击 alert 按钮
page.locator('#b1').click()
```



### 跟踪（tracing）

```python
from playwright.sync_api import sync_playwright

p = sync_playwright().start()
browser = p.chromium.launch(headless=False)

# 创建 BrowserContext对象
context = browser.new_context()
# 启动跟踪功能
context.tracing.start(snapshots=True, sources=True, screenshots=True)

page = context.new_page()
page.goto("https://www.byhy.net/_files/stock1.html")
# 结束跟踪
context.tracing.stop(path="trace.zip")
browser.close()
p.stop()
```

当前工作目录下面多了 trace.zip 这个跟踪数据文件。

怎么查看这个跟踪文件呢？有2种方法：

- 直接访问 [trace.playwright.dev](https://trace.playwright.dev/) 这个网站，上传 跟踪文件
- 执行命令 `playwright show-trace trace.zip`



### **事件监听**

Page 对象提供了一个 on 方法，它可以用来监听页面中发生的各个事件，比如 close、console、load、request、response 等等

```python
from playwright.sync_api import sync_playwright
def run(playwright):
    chromium = playwright.chromium
    browser = chromium.launch()
    page = browser.new_page()
    # 监听请求和响应事件
    page.on("request", lambda request: print(">>", request.method, request.url))
    page.on("response", lambda response: print("<<",response.status,response.url))
    page.goto("https://example.com")
    browser.close()
with sync_playwright() as playwright:
    run(playwright)
```

```python
# 可预期监听请求
with page.expect_request('**/*login*.png') as first:
	page.goto('http://www.baidu.com')
print(first.value.url)

# 异步模式
async with page.expect_request('**/*login*.png') as first:
	await page.goto('http://www.baidu.com')
first_request = await first.value
print(first_request.url)
————————————————
#一次性监听器
page.once('dialog', lambda dialog: dialog.accept('2022'))
page.evaluate('prompt("Enter a number: ")')
```





### 移动端浏览器支持

移动端支持Safari 浏览器、谷歌、不支持火狐，可以传入的设备有iPhone和Pixel 2 （Pixel 2 是谷歌的一款安卓手机）

示例：模拟打开 iPhone 12 Pro Max 上的 Safari 浏览器，然后手动设置定位，并打开百度地图并截图

故宫的经纬度是 39.913904, 116.39014，我们可以通过 geolocation 参数传递给 Webkit 浏览器并初始化

```python
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    iphone_12_pro_max = p.devices['iPhone 12 Pro Max']
    browser = p.webkit.launch(headless=False)
    context = browser.new_context(
        **iphone_12_pro_max,
        locale='zh-CN',
        geolocation={'longitude': 116.39014, 'latitude': 39.913904},
        permissions=['geolocation']
    )
    page = context.new_page()
    page.goto('https://amap.com') 	page.wait_for_load_state(state='networkidle')
    page.screenshot(path='location-iphone.png')
    browser.close()
```

### 处理新的窗口、弹窗，iframe

selenium处理iframe比较麻烦，但是playwright就比较简单，有不同方法

第一种，直接定位一个frame，在frame基础上操作

```python
# ********同步*********
# Get frame using the frame's name attribute
frame = page.frame('frame-login')
 
# Get frame using frame's URL
frame = page.frame(url=r'.*domain.*')
 
# Interact with the frame
frame.fill('#username-input', 'John') 
# *********异步***********
# Get frame using the frame's name attribute
frame = page.frame('frame-login')
# Get frame using frame's URL
frame = page.frame(url=r'.*domain.*')
# Interact with the frame
await frame.fill('#username-input', 'John')
```

第二种，直接定位到frame再定位到上面的元素，在元素基础上操作

示例：

```python
username = page.frame_locator('.frame-class').locator('#username-input')
username.fill('jonas')
```

处理弹窗，一般注册、或者点击一些按钮容易出现弹窗，我们可以利用page.expect_popup()来获取新窗口的iframe

示例：

```python
#可预期的
with page.expect_popup() as popup_info:
        # iframe中的id如果是动态的，所以我们只匹配关键字
        page.frame_locator("iframe[id^=x-URS-iframe]").locator("text=注册新帐号").click()
    register_page = popup_info.value
    # 点击邮箱地址输入框
  register_page.locator("input[id=\"username\"]").click()
    # 输入邮箱
  register_page.locator("input[id=\"username\"]").fill("TesterRoad")
    # 点击密码输入框
  register_page.locator("input[id=\"password\"]").click()
    # 输入密码
 register_page.locator("input[id=\"password\"]").fill("TesterRoad@126")

# 不可以预期的
def handle_popup(popup):
	popup.wait_for_load_state()
	print(popup.title())
page.on('popup', handle_popup)
page.remove_listener("request", handle_popup) # 移除事件监听

```





### 正则表达式

“^”表示匹配以某个模式开头的字符串，但是如果在模式外使用它，就可以表示取反

```txt
使用“(?!)”符号 在正则表达式中，“(?!)”符号表示负向零宽断言，它表示匹配后面不符合某个条件的字符串。例如，正则表达式“w+(?!d)”将匹配不以数字结尾的所有单词。

使用“[^ ]”符号 在正则表达式中，“[^ ]”符号表示不包含某个字符的字符串，其中“^”符号表示取反操作。例如，正则表达式“[^abc]”将匹配不包含字母“a”、“b”或“c”的所有字符串

```

正则表达式’^abc’将匹配以"abc"开头的字符串

“\b”表示单词边界，“\w+”表示匹配一个或多个单词字符，即字母、数字或下划线

“.”将匹配任何字符，“*”将匹配前面的字符零次或多次

“+”将匹配前面的字符一次或多次，“{n}”将匹配前面的字符恰好n次

“(abc)”将匹配“abc”并将其保存为捕获组

’\babc’将匹配以"abc"开头的字符串，但不会匹配"zabc"中的"abc"

### 参考：

[链接](https://blog.csdn.net/m0_72418211/article/details/132867964)

[链接1](https://blog.csdn.net/m0_51156601/article/details/126886040)

