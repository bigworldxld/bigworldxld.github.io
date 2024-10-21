---
title: chrome浏览器窗口（高级版）
categories: tools
img: /images/requestium.jpg
tags:
  - requestium
  - network
  - chrome
abbrlink: 23024
date: 2024-04-05 16:02:30
---

##  chrome浏览器窗口（高级版）

### 查看浏览器信息

在 chrome浏览器的地址栏中输入 `chrome://version`

到 **chrome浏览器** 安装的文件夹下，打开 **cmd窗口**，输入以下内容：

```python
chrome.exe --remote-debugging-port=9527 --user-data-dir="C:\Users\21173\AppData\Local\Google\Chrome\User Data"
```

启动 **chrome浏览器** 的调试模式，

- **user-data-dirr=“F:\selenium\AutomationProfile”** 是在单独的配置文件中启动 **chrome浏览器**，可以是chrome安装目录下现有的用户数据文件,使用的话就可以跳过一些页面的登录状态；也可以理解为新的浏览器，创建对应文件夹；
- 其中 **9527** 为端口号，可自行指定

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
options = Options()
options.add_experimental_option("debuggerAddress", "127.0.0.1:9527")
browser = webdriver.Chrome(options=options)

url = 'https://www.bilibili.com'
browser.get(url)
print(browser.title)	
```

```python
    option = webdriver.ChromeOptions()
    option.add_argument('--headless')
    option.add_argument("--user-data-dir=C:\\Users\\21173\\AppData\\Local\\Google\\Chrome\\User Data")
    option.add_argument("--remote-debugging-port=9222")
    option.add_argument("--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36")
    driver = webdriver.Chrome(options=option)
    driver.get(url)
```



### 执行cmd命令

使用os.popen即可执行cmd命令~（os.popen包装了sunprocess.Popen方法）

```python
import os
os.popen('start chrome --remote-debugging-port=9527 --user-data-dir="F:\selenium"')

# 方法二
# 先切换到chrome的可执行文件路径，再执行cmd命令。注意这里没有 start
os.chdir(r"C:\Program Files\Google\Chrome\Application")
os.popen('chrome --remote-debugging-port=9527 --user-data-dir="F:\selenium"')
```

os.system

```python
import os
# 方法一
os.system(r'start chrome --remote-debugging-port=9527 --user-data-dir="F:\selenium"')

# 方法二
# 先切换到chrome的可执行文件路径，再执行cmd命令。注意这里没有 start
os.chdir(r"C:\Program Files\Google\Chrome\Application")
os.system(r'chrome --remote-debugging-port=9527 --user-data-dir="F:\selenium"')
```

subprocess.Popen

```python
import os
import subprocess

# 先切换到chrome可执行文件的路径
os.chdir(r"C:\Program Files\Google\Chrome\Application")
# 然后使用Popen执行cmd命令，这里的chrome.exe 可替换为 chrome，注意这里没有 start
subprocess.Popen('chrome.exe --remote-debugging-port=9527 --user-data-dir="F:\selenium"')
```

### selenium操作chrome调试

[Chrome Devtools Protocal](https://chromedevtools.github.io/devtools-protocol/tot/Network/)

```python
from selenium import webdriver
if __name__ == '__main__':
    browser = webdriver.Chrome()
    browser.get('https://www.csdn.net/')
    # 获取远程链接的地址
    print('remote_url:', browser.caps['goog:chromeOptions']['debuggerAddress'])
    print('session_id:', browser.session_id)
    print(browser.title)
```

用调试模式执行以上代码，看到下图

- **`{'debuggerAddress': 'localhost:64829'}`** 

查看当前窗口的页面连接

http://localhost:64829/json

点击 `devtoolsFrontendUrl(开发工具前端Url)`，可以到调试界面，这个可以用作于远程调试

查看窗口远程链接

http://localhost:64829/json/version

- 圈出来的`webSocketDebuggerUrl(调试器Url)`，是远程链接的地址，若使用 **puppeteer**的话能用到

```python
import os
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
if __name__ == '__main__':
    os.system(r'start chrome --remote-debugging-port=9527 --user-data-dir="F:\selenium"')	
	# 完成登录扫码验证或者手机短信验证后，回车进入下一步
    input('输入空格继续程序...')
    options = Options()
  options.add_experimental_option("debuggerAddress", "127.0.0.1:9527")
    browser = webdriver.Chrome(options=options)
    print(browser.title)
