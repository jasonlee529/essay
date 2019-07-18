---
title: SpringCloud配置碎碎念
categories:
  - Spring
  - SpirngCloud
tags:
  - Spring
  - SpringCloud
  - Java
date: 2019-07-18 16:36:00
---
SpringCloud配置信息部分解释如下：

#### eureka.instance.prefer-ip-address

``` Java
/**
* Flag to say that, when guessing a hostname, the IP address of the server should be
* used in prference to the hostname reported by the OS.
*/
private boolean preferIpAddress = false;
```
这段的意思是是否需要将serverName 转换为IP地址注册到eureka-server中。
两个例子：
1115 端口
``` yml
server:
  port: 1115
eureka:
  instance:
    prefer-ip-address: true
```

1116端口
``` yml
server:
  port: 1116
eureka:
  instance:
    prefer-ip-address: false
```
结果：1115端口转换为了IP地址注册，1116端口没有注册转换名称给服务器。
``` jason
[
"URI：http://192.168.1.26:1115，service_id：CLIENT-CONSUMER",
"URI：http://peer1:1116，service_id：CLIENT-CONSUMER",
]
```
