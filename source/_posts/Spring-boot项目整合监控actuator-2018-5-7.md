---
title: Spring-boot项目整合监控actuator
date: 2018-05-07 11:53:18
categories:
- spring
- spring-boot
tags:
- spring
- spring-boot
- actuator
---
Spring-boot包含众多的starter，每一个starter都包含了一个方面的最佳实践，比如spring-boot-starter-web包含了构建web应用的依赖框架，spring-boot-starter-logging包含了日志输出所需框架的合集。而本文的主角spring-boot-starter-actuator包含了metrics的相关工具集。
## 什么是actuator?
>Spring Boot Actuator includes a number of additional features to help you monitor and manage your application when it’s pushed to production. You can choose to manage and monitor your application using HTTP or JMX endpoints. Auditing, health and metrics gathering can be automatically applied to your application. The user guide covers the features in more detail.
>Spring Boot Actuator包含了一些额外的特性用于生产环境中监控和管理spring　boot应用。actuator可以通过__HTTP__或者__JMX__对应用进行管控。可以自动收集审计、健康和度量信息。用户手册包含了更多的细节。

土话就是，线上应用都需要管控工具，省得大家再去造一个轮子了，spring-boot造好了一个，开箱即用，都到碗里来吧。


## actuator的dependency分析
以1.5.4.RELEASE版本为例
```shell
$ mvn dependency:tree
[INFO] \- org.springframework.boot:spring-boot-starter-actuator:jar:1.5.4.RELEASE:compile
[INFO]    +- org.springframework.boot:spring-boot-starter:jar:1.5.4.RELEASE:compile
[INFO]    |  +- org.springframework.boot:spring-boot:jar:1.5.4.RELEASE:compile
[INFO]    |  +- org.springframework.boot:spring-boot-autoconfigure:jar:1.5.4.RELEASE:compile
[INFO]    |  +- org.springframework.boot:spring-boot-starter-logging:jar:1.5.4.RELEASE:compile
[INFO]    |  |  +- ch.qos.logback:logback-classic:jar:1.1.11:compile
[INFO]    |  |  |  +- ch.qos.logback:logback-core:jar:1.1.11:compile
[INFO]    |  |  |  \- org.slf4j:slf4j-api:jar:1.7.22:compile
[INFO]    |  |  +- org.slf4j:jcl-over-slf4j:jar:1.7.25:compile
[INFO]    |  |  +- org.slf4j:jul-to-slf4j:jar:1.7.25:compile
[INFO]    |  |  \- org.slf4j:log4j-over-slf4j:jar:1.7.25:compile
[INFO]    |  +- org.springframework:spring-core:jar:4.3.9.RELEASE:compile
[INFO]    |  \- org.yaml:snakeyaml:jar:1.17:runtime
[INFO]    \- org.springframework.boot:spring-boot-actuator:jar:1.5.4.RELEASE:compile
[INFO]       +- com.fasterxml.jackson.core:jackson-databind:jar:2.8.8:compile
[INFO]       |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.8.0:compile
[INFO]       |  \- com.fasterxml.jackson.core:jackson-core:jar:2.8.8:compile
[INFO]       \- org.springframework:spring-context:jar:4.3.9.RELEASE:compile
[INFO]          +- org.springframework:spring-aop:jar:4.3.9.RELEASE:compile
[INFO]          +- org.springframework:spring-beans:jar:4.3.9.RELEASE:compile
[INFO]          \- org.springframework:spring-expression:jar:4.3.9.RELEASE:compile
```
核心依赖项目主要是spring-boot的相关基础组件和日志组件。
* sfl4j和logback相关的都是日志类的组件。
* jackson相关是json处理组件。
* spring-context和beans,core,aop,expression这些是任何spring项目几乎都需要的组件
* spring-boot还有spring-boot-autoconfig以及yaml是spring－boot的基础组建，提供默认配置和yml配置的解析支持。
* spring-boot-actuator提供了监控的特性

## 开箱即用
在pom.xml中添加：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>1.5.4.RELEASE</version>
</dependency>
```
然后直接启动就可以了，启动后可以查看最简单的http接口
```shell
$ curl localhost:8080/health
{
status: "UP",
diskSpace: {
status: "UP",
total: 196726059008,
free: 163859410944,
threshold: 10485760,
},
db: {
status: "UP",
database: "MySQL",
hello: 1,
},
}
```