```

### 获取Network数据

通过selenium去拿网页数据，往往是两个途径：

1. selenium.page_source，通过解析HTML
2. 通过中间人进行数据截获

应用场景

> Chrome浏览器-> 开发者工具 -> Network 中所有的数据包

Selenium3获取Network

> **这里指定9527端口打开浏览器，也可以不指定**

Selenium4获取Network

> 在Selenium 4中，`desired_capabilities` 已经被弃用，所以不能再在`webdriver.Chrome()`中使用它。需要将 `desired_capabilities` 转换为 `options`

日志没有response.body

步骤：通过Chromedriver获取到network日志。network 的response日志类型Network.responseReceived，里面有个参数 requestId ， chrome devtool Network.getResponseBody方法可以通过requestId 获取到response

```json
{'level':'INFO','message':
{
    "message":{
        "method":"Network.responseReceived",
        "params":{
            "frameId":"35489C704F5D95B9D01BD99C00BA3AFA",
            "loaderId":"BA3FE8EAD2BB30562343F3940AEB8CBC",
            "requestId":"BA3FE8EAD2BB30562343F3940AEB8CBC",
            "response":{
                "connectionId":12,
                "connectionReused":false,
                "encodedDataLength":230,
                "fromDiskCache":false,
                "fromPrefetchCache":false,
                "fromServiceWorker":false,
                "headers":{
                    "Access-Control-Allow-Credentials":"true",
                    "Access-Control-Allow-Origin":"*",
                    "Connection":"keep-alive",
                    "Content-Length":"571",
                    "Content-Type":"application/json",
                    "Date":"Tue, 24 Nov 2020 01:59:03 GMT",
                    "Server":"gunicorn/19.9.0"
                },
            	"headersText":"HTTP/1.1 200 OK,
                	Date: Tue, 24 Nov 2020 01:59:03 GMT
                	Content-Type: application/json
					Content-Length: 571
					Connection: keep-alive
					Server: gunicorn/19.9.0
					Access-Control-Allow-Origin: *
					Access-Control-Allow-Credentials: true",
                "mimeType":"application/json",
                "protocol":"http/1.1",
                "remoteIPAddress":"3.211.1.78",
                "remotePort":80,
                "requestHeaders":{           	       "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding":"gzip, deflate",
                    "Accept-Language":"en-US",
                    "Connection":"keep-alive",
                    "Host":"httpbin.org",
                    "Upgrade-Insecure-Requests":"1",
                    "User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:55.0) Gecko/20100101 Firefox/55.0"
                },
                "requestHeadersText":"GET /get HTTP/1.1
					Host: httpbin.org
					Connection: keep-alive
					Upgrade-Insecure-Requests: 1
					User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:55.0) Gecko/20100101 Firefox/55.0
			Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
					Accept-Encoding: gzip, deflate
					Accept-Language: en-US
",
                "responseTime":1606183143651.414,
                "securityState":"insecure",
                "status":200,
                "statusText":"OK",
                "timing":{
                    "connectEnd":282.386,
                    "connectStart":47.972,
                    "dnsEnd":47.972,
                    "dnsStart":46.748,
                    "proxyEnd":-1,
                    "proxyStart":-1,
                    "pushEnd":0,
                    "pushStart":0,
                    "receiveHeadersEnd":519.16,
                    "requestTime":43961290.624483,
                    "sendEnd":282.616,
                    "sendStart":282.547,
                    "sslEnd":-1,
                    "sslStart":-1,
                    "workerFetchStart":-1,
                    "workerReady":-1,
                    "workerRespondWithSettled":-1,
                    "workerStart":-1
                },
               "url":"http://httpbin.org/get"
            },
            "timestamp":43961291.145069,
            "type":"Document"
        }
    },
 "webview":"35489C704F5D95B9D01BD99C00BA3AFA"
}}
```

过滤请求包，一般来说，像图片、css&js文件等，往往是不需要的，所以可以对它们过滤

最后是获取数据包的 requestId，好比每个网络数据包的身份证。
在Selenium中调用cdp时候，需要传入 requestId，浏览器会验证是否存在该 requestId，

如果存在，则响应并返回数据；
如果不存在，则会抛出 WebDriverException 异常

```python
import json
from selenium import webdriver
from selenium.common.exceptions import WebDriverException
from selenium.webdriver.chrome.options import Options
options = Options()
caps = {
    "browserName": "chrome",
    'goog:loggingPrefs': {'performance': 'ALL'}  # 开启日志性能监听
}
# 将caps添加到options中
for key, value in caps.items():
    options.set_capability(key, value)
