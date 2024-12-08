---
title: 跨站和SameSite
img: 'https://bigworldxld.github.io/picx-images-hosting/images/跨站和SameSite.webp'
categories: test
tags:
  - Cookie
  - SameSite
  - CSRF
abbrlink: 52081
date: 2024-10-26 16:42:30
---
## 跨站和SameSite
Cookie 的 SameSite 属性可以让 Cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）。

SameSite 属性值可以有下面三种值：

Strict:仅允许一方请求携带 Cookie，即浏览器将只发送相同站点请求的 Cookie，即当前网页 URL 与请求目标 URL完全一致。
Lax:允许部分第三方请求携带 Cookie
None:无论是否跨站都会发送 Cookie
之前默认是 None，Chrome80 后默认是 Lax。

## 跨域和跨站
**跨域**
跨域对应的是同域，同域即是浏览器的同源策略，这里不单单是指相同域名，域名、协议、端口号必须完全一致，才属于同源。

跨域主要是针对请求的。如果跨域，浏览器会在控制台提示类似的Error。

**跨站**
跨站对应的是同站，也可以称呼叫做第一方（同站）或者第三方（跨站）。同源策略作为浏览器的安全基石，其「同源」判断是比较严格的。相对而言，Cookie中的「同站」判断就比较宽松：只要两个 URL 的 eTLD+1 相同即可，不需要考虑协议和端口。其中，eTLD 表示有效顶级域名，例如，.com、.http://co.uk、.http://github.io 等。eTLD+1 则表示，有效顶级域名+二级域名，例如 taobao.com 等。

Public Suffix List
假设这样的情况：

1. a.jzplp.com与b.jzplp.com 实际属于同站

2. jzplp.com.cn与other.com.cn 实际属于跨站

3. jzplp.github.io与other.github.io 实际属于跨站

 第一个属于同站。看第二个例子，com.cn是我们国家颁布的二级域名并不是指的某一个网站。显然这种域名后缀，并不能认为是同站。除了我国，还有其他国家和组织也会公布这种二级甚至更高级的域名。再看第三个例子，是Github提供的网站域名。使用github.io后缀的网站，显然也不能认为是同站。

因此，仅仅通过网址格式匹配，是无法判断是否同站的。因此，浏览器会维护一个列表：Public Suffix List。列表里面记录了这些需要特殊匹配的域名后缀，比如上面提到的com.cn与github.io等等，这个叫做有效顶级域名，eTLD。在遇到一个域名时，会首先匹配列表中的后缀，再把eTLD+1个字段相同表示为同站，不同表示为非同站。

例如 列表中的域名后缀为com.cn，那么eTLD+1个字段表示为jzplp.com.cn。那么a.jzplp.com.cn与b.jzplp.com.cn属于同站， jzplp.com.cn与other.com.cn属于跨站。

Public Suffix List列表的内容。这个列表会随着浏览器的更新而更新。

经实测，协议不同也会跨站。我是在 http 网页内用 iframe 嵌套 https 网页， 2个网页的 eTLD+1 是相同的。iframe 里的第三方网页，登录时设置 cookie 失败，接下去的请求也没有发送 cookie。

跨站一定跨域，跨域不一定跨站。

改变
接下来看下从 None 改成 Lax 到底影响了哪些地方的 Cookies 的发送？

| 请求类型 | 实例                               | Strict | Lax        | None       |
| -------- | ---------------------------------- | ------ | ---------- | ---------- |
| 链接     | `<a href="..."></a>`                 | 不发送 | 发送Cookie | 发送Cookie |
| 预加载   | `<link rel="prerender" href="..."/>` | 不发送 | 发送Cookie | 发送Cookie |
| get表单  | `<form method="GET" action="">`      | 不发送 | 发送Cookie | 发送Cookie |
| post表单 | `<form method="POST" action="">`     | 不发送 | 不发送     | 发送Cookie |
| iframe   | `<iframe src=""></iframe>`           | 不发送 | 不发送     | 发送Cookie |
| AJAX     | $.get(“...”)                       | 不发送 | 不发送     | 发送Cookie |
| image    | `<img src="...">`                  | 不发送 | 不发送     | 发送Cookie |

