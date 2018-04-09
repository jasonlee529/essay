---
title: SDKMAN搭建Java开发环境
date: 2018-04-09 09:44:12
categories:
- 工具
tags:
- 工具
- 管理工具
---

> 作为一个Java攻城狮，随着项目的数目和oracle的努力工作，很容易遇到安装多版本的jdk的问题。

### 为什么需要多个JDK?
JDK全称是Java SE Development Kit .Java历经了二十几年的发展，到现在已经发布到了JDK10的版本。随着版本的改进，新的特性越来越多的加入，类似5.0引入的泛型，8.0引入的lambda和stream API都对开发有极大的效率提升，不可避免的需要对技术进行升级。同时，主流的开发框架工具spring，mybatis，maven等都会跟随jdk版本进行升级，所以需要一个开发者同时具备多个开发环境。

除了jdk，java的开发环境中，还有maven/gradle，idea/myeclipse等工具需要随着jdk版本进行升级。
### 经典的配合方案
1. 到[oracle官网](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载jdk,下载对应自己系统版本的jdk安装包/压缩包。
2. 配置环境变量，__JAVA_HOME__,__CLASSPATH__,并将__JAVA_HOME\bin__目录加入__PATH__.
3. 去[maven官网](http://maven.apache.org/download.cgi)下载maven的安装文件。
4. 配置环境变量__M2_HOME__。

如果是gradle用户，请替换3-4步骤。
### 如何拥有多个JDK?

#### 软链接方案
软链接的做法很简单，比如：
``` shell
$: ll
drwxr-xr-x  8 jason jason      4096 12月 20 08:24 jdk1.8.0_161
$: ls -n jdk1.8.0_161 jdk
$: ll
lrwxrwxrwx  1 jason jason        12 3月  21 09:33 jdk -> jdk1.8.0_161/
drwxr-xr-x  8 jason jason      4096 12月 20 08:24 jdk1.8.0_161/
```
然后将软链接__jdk__配置为__JAVA_HOME__。当增加jdk版本时，需要这么做，
``` shell
$: ll
lrwxrwxrwx  1 jason jason        12 3月  21 09:33 jdk -> jdk1.8.0_161/
drwxr-xr-x  8 jason jason      4096 12月 20 08:24 jdk1.8.0_161
drwxr-xr-x  8 jason jason      4096 12月 20 08:24 jdk1.7.0_81
$: rm -rf jdk
$: ls -n jdk1.7.0_81 jdk
$: ll
lrwxrwxrwx  1 jason jason        12 3月  21 09:33 jdk -> jdk1.7.0_81/
drwxr-xr-x  8 jason jason      4096 12月 20 08:24 jdk1.8.0_161
drwxr-xr-x  8 jason jason      4096 12月 20 08:24 jdk1.7.0_81
```
类似的maven也可以用同样的方式实现，但是这个办法问题在于，每个版本的jdk或者maven需要手工去下载，然后手工去更改软链接。比较费时，这时候，作为能机器做的绝不手工的程序员能想到了，可以写一个shell脚本，切换版本啊，幸运的是已经有人提供了这个服务，请出主角SDKMAN.
#### SDKMAN ! The Software Development Kit Manager
顾名思义，软件开发工具吧的管理工具。那么sdkman可以管理哪些kit呢？ant,java,gradle,maven,springboot-cli,groovy，kotlin,scala这些基于jvm的开发语言都包括进来。
##### 如何安装？
安装很简单，下载一个shell脚本，执行后就可以了。
``` shell
$ curl -s "https://get.sdkman.io" | bash
$ source "$HOME/.sdkman/bin/sdkman-init.sh"
$ sdk version
sdkman 5.0.0+51
```
##### 日常使用
以安装java为例
``` shell
$ sdk list java

================================================================================
Available Java Versions
================================================================================
     9.0.4-zulu                                                                    
     9.0.4-oracle                                                                  
     9.0.4-openjdk                                                                 
     8.0.163-zulu                                                                  
     8.0.161-oracle                                                                
     7.0.141-zulu                                                                  
     6.0.93-zulu                                                                   
     10.0.0-zulu                                                                   
     10.0.0-oracle                                                                 
     10.0.0-openjdk                               
$ sdk install java 8.0.161-oracle

Oracle requires that you agree with the Oracle Binary Code License Agreement
prior to installation. The license agreement can be found at:

  http://www.oracle.com/technetwork/java/javase/terms/license/index.html

Do you agree to the terms of this agreement? (Y/n): y


Downloading: java 8.0.161-oracle
In progress...

######################################################################## 100.0%

Repackaging Java 8.0.161-oracle...

Done repackaging...

Installing: java 8.0.161-oracle
Done installing!


Setting java 8.0.161-oracle as default.
$ java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
$ sdk list java

================================================================================
Available Java Versions
================================================================================
     9.0.4-zulu                                                                    
     9.0.4-oracle                                                                  
     9.0.4-openjdk                                                                 
     8.0.163-zulu                                                                  
 > * 8.0.161-oracle                                                                
     7.0.141-zulu                                                                  
     6.0.93-zulu                                                                   
     10.0.0-zulu                                                                   
     10.0.0-oracle                                                                 
     10.0.0-openjdk                                    
```
以上有一点需要主要，下载jdk的时候需要同意oracle的协议，这个在oracle官网下载的时候也需要确认。
一个jdk就配置好了。
如果需要切换一个jdk版本呢？
``` shell
$ sdk install java 7.0.141-zulu  

Downloading: java 7.0.141-zulu

In progress...

######################################################################## 100.0%

Repackaging Java 7.0.141-zulu...

Done repackaging...

Installing: java 7.0.141-zulu
Done installing!

Do you want java 7.0.141-zulu to be set as default? (Y/n): n
$ sdk default java 7.0.141-zulu

Default java version set to 7.0.141-zulu
$ java -version
openjdk version "1.7.0_141"
OpenJDK Runtime Environment (Zulu 7.18.0.3-linux64) (build 1.7.0_141-b11)
OpenJDK 64-Bit Server VM (Zulu 7.18.0.3-linux64) (build 24.141-b11, mixed mode)
```
版本顺利改变。
> TIPS zulu的jdk是基于openjdk的一个改进版本，是另一个jvm的实现。

这样SDKMAN就完成了jdk的切换，类似的道理maven，groovy等都可以这样实现。

##### 总结下吧
SDKMAN安装简单，方便切换多个版本，主要的是省去了去各个网站的单独下载。还是挺方便的。