# 启动浏览器
browser = webdriver.Chrome(options=options)
browser.get('https://www.baidu.com')
def filter_type(_type: str):
    types = [
        'application/javascript', 'application/x-javascript', 'text/css', 'webp', 'image/png', 'image/gif',
        'image/jpeg', 'image/x-icon', 'application/octet-stream'
    ]
    if _type not in types:
        return True
    return False
performance_log = browser.get_log('performance')  # 获取名称为 performance 的日志
for packet in performance_log:
    message = json.loads(packet.get('message')).get('message')  # 获取message的数据
    if message.get('method') != 'Network.responseReceived':  # 如果method 不是 responseReceived 类型就不往下执行
        continue
    packet_type = message.get('params').get('response').get('mimeType')  # 获取该请求返回的type
    if not filter_type(_type=packet_type):  # 过滤type
        continue
    requestId = message.get('params').get('requestId')  # 唯一的请求标识符。相当于该请求的身份证
    url = message.get('params').get('response').get('url')  # 获取 该请求  url
    try:
        resp = browser.execute_cdp_cmd('Network.getResponseBody', {'requestId': requestId})  # selenium调用 cdp
        print(f'type: {packet_type} url: {url}')
        print(f'response: {resp}')
    except WebDriverException:  # 忽略异常
        pass
```

注意事项：

+ CDP主要关注**HTTP**和**HTTPS**请求，其它协议的可能无法捕获
+ 页面加载过程中的限制或**动态加载的内容**，等待一段时间，等页面完全加载好，再获取 CDP 日志
+ **资源策略和跨域限制**

### **Python字符串前缀u、r、b、f含义**

**1、字符串前加 u**

**例子：**

u"字符串中有中文"

**含义：**

前缀u表示该字符串是unicode编码，Python2中用，用在含有中文字符的字符串前，防止因为编码问题，导致中文出现乱码。另外一般要在文件开关标明编码方式采用utf8。Python3中，所有字符串默认都是unicode字符串。

**2、字符串前加 r**

**例子：**

r"\n\t"

**含义：**

在普通字符串中，反斜线是转义符，代表一些特殊的内容，如换行符\n。

前缀r表示该字符串是原始字符串，即\不是转义符，只是单纯的一个符号。

常用于特殊的字符如换行符、正则表达式、文件路径。

注意不能在原始字符串结尾输入反斜线，否则Python不知道这是一个字符还是换行符(字符串最后用\表示换行)，会报错：

SyntaxError: EOL while scanning string literal

 那如果是一个文件夹路径就是以\结尾怎么办呢，可以再接一个转义\的字符串：

```python
>>>print r'C:\Program File\my\path''\\'
C:\Program File\my\path\
```

**3、字符串前加 b**

**例子：**

```
b'<h1>Hello World!</h1>'
```

**含义：**

前缀b表示该字符串是bytes类型。用在Python3中，Python3里默认的str是unicode类。Python2的str本身就是bytes类，所以可不用。
常用在如网络编程中，服务器和浏览器只认bytes类型数据。如：send 函数的参数和 recv 函数的返回值都是 bytes 类型。
在 Python3 中，bytes 和 str 的互相转换方式是

```python
str.encode('utf-8')
bytes.decode('utf-8')
```

### 频繁更换ip

```python
from selenium import webdriver
from selenium.webdriver.common.desired_capabilities  import DesiredCapabilities
from selenium.webdriver.common.proxy import Proxy
from selenium.webdriver.common.proxy import ProxyType
option = webdriver.ChromeOptions()
option.add_argument('--no-sandbox')
option.add_argument('--headless')
#设置user-agent
option.add_argument('user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:55.0) Gecko/20100101 Firefox/55.0')
caps = DesiredCapabilities.CHROME
driver = webdriver.Chrome(options=option,desired_capabilities=caps)
p = '222.185.28.38:32374'
proxy = Proxy(
    {
        'proxyType':ProxyType.MANUAL,
        'httpProxy':'{}'.format(p),
    }
    )
