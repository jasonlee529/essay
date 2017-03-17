---
title: Spring.profiles多环境配置最佳实践
date: 2017-03-17 11:20:11
categories:
 - Spring
tags:
 - Spring
 - Java
---
Spring的profiles机制，是应对多环境下面的一个解决方案,比较常见的是开发和测试环境的配置。

## 配置项目

Spring的profiles有两个变量可以配置
* spring.profiles.default 默认值，优先级低。当active没有配置时，使用此变量。
* spring.profiles.active 优先级高，指定当前容器使用哪个profile。

## 一般用法
### 声明多profile
如果使用spring的profiles机制，第一步要在applicationContext.xml中配置多环境实例。
``` xml
 <beans profile="development">
    <!-- 开发环境，具体加载bean或者properties文件 -->
 </beans>
 <beans profile="test">
   <!-- 测试环境，具体加载bean或者properties文件 -->
 </beans>   
```
### 激活profile
在J2EE项目中，一般通过web.xml配置。
``` xml
<context-param>
   <param-name>spring.profiles.default</param-name>
   <param-value>development</param-value>
</context-param>
```
或者
``` xml
<context-param>
   <param-name>spring.profiles.default</param-name>
   <param-value>test</param-value>
</context-param>
```
两个方式都可以，一般推荐第一种。主要原因在于：方便扩展，没有指定具体profile，可通过其他方式覆盖；一般情况下开发环境是使用最多的情况。

### 测试环境激活profile
Spring-test 中提供了一个__ActiveProfiles__的注解，可以标注当前测试用例使用的具体profile
``` java
@ActiveProfiles("test")
public abstract class SpringTransactionalTestCase {
}
```

### 使用java运行参数指定profiles
java运行时可以指定jvm内存参数，这个大家都知道。形式类似下面这样去指定java虚拟机启动的内存设置。
``` xml
JAVA_OPTS=" -Xms1024m -Xmx1024m  -XX:PermSize=512m -XX:MaxPermSize=512m"
```
Java也可以指定启动的环境变量，比如CATALINA_HOME啊之类的，通过ps命令，可以清楚的看到，java启动的时候不同的软件是通过不同的-D参数来达到效果的。tomcat的启动情况:
``` shell
root      2284     1  0 3月16 ?       00:01:54 /opt/jdk/bin/java -Djava.util.logging.config.file=/opt/apache-tomcat-8.0.32/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -server -Xms1024m -Xmx1024m -XX:PermSize=512m -XX:MaxPermSize=512m -Djava.endorsed.dirs=/opt/apache-tomcat-8.0.32/endorsed -classpath /opt/apache-tomcat-8.0.32/bin/bootstrap.jar:/opt/apache-tomcat-8.0.32/bin/tomcat-juli.jar -Dcatalina.base=/opt/apache-tomcat-8.0.32 -Dcatalina.home=/opt/apache-tomcat-8.0.32 -Djava.io.tmpdir=/opt/apache-tomcat-8.0.32/temp org.apache.catalina.startup.Bootstrap start
```
回到正题，那这样，是不是也可以通过这种形式，来指定profile的呢？答案是肯定的。
修改tomcat启动脚本，直接修改JAVA_OPTS:
``` xml
JAVA_OPTS=" -Xms1024m -Xmx1024m  -XX:PermSize=512m -XX:MaxPermSize=512m -Dspring.profiles.active=test"
```
启动tomcat，发现整个系统启动时，使用profile=test的bean被激活了，证明配置生效。

## 总结
使用java的系统参数-D方式即减轻了耦合性，也降低了开发和维护之间交流协作的部分。当有多个部署环境时，提前部署好指定的profile,这点在datasource的指定上面极为好用。
