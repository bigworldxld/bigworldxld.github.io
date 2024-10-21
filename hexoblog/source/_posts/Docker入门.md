---
title: Docker入门
img: /images/docker.jpg
cover: true
coverImg: /images/docker.jpg
tags: docker
categories: java
abbrlink: 11205
date: 2022-08-03 09:56:08
---

## docker 入门

![image-20200514192040435](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNDE5MjA0MDQzNS5wbmc?x-oss-process=image/format,png)

Docker的思想就来自于集装箱！

JRE – 多个应用(端口冲突) – 原来都是交叉的！
隔离：Docker核心思想！打包装箱！每个箱子是互相隔离的。

Docker通过隔离机制，可以将服务器利用到极致！

### Docker的基本组成

镜像（image)：

docker镜像就好比是一个目标，可以通过这个目标来创建容器服务，tomcat镜像==>run==>容器（提供服务器），通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中的）。

容器(container)：

Docker利用容器技术，独立运行一个或者一组应用，通过镜像来创建的.
启动，停止，删除，基本命令
目前就可以把这个容器理解为就是一个简易的 Linux系统。

仓库(repository)：

仓库就是存放镜像的地方！
仓库分为公有仓库和私有仓库。(很类似git)
Docker Hub是国外的。
阿里云…都有容器服务器(配置镜像加速!)

### 安装Docker

Linux要求内核3.0以上

```shell
uname -r 
```

帮助文档：https://docs.docker.com/engine/install/

```shell
#1.卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#2.需要的安装包
yum install -y yum-utils
#3.设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
#默认是从国外的，不推荐
#推荐使用国内的
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#更新yum软件包索引
yum makecache fast
#4.安装docker相关的 docker-ce 社区版 而ee是企业版
yum install docker-ce docker-ce-cli containerd.io
#5、启动docker
systemctl start docker
#6. 使用docker version查看是否按照成功
docker version
#7. 测试
docker run hello-world
```

卸载docker

```shell
#1. 卸载依赖
yum remove docker-ce docker-ce-cli containerd.io
#2. 删除资源
rm -rf /var/lib/docker
# /var/lib/docker 是docker的默认工作路径！
```

### 阿里云镜像加速

1、登录阿里云找到容器服务

![image-20200515102112851](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTEwMjExMjg1MS5wbmc?x-oss-process=image/format,png)

2、找到镜像加速器

![image-20200515102009470](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTEwMjAwOTQ3MC5wbmc?x-oss-process=image/format,png)

3、配置使用

![image-20200515102216898](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTEwMjIxNjg5OC5wbmc?x-oss-process=image/format,png)

**docker run 流程图**

![image-20200515102637246](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTEwMjYzNzI0Ni5wbmc?x-oss-process=image/format,png)

### 为什么Docker比Vm快

1、docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。
2、docker利用的是宿主机的内核,而不需要Guest OS。

GuestOS： VM（虚拟机）里的的系统（OS）;

HostOS：物理机里的系统（OS）；

![image-20200515104117329](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTEwNDExNzMyOS5wbmc?x-oss-process=image/format,png)

因此,当新建一个 容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。而docker由于直接利用宿主机的操作系统,则省略了这个复杂的过程,因此新建一个docker容器只需要几秒钟。

### Docker的常用命令

帮助命令

> docker version    #显示docker的版本信息。
> docker info       #显示docker的系统信息，包括镜像和容器的数量
> docker 命令 --help #帮助命令


帮助文档的地址：https://docs.docker.com/engine/reference/commandline/build/

> 镜像命令
> docker images #查看所有本地主机上的镜像 可以使用docker image ls代替
>
> docker search 搜索镜像
>
> docker pull 下载镜像 
>
> docker rmi 删除镜像 
> 

docker rmi 删除镜像

➜  ~ docker rmi -f 镜像id #删除指定的镜像
➜  ~ docker rmi -f 镜像id 镜像id 镜像id 镜像id#删除指定的镜像
➜  ~ docker rmi -f $(docker images -aq) #删除全部的镜像容器命令
docker run 镜像id 新建容器并启动

docker ps 列出所有运行的容器 docker container list

docker rm 容器id 删除指定容器