可以看出，对大部分 web 应用而言，Post 表单，iframe，AJAX，Image 这四种情况从以前的跨站会发送三方 Cookie，变成了不发送。

Post表单：应该的，学 CSRF 总会举表单的例子。
iframe：iframe 嵌入的 web 应用有很多是跨站的，都会受到影响。
AJAX：可能会影响部分前端取值的行为和结果。
Image：图片一般放 CDN，大部分情况不需要 Cookie，故影响有限。但如果引用了需要鉴权的图片，可能会受到影响。
除了这些还有 script 的方式，这种方式也不会发送 Cookie。

**解决**
解决方案就是设置 SameSite 为 none。

不过也会有两点要注意的地方：

1：HTTP 接口不支持 SameSite=none。
如果你想加 SameSite=none 属性，那么该 Cookie 就必须同时加上 Secure 属性，表示只有在 HTTPS 协议下该 Cookie 才会被发送。
服务端和js都可以这么设置。
2：需要 UA 检测，部分浏览器不能加 SameSite=none。
IOS 12 的 Safari 以及老版本的一些 Chrome 会把 SameSite=none 识别成 SameSite=Strict，所以服务端必须在下发 Set-Cookie 响应头时进行 User-Agent 检测，对这些浏览器不下发 SameSite=none 属性

## CSRF

攻击者会导致受害者用户无意中执行操作。例如，这可能是为了更改其帐户的电子邮件地址、更改其密码或进行资金转账

CSRF 攻击成为可能，必须满足三个关键条件

**相关操作**：例如修改其他用户的权限或对用户特定数据的任何操作（例如更改用户自己的密码

**基于 Cookie 的会话处理**：应用程序仅依赖会话 Cookie 来识别发出请求的用户。没有其他机制可用于跟踪会话或验证用户请求

**没有不可预测的请求参数**：执行该操作的请求不包含攻击者无法确定或猜测其值的任何参数

**最常见防御措施**

 CSRF 令牌是由服务器端应用程序生成并与客户端共享的唯一、秘密且不可预测的值。

SameSite 是一种浏览器安全机制，用于确定何时将网站的 Cookie 包含在来自其他网站的请求中。自 2021 年以来，Chrome 默认强制实施 SameSite：Lax 限制

**基于 Referer 的验证** - 某些应用程序使用 HTTP Referer 标头来尝试防御 CSRF 攻击，通常是通过验证请求是否来自应用程序自己的域

**XSS 和 CSRF 有什么区别？**

XSS允许攻击者在受害者用户的浏览器中执行任意 JavaScript。

CSRF允许攻击者诱使受害者用户执行他们不打算执行的操作

CSRF 可以被描述为“单向”漏洞;XSS 是“双向”的，因为攻击者注入的脚本可以发出任意请求、读取响应并将数据泄露到攻击者选择的外部域

**CSRF 令牌验证中的常见缺陷**

**验证取决于请求方法**:某些应用程序在请求使用 POST 方法时正确验证令牌，但在使用 GET 方法时跳过验证

**验证取决于令牌的存在:**某些应用程序会在令牌存在时正确验证令牌，但如果省略令牌，则会跳过验证。

**令牌未绑定到用户会话**:某些应用程序不验证令牌是否与发出请求的用户属于同一会话。相反，应用程序维护它已颁发的令牌的全局池，并接受此池中显示的任何令牌

**令牌与非会话 Cookie 绑定**:某些应用程序确实将 CSRF 令牌绑定到 Cookie，但不绑定到用于跟踪会话的同一 Cookie。当应用程序使用两个不同的框架时，这很容易发生，一个用于会话处理，另一个用于 CSRF 保护，它们没有集成在一起

