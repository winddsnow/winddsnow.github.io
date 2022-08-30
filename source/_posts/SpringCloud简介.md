---
title: SpringCloud简介
categories:
  - 编程
tags:
  - Java
  - spring
  - springcloud
  - 微服务
date: 2022-08-29 08:55:47
---

# 认识微服务
- 微服务是系统架构的一种设计风格,将一个原本独立的服务拆分成多个小型服务,每个服务独立运行在在各自的进程中,服务之间通过 HTTP RESTful API 进行通信.每个小型的服务都围绕着系统中的某个耦合度较高的业务进行构建。
- 微服务是一种经过良好设计的分布式架构方案，而全球的互联网公司都在积极尝试自己的微服务落地方案。其中在java领域最引人注目的是SpringCloud提供的方案。
1. 单一职责:微服务拆分粒度更小，每个服务都应对唯一的业务能力，做到单一职责
2. 自治:团队独立、技术独立、数据独立，独立部署和交付
3. 面向服务:服务提供统一标准的接口，与语言无关、与技术无关
4. 隔离性强:服务调用做好隔离、容错、降级，避免出现级联问题

# SpringCloud
- SpringCloud是目前国内使用最广泛的微服务技术栈。官网地址：https://spring.io/projects/spring-cloud。
- SpringCloud集成了各种微服务功能组件，并**基于SpringBoot**实现了这些组件的自动装配，从而提供了良好的开箱即用体验
- SpringCloud:SpringCloud是微服务架构的一站式解决方案，集成了各种优秀微服务功能组件

### 服务注册发现
- Eureka
- Nacos
- Consul

### 服务远程调用
- OpenFeign
- Dubbo

### 服务链路监控
- Zipkin
- Sleuth

### 统一配置管理
- SpringCloudConfig
- Nacos

### 统一网关路由
- SpringCloudGateway
- Zuul

### 流控、降级、保护
- Hystix
- Sentinel

## 服务提供者、服务消费者

- 服务提供者：一次业务中，被其它微服务调用的服务。（提供接口给其它微服务）
- 服务消费者：一次业务中，调用其它微服务的服务。（调用其它微服务提供的接口）

#  注册中心-Eureka
## 搭建注册中心
1. pom.xml引入依赖
```xml
<dependencies>
    <!--EurekaServer包-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```
2. 编写启动类，添加@EnableEurekaServer注解
3. 添加application.yml文件，编写配置
```yaml
server:
  port: 8001    #端口号
spring:
  application:
    name: eureka-server # 应用名称，会在Eureka中作为服务的id标识（serviceId）
eureka:
  client:
    register-with-eureka: false   #是否将自己注册到Eureka中
    fetch-registry: false   #是否从eureka中获取服务信息
    service-url:
      defaultZone: http://localhost:8001/eureka
```