docker start 容器id #启动容器
docker restart 容器id #重启容器
docker stop 容器id #停止当前正在运行的容器
docker kill 容器id #强制停止当前容器

新建容器并启动

> docker run [可选参数] image | docker container run [可选参数] image 
> #参书说明
> --name="Name"		容器名字 tomcat01 tomcat02 用来区分容器
> -d					后台方式运行
> -it 				使用交互方式运行，进入容器查看内容
> -p					指定容器的端口 -p 8080(宿主机):8080(容器)
> 		-p ip:主机端口:容器端口
> 		-p 主机端口:容器端口(常用)
> 		-p 容器端口
> 		容器端口
> -P(大写) 				随机指定端口

测试、启动并进入容器

docker run -it centos /bin/bash
列出所有运行的容器

> #docker ps命令 #列出当前正在运行的容器
>   -a, --all             Show all containers (default shows just running)
>   -n, --last int        Show n last created containers (includes all states) (default -1)
>   -q, --quiet           Only display numeric IDs
>

**退出容器**

```shell
exit #容器直接退出
ctrl +P +Q #容器不停止退出
```

**删除容器**

```shell
docker rm 容器id   #删除指定的容器，不能删除正在运行的容器，如果要强制删除 rm -rf
docker rm -f $(docker ps -aq)  #删除指定的容器
docker ps -a -q|xargs docker rm  #删除所有的容器
```

**启动和停止容器的操作**

```shell
docker start 容器id	#启动容器
docker restart 容器id	#重启容器
docker stop 容器id	#停止当前正在运行的容器
docker kill 容器id	#强制停止当前容器
```

**后台启动命令**

```shell
# 命令 docker run -d 镜像名
➜  ~ docker run -d centos
```

查看日志

> docker logs --help
> ➜  ~ docker run -d centos /bin/sh -c "while true;do echo 6666;sleep 1;done" #模拟日志      
> #显示日志
> -tf		#显示日志信息（一直更新）
> --tail number #需要显示日志条数
> docker logs -t --tail n 容器id #查看n行日志
> docker logs -ft 容器id #跟着日志

**查看容器中进程信息** ps

```shell
docker top 容器id
```

**查看镜像的元数据**

```shell
# 命令
docker inspect 容器id
```

**进入当前正在运行的容器**

```shell
# 我们通常容器都是使用后台方式运行的，需要进入容器，修改一些配置

# 命令 
docker exec -it 容器id bashshell
# 方式二
docker attach 容器id
```

**从容器内拷贝到主机上**

```shell
docker cp 容器id:容器内路径   主机目的路径
#进入docker容器内部
docker exec -it  55321bcae33d /bin/bash 
```

Docker 安装Nginx

> #1. 搜索镜像 search 建议大家去docker搜索，可以看到帮助文档
> #2. 拉取镜像 pull
> #3、运行测试
>
> -d 后台运行
>
> --name 给容器命名
>
> -p 宿主机端口：容器内部端口
>
> ➜  ~ docker run -d --name nginx00 -p 82:80 nginx
> 75943663c116f5ed006a0042c42f78e9a1a6a52eba66311666eee12e1c8a4502
> ➜  ~ docker ps
> CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
> 75943663c116        nginx               "nginx -g 'daemon of…"   41 seconds ago      Up 40 seconds       0.0.0.0:82->80/tcp   nginx00
> ➜  ~ curl localhost:82   #测试

作业：部署es+kibana

```shell
# es 暴露的端口很多！
# es 的数据一般需要放置到安全目录！挂载
# --net somenetwork ? 网络配置

# 启动elasticsearch
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.6.2
# 测试一下es是否成功启动
➜  ~ curl localhost:9200
➜  ~ docker stats # 查看docker容器使用内存情况
```

![image-20200515152552722](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTE1MjU1MjcyMi5wbmc?x-oss-process=image/format,png)

```shell
#关闭，添加内存的限制，修改配置文件 -e 环境配置修改
➜  ~ docker rm -f d73ad2f22dd3                                            
➜  ~ docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms64m -Xmx512m" elasticsearch:7.6.2
```

### 可视化

**什么是portainer？**