**令牌只是在 cookie 中复制**:某些应用程序不维护已颁发令牌的任何服务器端记录，而是在 Cookie 和请求参数中复制每个令牌。验证后续请求时，应用程序只需验证 request 参数中提交的令牌是否与 Cookie 中提交的值匹配。

域名后缀，亦被称为顶级域名(Top Level Domain)，是指代表一个域名类型的符号。 不同后缀的域名有不同的含义。域名共分为两类：**[国别域名](https://baike.baidu.com/item/国别域名/880897?fromModule=lemma_inlink)**（ccTLD），例如中国的[.cn](https://baike.baidu.com/item/.cn/0?fromModule=lemma_inlink)、美国的[.us](https://baike.baidu.com/item/.us/6688112?fromModule=lemma_inlink)、[俄罗斯](https://baike.baidu.com/item/俄罗斯/125568?fromModule=lemma_inlink)的.ru、以及国际**[通用域名](https://baike.baidu.com/item/通用域名/8221919?fromModule=lemma_inlink)**（[gTLD](https://baike.baidu.com/item/gTLD/6758550?fromModule=lemma_inlink)），例如[.com](https://baike.baidu.com/item/.com/812899?fromModule=lemma_inlink)、[.xyz](https://baike.baidu.com/item/.xyz/15250717?fromModule=lemma_inlink)、[.top](https://baike.baidu.com/item/.top/8771482?fromModule=lemma_inlink)、[.wang](https://baike.baidu.com/item/.wang/13476945?fromModule=lemma_inlink)、[pub](https://baike.baidu.com/item/pub/10691360?fromModule=lemma_inlink)、[.xin](https://baike.baidu.com/item/.xin/18761126?fromModule=lemma_inlink)、[.net](https://baike.baidu.com/item/.net/0?fromModule=lemma_inlink)等1000多种

`https://app.example.com:443`

+ https://为schema
+ .com为TLD
+ .example.com为TLD+1
+ TLD+1和schema统称为站site
+ `https://app.example.com:443`整体称为origin

| **请求方**                | **请求**                       | **同站点？**      | **同源？**     |
| ------------------------- | ------------------------------ | ----------------- | -------------- |
| `https://example.com`     | `https://example.com`          | 是的              | 是的           |
| `https://app.example.com` | `https://intranet.example.com` | 是的              | 否：域名不匹配 |
| `https://example.com`     | `https://example.com:8080`     | 是的              | 否：端口不匹配 |
| `https://example.com`     | `https://example.co.uk`        | 否：不匹配的 eTLD | 否：域名不匹配 |
| `https://example.com`     | `http://example.com`           | 否：不匹配的方案  | 否：不匹配的方 |

**绕过基于 Referer 的 CSRF 防御**
Referer 标头
HTTP Referer 标头（在 HTTP 规范中无意中拼写错误）是一个可选的请求标头，其中包含链接到所请求资源的网页的 URL。它通常由浏览器在用户触发 HTTP 请求时自动添加，包括通过单击链接或提交表单。存在各种方法，允许链接页面保留或修改 header 的值。这样做通常是出于隐私原因
1：如果省略标头，则跳过验证
`<meta name="referrer" content="never">`
2：如果应用程序验证 中的域是否以预期值开头，则攻击者可以将其作为自己域的子域，如果应用程序只是验证 是否包含自己的域名，则攻击者可以将所需的值放在 URL 中的其他位置，http://attacker-website.com/csrf-attack?vulnerable-website.com

可确保发送完整的 URL，包括查询字符串。Referrer-Policy: unsafe-urlReferrer



**防止 CSRF 漏洞**

使用 CSRF 令牌必须满足以下条件：

- 高熵不可预测，就像一般的会话令牌一样。
- 绑定到用户的会话。
- 在执行相关操作之前，在每种情况下都经过严格验证

使用严格的 SameSite Cookie 限制

警惕跨域、同一站点的攻击