## 服务注册
- 服务提供者注册
1. pom.xml
```XML
<!--EurekaClient包-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
2.  修改application.yml
```YAML
eureka:
  client:
    service-url:
      # EurekaServer的地址
      defaultZone: http://localhost:8001/eureka
  instance:
    #以IP地址注册到服务中心
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${server.port}:@project.version@
```
- 服务消费者注册
与提供者一致
## 总结
- 搭建EurekaServer
  - 引入eureka-server依赖
  - 启动类上添加@EnableEurekaServer注解
  - 在application.yml中配置eureka地址
- 服务注册
  - 引入eureka-client依赖
  - 在application.yml中配置eureka地址

- 服务发现
  - 引入eureka-client依赖
  - 在application.yml中配置eureka地址
  - 给RestTemplate添加@LoadBalanced注解
  - 用服务提供者的服务名称远程调用(由原来的ip:port改服务名(spring.application.name))

# 负载均衡Ribbon
## Ribbon是什么？
Ribbon是Netflix发布的负载均衡器，有助于控制HTTP客户端行为。为Ribbon配置服务提供者地址列表后，Ribbon就可基于负载均衡算法，自动帮助服务消费者请求。
概念：Ribbon是基于Http协议请求的客户端负载均衡器，能实现很丰富的负载均衡算法。

## 负载均衡流程图

```properties
1:用户发起请求，会先到达xxxxx-order服务
2:xxxxx-order服务通过Ribbon负载均衡器从eurekaserver中获取服务列表
3:获取了服务列表后，轮询（负载均衡算法）调用
```

## 负载均衡算法
| **内置负载均衡规则类**    | **规则描述**                                                 |
| ------------------------- | ------------------------------------------------------------ |
| RoundRobinRule            | **简单轮询**服务列表来选择服务器。                           |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略： <br/>（1）在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加。<br/>（2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的`<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit`属性进行配置。 |
| WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。 |
| ZoneAvoidanceRule【默认】 | 以区域可用的服务器为基础进行服务器的选择。使用Zone对服务器进行分类，这个Zone可以理解为一个机房、一个机架等。而后再对Zone内的多个服务做轮询。它是Ribbon默认的负载均衡规则。 |
| BestAvailableRule         | 忽略哪些短路的服务器，并选择并发数较低的服务器。             |
| RandomRule                | 随机选择一个可用的服务器。                                   |
| RetryRule                 | 重试机制的选择逻辑                                           |

## Ribbon负载均衡算法使用
Ribbon负载均衡算法的使用有2种方式
- 代码方式: 全局，所有的服务提供提供者采用相同的负载均衡策略

  - 注册IRule接口的实现类(负载均衡算法)：在`xxxxx-order`的启动类中添加如下负载均衡注册代码：

```java
    /**
     * 随机负载均衡算法
     * @return
     */
    @Bean
    public IRule randomRule() {
        return new RandomRule();
    }
```

- 配置方式 局部配置，为每个服务提供者制定不同的负载均衡策略
- 为指定服务配置负载均衡算法：在`xxxxx-order`的核心配置文件中添加如下配置：
```yaml
    #注意配置到跟节点

    #指定服务使用指定负载均衡算法
    xxxxx-user:
      ribbon:
        NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #负载均衡规则
```

## 从懒加载 变为 饥饿加载

Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长。
而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，在`xxxxx-order`的核心配置文件中，添加如下配置开启饥饿加载：

```yaml
#注意配置到根节点

#饥饿加载
ribbon:
  eager-load:
    clients: xxxxx-user #开启饥饿加载 
    enabled: true #指定对user这个服务饥饿加载
```

## Ribbon总结
- Ribbon负载均衡规则
  - 规则接口是IRule
  - 默认实现是ZoneAvoidanceRule，根据zone选择服务列表，然后轮询

- 负载均衡自定义方式
  - 代码方式：配置灵活，但修改时需要重新打包发布，全局配置
  - 配置文件方式：直观，方便，无需重新打包发布，但是无法做全局配置，指定某个提供者的负载均衡策略【推荐】

- 饥饿加载, 拉取服务提供者的方式
  - 开启饥饿加载
  - 指定饥饿加载的微服务名称

##  [Ribbin负载均衡原理与源码调试](https://winddsnow.github.io/2022/08/30/Ribbin%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E6%BA%90%E7%A0%81%E5%8E%9F%E7%90%86%E4%B8%8E%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%95/)


# http客户端Feign
- Feign是一个声明式的http客户端，官方地址：https://github.com/OpenFeign/feign 其作用就是帮助我们优雅的实现http请求的发送

## Feign入门案例
定义和使用Feign客户端的步骤如下：

```properties
1:引入依赖包 spring-cloud-starter-openfeign
2:添加注解@EnableFeignClients开启Feign功能
3:消费者端定义远程调用接口，在接口中远程调用的【服务名字】、【方法签名】
4:消费者端注入接口，执行远程调用(接口)
```

### 引入依赖
```xml
<!--openfeign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 开启Feign功能
在启动类添加`@EnableFeignClients`注解开启Feign功能
```java
@SpringBootApplication
@EnableFeignClients
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
    //...其他略
}
```
### 定义远程调用接口
创建接口`xxxClient`
```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * order调用user服务(代替了 String url = "http://xxx-user/user/" + orderInfo.getUserId();)
 * 1.接口上使用@FeignClient(value="被调用服务名")
 * 2.定义被调用接口中的方法（基于被调用的controller编写）
 *  2.1 requestMapping中的路径必须是全路径(controller类上的+方法上的)
 *  2.2 使用PathVariable注解，必须取别名
 */
@FeignClient(value = "xxxx-user")
public interface UserClient {

    /**
     * 调用用户微服中controller的方法
     */
    @GetMapping(value = "/user/{id}")
    public User one(@PathVariable(value = "id") Long id);
}
```
主要是基于SpringMVC的注解来声明远程调用的信息，比如：

