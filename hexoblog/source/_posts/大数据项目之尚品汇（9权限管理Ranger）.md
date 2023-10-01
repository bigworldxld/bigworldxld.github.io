---
title: 大数据项目之尚品汇（9权限管理Ranger）
date: 2022-02-14 13:48:57
categories: data warehouse
img: /images/datawarehouse.jpg
tags:
	- hadoop          
	- spark
	- zookeeper
	- hive
	- kafka
	- flume
	- Azkaban
	- sqoop
	- superset
	- hbase
	- Presto
	- kylin
	- zabbix
	- Kerberos
	- ranger
	- atlas
---

# 第1章 Ranger概述

## 1.1 什么是Ranger

Apache Ranger是一个Hadoop平台上的全方位数据安全管理框架，它可以为整个Hadoop生态系统提供全面的安全管理。

随着企业业务的拓展，企业可能在多用户环境中运行多个工作任务，这就需要一个可以对安全策略进行集中管理，配置和监控用户访问的框架。Ranger由此产生！

Ranger的官网：https://ranger.apache.org/

## 1.2 Ranger的目标

- 允许用户使用UI或REST API对所有和安全相关的任务进行集中化的管理
- 允许用户使用一个管理工具对操作Hadoop体系中的组件和工具的行为进行细粒度的授权
- 支持Hadoop体系中各个组件的授权认证标准
- 增强了对不同业务场景需求的授权方法支持，例如基于角色的授权或基于属性的授权
- 支持对Hadoop组件所有涉及安全的审计行为的集中化管理

## 1.3 Ranger支持的框架

- Apache Hadoop
- Apache Hive
- Apache HBase
- Apache Storm
- Apache Knox
- Apache Solr
- Apache Kafka
- YARN
- NIFI

## 1.4 Ranger的架构

{% asset_img Ranger的架构.png This is an example image %}

## 1.5 Ranger的工作原理

Ranager的核心是Web应用程序，也称为RangerAdmin模块，此模块由管理策略，审计日志和报告等三部分组成。

管理员角色的用户可以通过RangerAdmin提供的web界面或REST APIS来定制安全策略。这些策略会由Ranger提供的轻量级的针对不同Hadoop体系中组件的插件来执行。插件会在Hadoop的不同组件的核心进程启动后，启动对应的插件进程来进行安全管理！

# 第2章 Ranger的安装

## 2.1 环境说明

Ranger2.0要求对应的Hadoop为3.x以上，Hive为3.x以上版本，JDK为1.8以上版本。Hadoop及Hive等需开启用户认证功能，本文基于开启Kerberos安全认证的Hadoop和Hive环境。

注：本文中所涉及的Ranger相关组件均安装在hadoop102节点。

## 2.2 创建系统用户和Kerberos主体

Ranger的启动和运行需使用特定的用户，故须在Ranger所在节点创建所需系统用户并在Kerberos中创建所需主体。

**1.**** 创建 ***\*ranger\**** 系统用户**

```shell
[root@hadoop102 ~]# useradd ranger -G hadoop

[root@hadoop102 ~]# echo ranger | passwd --stdin ranger
```

**2.**** 检查 ***\*HTTP\**** 主体是否正常（该主体在 ***\*Hadoop\**** 开启 ***\*Kerberos\**** 时已创建）**

1）使用keytab文件认证HTTP主体

```shell
[root@hadoop102 ~]# kinit -kt /etc/security/keytab/spnego.service.keytab HTTP/hadoop102@EXAMPLE.COM
```

2）查看认证状态

```shell
[root@hadoop102 ~]# klist
```

3）注销认证

```shell
[root@hadoop102 ~]# kdestroy
```

**3.**** 创建 ***\*rangeradmin\**** 主体**

1）创建主体

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"addprinc -randkey rangeradmin/hadoop102"
```

2）生成keytab文件

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"xst -k /etc/security/keytab/rangeradmin.keytab rangeradmin/hadoop102"
```

3）修改keytab文件所有者

```shell
[root@hadoop102 ~]# chown ranger:ranger /etc/security/keytab/rangeradmin.keytab
```

**4.**** 创建 ***\*rangerlookup\**** 主体**

1）创建主体

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"addprinc -randkey rangerlookup/hadoop102"
```

2）生成keytab文件

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"xst -k /etc/security/keytab/rangerlookup.keytab rangerlookup/hadoop102"
```

3）修改keytab文件所有者

```shell
[root@hadoop102 ~]# chown ranger:ranger /etc/security/keytab/rangerlookup.keytab
```

**5.**** 创建 ***\*rangerusersync\**** 主体**