proxy.add_to_capabilities(caps)
driver.start_session(caps)
driver.get('http://httpbin.org/get')
print(driver.page_source)
#更换proxy后 重新执行proxy.add_to_capabilities(caps) 和 driver.start_session(caps)即可。不用重启driver更新代理成功。
```

### Selenium Webdriver常用方法

操作Element对象

```python
find_element_by_id("user_name").clear()  # 清楚元素内容
find_element_by_id("user_name").send_keys("username")  # 按键输入
find_element_by_id("dl_an_submit").click()  # 单机元素
find_element_by_id("dl_an_submit").submit()  # 提交表单
```

- Element常用方法

```python
driver.find_element_by_name('tj_trnews').size  # 元素尺寸
driver.find_element_by_name('tj_trnews').text  # 返回元素文本
driver.find_element_by_name('tj_trnews').get_attribute('class')  # 获取属性值
driver.find_element_by_name('tj_trnews').is_displayed()  # 是否用户可见
```

鼠标事件

```python
from selenium.webdriver.common.action_chains import ActionChains

 el = driver.find_element_by_name('tj_trnews')  # 目标元素

ActionChains(driver).context_click(el).perform()  # 右击目标元素
ActionChains(driver).double_click(el).perform()  # 双击目标元素

source = driver.find_element_by_id('lg')   # 目标元素原始位置
target = driver.find_element_by_id('kw')  # 拖动的目标位置
ActionChains(driver).drag_and_drop(source, target).perform()  # 拖动元素

ActionChains(driver).move_to_element(el).perform()  # 鼠标移动的目标元素上
ActionChains(driver).click_and_hold(el).perform()  # 移动到目标元素按下鼠标左键
```

键盘事件

```python
from selenium.webdriver.common.keys import Keys

driver.get("https://www.baidu.com/")
driver.find_element_by_id("kw").send_keys("selenium")  # 输入框输入内容
driver.find_element_by_id("kw").send_keys(Keys.BACK_SPACE)  # 从后删除一个字符（删除键）
driver.find_element_by_id("kw").send_keys(Keys.SPACE)  # 输入空格
driver.find_element_by_id("kw").send_keys(Keys.CONTROL,'a')  # ctrl + a 全选输入框内容 
driver.find_element_by_id("kw").send_keys(Keys.CONTROL,'x')  # ctrl + x 剪切输入框内容
driver.find_element_by_id("kw").send_keys(Keys.CONTROL,'v')  # ctrl + v 粘贴
driver.find_element_by_id("su").send_keys(Keys.ENTER)  # 回车
```

设置等待时间

-  强制等待sleep() 强制等待比较暴力，调用time模块的sleep()方法 
-  隐性等待implicitly_wait()是设置了最大等待时间，如果在规定时间内加载完成，则继续执行下面操作，否则一直等到时间截止再执行下一步。如果需要的元素已经加载出来了，但是页面整体还没有加载完成，程序也会一直等待，也并不智能。 
-  显性等待WebDriverWait() 需要传入一个判断条件的匿名函数，每隔一段时间去判断条件函数，如果条件成立则继续下一步，如果不成立则继续等待。

```python
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
import time
driver = webdriver.Firefox()
driver.get("http://www.baidu.com")
# 强制等待
time.sleep(5)
# 隐形等待
driver.implicitly_wait(30)
# 显性等待
element=WebDriverWait(driver, 10).until(lambda driver:driver.find_element_by_id("kw"))  # 当找到id为kw的元素时才执行下一步。超时时间为10秒，默认每0.5秒检测一次。
```

定位frame中的对象

```javascript
driver.switch_to.frame()
frame方法接收三种参数:frame name、index 和webelement
driver.switch_to.frame('frame_name')
driver.switch_to.frame(0) #第一个frame
driver.switch_to.default_content() #回到主体框架
driver.switch_to.frame(driver.find_elements_by_tag_name("iframe")[0])
```

多窗口切换

要想在多个窗口之间切换，首先要获得每一个窗口的唯一标识符号（句柄）。通过获得的句柄来区别分不同的窗口，从而切换不同窗。

```javascript
driver.get("http://example.com")  # 打开一个窗口
now_handle = drvier.current_window_handle  # 获取当前窗口句柄