- 服务名称：user
- 请求方式：GET
- 请求路径：/user/{username}
- 请求参数：String username
- 返回值类型：User

### 远程调用
修改`order`的`OrderServiceImpl.one()`方法，执行远程调用
```java
@Autowired
private UserClient userClient;

/**
 * 根据ID查询订单信息
 */
@Override
public OrderInfo findById(Long id) {
    //1.查询订单
    OrderInfo orderInfo = orderDao.selectById(id);
    //2.根据订单查询用户信息->需要调用  【item-user】  服务
    User user = userClient.one(orderInfo.getUserId());
    //3.封装user
    orderInfo.setUser(user);
    //4.返回订单信息
    return orderInfo;
}
```

### Feign其他功能
Feign运行自定义配置来覆盖默认配置，可以修改的配置如下：

| 类型                   | 作用             | 说明                                                   |
| ---------------------- | ---------------- | ------------------------------------------------------ |
| **feign.Logger.Level** | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL         |
| feign.codec.Decoder    | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为java对象 |
| feign.codec.Encoder    | 请求参数编码     | 将请求参数编码，便于通过http请求发送                   |
| feign. Contract        | 支持的注解格式   | 默认是SpringMVC的注解                                  |
| feign. Retryer         | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试 |

```properties
NONE:默认的，不显示任何日志
BASIC:仅记录请求方法、URL、响应状态码以及执行时间
HEADERS:除了BASIC中定义的信息以外，还有请求和响应的头信息
FULL:除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据
```
### Feign日志配置,结合SpringBoot日志
```yaml
logging:
  pattern:
    # 输出到控制台
    console: '%d{HH:mm:ss} %-5level %msg [%thread] - %logger{15}%n\'
  level:
    root: info # 全局用info
    # 只有在com.xxxx包下才输出debug信息
    com:
      xxxx: debug
```

配置Feign日志有两种方式：
- 配置文件方式
 - 全局生效

```yaml
    feign:
      client:
        config:
          default: #这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
            loggerLevel: HEADERS #日志级别
```

- 局部生效

```yaml
    feign:
      client:
        config:
          xxxxx-user: #指定服务
            loggerLevel: HEADERS #日志级别
```

- 代码方式
 - 注册日志级别

```java
    /**
     * 注册日志级别
     * @return
     */
    @Bean
    public Logger.Level feignLogLevel() {
        return Logger.Level.FULL;
    }
```

- 全局生效

```properties
    #如果是全局配置，则把它放到@EnableFeignClients这个注解中
    @EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class)
```

- 局部生效

```properties
    #如果是局部配置，则把它放到@FeignClient这个注解中
    @FeignClient(value = "xxxxx-user",configuration = FeignClientConfiguration.class)
```

### Feign性能优化
Feign底层的客户端实现：

- URLConnection：默认实现，不支持连接池
- Apache HttpClient ：支持连接池
- OKHttp：支持连接池



因此优化Feign的性能主要包括：

- 使用连接池代替默认的URLConnection
- 日志级别，最好用basic或none


Feign切换Apache HttpClient步骤如下：
```properties
1:引入依赖
2:配置连接池
```

1)引入依赖
在`xxxxx-order`中引入如下依赖：
```xml
<!--httpClient依赖-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

2)配置连接池
在`xxxxx-order`的核心配置文件`application.yml`中添加如下配置：

```yaml
feign:
  client:
    config:
      default: #这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: BASIC #日志级别
      xxxxx-user: #指定服务
        loggerLevel: BASIC #日志级别
  httpclient:
    enabled: true #开启feign对HttpClient的支持
    max-connections: 200 #最大的连接数
    max-connections-per-route: 50 #每个路径的最大连接数