Docker图形化界面管理工具！提供一个后台面板供我们操作！

```shell
docker run -d -p 8080:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

进入之后的面板

![image-20200515155113693](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNTE1NTExMzY5My5wbmc?x-oss-process=image/format,png)

Docker 镜像都是只读的，当容器启动时，一个新的可写层加载到镜像的顶部！

这一层就是我们通常说的容器层，容器之下的都叫镜像层！

commit镜像

> docker commit 提交容器成为一个新的副本
>
> 命令和git原理类似
>
> docker commit -m="描述信息" -a="作者" 容器id 目标镜像名:[TAG]

### 容器数据卷

容器之间可以有一个数据共享的技术！[Docker](https://so.csdn.net/so/search?q=Docker&spm=1001.2101.3001.7020)容器中产生的数据，同步到本地！

这就是卷技术！目录的[挂载](https://so.csdn.net/so/search?q=挂载&spm=1001.2101.3001.7020)，将我们容器内的目录，挂载到Linux上面！

使用数据卷

> 方式一 ：直接使用命令挂载 -v
>
> -v, --volume list                    Bind mount a volume
>
> docker run -it -v 主机目录:容器内目录  -p 主机端口:容器内端口
> ➜ ~ docker run -it -v /home/ceshi:/home centos /bin/bash
> #通过 docker inspect 容器id 查看

具名和匿名挂载

> -v 容器内路径!
> docker run -d -P --name nginx01 -v /etc/nginx nginx
>
> 查看所有的volume的情况
>
> ➜  ~ docker volume ls    
> DRIVER              VOLUME NAME
> local               33ae588fae6d34f511a769948f0d3d123c9d45c442ac7728cb85599c2657e50d
> local            
>
> 这里发现，这种就是匿名挂载，我们在 -v只写了容器内的路径，没有写容器外的路劲！
>
> 具名挂载
>
> ➜  ~ docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
> ➜  ~ docker volume ls                  
> DRIVER              VOLUME NAME
> local               juming-nginx
>
> 通过 -v 卷名：容器内路径

所有的docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes/xxxx/_data`下

> 三种挂载： 匿名挂载、具名挂载、指定路径挂载
>
> -v 容器内路径			#匿名挂载
> -v 卷名：容器内路径		#具名挂载
> -v /宿主机路径：容器内路径 #指定路径挂载 docker volume ls 是查看不到的

```shell
# 通过 -v 容器内路径： ro rw 改变读写权限
ro #readonly 只读
rw #readwrite 可读可写
docker run -d -P --name nginx05 -v juming:/etc/nginx:ro nginx
docker run -d -P --name nginx05 -v juming:/etc/nginx:rw nginx
# ro 只要看到ro就说明这个路径只能通过宿主机来操作，容器内部是无法操作！
```

### DockerFile介绍

dockerfile是用来构建docker镜像的文件！命令参数脚本！

构建步骤：

1、 编写一个dockerfile文件

2、 docker build 构建称为一个镜像

3、 docker run运行镜像

4、 docker push发布镜像（DockerHub 、阿里云仓库)**基础知识：**