driver.find_element_by_name('example').click()  # 点击某个元素打开新的窗口(target="_black"的元素)
all_handle = drvier.window_handles  # 获取所有窗口句柄
drvier.switch_to.window(now_handle)  # 切换为第一窗口
driver.close()  # 关闭当前窗口
```

下拉框处理

webdriver处理下拉框首先定位到下拉框内容，然后click某个option即可

```python
m=driver.find_element_by_id("ShippingMethod")  # 首先定位到下拉框
m.find_element_by_xpath("//option[@value='10.69']").click()  # 然后点击下拉框选项
```

执行JavaScrapt

```python
driver.execute_script('Java Scrapt Code')
# 例如下拉浏览器滚动条
driver.execute_script('window.scrollTo(0, document.body.scrollHeight)')
```

Cookie处理

webdriver可以对cookie进行读取、增加、删除

```python
get_cookies()   # 获得所有 cookie 信息
get_cookie(name)   # 返回特定 name 有 cookie 信息
add_cookie(cookie_dict)   # 添加 cookie，必须有 name 和 value 值
delete_cookie(name)   # 删除特定(部分)的 cookie 信息
delete_all_cookies()   # 删除所有 cookie 信息
```

处理弹出窗口和警告：使用Selenium的方法来处理网页中的弹出窗口和警告。

```python
from requestium import Session
s = Session(webdriver_path='path/to/your/webdriver')
s.get('http://example.com')
# 点击会导致弹出窗口的按钮
s.driver.find_element_by_id('my-button').click()
# 接受弹出的确认对话框
alert = s.driver.switch_to.alert
alert.accept()
# 或者，如果要取消对话框
# alert.dismiss()
# 处理警告消息
s.get('http://example.com')
# 执行会导致警告的JavaScript代码
s.driver.execute_script("window.alert('Hello, World!');")
# 获取并接受警告
alert = s.driver.switch_to.alert
alert.accept()
```

截图和录像：使用Selenium的方法来截取网页的屏幕截图或录制视频。录制视频功能可能需要额外的库（如ffmpeg）和配置

```python
from requestium import Session
import time
s = Session(webdriver_path='path/to/your/webdriver')
s.get('http://example.com')
# 截取屏幕截图并保存为图片文件
s.driver.get_screenshot_as_file('screenshot.png')
# 或者，获取截图的Base64编码
screenshot_base64 = s.driver.get_screenshot_as_base64()
# 录制网页操作的视频（需要安装额外的库如ffmpeg）
s.driver.start_recording_screen(video_name='recording.mp4')
# 进行一些操作...
time.sleep(5)  # 操作持续5秒
s.driver.stop_recording_screen()
```

### requestium模块

可以使用requestium模块,由于Requestium是基于Selenium和Requests的，使用前确保安装这两个库

```python
from selenium import webdriver
from requestium import Session, Keys
firefox_driver = webdriver.Firefox()
s = Session(driver=firefox_driver)
```

不需要解析响应，当调用 xpath，css 或 re 时，它会自动完成

```python
response = s.get('http://samplesite.com/sample_path')
cmnt_karma = response.xpath("//span[@class='karma comment-karma']//text()").extract_first()
# Extracts the first match
identifier = response.re_first(r'ID_\d\w\d', default='ID_1A1')
# Extracts all matches as a list
users = response.re(r'user_\d\d\d')
```

**添加 Cookie**

ensure_add_cookie 方法使得添加 Cookie 更加稳健。Selenium 需要浏览器在能够添加 Cookie 之前处于 Cookie 的域中。

此方法提供了几种解决方法。

+ 如果浏览器不在 Cookie 域中，它会先获取域然后再添加 Cookie。
+ 它还允许你在添加 Cookie 之前覆盖域，并避免执行此 GET。域可以被覆盖为 ‘ ’，这将把 Cookie 的域设置为驱动程序当前所在的任何域。
+ 如果无法添加 cookie，它会尝试使用限制性较小的域（例如：home.site.com -> site.com）进行添加，然后在失败之前

```python
cookie = {"domain": "www.site.com",
          "secure": false,
          "value": "sd2451dgd13",
          "expiry": 1516824855.759154,
          "path": "/",
          "httpOnly": true,
          "name": "sessionid"}