1）创建主体

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"addprinc -randkey rangerusersync/hadoop102"
```

2）生成keytab文件

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"xst -k /etc/security/keytab/rangerusersync.keytab rangerusersync/hadoop102"
```

3）修改keytab文件所有者

```shell
[root@hadoop102 ~]# chown ranger:ranger /etc/security/keytab/rangerusersync.keytab
```

## 2.2 安装RangerAdmin

### 2.2.1 数据库环境准备

（1）登录MySQL

```shell
[root@hadoop102 ~]# mysql -uroot -p000000
```

（2）在MySQL数据库中创建Ranger存储数据的数据库

```shell
mysql&gt; create database ranger;
```

（3）更改mysql密码策略，为了可以采用比较简单的密码

```shell
mysql&gt; set global validate_password_length=4;

mysql&gt; set global validate_password_policy=0;
```

（4）创建用户

```shell
mysql&gt; grant all privileges on ranger.* to ranger@'%' identified by 'ranger';
```

### 2.2.2 安装RangerAdmin

1.在hadoop102的/opt/module路径上创建一个ranger

```shell
[root@hadoop102 ~]# mkdir /opt/module/ranger
```

2.解压软件

```shell
[root@hadoop102 software]# tar -zxvf ranger-2.0.0-admin.tar.gz -C /opt/module/ranger
```

3.进入/opt/module/ranger/ranger-2.0.0-admin路径，对install.properties配置

```shell
[root@hadoop102 ranger-2.0.0-admin]# vim install.properties
```

修改以下配置内容：

```properties
#mysql驱动

SQL_CONNECTOR_JAR=/opt/software/mysql-connector-java-5.1.48.jar

#mysql的主机名和root用户的用户名密码

db_root_user=root

db_root_password=000000

db_host=hadoop102

#ranger需要的数据库名和用户信息，和2.2.1创建的信息要一一对应

db_name=ranger

db_user=ranger

db_password=ranger

#Ranger各组件的admin用户密码

rangerAdmin_password=atguigu123

rangerTagsync_password=atguigu123

rangerUsersync_password=atguigu123

keyadmin_password=atguigu123

#ranger存储审计日志的路径，默认为solr，这里为了方便暂不设置

audit_store=

#策略管理器的url,rangeradmin安装在哪台机器，主机名就为对应的主机名

policymgr_external_url=http://hadoop102:6080

#启动ranger admin进程的linux用户信息

unix_user=ranger

unix_user_pwd=ranger

unix_group=ranger

#Kerberos相关配置

spnego_principal=HTTP/hadoop102@EXAMPLE.COM

spnego_keytab=/etc/security/keytab/spnego.service.keytab

admin_principal=rangeradmin/hadoop102@EXAMPLE.COM

admin_keytab=/etc/security/keytab/rangeradmin.keytab

lookup_principal=rangerlookup/hadoop102@EXAMPLE.COM

lookup_keytab=/etc/security/keytab/rangerlookup.keytab

hadoop_conf=/opt/module/hadoop-3.1.3/etc/hadoop
```

4.在/opt/module/ranger/ranger-2.0.0-admin目录下执行安装脚本

```shell
[root@hadoop102 ranger-2.0.0-admin]# ./setup.sh
```

出现以下信息，说明安装完成

2020-04-30 13:58:18,051 [I] Ranger all admins default password change request processed successfully..

Installation of Ranger PolicyManager Web Application is completed.

5.修改/opt/module/ranger/ranger-2.0.0-admin/conf/ranger-admin-site.xml配置文件中的以下属性。

```shell
[root@hadoop102 ranger-2.0.0-admin]# vim /opt/module/ranger/ranger-2.0.0-admin/conf/ranger-admin-site.xml
```

修改如下参数

```xml
<property>
    <name>ranger.jpa.jdbc.password</name>
    <value>ranger</value>
    <description />
</property>

<property>
    <name>ranger.service.host</name>
    <value>hadoop102</value>
</property>
```

### 2.2.3 启动RangerAdmin

1.启动ranger-admin（以ranger用户启动）

```shell
[root@hadoop102 ranger-2.0.0-admin]# sudo -i -u ranger ranger-admin start
```

Starting Apache Ranger Admin Service

Apache Ranger Admin Service with pid 7058 has started.

ranger-admin在安装时已经配设置为开机自启动，因此之后无需再手动启动！

2.查看启动后的进程

```shell
[root@hadoop102 ranger-2.0.0-admin]# jps

7058 EmbeddedServer

8132 Jps
```

