---
title: 大数据项目之尚品汇（10元数据管理Atlas）
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
abbrlink: 14181
date: 2022-02-15 13:51:56
---

# 第1章 Atlas入门

## 1.1 Atlas概述

Apache Atlas为组织提供开放式元数据管理和治理功能，用以构建其数据资产目录，对这些资产进行分类和管理，并为数据分析师和数据治理团队，提供围绕这些数据资产的协作功能。

Atlas的具体功能如下：

| **元数据分类** | **支持对元数据进行分类管理，例如个人信息，敏感信息等**       |
| -------------- | ------------------------------------------------------------ |
| **元数据检索** | **可按照元数据类型、元数据分类进行检索，支持全文检索**       |
| **血缘依赖**   | **支持表到表和字段到字段之间的血缘依赖，便于进行问题回溯和影响分析等** |

1）表与表之间的血缘依赖

{% asset_img 血缘依赖.png This is an example image %}

2）字段与字段之间的血缘依赖

{% asset_img 字段与字段.png This is an example image %}

## 1.2 Atlas架构原理

{% asset_img Atlas架构原理.png This is an example image %}

# 第2章 Atlas安装

1）Atlas官网地址：https://atlas.apache.org/

2）文档查看地址：https://atlas.apache.org/2.1.0/index.html

3）下载地址：https://www.apache.org/dyn/closer.cgi/atlas/2.1.0/apache-atlas-2.1.0-sources.tar.gz

## 2.1 安装环境准备

Atlas安装分为：集成自带的HBase + Solr；集成外部的HBase + Solr。通常企业开发中选择集成外部的HBase + Solr，方便项目整体进行集成操作。

以下是Atlas所以依赖的环境及集群规划。本文只包含Solr和Atlas的安装指南，其余所依赖服务的安装请参考前边章节。

| 服务名称      | 子服务         | 服务器hadoop102 | 服务器hadoop103 | 服务器hadoop104 |
| ------------- | -------------- | --------------- | --------------- | --------------- |
| JDK           |                | √               | √               | √               |
| Zookeeper     | QuorumPeerMain | √               | √               | √               |
| Kafka         | Kafka          | √               | √               | √               |
| HBase         | HMaster        | √               |                 |                 |
| HRegionServer | √              | √               | √               |                 |
| Solr          | Jar            | √               | √               | √               |
| Hive          | Hive           | √               |                 |                 |
| Atlas         | atlas          | √               |                 |                 |
| 服务数总计    |                | 13              | 7               | 7               |

### 2.1.1 安装Solr-7.7.3

1.在每台节点创建系统用户solr

```shell
[root@hadoop102 ~]# useradd solr

[root@hadoop102 ~]# echo solr | passwd --stdin solr

[root@hadoop103 ~]# useradd solr

[root@hadoop103 ~]# echo solr | passwd --stdin solr

[root@hadoop104 ~]# useradd solr

[root@hadoop104 ~]# echo solr | passwd --stdin solr
```

2.解压solr-7.7.3.tgz到/opt/module目录，并改名为solr

```shell
[root@hadoop102 ~]# tar -zxvf solr-7.7.3.tgz -C /opt/module/

[root@hadoop102 ~]# mv solr-7.7.3/ solr
```

3.修改solr目录的所有者为solr用户

```shell
[root@hadoop102 ~]# chown -R solr:solr /opt/module/solr
```

4.修改solr配置文件

修改/opt/module/solr/bin/solr.in.sh文件中的以下属性

```shell
ZK_HOST="hadoop102:2181,hadoop103:2181,hadoop104:2181"
```

5.分发solr

```shell
[root@hadoop102 ~]# xsync /opt/module/solr
```

6.启动solr集群

1）启动Zookeeper集群

```shell
[root@hadoop102 ~]# zk.sh start
```

2）启动solr集群

出于安全考虑，不推荐使用root用户启动solr，此处使用solr用户，在所有节点执行以下命令启动solr集群