s.driver.ensure_add_cookie(cookie, override_domain='')
```

处理Cookies和Headers：使用Requests的方式设置和管理Cookies和Headers

```python
from requestium import Session
s = Session(webdriver_path='path/to/your/webdriver')
# 设置请求头
headers = {'User-Agent': 'My User Agent'}
s.headers.update(headers)
# 发送带Header的GET请求
s.get('http://example.com', headers=headers)
# 获取并保存Cookies
cookies = s.cookies
# 在后续请求中使用已保存的Cookies
s.get('http://another-example.com', cookies=cookies)
```

**等待元素**

ensure_element_by_ 方法等待元素在浏览器中加载，并在加载后立即返回。它以 Selenium的 find_element_by_ 方法命名（如果找不到元素，它们会立即引发异常）

Requestium 可以等待一个元素处于以下任何状态：

- 存在（默认）
- 可点击，clickable
- 看得见的，visible
- 不可见（可用于等待加载... GIF 消失等）

这些方法对于单页面 Web 应用程序非常有用，其中站点动态地更改其元素。通常用 ensure_element_by_ 调用替换我们的 find_element_by_ 调用，因为它们更灵活。使用这些方法获取的元素具有新的 ensure_click 方法，这使得点击不太容易失败

s.driver.ensure_element_by_xpath("//li[@class='b1']", state='clickable', timeout=5).ensure_click()

state='clickable'；

异步请求和多线程处理：虽然Requestium本身不直接支持异步请求或多线程处理，但你可以结合其他库（如asyncio或concurrent.futures）来实现这些功能。

```python
import asyncio
from requestium import Session
async def fetch_page(session, url):
    await asyncio.sleep(1)
    session.driver.get(url)
    return session.driver.page_source
async def main():
    s = Session(webdriver_path='E:\python\chromedriver.exe')

    tasks = []
    urls = ['https://www.baidu.com', 'https://www.bilibili.com/', 'https://www.jd.com']
    for url in urls:
        tasks.append(fetch_page(s, url))
    results = await asyncio.gather(*tasks)
    s.driver.quit()
    for result in results:
        print(result)
asyncio.run(main())
```

问题：

+ python异步协程爬虫报错：【TypeError: object int can‘t be used in ‘await‘ expression】

错误的原因可能是在async函数中使用了一个整数类型的变量作为await的参数。await只能用于返回协程对象的异步函数，无法使用在普通的同步操作上

解决这个问题，需要确保await的参数是一个异步函数的返回值。如果我们只是想等待一个时间段后再执行下一个操作，可以使用asyncio.sleep()函数作为协程对象来等待一定的时间

+ OSError: [WinError 6\] 句柄无效的解决办法

解决方案：
关闭driver 时 ， 使用 driver.quit()代替 driver.close()

区别
close：只会关闭焦点所在的当前窗口
quit：会关闭所有关联的窗口

------------------------------------------------

webdriver和requests相互切换

切换回使用 Requests:transfer_driver_cookies_to_session()

切换到使用 Selenium Webdriver 来运行:

transfer_session_cookies_to_driver()提升效率

```python
import time,sys,os
from requestium import Session, Keys
s = Session(webdriver_path='/Users/lib/Documents/chromedriver',
            browser='chrome',
            default_timeout=15,
            webdriver_options={'arguments': ['disable-gpu',"--user-agent=Mozilla/5.0 (iPod; U; CPU iPhone OS 2_1 like Mac OS X; ja-jp) AppleWebKit/525.18.1 (KHTML, like Gecko) Version/3.1.1 Mobile/5F137 Safari/525.20"]})
            #webdriver_options={'arguments': ['headless','--proxy-server=http://user:pwd@ip2.hahado.cn:40275']})
            #webdriver_options={'arguments': ['headless','--proxy-server=http://116.11.254.37:80']})
