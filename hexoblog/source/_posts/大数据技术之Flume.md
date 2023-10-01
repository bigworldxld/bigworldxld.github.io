---
title: 大数据技术之Flume
date: 2022-02-05 23:19:34
categories: hadoop生态圈
img: /images/Flume.jpg
tags: Flume
---

# 大数据技术之Flume

## 第2章 Flume入门

### 2.1 Flume安装部署

### 2.1.1 安装地址

1. [Flume官网](http://flume.apache.org/)
2. [文档查看](http://flume.apache.org/FlumeUserGuide.html)
3. [下载地址](http://archive.apache.org/dist/flume/)

### 2.1.2 安装部署

（1）将apache-flume-1.9.0-bin.tar.gz上传到linux的/opt/software目录下
（2）解压apache-flume-1.9.0-bin.tar.gz到/opt/module/目录下

```shell
[atguigu@hadoop102 software]$ tar -zxf /opt/software/apache-flume-1.9.0-bin.tar.gz -C /opt/module/
```

（3）修改apache-flume-1.9.0-bin的名称为flume

```shell
[atguigu@hadoop102 module]$ mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume
```

（4）将lib文件夹下的guava-11.0.2.jar删除以兼容Hadoop 3.1.3

```shell
[atguigu@hadoop102 module]$ rm /opt/module/flume/lib/guava-11.0.2.jar
```

注意：删除guava-11.0.2.jar的服务器节点，一定要配置hadoop环境变量。否则会报如下异常。
Caused by: java.lang.ClassNotFoundException: com.google.common.collect.Lists
        at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 1 more
（5）将flume/conf下的flume-env.sh.template文件修改为flume-env.sh，并配置flume-env.sh文件

```shell
[atguigu@hadoop102 conf]$ mv flume-env.sh.template flume-env.sh
[atguigu@hadoop102 conf]$ vi flume-env.sh
export JAVA_HOME=/opt/module/jdk1.8.0_212
```


