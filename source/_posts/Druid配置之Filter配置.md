---
title: Druid配置之Filter配置
tags:
  - Druid
  - Spring
  - Java
categories: []
date: 2019-04-16 15:28:00
---
Druid作为阿里巴巴出品的数据库连接池产品，虽然从坊间传闻hikaricp性能更好，奈何Druid还有监控功能啊，线上的SQL问题统计方便很多，所以必须要了解下Druid的配置

以下对Druid的依赖管理略过。
#### JavaEE版本

StatViewServlet是一个标准的javax.servlet.http.HttpServlet，需要配置在你web应用中的WEB-INF/web.xml中。
``` xml
  <servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
  </servlet>
  <servlet-mapping>
      <servlet-name>DruidStatView</servlet-name>
      <url-pattern>/druid/*</url-pattern>
  </servlet-mapping>
 ```
根据配置中的url-pattern来访问内置监控页面，如果是上面的配置，内置监控页面的首页是/druid/index.html

打开上面的配置能够打开页面，相关的sql统计功能还需要打开统计功能：
``` xml
 <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <!-- Connection Info -->
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="filters" value="stat,wall,config" />
    </bean>
```
这样就可以查看到统计功能了。

一般JavaEE的工程具备一个contextPath，访问路径就是：http://ip:port/${contextPath}/druid/index.html




#### SpringBoot版本
Spring Boot的配置更简单了，没有web.xml文件了，properties/yml的文件配置就够了。
``` yml
spring:
  datasource:
    druid:
      web-stat-filter:
        enabled: true
      stat-view-servlet:
        enabled: true
```
就开启了druid的监控，但是这个有个问题，就是不认__server.serverlet.path__配置项。

#### 补充


	如果用了shiro或者Spring-Security等权限管理，一定要关闭对druid的权限验证，而且也不推荐将druid合并到业务帐号里面。