#url = 'https://icanhazip.com/'
url = 'https://www.che300.com/pinggu/v9c9m30614r2016-06g2.8'
url = 'https://magi.com/search?q=%E6%96%B0%E5%86%A0'
url = 'http://m.baidu.com/s?word=%E6%88%BF%E5%B1%B1%E6%96%B0%E7%9B%98'
s.driver.get(url)
html =  s.driver.page_source
print (html)
#pic = response.xpath('//img/@src').extract_first()
pic = None
#if pic is not None:
if u'访问太过于频繁,请输入验证码后再次访问' in html:
  #print pic
  code = raw_input('请输入验证码:')
  print ('您输入的验证码是：%s' % code)
  s.driver.find_element_by_xpath("//input[@name='code']").send_keys(code,Keys.ENTER)
  time.sleep(1)
  s.driver.ensure_element_by_xpath("//input[@type='submit']").click()
  #self.s.transfer_driver_cookies_to_session()

#s.transfer_session_cookies_to_driver()
time.sleep(3)
print (s.driver.page_source)
s.driver.quit()
```

### requestium实现自动登录获取网页cookie信息并爬取数据

（1）有些网站设置了反爬机制，我们需要手动添加headers，加入user-agent参数

```python
from requestium import Session, Keys
import time
import json
import requests

#将cookie信息添加进requests组件中	
def add_cookies(cookie,s):
	u"往session添加cookies"
	c = requests.cookies.RequestsCookieJar()
	for i in cookie:    #添加cookie到CookieJar
		#print(i)
		c.set(i["name"], i["value"])
	s.cookies.update(c)     #更新session里的cookie
	return s
    
#获取cookie信息
def getCookie():
	#webdriver驱动地址根据实际修改
	s = Session(webdriver_path='D:/chromedriver_win32/chromedriver.exe',
		browser='chrome',
		default_timeout=15,
		webdriver_options={'arguments': ['headless']})
	s.driver.get('填写登录网址')
	time.sleep(1)
	s.driver.ensure_element_by_id('username').clear()
	s.driver.ensure_element_by_id('username').send_keys('用户名')
	s.driver.ensure_element_by_id('password').clear()
	s.driver.ensure_element_by_id('password').send_keys('密码')
	s.driver.ensure_element_by_id('login').click()
	time.sleep(3)
	url = s.driver.current_url
	cookie = s.driver.get_cookies()
	#print(cookie)
	jsonCookies = json.dumps(cookie) #转字符串
	s.driver.close()
	return cookie

if __name__ == '__main__':  
	s = requests.session()
	cookie = getCookie()
	#为requests组件加入cookie信息
	add_cookies(cookie,s)
	#请求需要的数据
	data = {
		'response_data': json.dumps(response_data),
		'filter_condition_key':'', 
		'_search': 'False',
		'nd': 1576549859948,
		'rows': 100,
		'page': 1,
		'sidx': '', 
		'sord': 'asc',
		'Q^ALERT_TIME_START_^DL': 'xxx',
		'Q^ALERT_TIME_START_^DG': 'xxx'
	}
	#反爬破解机制，加入自己网页的user-agent
	headers={'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.87 Safari/537.36 SLBrowser/6.0.1.4221'}
	response = s.post('需要抓取数据的网址', headers,data=data)
	if response.content:
		print(response.json())
```



### 附录

[selenium官网](https://selenium-python-zh.readthedocs.io/en/latest/)

[参考](https://blog.csdn.net/qq_29404831/article/details/110086552?app_version=6.3.1&code=app_1562916241&csdn_share_tail=%7B%22type%22%3A%22blog%22%2C%22rType%22%3A%22article%22%2C%22rId%22%3A%22110086552%22%2C%22source%22%3A%22airgzne%22%7D&uLinkId=usr1mkqgl919blen&utm_source=app)

[参考1](https://mp.weixin.qq.com/s/-41OaGfrS6A9iKVsKFLAjw)