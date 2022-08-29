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
- - SpringCloud集成了各种微服务功能组件，并**基于SpringBoot**实现了这些组件的自动装配，从而提供了良好的开箱即用体验
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