1、每个保留关键字(指令）都是必须是大写字母

2、执行从上到下顺序

3、#表示注释

4、每一个指令都会创建提交一个新的镜像曾，并提交！

![image-20200516131756997](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNjEzMTc1Njk5Ny5wbmc?x-oss-process=image/format,png)

DockerFile的指令

> FROM				# 基础镜像，一切从这里开始构建
> MAINTAINER			# 镜像是谁写的， 姓名+邮箱
> RUN					# 镜像构建的时候需要运行的命令
> ADD					# 步骤，tomcat镜像，这个tomcat压缩包！添加内容 添加同目录
> WORKDIR				# 镜像的工作目录
> VOLUME				# 挂载的目录
> EXPOSE				# 保留端口配置
> CMD					# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代。
> ENTRYPOINT			# 指定这个容器启动的时候要运行的命令，可以追加命令
> ONBUILD				# 当构建一个被继承 DockerFile 这个时候就会运行ONBUILD的指令，触发指令。
> COPY				# 类似ADD，将我们文件拷贝到镜像中
> ENV					# 构建的时候设置环境变量！

实战测试
创建一个自己的centos

vim mydockerfile-centos

```sh
FROM centos
MAINTAINER bigworldxld

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "-----end----"
CMD /bin/bash
```

2、通过这个文件构建镜像

 命令 docker build -f 文件路径 -t 镜像名:[tag] 

docker build -f mydockerfile-centos -t mycentos:0.1 .

**测试运行**

![image-20200516141629583](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNjE0MTYyOTU4My5wbmc?x-oss-process=image/format,png)

> CMD 和 ENTRYPOINT区别

```shell
CMD					# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代。
ENTRYPOINT			# 指定这个容器启动的时候要运行的命令，可以追加命令
```

## 实战：Tomcat镜像

1、准备镜像文件

 准备tomcat 和 jdk到当前目录，编写好README

![image-20200516162443652](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNjE2MjQ0MzY1Mi5wbmc?x-oss-process=image/format,png)

2、编写dokerfile

```sh
FROM centos #
MAINTAINER bigworldxld
COPY README /usr/local/README #复制文件
ADD jdk-8u231-linux-x64.tar.gz /usr/local/ #复制解压
ADD apache-tomcat-9.0.35.tar.gz /usr/local/ #复制解压
RUN yum -y install vim
ENV MYPATH /usr/local #设置环境变量
WORKDIR $MYPATH #设置工作目录
ENV JAVA_HOME /usr/local/jdk1.8.0_231 #设置环境变量
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.35 #设置环境变量
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib #设置环境变量 分隔符是：
EXPOSE 8080 #设置暴露的端口
CMD /usr/local/apache-tomcat-9.0.35/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.35/logs/catalina.out # 设置默认命令
```

3、构建镜像

```shell
# 因为dockerfile命名使用默认命名 因此不用使用-f 指定文件
$ docker build -t mytomcat:0.1 .
```

4、run镜像

```shell
$ docker run -d -p 8080:8080 --name tomcat01 -v /home/kuangshen/build/tomcat/test:/usr/local/apache-tomcat-9.0.35/webapps/test -v /home/kuangshen/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.35/logs mytomcat:0.1
```

5、访问测试

6、发布项目(由于做了卷挂载，我们直接在本地编写项目就可以发布了！)

发布自己的镜像
Dockerhub

1、地址 https://hub.docker.com/

2、确定这个账号可以登录

3、登录

> $ docker login --help
> Usage:  docker login [OPTIONS] [SERVER]
>
> Log in to a Docker registry.
> If no server is specified, the default is defined by the daemon.
>
> Options:
>   -p, --password string   Password
>       --password-stdin    Take the password from stdin
>   -u, --username string   Username

4、提交 push镜像

```sh
# 会发现push不上去，因为如果没有前缀的话默认是push到 官方的library
# 解决方法
# 第一种 build的时候添加你的dockerhub用户名，然后在push就可以放到自己的仓库了
$ docker build -t bigworldxld/mytomcat:0.1 .
# 第二种 使用docker tag #然后再次push
$ docker tag 容器id bigworldxld/mytomcat:1.0 #然后再次push
```

> 阿里云镜像服务上

看官网 很详细https://cr.console.aliyun.com/repository/

```shell
$ docker login --username=bigworldxld registry.cn-hangzhou.aliyuncs.com
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/bigworldxld/mybuntu:[镜像版本号]
$ docker push registry.cn-hangzhou.aliyuncs.com/bigworldxld/mybuntu:[镜像版本号]
```

## SpringBoot微服务打包Docker镜像

1、构建SpringBoot项目

2、打包运行

mvn package

3、编写dockerfile

```shell
FROM java:8
COPY *.jar /app.jar
CMD ["--server.port=8080"]
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]
```

4、构建镜像

```shell
# 1.复制jar和DockerFIle到服务器
# 2.构建镜像
$ docker build -t xxxxx:xx  .
```

4、构建镜像

5、发布运行

## 小结

![image-20200516171155667](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL2NoZW5nY29kZXgvY2xvdWRpbWcvbWFzdGVyL2ltZy9pbWFnZS0yMDIwMDUxNjE3MTE1NTY2Ny5wbmc?x-oss-process=image/format,png)





















