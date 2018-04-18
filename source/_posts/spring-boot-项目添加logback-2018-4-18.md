---
title: spring-boot 项目添加logback
date: 2018-04-18 15:01:49
categories:
- spring
- spring-boot
tags:
- spring
- spring-boot
- log
- logback
---

使用spring-boot项目中添加日志输出，java的日志输出一共有两个大的方案log4j/log4j2 ，logback。log4j2算是对log4j的一个升级版本。
常规做法是引入slf4j作为日志入口，log4j或者logback选择一个做实现。spring的项目里面，只有spring-boot-starter-web用的是log4j，其他的用过的starter全部都是logback。

## 依赖 dependency-spring-boot-starter-logging
spring-boot-starter-logging还有两个很容易混淆的变种的starter  
__spring-boot-starter-log4j__ 和 __spring-boot-starter-log4j2__

前者在版本更新到了1.3之后就停止了，后者持续更新，需要注意下。
## dependencies
spring-boot-starter-logging 依赖有

| groupId                   | artifactId      | version |
|---------------------------|-----------------|---------|
| ch.qos.logback            | logback-classic | 1.2.3   |
| org.apache.logging.log4j  | log4j-to-slf4j  | 2.10.0  |
| org.slf4j                 | jul-to-slf4j    | 1.7.25  |


上面的列表中可知log的最終打印是使用logback實現的。项目中优先选用 __spring-boot-starter-logging__

## logback配置
先看一个典型的配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <property name="name" value="spring-boot-logging"/>
    <contextName>${name}</contextName>
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="rollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/infisa.${name}.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/infisa.clinic.%d{yyyy-MM-dd}.log</fileNamePattern>
        </rollingPolicy>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- project default level -->
    <logger name="cn.infisa" level="info"/>

    <!-- 定义根日志级别 -->
    <root level="info">
        <appender-ref ref="rollingFile"/>
        <appender-ref ref="console"/>
    </root>
   
</configuration>
```
### configuration
configuration是logback.xml的根节点，所有的配置项目都要位于configuration元素下面，可配置的属性有：

| 属性名称         | 默认值  | 说明|
|----------------|--------|---------------------------|
| scan           | true   |当此属性设置为true时，配置文件如果发生改变，将会被重新加载|
| scanPeriod     | 1min   | 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。|
| debug          | true   | 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。|


#### contextName
每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用<contextName>设置成其他名字，用于区分不同应用程序的记录。一旦设置，不能修改。

#### property
用来定义变量值的标签，<property> 有两个属性，name和value；其中name的值是变量的名称，value的值时变量定义的值。通过<property>定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量。
例如使用<property>定义上下文名称，然后在<contentName>设置logger上下文时使用。
#### appender
负责写日志的组件，它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名.
{% appenderClassDiagram.jpg  appender类图 %}
##### ConsoleAppender
日志输出到控制台，有以下子节点：
<encoder>：对日志进行格式化。
<target>：字符串System.out(默认)或者System.err（区别不多说了）

##### FileAppender
 把日志添加到文件，有以下子节点：
　　　　　　<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
　　　　　　<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
　　　　　　<encoder>：对记录事件进行格式化。（具体参数稍后讲解 ）
　　　　　　<prudent>：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。
##### RollingFileAppender
滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。有以下子节点：
　　　　　　<file>：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
　　　　　　<append>：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。	
　　　　　　<rollingPolicy>:当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类
　　　　　　class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy"： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：
　　　　　　　　<fileNamePattern>：必要节点，包含文件名及“%d”转换符，“%d”可以包含一个java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。
如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；
如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。	
　　　　　　　　<maxHistory>:
可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且<maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。

　　　　　　class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy"： 查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:
　　　　　　　　<maxFileSize>:这是活动文件的大小，默认值是10MB。
　　　　　　　　<prudent>：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。	

　　　　　　<triggeringPolicy >: 告知 RollingFileAppender 合适激活滚动。
　　　　　　class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy" 根据固定窗口算法重命名文件的滚动策略。有以下子节点：
　　　　　　　　<minIndex>:窗口索引最小值
　　　　　　　　<maxIndex>:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
　　　　　　　　<fileNamePattern>:必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip
#### logger
用来设置某一个包或具体的某一个类的日志打印级别、以及指定<appender>。<loger>仅有一个name属性，一个可选的level和一个可选的addtivity属性。
可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger
　　　　name: 用来指定受此loger约束的某一个包或者具体的某一个类。
　　　　level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。 如果未设置此属性，那么当前loger将会继承上级的级别。
addtivity: 是否向上级loger传递打印信息。默认是true。同<loger>一样，可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger。
#### root 
它也是<loger>元素，但是它是根loger,是所有<loger>的上级。只有一个level属性，因为name已经被命名为"root",且已经是最上级了。
　　　　level: 用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL和OFF，不能设置为INHERITED或者同义词NULL。 默认是DEBUG。	