---
title: Hexo,Github开通博客
date: 2017-02-22 15:27:43
tags:
---
通过hexo本地编写博客文件，github做远程仓库保存，travis-ci做集成服务.

>1. 开通GitHub账号,创建目标库
>2. 配置本地NodeJs,Hexo,git环境
>3. 初始化Hexo博客库
>4. 开通GitPages服务
>5. 添加Travis-CI服务

## 1.开通GitHub账号
自行搜索把，已注册用户跳过。

创建一个Respository。

## 2.配置本地的Nodejs,Hexo,git环境
本人使用的是linux环境，装机自带git。没有的可以通过apt安装
```shell
$ sudo apt install git
```
git安装完成后推荐使用github的ssh通道上传下载代码，操作方便简单。
nodejs 可以自行下载配置也可以使用apt安装，不过鉴于npm官方库国内访问较慢，推荐使用阿里云镜像加速，[npm阿里云镜像配置](https://yq.aliyun.com/articles/5543 "")

node和git都配置好以后，
