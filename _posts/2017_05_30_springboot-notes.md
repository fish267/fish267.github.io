---
author: Fish
layout: post
title: SpringBoot 笔记
categories: java
tags:  
---

营销测试工具工程实现, 基于<code> SpringBoot </code>, 码了一段时间, 整理一下笔记, 真的**特别好用!!!** 最直观的感受有下面三方面:

- 无 xml 配置!!!
- 内嵌 tomcat
- 组件选配

## 1. 创建工程

## 1.1 使用 IDEA 创建

个人推荐使用 IDEA 来创建, 省事. 不需要下载, 解压, 顺带还指定安装目录.

<!--more-->

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/shiheng.fsh/cloud_notes/0074c33e35472c695d627c1691471836/image.png)
![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/shiheng.fsh/cloud_notes/306586331500e42c7e17603dfa86742f/image.png)
![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/shiheng.fsh/cloud_notes/53129680a09a6cbebb616f8355672ab7/image.png)

## 1.2 官网创建工程

打开官网 [http://start.spring.io/](http://start.spring.io/), 填入 Group Artifact 等, 勾选依赖的组件, 点击 <b>Generate Project</b>, 下载 ZIP. 然后导入IDEA.

<hr>

创建后的目录结构如下:

```shell

├── mvnw
├── mvnw.cmd
├── pom.xml
├── promoboot.iml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       └── java
└── target
    ├── classes
    │   ├── application.properties
    │   └── com
    ├── generated-sources
    │   └── annotations
    ├── generated-test-sources
    │   └── test-annotations
    └── test-classes
        └── com
```

# 2. 启动工程

在启动文件中, 添加个 Controller 并启动测试一下, 默认的 <b>server.port=8080</b> 

```java
@Controller
@SpringBootApplication
public class PromobootApplication {

    public static void main(String[] args) {
        SpringApplication.run(PromobootApplication.class, args);
    }

    @RequestMapping("/")
    @ResponseBody
    String home() {
        return "Hello World!";
    }
}
```

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/shiheng.fsh/cloud_notes/07be60c8fd5763ba4850806864c66669/image.png)

<b>启动工程的方式有如下几个:</b>

## 2.1 SpringBoot启动 --- IDEA 启动

直接启动<code>main</code>函数

## 2.2 SpringBoot启动 --- mvn 命令启动

执行下面命令, 直接启动:

```shell
mvn spring-boot:run
```

## 2.3 SpringBoot启动 ---  打包部署启动

打包并部署放到生产环境

首先创建 Jar 包

```shell
mvn package
```

然后使用 <code>java -jar</code> 命令启动

```shell
java -jar target/promoboot-0.0.1-SNAPSHOT.jar
```

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/6913bef01b37bb9b627fec52f0fb22b4/image.png)

## 2.4 SpringBoot 热部署

改完代码, 每次启动工程挺费时的, SpringBoot 支持热部署需要添加 <code>spring-boot-devtools</code> 依赖, 可以手动添加, 也可以在创建 SpringBoot 时勾选 DevTools, 如下图:


![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/8d0cf593982b85e0f874c81a3213e58b/image.png)



下面说一下手工添加的方式:

-  手动添加 devtools 依赖, 设置 optional = true

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> 
</dependency>
```

-  使用 <code>mvn spring-boot:run</code> 启动

```shell
mvn spring-boot:run
```

可以看到多了一条监听 class 变更的日志

```shell
18:26:43.631 [main] DEBUG org.springframework.boot.devtools.restart.ChangeableUrls - Matching URLs for reloading : [file:/Users/fish/AliDrive/git/springbootdemo/target/classes/]
```

此时修改代码, 就可以无脑看执行结果了!

<b>另外, IDEA 设置一下字段编译, 如下图 </b>

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/748b590633cdfd7f1089c36ec95f5d9b/image.png)


## 2.5 开启本地 Debug 端口

参考文档[Debug the application](http://docs.spring.io/spring-boot/docs/current/maven-plugin/examples/run-debug.html) , 开启本地 Debug 端口使用如下方法:

```java
mvn spring-boot:run -Drun.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"
```

# 3. 配置文件读取与环境隔离

初始化的工程, 自带了<code>application.properties</code>配置文件. 


## 3.1 使用 yaml 替换 properties
其实框架默认支持 <code>Ruby on Rails</code> 惯用的 <code>yaml</code> 格式配置文件, 非常建议大家使用<b> yaml</b> 来替换掉 <code>properties</code> , 配置文件看起来短多了, 对比如下:


- 使用 <code>properties</code> 

```shell
server.port = 8080
server.context-path = '/boot'
```

- 使用 <code>yaml</code>

```shell
server:
  port: 8080
  context-path: '/boot'
