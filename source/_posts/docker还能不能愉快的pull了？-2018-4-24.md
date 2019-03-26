---
title: docker还能不能愉快的pull了？
categories:
  - docker
tags:
  - docker
keywords: docker
date: 2018-04-24 17:58:00
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
``` shell
$ export ALL_PROXY=socks5://localhost:port 
$ docker pull image
```
## 添加registry
registry有很多种，内网版本，阿里云、DaoCloud等任意一家云厂商的registry。

如果公司的局域网，一定要搭建一个私服registry，速度提升的速度不要太明显了。推荐一下Nexus3,可以作为Maven、npm和docker三方私服。

如果在家办公，就一个电脑需要pull，那直接用云厂商的服务更好。

以阿里云为例：
在阿里云的产品目录中找到“容器镜像服务-镜像中心-镜像加速器”这个功能，如果第一次打开可能会提示开通。
里面有具体的安装方式。
``` shell
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://id.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
