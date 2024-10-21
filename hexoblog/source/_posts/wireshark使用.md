---
title: wireshark使用
img: /images/wireshark.jpg
categories: tools
tags:
  - wireshark
  - ISO
abbrlink: 43829
date: 2024-10-09 18:55:36
---

## 安装

 软件下载路径：[wireshark官网](https://www.wireshark.org/)

Wireshark提供[Mac版本](https://www.wireshark.org/download.html)，按照系统版本(终端使用uname -a)选择下载，下载完成后，按照软件提示一路Next安装。

 说明：如果你是[Win10系统](https://so.csdn.net/so/search?q=Win10系统&spm=1001.2101.3001.7020)，安装完成后，选择抓包但是不显示网卡，下载win10pcap兼容性安装包。下载路径：[win10pcap兼容性安装包](http://www.win10pcap.org/download/)

## 使用

Wireshark在第一个界面就把当前系统所包含的网卡列出来了，直接点击任何一项就可以开始监听通过该网卡的所有网络流量。

当我们把iPhone通过usb连接macbook时，Wireshark并不能直接监听通过iPhone的网络流量，需要通过一个系统程序在我们的Mac系统上，建立一个映射到iPhone的虚拟网卡，在terminal中输入如下命令即可：

```bash
pip install tidevice3
t3 list
rvictl -s udid
```

格式是rvictl -s [设备udid]，设备的udid可以通过itunes或者itools获取，执行命令之后Wireshark能立即识别新增加的rvi0网卡，也就是上图中高亮的部分，双击rvi0这一项，Wireshare即进入如下界面开始监听iPhone设备上的所有流量。

注意：勾选混淆模式选项

## 界面介绍

Wireshark的流量监控界面主要分为四块，由上至下
![wireshark界面](https://bigworldxld.github.io/picx-images-hosting/images/wireshark界面.67xgj8i223.webp)
 说明：数据包列表区中不同的协议使用了不同的颜色区分。协议颜色标识定位在菜单栏View --> Coloring Rules。如下所示
{% asset_img wireshark颜色过滤.png This is an example image %}


**WireShark 主要分为这几个界面**

1. Display Filter(显示过滤器)， 用于设置过滤条件进行数据包列表过滤。菜单路径：Analyze --> Display Filters。

{% asset_img wireshark过滤器.png This is an example image %}


2. Packet List Pane(数据包列表)， 显示捕获到的数据包，每个数据包包含编号，时间戳，源地址，目标地址，协议，长度，以及数据包信息。 不同协议的数据包使用了不同的颜色区分显示。

{% asset_img wireshark数据包列表.png This is an example image %}


3. Packet Details Pane(数据包详细信息), 在数据包列表中选择指定数据包，在数据包详细信息中会显示数据包的所有详细信息内容。数据包详细信息面板是最重要的，用来查看协议中的每一个字段。各行信息分别为

 （1）Frame:  物理层的数据帧概况

 （2）Ethernet II: 数据链路层以太网帧头部信息

 （3）Internet Protocol Version 4: 互联网层IP包头部信息

 （4）Transmission Control Protocol: 传输层T的数据段头部信息，此处是TCP

 （5）Hypertext Transfer Protocol: 应用层的信息，此处是HTTP协议

{% asset_img wireshark_ISO.png This is an example image %}


**TCP包的具体内容**

 从下图可以看到wireshark捕获到的TCP包中的每个字段。
{% asset_img wireshark传输层.png This is an example image %}


## 使用Filter过滤包

1、抓包过滤器语法和实例

  抓包过滤器类型Type（host、net、port）、方向Dir（src、dst）、协议Proto（ether、ip、tcp、udp、http、icmp、ftp等）、逻辑运算符（&& 与、|| 或、！非）

（1）协议过滤

 比较简单，直接在抓包过滤框中直接输入协议名即可。

 tcp，只显示TCP协议的数据包列表

 http，只查看HTTP协议的数据包列表

 icmp，只显示ICMP协议的数据包列表

（2）IP过滤

 host 192.168.1.104

 src host 192.168.1.104

 dst host 192.168.1.104

（3）端口过滤

 port 80

 src port 80

 dst port 80

（4）逻辑运算符&& 与、|| 或、！非

 src host 192.168.1.104 && dst port 80 抓取主机地址为192.168.1.80、目的端口为80的数据包

 host 192.168.1.104 || host 192.168.1.102 抓取主机为192.168.1.104或者192.168.1.102的数据包

 ！broadcast 不抓取广播数据包

2、显示过滤器语法和实例

（1）比较操作符

 比较操作符有== 等于、！= 不等于、> 大于、< 小于、>= 大于等于、<=小于等于。

（2）协议过滤

 比较简单，直接在Filter框中直接输入协议名即可。**注意：协议名称需要输入小写。**

 tcp，只显示TCP协议的数据包列表

 http，只查看HTTP协议的数据包列表

 icmp，只显示ICMP协议的数据包列表

（3） ip过滤

  ip.src ==192.168.1.104或者  ip.src_host ==192.168.1.104 显示源地址为192.168.1.104的数据包列表

  ip.dst==192.168.1.104, 显示目标地址为192.168.1.104的数据包列表

  ip.addr == 192.168.1.104 显示源IP地址或目标IP地址为192.168.1.104的数据包列表

（4）端口过滤

 tcp.port ==80, 显示源主机或者目的主机端口为80的数据包列表。

 tcp.srcport == 80, 只显示TCP协议的源主机端口为80的数据包列表。

 tcp.dstport == 80，只显示TCP协议的目的主机端口为80的数据包列表。

（5） Http模式过滤

 http.request.method=="GET",  只显示HTTP GET方法的。

（6）逻辑运算符为 and/or/not

 过滤多个条件组合时，使用and/or。比如获取IP地址为192.168.1.104的ICMP数据包表达式为ip.addr == 192.168.1.104 and icmp

（7）按照数据包内容过滤。假设我要以IMCP层中的内容进行过滤，可以单击选中界面中的码流，在下方进行选中数据

后面条件表达式就需要自己填写。如下我想过滤出data数据包中包含"abcd"内容的数据流。**包含的关键词是contains 后面跟上内容。**

data contains “abcd”

 调整数据包列表中时间戳显示格式。调整方法为**View -->Time Display Format --> Date and Time of Day**


## 流量跟踪

默认情况下将不同网络连接的流量都混在一起展示，即使给不同协议的包上色之后，要单独查看某个特定连接的流量依然不怎么方便

方式一：Follow Stream

当我们选中某个包之后，右键弹出的菜单里，有个Fllow选项允许我们将当前包所属于的完整流量单独列出来

Wireshark支持我们常见的四种Stream，TCP，UDP，HTTP，SSL。比如我们选中Follow TCP Stream之后可以得到如下的详细分析输出（样本为监控iPhone手机的流量）：

将iPhone和Server之间某次的连接流量完整的呈现出来，包括iPhone发送了多少个包，Server回了多少个包，以及iPhone上行和下行的流量，还提供流量编解码选择，文本搜索功能等

方式二：Flow Graph

Flow Graph可以通过菜单Statistics->Flow Graph来生成，这样我们可以得到另一种形式的流量呈现

Follow Stream更适合分析针对某一个服务器地址的流量，而Flow Graph更适合分析某个App的整体网络行为，包含从DNS解析开始到和多个服务器交互等

比如Statistics->HTTP->Requests可以得到如下按主机分门别类的HTTP请求分析图

## ISO

**网络层：**

ARP (Address Resolution Protocol)根据IP地址获取物理地址的一个TCP/IP协议

**ICMP**（ping协议，测试网络连通性）

b ping a
a ping b
数据包都是有去有回
防火墙，不通的那个设备，禁ping
windows防火墙，出站连接，默认放行；入站连接，默认禁止，解决：设置新的入站规则，添加所需要的协议类型

win10下路由跟踪

tracert -d www.baidu.com
测试，我这台电脑，到达百度服务器中间经过了哪些设备

windows,发出的是个icmp探测包，ttl=1，探测出第一跳

​                  发出的是个icmp探测包，ttl=2，跳探测出第二跳

​				  发出的是个icmp探测包，ttl=3，探测出第三跳
因为ttl =1的包，数据包每到达一个三层设备，ttl 减去1，在第一跳减到0， 如果ttl=0，丢弃,谁把我的包，丢了，谁给我一个报错消息，报错消息的内容，ttl没了所以丢包了

ttl=2，第二跳设备，丢我包。他给我发报错。 他就是第二跳。

**传输层：**

**TCP**（传输层）

三次握手： Step1：客户端发送一个SYN=1，ACK=0标志的数据包给服务端，请求进行连接，这是第一次握手；
SYN=1,ACK=0,seq=x
tcp.flags.syn==1 and tcp.flags.ack==0
  Step2：服务端收到请求并且允许连接的话，就会发送一个SYN=1，ACK=1标志的数据包给发送端，告诉它，可以通讯了，并且让客户端发送一个确认数据包，这是第二次握手；
SYN=1,ACK=1,seq=y,ack=x+1
  Step3：服务端发送一个SYN=0，ACK=1的数据包给客户端端，告诉它连接已被确认，这就是第三次握手。TCP连接建立，开始通讯。
ACK=1,seq=x+1,ack=y+1



四次挥手标志分别为：

- "[FIN, ACK]"
- "[ACK]"
- "[FIN, ACK]"
- "[ACK]"

**应用层：**

**DNS**，基于TCP，端口号53
DNS系统的作用：提供了主机名字和以地址间的相互转换
DNS系统的模式：采用客户端/服务器模式

**FTP**协议是互联网上广泛使用的文件传输协议
客户端/服务器模式，基于TCP
FTP采用双TCP连接方式

**TFTP**(简单文件传输协议)也是采用客户机/服务器模式的文件传输协议
适用于客户端和服务器之间不需要复杂交互的环境
基于UDP之上，端口号69
TFTP仅提供简单的文件传输功能(上传、下载)
TFTP没有存取授权与认证机制，不提供目录列表功能
TFTP协议传输是由客户端发起的

arp -a

查看所有的mac地址

Telnet和SSH是两种常用的远程登录协议主要区别：

‌**安全性**：‌Telnet是一种明文传输协议，所有的数据（包括用户名、密码和传输的文件内容）都是以明文形式在网络中传输，因此容易被窃听和篡改，不适合用于敏感信息的传输。‌12

SSH（Secure Shell）是一种加密的远程登录协议，它使用公钥加密和密钥交换技术，确保数据在传输过程中是加密的，从而保护了用户的隐私和数据的安全性。

**身份验证方式**‌：Telnet只支持基于用户名和密码的身份验证。‌SSH支持多种身份验证方式，包括用户名和密码、SSH密钥等，提供了更高的安全性。

‌**端口号**‌：Telnet使用的默认端口号是23。SSH使用的默认端口号是22。

**功能**：‌Telnet只能提供基本的远程登录功能，支持全双工通信，即客户端和服务器可以同时发送和接收数据。‌SSH除了远程登录外，还支持文件传输、端口转发、远程命令执行等功能，使得用户能够更加方便地远程管理计算机

**协议号，和端口号，啥区别？**
协议号，在网络层中，ip头部里的字段，标识这个数据包是tcp（6），还是udp（17），还是icmp（1），还是gre（50），还是ospf（89），还是vrrp

端口号，在传输层中， tcp，udp头部里的字段，标识这个数据包是http(80)，还是dns(53)还是ftp(21)，还是tftp（69），还是smtp 

参考：[wireshark抓包教程详解](https://blog.csdn.net/lixinkuan328/article/details/122985439?ops_request_misc=%257B%2522request%255Fid%2522%253A%25226134A3B1-0118-43FC-A2E1-92114CF59D0C%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=6134A3B1-0118-43FC-A2E1-92114CF59D0C&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-122985439-null-null.142^v100^pc_search_result_base7&utm_term=wireshark%E6%8A%93%E5%8C%85%E5%8F%8A%E5%88%86%E6%9E%90&spm=1018.2226.3001.4187)

[Wireshark抓包](https://blog.csdn.net/qq_30513483/article/details/73603325?ops_request_misc=%257B%2522request%255Fid%2522%253A%25228B17844C-BEEB-489E-BFB2-A49BB7EB6703%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=8B17844C-BEEB-489E-BFB2-A49BB7EB6703&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-2-73603325-null-null.142^v100^pc_search_result_base7&utm_term=ios%20wireshark%E6%8A%93%E5%8C%85%E5%8F%8A%E5%88%86%E6%9E%90&spm=1018.2226.3001.4187)