```

此外, <code>yaml</code> 语法还支持变量定义等, 详细用法可以查看文档 [YAML Syntax](http://docs.ansible.com/ansible/YAMLSyntax.html)


## 3.2 开发/测试/生产 环境配置隔离

对于程序员, 手动改配置是一项不能忍的工作, 好在 spingboot 完美的支持配置文件隔离. 操作如下:

新增 开发/测试/生产 配置文件: 

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/d296f18bc917029cf1c6e4fced834432/image.png)

配置 <code>application.properties</code> 中的内容, 比如启用 dev 环境, 设置 active = dev, 就启用了<code> application-dev.yaml</code> : 

```shell
spring:
  profiles:
    active: dev
```

本地开发时, 使用 <code>mvn spring-boot:run </code>启动的工程, 可以自动生效.

## 3.2 线上部署, 配置环境隔离

上面提到过 Jar 包的部署方式, 同一套代码部署到不同环境时, 不需要修改配置文件, 使用下面的命令来部署: 


- 将服务部署到生产环境(prod)

```shell
mvn install && java -jar target/promoboot-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
```

## 3.4 读取配置文件中的值

类似 sofa 框架的 <code>@AppConfig</code> 注解, SpringBoot 在读取配置文件时, 关键词是

```java
// 获取单个配置值
@Value("${xx.oo}")

// 自定义对象注入
@Component
@ConfigurationProperties
```  

### 3.4.1 读取单个配置的值

<code>application.yaml </code>配置如下:

```yaml
params:
  env: stable
```

代码中使用如下方式获取:

```java
@Value("${params.env}")
private String env;
```

### 3.4.2 自定义个对象进行注入

如果某一个类型的参数实在多, 使用上面的方式非常繁琐, SpringBoot 支持直接映射到对象中.

```yaml
apple:
  size: 20
  color: red
  weight: 300
```

然后定义个 Apple 的类, 添加上 <code>Component</code> 和 <code>ConfigurationProperties</code> 注解.

```java
@Component
@ConfigurationProperties(prefix = "apple")
public class Apple {

    private String name;

    private int size;

    private String color;

    private int weight;

    public String getName() {
        return name;
    }
    ......
```

最后, 引用方式如下:

```java
@Autowired
private Apple apple;
```

这样<code> Apple </code>的配置就映射进来了, 当然你也可以在他的属性上添加 <code>@NotEmpty</code> 等注解. 比如在 color 属性上添加 @NotEmpty, 配置文件中置空, 系统自动编译会直接报错的:

```shell
***************************
APPLICATION FAILED TO START
***************************

Description:

Binding to target Apple{name='lol', size=20, cllor='', weight=300} failed:

    Property: apple.color
    Value:
    Reason: may not be empty
```


# 4. Controller


- SpringMVC 

![rcontroller-mvc](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/77a5032e067c079990a793917b5a3117/rcontroller-mvc.jpg)

- SpringBoot

![rcontroller-rest](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/2b8f2ab1d4370606619d382ea8dec013/rcontroller-rest.jpg)


从上面两个图可见, 两个框架提供的功能是一样的, SpringBoot 更简单, 还挺好用:

```java
@RestController = @Controller + @ResponseBody

@GetMapping(value="xx") = @RequestMapping(value = "xx", method = RequestMethod.GET)
@PostMapping(value="xx") = @RequestMapping(value = "xx", method = RequestMethod.POST)
```

# 5. 数据库操作 JPA

操作数据库, 常见的 JDBC, MyBatis等, 不再赘述. 项目中使用的是 <code>JPA</code>, 非常简洁.

## 5.1 什么是 JPA

    JPA(Java Persistence API)是Sun官方提出的Java持久化规范, 它为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据, 目的是整合现有的 ORM 技术.

## 5.2 工程配置

### 5.2.1 添加依赖

比如使用 mysql 数据库, 那么在<code> pom.xml </code>中添加 <code>jpa mysql</code> 依赖

```xml
        <!--使用spring-jpa-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!--mysql 依赖添加-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
```

### 5.2.2 数据库配置

编辑<code> application-dev.yaml </code>文件

```mysql
spring:
# 数据库配置
  datasource:
    username: oooo
    password: xxxx
    url: jdbc:mysql://dev.lab.alipay.net:3306/promotion
    driver-class-name: com.mysql.jdbc.Driver
    # 配置 hibernate 属性
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

ddl-auto 是 hibernate 的配置属性, 对应的作用是: 

```
create: 删表, 重新创建表(慎用)
update: 常用配置, ，第一次加载Hibernate时创建数据表, 以后加载HIbernate时只会根据model更新
validate: 验证数据库表结构 
```


### 5.2.3 创建实体

比如创建一个 <code>Robot</code> 实体, 自定义一些属性:

```java
@Entity
@Table(name = "robot")
public class RobotDO {

    @Id
    @GeneratedValue
    private long id;

    @Column(nullable = false)
    private String robotName;

    private String size;

    @Column(name = "robot_type")
    private String type;

    private Date gmtCreate;

    private Date gmtModified;

```

注解解释:

    
    @Id:            主键
    @GeneratedValue: 数据库表 primary key, 自增长

