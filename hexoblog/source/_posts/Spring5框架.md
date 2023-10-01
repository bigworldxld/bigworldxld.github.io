---
title: Spring5框架
date: 2022-03-26 22:12:04
tags: Spring
categories: java
img: /images/Spring.jpg
---

# Spring5 框架

## Spring5 框架概述

1、Spring 是轻量级的开源的 JavaEE 框架
2、Spring 可以解决企业应用开发的复杂性
3、Spring 有两个核心部分：IOC 和 Aop
（1）IOC：控制反转，把创建对象过程交给 Spring 进行管理
（2）Aop：面向切面，不修改源代码进行功能增强
4、Spring 特点
（1）方便解耦，简化开发
（2）Aop 编程支持
（3）方便程序测试
（4）方便和其他框架进行整合
（5）方便进行事务操作
（6）降低 API 开发难度
5、现在课程中，选取 Spring 版本 5.x

## Spring5 案例

### IOC 操作 Bean 管理(基于注解方式)

1、什么是注解

（1）注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值..)
（2）使用注解，注解作用在类上面，方法上面，属性上面
（3）使用注解目的：简化 xml 配置
2、Spring 针对 Bean 管理中创建对象提供注解
（1）@Component
（2）@Service
（3）@Controller
（4）@Repository

* 上面四个注解功能是一样的，都可以用来创建 bean 实例
  3、基于注解方式实现对象创建

3、基于注解方式实现对象创建
第一步 引入依赖
第二步 开启组件扫描

```xml
<!--开启组件扫描
 1 如果扫描多个包，多个包使用逗号隔开
 2 扫描包上层目录
-->
<context:component-scan base-package="com.atguigu"></context:component-scan>
```

第三步 创建类，在类上面添加创建对象注解
//在注解里面 value 属性值可以省略不写，
//默认值是类名称，首字母小写
//UserService -- userService

```java
@Component(value = "userService") //<bean id="userService" class=".."/>
public class UserService {
    public void add() {
        System.out.println("service add.......");
    }
}
```

4、开启组件扫描细节配置

```xml
<!--示例 1
 use-default-filters="false" 表示现在不使用默认 filter，自己配置 filter
 context:include-filter ，设置扫描哪些内容
-->
<context:component-scan base-package="com.atguigu" use-default-filters="false">
    <context:include-filter type="annotation" 

                            expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
<!--示例 2
 下面配置扫描包所有内容
 context:exclude-filter： 设置哪些内容不进行扫描
-->
<context:component-scan base-package="com.atguigu">
    <context:exclude-filter type="annotation"        expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

5、基于注解方式实现属性注入
（1）@Autowired：根据属性类型进行自动装配
第一步 把 service 和 dao 对象创建，在 service 和 dao 类添加创建对象注解
第二步 在 service 注入 dao 对象，在 service 类添加 dao 类型属性，在属性上面使用注解

```java
@Service
public class UserService {
    //定义 dao 类型属性
    //不需要添加 set 方法
    //添加注入属性注解
    @Autowired 
    private UserDao userDao;
    public void add() {
        System.out.println("service add.......");
        userDao.add();
    }
}
```

（2）@Qualifier：根据名称进行注入
这个@Qualifier 注解的使用，和上面@Autowired 一起使用
//定义 dao 类型属性
//不需要添加 set 方法
//添加注入属性注解
@Autowired //根据类型进行注入
@Qualifier(value = "userDaoImpl1") //根据名称进行注入

private UserDao userDao;
（3）@Resource：可以根据类型注入，可以根据名称注入
//@Resource //根据类型进行注入
@Resource(name = "userDaoImpl1") //根据名称进行注入
private UserDao userDao;
（4）@Value：注入普通类型属性
@Value(value = "abc")
private String name;
6、完全注解开发 
（1）创建配置类，替代 xml 配置文件
@Configuration //作为配置类，替代 xml 配置文件

```xml
@ComponentScan(basePackages = {"com.atguigu"})
public class SpringConfig {
}
```


（2）编写测试类

```java
@Test
public void testService2() {
 //加载配置类
 ApplicationContext context
 = new AnnotationConfigApplicationContext(SpringConfig.class);
 UserService userService = context.getBean("userService", 
UserService.class);
 System.out.println(userService);
 userService.add();
}
```

### AOP 操作（AspectJ 注解）

1、创建类，在类里面定义方法

```java
public class User {
    public void add() {
        System.out.println("add.......");
    }
}
```

2、创建增强类（编写增强逻辑）
（1）在增强类里面，创建方法，让不同方法代表不同通知类型
//增强的类

```java
public class UserProxy {
    public void before() {//前置通知
        System.out.println("before......");
    }
}
```

3、进行通知的配置
（1）在 spring 配置文件中，开启注解扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 xmlns:context="http://www.springframework.org/schema/context" 
 xmlns:aop="http://www.springframework.org/schema/aop" 
 xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans.xsd 
 http://www.springframework.org/schema/context 
http://www.springframework.org/schema/context/spring-context.xsd 
 http://www.springframework.org/schema/aop 
http://www.springframework.org/schema/aop/spring-aop.xsd">
 <!-- 开启注解扫描 -->
 <context:component-scan base-package="com.atguigu.spring5.aopanno"></context:component-scan>
```

（2）使用注解创建 User 和 UserProxy 对象

分别在增强类和被增强类上加@Component

（3）在增强类上面添加注解 @Aspect
//增强的类
@Component
@Aspect //生成代理对象
public class UserProxy {}
（4）在 spring 配置文件中开启生成代理对象

```xml
<!-- 开启 Aspect 生成代理对象-->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```

