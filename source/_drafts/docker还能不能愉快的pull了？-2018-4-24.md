---
title: docker还能不能愉快的pull了？
date: 2018-04-24 17:58:56
categories:
- docker
tags:
- docker
---

最近貌似docker的镜像又被墙了，官方镜像本来就慢的要杀人，第三方的elk镜像不知道怎么也挂了，pull的时候一直报超时？

docker还有一个叫做http_proxy的东西，可通过代理获取镜像？
但是，有多台机器都要用到镜像？都陪代理吗？不行，心疼代理钱。那只能再搞一个私有的docker registry。

看看，我原本只是要一个镜像，结果平白添加了两件事？

## 还原问题
``` shell
$ docker pull docker.elastic.co/elasticsearch/elasticsearch:6.2.4

```

## 添加代理


## 添加registry