    @Column(name=""): hibernate 自动根据属性名称创建数据库列名, 也可以手动定义列名

    @Column(nullable = false): create table robot (robot_name varchar(255) not null,) .....

    @Table: 数据库表名默认和类名一直, 使用该注解自定义

    @CreationTimestamp: 自动填入 gmtCreate 时间

    @UpdateTimestamp: 自动填入更新时间



### 5.2.4 重启工程, 生成数据表

启动 springboot 工程, 可以看到数据库中新增了表 robot

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/303a65d68066d8f880201ae6041989cc/image.png)

### 5.2.5 测试代码

定义个接口, 继承自 <code> JpaRepository </code>:

```java
public interface RobotDAO extends JpaRepository<RobotDO, Integer> {

}
```

查看父类源码, 自带了 CRUD, 直接写个 Controller 来测试一下 (测试代码比较偷懒, 没有写 Service, 直接注入 DAO, 不推荐):

```java
@SpringBootApplication
@RestController
public class TestController {

    Logger logger = Logger.getLogger(this.getClass());

    @Autowired
    private Apple apple;

    @Autowired
    private RobotDAO robotDAO;

    @GetMapping(value = "/test")
    public Object testController() {
        logger.info("=====================Visiting  test controller======================");

        logger.info("测试插入:");
        RobotDO robotDO = new RobotDO();
        robotDO.setRobotName("Wall-E");
        robotDO.setSize("Huge");
        robotDO.setType("AI");
        robotDAO.save(robotDO);

        logger.info("测试查找:");
        logger.info(robotDAO.findAll());

        return apple;
    }

```

日志输出如下, 设置 <code>show-sql: true</code> 可以方便看出执行的具体 SQL:

```shell
2017-05-30 15:39:41.570  INFO 16254 --- [nio-9999-exec-1] troller$$EnhancerBySpringCGLIB$$efd51a80 : =====================Visiting  test controller======================
2017-05-30 15:39:41.570  INFO 16254 --- [nio-9999-exec-1] troller$$EnhancerBySpringCGLIB$$efd51a80 : 测试插入:
Hibernate: insert into robot (gmt_create, gmt_modified, robot_name, size, robot_type) values (?, ?, ?, ?, ?)
2017-05-30 15:39:41.585  INFO 16254 --- [nio-9999-exec-1] troller$$EnhancerBySpringCGLIB$$efd51a80 : 测试查找:
2017-05-30 15:39:41.588  INFO 16254 --- [nio-9999-exec-1] o.h.h.i.QueryTranslatorFactoryInitiator  : HHH000397: Using ASTQueryTranslatorFactory
Hibernate: select robotdo0_.id as id1_0_, robotdo0_.gmt_create as gmt_crea2_0_, robotdo0_.gmt_modified as gmt_modi3_0_, robotdo0_.robot_name as robot_na4_0_, robotdo0_.size as size5_0_, robotdo0_.robot_type as robot_ty6_0_ from robot robotdo0_
2017-05-30 15:39:41.598  INFO 16254 --- [nio-9999-exec-1] troller$$EnhancerBySpringCGLIB$$efd51a80 : [
RobotDO{id=1, robotName='Wall-E', size='Huge', type='AI', gmtCreate=2017-05-30 15:38:24.0, gmtModified=2017-05-30 15:38:24.0}, 
RobotDO{id=2, robotName='Wall-E', size='Huge', type='AI', gmtCreate=2017-05-30 15:38:55.0, gmtModified=2017-05-30 15:38:55.0}, 
RobotDO{id=3, robotName='Wall-E', size='Huge', type='AI', gmtCreate=Tue May 30 15:39:41 CST 2017, gmtModified=Tue May 30 15:39:41 CST 2017}
]
```


看一下数据库:

![image](http://024028.oss-cn-hangzhou-zmf.aliyuncs.com/uploads/app_release/checkroute/7f3f2ee934d8f64fcf38aa930f71621e/image.png)


### 5.2.5 事务操作

多表操作, 需要放在一个事务里, 在方法上面添加 <code>@Transactional</code> 即可.

# 6. AOP


面向切面编程, 是 spring 的一大特性, 举个 controller 切面例子, 看一下 springboot 的 aop 使用.

参考的是这个文档 [Spring AOP Example Tutorial](http://www.journaldev.com/2583/spring-aop-example-tutorial-aspect-advice-pointcut-joinpoint-annotations)

## 6.1 AOP 依赖引入

```xml
        <!--AOP依赖添加-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

## 6.2 AOP 实现

```java
@Aspect
@Component
@Order(1)
public class ControllerAspect {
    private Logger logger = Logger.getLogger(this.getClass());

    @Pointcut("execution(public * com.alipay.liuqi.controller.TestController.testController())")
    public void webController() {

    }

    @Before(value = "webController()")
    public void beforeWebController() {
        logger.info("===BeforeWebController: start visiting===");
    }

    @After(value = "webController()")
    public void afterWebController() {
        logger.info("====AfterWebController: end visiting===");
    }

    @AfterThrowing(value = "webController()")
    public void afterThrowWebController() {
        logger.info("====AfterThrowWebController: throw exception ===");
    }

    @AfterReturning(value = "webController()")
    public void afterReturnWebControlelr() {
        logger.info("====AfterReturnWebControlelr: return normal ===");
    }

    @Around(value = "webController()")
    public void aroundWebController(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        logger.info("====AroundWebController: start around ===");
        proceedingJoinPoint.proceed();
        logger.info("====AroundWebController: end around ===");
    }
}

```

执行一下, 日志打印出来:

```shell
2017-05-30 17:12:21.183  INFO 16254 --- [nio-9999-exec-2] c.alipay.liuqi.aspect.ControllerAspect   : ====AroundWebController: start around ===
2017-05-30 17:12:21.184  INFO 16254 --- [nio-9999-exec-2] c.alipay.liuqi.aspect.ControllerAspect   : ===BeforeWebController: start visiting===
2017-05-30 17:12:21.189  INFO 16254 --- [nio-9999-exec-2] troller$$EnhancerBySpringCGLIB$$efd51a80 : I am controller
2017-05-30 17:12:21.194  INFO 16254 --- [nio-9999-exec-2] c.alipay.liuqi.aspect.ControllerAspect   : ====AroundWebController: end around ===
2017-05-30 17:12:21.194  INFO 16254 --- [nio-9999-exec-2] c.alipay.liuqi.aspect.ControllerAspect   : ====AfterWebController: end visiting===
2017-05-30 17:12:21.194  INFO 16254 --- [nio-9999-exec-2] c.alipay.liuqi.aspect.ControllerAspect   : ====AfterReturnWebControlelr: return normal ===
```

## 6.3 AOP 小结

### 其他说明

1. Pointcut 支持正则表达式
2. 除了 @Before 和 @After, 还有其他的注解, 说明如下:

```xml
@AfterReturning: 切面点正常执行后, 才执行 AfterReturning 注解下面代码

@AfterThrowing: 切面点抛出异常后, 执行 AfterThrowing 注解对应的代码

@Around: 可以同时在所拦截方法的前后, 执行一段逻辑, 比如可以统计一下方法执行的时间

```

切面小结:

1. 定义切面点
2. 实现切面点前后要做的事情
3. 排好优先级


# 7. 其他

## 7.1 使用  LiveReload  热刷新

热部署提到了修改完代码, 不需要重启工程, 但是有时候前端页面还是需要刷新的, 查看热部署文档时, 看到了 [LiveReload](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#using-boot-devtools-livereload), 热刷新简直好用极了!!!

### 7.1.1 添加 devtools 依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional> <!-- 这个需要为 true 热部署才有效 -->
        </dependency>
```

### 7.1.2 设置 livereload 属性为 true

修改 <code>application-dev.profiles</code> :

```xml
# 配置热部署, 热刷新
spring:
  devtools:
    livereload:
      enabled: true
```

### 7.1.3 浏览器安装 livereload 插件

[插件地址](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei)

启动工程后, 日志会显示 livereload 占用的端口:

```xml
2017-05-30 17:12:09.505  INFO 16254 --- [  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
```

## 7.2 未提到的

健康性检查是工程非常重要的指标, 因为没怎么用到, 暂未提及.

- 接口测试
- 健康性检查(Spring Boot Actuator)


# 参考文档

1. [SpringBoot 官方文档](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-documentation)

2. [SpringBoot进阶之Web进阶--视频教程](http://www.imooc.com/learn/810)

3. [Spring Boot中使用Spring-data-jpa让数据访问更简单、更优雅](http://blog.didispace.com/springbootdata2/)

4. [stackoverflow--JPA 时间设置]((https://stackoverflow.com/questions/221611/creation-timestamp-and-last-update-timestamp-with-hibernate-and-mysql)

5. [https://github.com/spring-projects/spring-boot](https://github.com/spring-projects/spring-boot)