3.访问Ranger的WebUI，地址为：[http://hadoop102:6080](http://hadoop103:6080/)

{% asset_img WebUI.png This is an example image %}

4.停止ranger（此处不用执行）

```shell
[root@hadoop102 ranger-2.0.0-admin]# sudo -i -u ranger ranger-admin stop
```

### 2.2.4 登录Ranger

默认可以使用用户名：admin，密码为之前配置的atguigu123进行登录！登录后界面如下：

{% asset_img 登录后界面.png This is an example image %}

# 第3章 安装 RangerUsersync

## 3.1 RangerUsersync简介

RangerUsersync作为Ranger提供的一个管理模块，可以将Linux机器上的用户和组信息同步到RangerAdmin的数据库中进行管理。

## 3.2 RangerUsersync安装

1.解压软件

```shell
[root@hadoop102 software]# tar -zxvf ranger-2.0.0-usersync.tar.gz -C /opt/module/ranger/
```

2.配置软件

在/opt/module/ranger/ranger-2.0.0-usersync目录下修改以下文件

```shell
[root@hadoop102 ranger-2.0.0-usersync]# vim install.properties
```

修改以下配置信息

```properties
#rangeradmin的url

POLICY_MGR_URL =http://hadoop102:6080

#同步间隔时间，单位(分钟)

SYNC_INTERVAL = 1

#运行此进程的linux用户

unix_user=ranger

unix_group=ranger

#rangerUserSync用户的密码，参考rangeradmin中install.properties的配置

rangerUsersync_password=atguigu123

#Kerberos相关配置

usersync_principal=rangerusersync/hadoop102@EXAMPLE.COM

usersync_keytab=/etc/security/keytab/rangerusersync.keytab

hadoop_conf=/opt/module/hadoop-3.1.3/etc/hadoop
```

3.在/opt/module/ranger/ranger-2.0.0-usersync目录下执行安装脚本

```shell
[root@hadoop102 ranger-2.0.0-usersync]# ./setup.sh
```

出现以下信息，说明安装完成

ranger.usersync.policymgr.password has been successfully created.

Provider jceks://file/etc/ranger/usersync/conf/rangerusersync.jceks was updated.

[I] Successfully updated password of rangerusersync user

4.修改/opt/module/ranger/ranger-2.0.0-usersync/conf/ranger-ugsync-site.xml配置文件中的以下参数

```xml
<property>
      <name>ranger.usersync.enabled</name>
      <value>true</value>
</property>
```

## 3.3 RangerUsersync启动

1.启动之前，在ranger admin的web-UI界面，查看用户信息如下：

{% asset_img 查看用户信息.png This is an example image %}

2.启动RangerUserSync（使用ranger用户启动）

```shell
[root@hadoop102 ranger-2.0.0-usersync]# sudo -i -u ranger ranger-usersync start

Starting Apache Ranger Usersync Service
Apache Ranger Usersync Service with pid 7510 has started.
```

3.启动后，再次查看用户信息：

{% asset_img 再次查看.png This is an example image %}

说明ranger-usersync工作正常！

ranger-usersync服务也是开机自启动的，因此之后不需要手动启动！

# 第4章 安装Ranger Hive-plugin

## 4.1 Ranger Hive-plugin简介

Ranger Hive-plugin是Ranger对hive进行权限管理的插件。需要注意的是，Ranger Hive-plugin只能对使用jdbc方式访问hive的请求进行权限管理，hive-cli并不受限制。

## 4.2 Ranger Hive-plugin安装

1.解压软件

```shell
[root@hadoop102 software]# tar -zxvf ranger-2.0.0-hive-plugin.tar.gz -C /opt/module/ranger/
```

2.配置软件

```shell
[root@hadoop102 ranger-2.0.0-hive-plugin]# vim install.properties
```

修改以下内容

```properties
#策略管理器的url地址

POLICY_MGR_URL=http://hadoop102:6080

#组件名称

REPOSITORY_NAME=hive

#hive的安装目录

COMPONENT_INSTALL_DIR_NAME=/opt/module/hive

#hive组件的启动用户

CUSTOM_USER=hive

#hive组件启动用户所属组

CUSTOM_GROUP=hadoop
```

3.启用Ranger Hive-plugin，在/opt/module/ranger/ranger-2.0.0-hive-plugin下执行以下命令

```shell
[root@hadoop102 ranger-2.0.0-hive-plugin]# ./enable-hive-plugin.sh
```

查看$HIVE_HOME/conf目录是否出现以下配置文件，如出现则表示Hive插件启用成功。

```shell
[root@hadoop102 ranger-2.0.0-hive-plugin]# ls $HIVE_HOME/conf | grep -E hiveserver2|ranger
```

hiveserver2-site.xml

ranger-hive-audit.xml

ranger-hive-security.xml

ranger-policymgr-ssl.xml

ranger-security.xml

4.重启Hiveserver2，需使用hive用户启动。

```shell
[root@hadoop102 ~]# sudo -i -u hive hiveserver2
```

## 4.3 在ranger admin上配置hive插件

1.授予hive用户在Ranger中的Admin角色

1）点击hive用户

{% asset_img 点击hive用户.png This is an example image %}

2）将角色设置为Admin

{% asset_img 设置为Admin.png This is an example image %}

2.配置Hive插件

1）点击Access Manager，添加Hive Manager。

{% asset_img 添加Hive Manager.png This is an example image %}

2）配置服务详情

{% asset_img 服务详情.png This is an example image %}

注：

Service Name：hive

Username：rangerlookup

Password：rangerlookup

jdbc.driverClassName：org.apache.hive.jdbc.HiveDriver

jdbc.url：jdbc:hive2://hadoop102:10000/;principal=hive/hadoop102@EXAMPLE.COM

3）测试连接

点击测试连接

{% asset_img 点击测试.png This is an example image %}

点击测试连接后会提示连接失败，具体原因是rangerlookup用户没有访问hive表的权限，这是因为到目前为止，我们还未使用Ranger向任何用户赋予任何权限，故此时连接失败为正常现象。

{% asset_img 正常现象.png This is an example image %}

4）保存Hive Manager

（1）点击Add按钮

{% asset_img Add按钮.png This is an example image %}

（2）点击下图所示hive按钮

{% asset_img hive按钮.png This is an example image %}

下图内容表示，目前rangerlookup用户已经拥有了Hive所有资源的所有权限。

{% asset_img Hive所有资源.png This is an example image %}

5）重新测试连接

（1）点击下图编辑按钮

{% asset_img 编辑按钮.png This is an example image %}

（2）重新点击Test Connection

{% asset_img 点击Test.png This is an example image %}

（3）连接成功

{% asset_img 连接成功.png This is an example image %}

# 第5章 使用Ranger对Hive进行权限管理

## 5.1 权限控制初体验

1.查看默认的访问策略，此时只有rangerlookup用户拥有对所有库、表和函数的访问权限，故理论上其余用户是不能访问任何Hive资源的。

{% asset_img 默认的访问策略.png This is an example image %}

2.验证：使用atguigu用户尝试进行认证，认证成功后，使用beeline客户端连接Hiveserver2

1）使用atguigu用户认证，并按照提示输入密码

```shell
[atguigu@hadoop102 ~]$ kinit atguigu
```

2）登录beeline客户端

```shell
[atguigu@hadoop102 ~]$ beeline -u "jdbc:hive2://hadoop102:10000/;principal=hive/hadoop102@EXAMPLE.COM"
```

3）执行以下sql语句，验证当前用户为atguigu

{% asset_img 验证当前用户.png This is an example image %}

4）执行use gmall语句，结果如图所示，atguigu用户没有对gmall库的使用权限

{% asset_img 执行use gmall.png This is an example image %}

5）赋予atguigu用户对gmall数据库的访问权限

1）点击Add New Policy

{% asset_img 点击Add.png This is an example image %}

2）配置授权策略

如下图所示，将gmall库的所有表的所有权限均授予给了atguigu用户。

{% asset_img 配置授权策略.png This is an example image %}

3）等待片刻，在回到beeline客户端，重新执行use gmall语句，此时atguigu用户已经能够使用gmall库，并且可访问gmall库下的所有表了。

## 5.2 Ranger授权模型

Ranger所采用的权限管理模型可归类为RBAC（Role-Based Access Control ）基于角色的访问控制。基础的RBAC模型共包含三个实体，分别是用户（user）、角色（role）和权限（permission）。用户需划分为某个角色，权限的授予对象也是角色，例如用户张三为管理角色，那他就拥有了管理员角色的所有权限。

Ranger的权限管理模型比基础的RBAC模型要更加灵活，以下是Ranger的权限管理模型。

{% asset_img 权限管理模型.png This is an example image %}

# 第6章 官网其他权限配置

更多配置，可以参考官网介绍：https://cwiki.apache.org/confluence/display/RANGER/Row-level+filtering+and+column-masking+using+Apache+Ranger+policies+in+Apache+Hive

[大数据项目之尚品汇（10元数据管理Atlas）](https://bigworldxld.github.io/2022/02/15/undefined.html)