```shell
[root@hadoop102 ~]#sudo -i -u solr /opt/module/solr/bin/solr start

[root@hadoop103 ~]# sudo -i -u solr /opt/module/solr/bin/solr start

[root@hadoop104 ~]# sudo -i -u solr /opt/module/solr/bin/solr start
```

出现 **Happy Searching!** 字样表明启动成功。

{% asset_img 字样表明启动.png This is an example image %}

说明：上述警告内容是：solr推荐系统允许的最大进程数和最大打开文件数分别为65000和65000，而系统默认值低于推荐值。如需修改可参考以下步骤，修改完需要重启方可生效， **此处可暂不修改。**

（1）修改打开文件数限制

修改/etc/security/limits.conf文件，增加以下内容

```properties
* soft nofile 65000

* hard nofile 65000
```

（2）修改进程数限制

修改/etc/security/limits.d/20-nproc.conf文件

```properties
* soft nproc **65000**
```

（3）重启服务器

7.访问web页面

默认端口为8983，可指定三台节点中的任意一台IP，[http://hadoop102:8983](http://hadoop102:8983/)

{% asset_img web页面.png This is an example image %}

提示：UI界面出现Cloud菜单栏时，Solr的Cloud模式才算部署成功。

### 2.1.8 安装Atlas2.1.0

1.把apache-atlas-2.1.0-bin.tar.gz 上传到hadoop102的/opt/software目录下

2.解压apache-atlas-2.1.0-server.tar.gz 到/opt/module/目录下面

```shell
[root@hadoop102 software]# tar -zxvf apache-atlas-2.1.0-bin.tar.gz -C /opt/module/
```

3.修改apache-atlas-2.1.0的名称为atlas

```shell
[root@hadoop102 ~]# mv /opt/module/apache-atlas-2.1.0 /opt/module/atlas
```

## 2.2 Atlas配置

### 2.2.1 Atlas集成Hbase

1.修改/opt/module/atlas/conf/atlas-application.properties配置文件中的以下参数

```shell
atlas.graph.storage.hostname=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

2.修改/opt/module/atlas/conf/atlas-env.sh配置文件，增加以下内容

```shell
export HBASE_CONF_DIR=/opt/module/hbase/conf
```

### 2.2.2 Atlas集成Solr

1.修改/opt/module/atlas/conf/atlas-application.properties配置文件中的以下参数

```properties
atlas.graph.index.search.backend=solr

atlas.graph.index.search.solr.mode=cloud

atlas.graph.index.search.solr.zookeeper-url=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

2.创建solr collection

```shell
[root@hadoop102 ~]# sudo -i -u solr /opt/module/solr/bin/solr create -c vertex_index -d /opt/module/atlas/conf/solr -shards 3 -replicationFactor 2

[root@hadoop102 ~]# sudo -i -u solr /opt/module/solr/bin/solr create -c edge_index -d /opt/module/atlas/conf/solr -shards 3 -replicationFactor 2

[root@hadoop102 ~]# sudo -i -u solr /opt/module/solr/bin/solr create -c fulltext_index -d /opt/module/atlas/conf/solr -shards 3 -replicationFactor 2
```

### 2.2.3 Atlas集成Kafka

修改/opt/module/atlas/conf/atlas-application.properties配置文件中的以下参数

```properties
atlas.notification.embedded=false

atlas.kafka.data=/opt/module/kafka/data

atlas.kafka.zookeeper.connect= hadoop102:2181,hadoop103:2181,hadoop104:2181/kafka

atlas.kafka.bootstrap.servers=hadoop102:9092,hadoop103:9092,hadoop104:9092
```

### 2.2.4 Atlas Server配置

1.修改/opt/module/atlas/conf/atlas-application.properties配置文件中的以下参数

```properties
######### Server Properties #########

atlas.rest.address=http://hadoop102:21000

# If enabled and set to true, this will run setup steps when the server starts

atlas.server.run.setup.on.start=false

\######### Entity Audit Configs #########

atlas.audit.hbase.zookeeper.quorum=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

2.记录性能指标，进入/opt/module/atlas/conf/路径，修改当前目录下的atlas-log4j.xml

```shell
[root@hadoop101 conf]# vim atlas-log4j.xml
```

```xml
#去掉如下代码的注释
<appender name="perf_appender" class="org.apache.log4j.DailyRollingFileAppender">
    <param name="file" value="${atlas.log.dir}/atlas_perf.log" />
    <param name="datePattern" value="'.'yyyy-MM-dd" />
    <param name="append" value="true" />
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d|%t|%m%n" />
    </layout>
</appender>

<logger name="org.apache.atlas.perf" additivity="false">
    <level value="debug" />
    <appender-ref ref="perf_appender" />
</logger>
```

### 2.2.5 Kerberos相关配置

若Hadoop集群开启了Kerberos认证，Atlas与Hadoop集群交互之前就需要先进行Kerberos认证。若Hadoop集群未开启Kerberos认证，则本节可跳过。

1.为Atlas创建Kerberos主体，并生成keytab文件

```shell
[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"addprinc -randkey atlas/hadoop102"

[root@hadoop102 ~]# kadmin -padmin/admin -wadmin -q"xst -k /etc/security/keytab/atlas.service.keytab atlas/hadoop102"
```

2.修改/opt/module/atlas/conf/atlas-application.properties配置文件，增加以下参数

```properties
atlas.authentication.method=kerberos

atlas.authentication.principal=atlas/hadoop102@EXAMPLE.COM

atlas.authentication.keytab=/etc/security/keytab/atlas.service.keytab
```

### 2.2.6 Atlas集成Hive

1.安装Hive Hook

1）解压Hive Hook

```shell
[root@hadoop102 ~]# tar -zxvf apache-atlas-2.1.0-hive-hook.tar.gz
```

2）将Hive Hook依赖复制到Atlas安装路径

```shell
[root@hadoop102 ~]# cp -r apache-atlas-hive-hook-2.1.0/* /opt/module/atlas/
```

3）修改/opt/module/hive/conf/hive-env.sh配置文件

注：需先需改文件名

```shell
[root@hadoop102 ~]# mv hive-env.sh.template hive-env.sh
```

增加如下参数

```shell
export HIVE_AUX_JARS_PATH=/opt/module/atlas/hook/hive
```

2.修改Hive配置文件，在/opt/module/hive/conf/hive-site.xml文件中增加以下参数，配置Hive Hook。

```xml
<property>
      <name>hive.exec.post.hooks</name>
      <value>org.apache.atlas.hive.hook.HiveHook</value>
</property>
```

3.修改/opt/module/atlas/conf/atlas-application.properties配置文件中的以下参数

```properties
######### Hive Hook Configs #######

atlas.hook.hive.synchronous=false

atlas.hook.hive.numRetries=3

atlas.hook.hive.queueSize=10000

atlas.cluster.name=primary
```

4）将Atlas配置文件/opt/module/atlas/conf/atlas-application.properties

拷贝到/opt/module/hive/conf目录

```shell
[root@hadoop102 ~]# cp /opt/module/atlas/conf/atlas-application.properties /opt/module/hive/conf/
```

## 2.4 Atlas启动

1.启动Atlas所依赖的环境

1）启动Hadoop集群

（1）在NameNode节点执行以下命令，启动HDFS

```shell
[root@hadoop102 ~]# start-dfs.sh
```

（2）在ResourceManager节点执行以下命令，启动Yarn

```shell
[root@hadoop103 ~]# start-yarn.sh
```

2）启动Zookeeper集群

```shell
[root@hadoop102 ~]# zk.sh start
```

3）启动Kafka集群

```
[root@hadoop102 ~]# kf.sh start
```

4）启动Hbase集群

在HMaster节点执行以下命令，使用hbase用户启动HBase

```shell
[root@hadoop102 ~]# sudo -i -u hbase start-hbase.sh
```

5）启动Solr集群

在所有节点执行以下命令，使用solr用户启动Solr

```shell
[root@hadoop102 ~]# sudo -i -u solr /opt/module/solr/bin/solr start

[root@hadoop103 ~]# sudo -i -u solr /opt/module/solr/bin/solr start

[root@hadoop104 ~]# sudo -i -u solr /opt/module/solr/bin/solr start
```

6）进入/opt/module/atlas路径，启动Atlas服务

```shell
[root@hadoop102 atlas]# bin/atlas_start.py
```

提示：

（1）错误信息查看路径：/opt/module/atlas/logs/*.out和application.log

（2）停止Atlas服务命令为atlas_stop.py

7）访问Atlas的WebUI

访问地址：[http://hadoop102:21000](http://hadoop102:21000/)

注意：等待若干分钟。

账户：admin

密码：admin

{% asset_img Atlas的WebUI.png This is an example image %}

# 第3章 Atlas使用

Atlas的使用相对简单，其主要工作是同步各服务（主要是Hive）的元数据，并构建元数据实体之间的关联关系，然后对所存储的元数据建立索引，最终未用户提供数据血缘查看及元数据检索等功能。

Atlas在安装之初，需手动执行一次元数据的全量导入，后续Atlas便会利用Hive Hook增量同步Hive的元数据。

## 3.1 Hive元数据初次导入

Atlas提供了一个Hive元数据导入的脚本，直接执行该脚本，即可完成Hive元数据的初次全量导入。

1.导入Hive元数据

执行以下命令

```shell
[root@hadoop102 ~]# /opt/module/atlas/hook-bin/import-hive.sh
```

按提示输入用户名：admin；输入密码：admin

```shell
Enter username for atlas :- admin

Enter password for atlas :-
```

等待片刻，出现以下日志，即表明导入成功

Hive Meta Data import was successful!!!

2.查看Hive元数据

1）搜索hive_table类型的元数据，可已看到Atlas已经拿到了Hive元数据

{% asset_img Hive元数据.png This is an example image %}

2）任选一张表查看血缘依赖关系

发现此时并未出现期望的血缘依赖，原因是Atlas是根据Hive所执行的SQL语句获取表与表之间以及字段与字段之间的依赖关系的，例如执行insert into table_a select * from table_b语句，Atlas就能获取table_a与table_b之间的依赖关系。此时并未执行任何SQL语句，故还不能出现血缘依赖关系。

{% asset_img 血缘依赖关系.png This is an example image %}

## 3.2 Hive元数据增量同步

Hive元数据的增量同步，无需人为干预，只要Hive中的元数据发生变化（执行DDL语句），Hive Hook就会将元数据的变动通知Atlas。除此之外，Atlas还会根据DML语句获取数据之间的血缘关系。

### 3.2.1 全流程调度

为查看血缘关系效果，此处使用Azkaban将数仓的全流程调度一次。

**1.**** 新数据准备**

**1**** ）用户行为日志**

（1）启动日志采集通道，包括Zookeeper，Kafka，Flume等

（2）修改hadoop102，hadoop103两台节点的/opt/module/applog/application.yml文件，将模拟日期改为2020-06-17如下

\#业务日期

mock.date: "2020-06-17"

（3）执行生成日志的脚本

```shell
lg.sh
```

（4）等待片刻，观察HDFS是否出现2020-06-17的日志文件

**2**** ）业务数据**

（1）修改/opt/module/db_log/application.properties，将模拟日期修改为2020-06-17，如下

\#业务日期

mock.date=2020-06-17

（2）进入到/opt/module/db_log路径，执行模拟生成业务数据的命令，如下

```shell
java -jar gmall2020-mock-db-2021-01-22.jar
```

（3）观察mysql的gmall数据中是否出现2020-06-17的数据

**2.**** 启动 ****Azkaban**

注意需使用azkaban用户启动Azkaban

1）启动Executor Server

在各节点执行以下命令，启动Executor

```shell
[root@hadoop102 ~]# sudo -i -u azkaban bash -c "cd /opt/module/azkaban/azkaban-exec;bin/start-exec.sh"

[root@hadoop103 ~]# sudo -i -u azkaban bash -c "cd /opt/module/azkaban/azkaban-exec;bin/start-exec.sh"

[root@hadoop104 ~]# sudo -i -u azkaban bash -c "cd /opt/module/azkaban/azkaban-exec;bin/start-exec.sh"
```

2）激活Executor Server，任选一台节点执行以下激活命令即可

```shell
[root@hadoop102 ~]# curl http://hadoop102:12321/executor?action=activate

[root@hadoop102 ~]# curl http://hadoop103:12321/executor?action=activate

[root@hadoop102 ~]# curl http://hadoop104:12321/executor?action=activate
```

3）启动Web Server

```shell
[root@hadoop102 ~]# sudo -i -u azkaban bash -c "cd /opt/module/azkaban/azkaban-web;bin/start-web.sh"
```

**3.**** 全流程调度**

1）工作流参数

{% asset_img 工作流参数.png This is an example image %}

2）运行结果

{% asset_img 运行结果.png This is an example image %}

### 3.2.2 查看血缘依赖

此时在通过Atlas查看Hive元数据，即可发现血缘依赖

{% asset_img 查看Hive元数据.png This is an example image %}

# 第4章 扩展内容

## 4.1 Atlas源码编译

### 4.1.1 安装Maven

1）Maven下载：https://maven.apache.org/download.cgi

2）把apache-maven-3.6.1-bin.tar.gz上传到linux的/opt/software目录下

3）解压apache-maven-3.6.1-bin.tar.gz到/opt/module/目录下面

```shell
[root@hadoop102 software]# tar -zxvf apache-maven-3.6.1-bin.tar.gz -C /opt/module/
```

4）修改apache-maven-3.6.1的名称为maven

```shell
[root@hadoop102 module]# mv apache-maven-3.6.1/ maven
```

5）添加环境变量到/etc/profile中

```shell
[root@hadoop102 module]#vim /etc/profile
#MAVEN_HOME

export MAVEN_HOME=/opt/module/maven

export PATH=$PATH:$MAVEN_HOME/bin
```

6）测试安装结果

```shell
[root@hadoop102 module]# source /etc/profile

[root@hadoop102 module]# mvn -v
```

7）修改setting.xml，指定为阿里云

```shell
[root@hadoop101 module]# cd /opt/module/maven/conf/

[root@hadoop102 maven]# vim settings.xml
```

```xml
<!-- 添加阿里云镜像-->
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
<url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
<mirror>
    <id>UK</id>
    <name>UK Central</name>
    <url>http://uk.maven.org/maven2</url>
    <mirrorOf>central</mirrorOf>
</mirror>
<mirror>
    <id>repo1</id>
    <mirrorOf>central</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://repo1.maven.org/maven2/</url>
</mirror>
<mirror>
    <id>repo2</id>
    <mirrorOf>central</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://repo2.maven.org/maven2/</url>
</mirror>
```

### 4.1.2 编译Atlas源码

1）把apache-atlas-2.1.0-sources.tar.gz上传到hadoop102的/opt/software目录下

2）解压apache-atlas-2.1.0-sources.tar.gz到/opt/module/目录下面

```shell
[root@hadoop101 software]# tar -zxvf apache-atlas-2.1.0-sources.tar.gz -C /opt/module/
```

3）下载Atlas依赖

```shell
[root@hadoop101 software]# export MAVEN_OPTS="-Xms2g -Xmx2g"

[root@hadoop101 software]# cd /opt/module/apache-atlas-sources-2.1.0/

[root@hadoop101 apache-atlas-sources-2.1.0]# mvn clean -DskipTests install

[root@hadoop101 apache-atlas-sources-2.1.0]# mvn clean -DskipTests package -Pdis
```

\#一定要在${atlas_home}执行

```shell
[root@hadoop101 apache-atlas-sources-2.1.0]# cd distro/target/

[root@hadoop101 target]# mv apache-atlas-2.1.0-server.tar.gz /opt/software/

[root@hadoop101 target]# mv apache-atlas-2.1.0-hive-hook.tar.gz /opt/software/
```

提示：执行过程比较长，会下载很多依赖，大约需要半个小时，期间如果报错很有可能是因为TimeOut造成的网络中断，重试即可。

## 4.2 Atlas内存配置

如果计划存储数万个元数据对象，建议调整参数值获得最佳的JVM GC性能。以下是常见的服务器端选项

1）修改配置文件/opt/module/atlas/conf/atlas-env.sh

```sh
#设置Atlas内存

export ATLAS_SERVER_OPTS="-server -XX: **SoftRefLRUPolicyMSPerMB** =0 -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+PrintTenuringDistribution -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=dumps/atlas_server.hprof -Xloggc:logs/gc-worker.log -verbose:gc -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=1m -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCTimeStamps"

\#建议JDK1.7使用以下配置

export ATLAS_SERVER_HEAP="-Xms15360m -Xmx15360m -XX:MaxNewSize=3072m -XX:PermSize=100M -XX:MaxPermSize=512m"

\#建议JDK1.8使用以下配置

export ATLAS_SERVER_HEAP="-Xms15360m -Xmx15360m -XX:MaxNewSize=5120m -XX:MetaspaceSize=100M -XX:MaxMetaspaceSize=512m"

\#如果是Mac OS用户需要配置

export ATLAS_SERVER_OPTS="-Djava.awt.headless=true -Djava.security.krb5.realm= -Djava.security.krb5.kdc="
```

参数说明： -XX:SoftRefLRUPolicyMSPerMB 此参数对管理具有许多并发用户的查询繁重工作负载的GC性能特别有用。

## 4.3 配置用户名密码

Atlas支持以下身份验证方法：File、Kerberos协议、LDAP协议

通过修改配置文件atlas-application.properties文件开启或关闭三种验证方法

```properties
atlas.authentication.method.kerberos=true|false

atlas.authentication.method.ldap=true|false

atlas.authentication.method.file=true|false
```

如果两个或多个身份证验证方法设置为true，如果较早的方法失败，则身份验证将回退到后一种方法。例如，如果Kerberos身份验证设置为true并且ldap身份验证也设置为true，那么，如果对于没有kerberos principal和keytab的请求，LDAP身份验证将作为后备方案。

**本文主要讲解采用文件方式修改用户名和密码设置。其他方式可以参见官网配置即可。**

1）打开/opt/module/atlas/conf/users-credentials.properties文件

```shell
[atguigu@hadoop102 conf]$ vim users-credentials.properties
```

```properties
#username=group::sha256-password

admin=ADMIN::8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918

rangertagsync=RANGER_TAG_SYNC::e3f67240f5117d1753c940dae9eea772d36ed5fe9bd9c94a300e40413f1afb9d
```

（1）admin是用户名称

（2）8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918是采用sha256加密的密码，默认密码为admin。

2）例如：修改用户名称为atguigu，密码为atguigu

（1）获取sha256加密的atguigu密码

```shell
[atguigu@hadoop102 conf]$ echo -n "atguigu"|sha256sum

2628be627712c3555d65e0e5f9101dbdd403626e6646b72fdf728a20c5261dc2
```

（2）修改用户名和密码

```shell
[atguigu@hadoop102 conf]$ vim users-credentials.properties

\#username=group::sha256-password

atguigu=ADMIN::2628be627712c3555d65e0e5f9101dbdd403626e6646b72fdf728a20c5261dc2

rangertagsync=RANGER_TAG_SYNC::e3f67240f5117d1753c940dae9eea772d36ed5fe9bd9c94a300e40413f1afb9d
```

[大数据项目之尚品汇（11数据质量管理）](https://bigworldxld.github.io/2022/02/16/undefined.html)