4、配置不同类型的通知
（1）在增强类的里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置

```java
package com.atguigu.spring5.dao;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * @author bigworldxld
 * @create 2022-03-26 15:14
 */
@Component
@Aspect//生成代理对象
//在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高

public class UserProxy {
    //相同切入点抽取
    @Pointcut(value = "execution(* com.atguigu.spring5.dao.User.add(..))")
    public void pointdemo() {
    }

    @Before(value = "pointdemo()")
    public void before() {//前置通知
        System.out.println("before......");
    }
    //后置通知（返回通知）
    @AfterReturning(value = "pointdemo()")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }
    //最终通知
    @After(value = "pointdemo()")
    public void after() {
        System.out.println("after.........");
    }
    //异常通知
    @AfterThrowing(value = "pointdemo()")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }
    //环绕通知
    @Around(value = "pointdemo()")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws
            Throwable {
        System.out.println("环绕之前.........");
        //被增强的方法执行
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后.........");
    }
}
```

5、相同的切入点抽取

```java
//相同切入点抽取
@Pointcut(value = "execution(* com.atguigu.spring5.aopanno.User.add(..))")
public void pointdemo() {
}
//前置通知
//@Before 注解表示作为前置通知
@Before(value = "pointdemo()")
public void before() {
 System.out.println("before.........");
}
```

6、有多个增强类多同一个方法进行增强，设置增强类优先级
（1）在增强类上面添加注解 @Order(数字类型值)，数字类型值越小优先级越高
@Component
@Aspect
@Order(1)
public class PersonProxy
7、完全使用注解开发 
（1）创建配置类，不需要创建 xml 配置文件 

```java
@Configuration
@ComponentScan(basePackages = {"com.atguigu"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {
}
```

### 事务操作（完全注解声明式事务管理）

1、创建配置类，使用配置类替代 xml 配置文件

```java
@Configuration //配置类
@ComponentScan(basePackages = "com.atguigu") //组件扫描
@EnableTransactionManagement //开启事务
public class TxConfig {
    //创建数据库连接池
    @Bean
    public DruidDataSource getDruidDataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");#这里使用的mysql8.0版本，驱动类名有所不同
        dataSource.setUrl("jdbc:mysql://localhost:3306/mybatis?serverTimezone=GMT");
        dataSource.setUsername("root");
        dataSource.setPassword("123");
        return dataSource;
    }
    //创建 JdbcTemplate 对象

    @Bean
    public JdbcTemplate getJdbcTemplate(DataSource dataSource) {
        //到 ioc 容器中根据类型找到 dataSource
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        //注入 dataSource
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }
    //创建事务管理器
    @Bean
    public DataSourceTransactionManager 
        getDataSourceTransactionManager(DataSource dataSource) {
        DataSourceTransactionManager transactionManager = new 
            DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```

###  SpringWebflux（基于注解编程模型）

SpringWebflux 实现方式有两种：注解编程模型和函数式编程模型
使用注解编程模型方式，和之前 SpringMVC 使用相似的，只需要把相关依赖配置到项目中，
SpringBoot 自动配置相关运行容器，默认情况下使用 Netty 服务器
第一步 创建 SpringBoot 工程，引入 Webflux 依赖

```properties
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

第二步 配置application.properties启动端口号

```properties
server.port=8888
```

第三步 创建包和相关类
⚫ 实体类

```java
package com.atguigu.springboot_project.bean;
/**
 * @author bigworldxld
 * @create 2022-03-22 16:29
 */
@data
public class User {
    private String name;
    private String gender;
    private Integer age;
}

```

⚫ 创建接口定义操作的方法
//用户操作接口

```java
public interface UserService {
 //根据 id 查询用户
 Mono<User> getUserById(int id);
 //查询所有用户
 Flux<User> getAllUser();
 //添加用户
 Mono<Void> saveUserInfo(Mono<User> user);
}
```

⚫ 接口实现类

```java
public class UserServiceImpl implements UserService {
    //创建 map 集合存储数据
    private final Map<Integer,User> users = new HashMap<>();
    public UserServiceImpl() {
        this.users.put(1,new User("lucy","nan",20));
        this.users.put(2,new User("mary","nv",30));
        this.users.put(3,new User("jack","nv",50));
    }
    //根据 id 查询
    @Override
    public Mono<User> getUserById(int id) {
        return Mono.justOrEmpty(this.users.get(id));
    }
    //查询多个用户
    @Override
    public Flux<User> getAllUser() {
        return Flux.fromIterable(this.users.values());
    }
    //添加用户
    @Override
    public Mono<Void> saveUserInfo(Mono<User> userMono) {
        return userMono.doOnNext(person -> {
            //向 map 集合里面放值
            int id = users.size()+1;
            users.put(id,person);
        }).thenEmpty(Mono.empty());
    }
}
```

⚫ 创建 controller

```java
@RestController
public class UserController {
    //注入 service
    @Autowired
    private UserService userService;
    //id 查询
    @GetMapping("/user/{id}")
    public Mono<User> geetUserId(@PathVariable int id) {
        return userService.getUserById(id);
    }
    //查询所有
    @GetMapping("/user")
    public Flux<User> getUsers() {
        return userService.getAllUser();
    }
    //添加
    @PostMapping("/saveuser")
    public Mono<Void> saveUser(@RequestBody User user) {
        Mono<User> userMono = Mono.just(user);
        return userService.saveUserInfo(userMono);
    }
}
```

⚫ 说明
SpringMVC 方式实现，同步阻塞的方式，基于 SpringMVC+Servlet+Tomcat
SpringWebflux 方式实现，异步非阻塞 方式，基于 SpringWebflux+Reactor+Netty