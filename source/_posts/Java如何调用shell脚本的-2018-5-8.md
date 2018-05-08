---
title: Java如何调用shell脚本的
date: 2018-05-08 11:59:42
categories:
- Java
tags:
- Java
- shell
---
有些时候会碰到这样的场景：java的功能里面要嵌入一个功能点，这个功能是通过是shell脚本实现的。这种时候就需要Java对脚本调用的支持了。

## 测试环境
Ubuntu16.04 i3-6100,12GB
## Hello World
来看一个基本的例子
```java
    Process exec = Runtime.getRuntime().exec(new String[] { "uname" ,"-a"});
    exec.waitFor();
    BufferedReader reader =
            new BufferedReader(new InputStreamReader(exec.getInputStream()));
    System.out.println(reader.readLine());

Linux jason-Inspiron-3650 4.4.0-121-generic #145-Ubuntu SMP Fri Apr 13 13:47:23 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
## 解读Process


## 与脚本交互


## 总结
