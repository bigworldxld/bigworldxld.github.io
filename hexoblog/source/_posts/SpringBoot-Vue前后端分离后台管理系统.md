---
title: SpringBoot+Vue前后端分离后台管理系统
date: 2022-06-22 18:45:00
img: /images/springboot+vue.jpg
categories: java
tags:
	- vue
	- springboot
	- nginx
---

## 安装vue的环境

1、首先我们上node.js官网([https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/?fileGuid=HXqVy6jx8qkWKPJq))，下载最新的长期版本，直接运行安装完成之后，我们就已经具备了node和npm的环境啦。

2、接下来，我们安装vue的环境

```sell
# 安装淘宝npm
npm install -g cnpm --registry=https://registry.npm.taobao.org
# vue-cli 安装依赖包
cnpm install --g vue-cli
# 打开vue的可视化管理工具界面
vue ui
```

安装了淘宝npm，cnpm是为了提高我们安装依赖的速度。vue ui是[@vue](https://github.com/vue)/cli3.0增加一个可视化项目管理工具，可以运行项目、打包项目，检查等操作。

## 安装element-ui

```sell
# 切换到项目根目录
cd vueadmin-vue
# 或者直接在idea中执行下面命令
# 安装element-ui
cnpm install element-ui --save
```

然后我们打开项目src目录下的main.js，引入element-ui依赖。

```plain
import Element from 'element-ui'
import "element-ui/lib/theme-chalk/index.css"
Vue.use(Element)
```

## 安装axios、qs、mockjs

- **axios**：一个基于 promise 的 HTTP 库，类ajax
- **qs**：查询参数序列化和解析库
- **mockjs**：为我们生成随机数据的工具库

接下来，我们来安装axios（[http://www.axios-js.com/](http://www.axios-js.com/?fileGuid=HXqVy6jx8qkWKPJq)），axios是一个基于 promise 的 HTTP 库，这样我们进行前后端对接的时候，使用这个工具可以提高我们的开发效率。

安装命令：

```plain
cnpm install axios --save
```

然后同样我们在main.js中全局引入axios。

```plain
import axios from 'axios'
Vue.prototype.$axios = axios //
```

组件中，我们就可以通过this.$axios.get()来发起我们的请求了哈。当然了，后面我们添加axios拦截的时候我们需要修改引入的编写。
同时，我们同步安装一个qs，什么是qs？qs是一个流行的查询参数序列化和解析库。可以将一个普通的object序列化成一个查询字符串，或者反过来将一个查询字符串解析成一个object,帮助我们查询字符串解析和序列化字符串。

```plain
cnpm install qs --save
```

然后因为后台我们现在还没有搭建，无法与前端完成数据交互，因此我们这里需要mock数据，因此我们引入mockjs（[http://mockjs.com/](http://mockjs.com/?fileGuid=HXqVy6jx8qkWKPJq)），方便后续我们提供api返回数据。

```plain
cnpm install mockjs --save-dev
```

然后我们在src目录下新建mock.js文件，用于编写随机数据的api，然后我们需要在main.js中引入这个文件：

- src/main.js

  ```plain
  require("./mock") //引入mock数据，关闭则注释该行
  ```

  后面我们mackjs会自动为我们拦截ajax，并自动匹配路径返回数据！

## 管理后台界面

- 开发环境：java1.8,node16.14,npm8.6,@vue/cli5.0,idea2019.3,mysqlserver8.0

- 实现技术：vue3,vuerouter,vuex,elementplus,springboot,mysql,mybatisplus,axios,hutool

- Spring Boot模块：Spring Web,MySQL Driver,Lombok,MyBatis Framework

- 实现功能：前后端分离实现后台管理页面及操作

- 前端端口为8080，后端端口为9090

  {% asset_img 端口1.png This is an example image %}

## 数据库准备

- 创建一个新的数据库命名为"springboot-vue"(或为其他名称，若为其他名称需要手动修改spring.datasource.url),字符集utf8mb4,排序规则unf8mb4_unicode_ci

- 新建表user:id(int,主键,非空,自增),username(varchar,255),password(varchar,255),nick_name(varchar,255),age(int),sex(varchar,255),address(varchar,255)

- 修改spring.datasource.password为你的数据库密码

- 本地的数据库

  {% asset_img 本地的数据库.png This is an example image %}

- Linux上的

  {% asset_img Linux数据库.png This is an example image %}

## 运行在本地

- 后端：使用idea导入springboot目录，初始化maven，运行即可
- 前端：npm i后使用npm run serve启动即可

{% asset_img 运行在本地.png This is an example image %}



## 打包到Liunx上运行

安装nginx

Linux安装Nginx.md

链接：https://blog.csdn.net/t8116189520/article/details/81909574

Niginx的配置

{% asset_img Nginx的配置.png This is an example image %}

命令启动

{% asset_img 命令启动.png This is an example image %}

{% asset_img 命令启动1.png This is an example image %}

[详细源码](https://github.com/bigworldxld/xld.github.io)