```
## Feign最佳实践
方式一（继承）：给消费者的FeignClient和提供者的controller定义统一的父接口作为标准。
- 服务紧耦合
- 父接口参数列表中的映射不会被继承

方式二（抽取）(**推荐**)：将FeignClient抽取为独立模块，并且把接口有关的POJO、默认的Feign配置都放到这个模块中，提供给所有消费者使用

Feign最佳实现流程如上图所示：

```properties
实现最佳实践方式二的步骤如下：
1:创建xxxxx-api，然后引入feign的starter依赖 xxxxx-pojo依赖
2:将xxxxx-order中编写的UserClient继承到xxxxx-api项目中
3:在xxxxx-order中引入xxxxx-api的依赖，controller实现xxxxx-api中的接口
4:重启测试
```
1)引入依赖
创建xxxxx-api，然后引入feign的starter依赖 xxxxx-user依赖
```xml
<dependencies>
     <!--openfeign-->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>

     <!--httpClient依赖-->
     <dependency>
         <groupId>io.github.openfeign</groupId>
         <artifactId>feign-httpclient</artifactId>
     </dependency>

     <dependency>
         <groupId>com.xxxxx</groupId>
         <artifactId>xxxxx-pojo</artifactId>
         <version>1.0-SNAPSHOT</version>
     </dependency>
</dependencies>
```

2)编写的UserClient
将xxxxx-order中编写的UserClient复制到xxxxx-api项目中
```java
package com.xxxxx.client;

import com.xxxxx.user.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * order调用user服务(代替了 String url = "http://xxxxx-user/user/" + orderInfo.getUserId();)
 * 1.接口上使用@FeignClient(value="被调用服务名")
 * 2.定义被调用接口中的方法（基于被调用的controller编写）
 *  2.1 requestMapping中的路径必须是全路径(controller类上的+方法上的)
 *  2.2 使用PathVariable注解，必须取别名
 */
@FeignClient(value = "xxxxx-user")
public interface UserClient {

    /**
     * 调用用户微服中controller的方法
     */
    @GetMapping(value = "/user/{id}")
    public User one(@PathVariable(value = "id") Long id);
}
```

3)在xxxxx-order中引入xxxxx-api的依赖

```xml
<!--引入feign-api-->
<dependency>
    <groupId>com.xxxxx</groupId>
    <artifactId>xxxxx-api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

4) 【注意】当定义的FeignClient不在SpringBootApplication的**扫描包范围**时，这些FeignClient无法使用。
有两种方式解决：
方式一：指定FeignClient所在包（推荐-范围大）
`@EnableFeignClients(basePackages = "com.xxxxx.user.feign")`

方式二：指定FeignClient字节码

`@EnableFeignClients(clients = {UserClient.class})`

### Feign总结

- Feign的使用步骤
  - 引入依赖
  - 启动类添加@EnableFeignClients注解，如果feignclient接口不在启动类包下，则需要添加扫包(basePackages )
  - 编写FeignClient接口
  - 使用FeignClient中定义的方法代替RestTemplate

- Feign的日志配置:
  - 方式一是配置文件，feign.client.config.xxx.loggerLevel
  - 如果xxx是default则代表全局
  - 如果xxx是服务名称，例如userservice则代表某服务

- 方式二是java代码配置Logger.Level这个Bean
  - 如果在@EnableFeignClients注解声明则代表全局
  - 如果在@FeignClient注解中声明则代表某服务

- Feign的优化
  - 日志级别尽量用basic
  - 使用HttpClient或OKHttp代替URLConnection
    - 引入feign-httpClient依赖
    - 配置文件开启httpClient功能，设置连接池参数

- Feign的最佳实践：
  - 让controller和FeignClient继承同一接口
  - 将FeignClient、POJO、Feign的默认配置都定义到一个项目中，供所有消费者使用
