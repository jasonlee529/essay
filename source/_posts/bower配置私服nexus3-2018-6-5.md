---
title: bower配置私服nexus3
date: 2018-06-05 19:30:30
categories:
- nexus
tags:
- bower
- nexus
- nexus3
---

> 内容来自 https://help.sonatype.com/repomanager3/bower-repositories#BowerRepositories-BrowsingBowerRepositoriesandSearchingPackages
> nexus3.0版本不仅提供了maven的私服，还可以托管docker、npm、bower甚至是python的仓库，为搭建统一的私服平台提供了便利。
> 
# Bower 仓库


##### * [1.简介](#1)  
##### * [2.代理仓库](#2)
##### * [3.本地私服仓库](#3)
##### * [4.公开仓库组](#4)
##### * [5.安装bower](#5)
##### * [6.配置私服下载](#6)
##### * [7.浏览和检索依赖](#7)
##### * [8.上传bower包](#8)

<h2 id="1">简介</h2>
 Bower是一个前端包管理工具。

NexusOSS是一个强大的Maven仓库管理器，它极大地简化了自己内部仓库的维护和外部仓库的访问。利用Nexus你可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个Artifact。Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。3.0版本之后加入了npm、bower、docker还有.net的仓库管理。

<h2 id="2">代理仓库</h2>
NexusOss可以对bower的外部仓库进行代理，比如代理官方库：https://registry.bower.io 。在官方库速度较慢的情况下非常好用。

创建bower代理库的步骤：
* 选择创建repository,类型为bower(proxy)
* 定义仓库名称：bower-proxy
* 定义代理的仓库地址，例如官方仓库： https://registry.bower.io
* 选择合适的存储

<h2 id="3">本地私服仓库</h2>
NexusOss支持创建本地私服用于管理bower包。私服仓库扮演了一个权威角色，定义包的URL和名称的关系。

创建本地私服仓库的操作步骤:

* 选择创建repository,类型为bower(hosted)
* 定义名称：bower-hosted
* 选择合适的存储

<h2 id="4">公开仓库组</h2>
推荐使用仓库组暴露Bower仓库地址。仓库组既可以暴露多个代理仓库和多个本地私服仓库，当代理仓库或者私服变动的时候不会影响到使用者。

创建步骤如下:

* 选择创建repository,类型为bower(group)
* 定义名称：bower-all
* 选择合适的存储
* 添加bower-proxy和bower-hosted仓库到组中
<h2 id="5">安装bower</h2>
通过npm安装：
``` shell
npm install -g bower 
$ bower -v
1.7.7 
```
使用Bower私服需要添加一个解析自定义URL的组件来完成解析NexusOss的工作。可以通过下面两种方式引入：
```shell
npm install -g bower-nexus3-resolver
```
或者引入到package.json中
```json
"devDependencies" : {
   "bower-nexus3-resolver" : "*"
}
```
<h2 id="6">配置私服下载</h2>
一旦配置了Bower私服，就需要配置.bowerrc文件来实现到私服的URL解析。一般需要配置.bowerrc文件，可以配置全局的.bowerrc，位于$HOME目录下，也可以配置在项目内部的.bowerrc文件中。
```json
{
  "registry" : {
    "search" : [ "http://localhost:8081/repository/bower-all" ]
   },
 "resolvers" : [ "bower-nexus3-resolver" ]
}
```
测试下效果：

``` shell
$ bower install jquery
bower jquery#*
  not-cached nexus+http://localhost:8081/repository/bower-all/jquery#*
bower jquery#*
  resolve nexus+http://localhost:8081/repository/bower-all/jquery#*
bower jquery#*
  resolved nexus+http://localhost:8081/repository/bower-all/jquery#2.2.0
bower jquery#^2.2.0  install jquery#2.2.0

jquery#2.2.0 bower_components/jquery
```


<h2 id="7">浏览和检索依赖</h2>
NexusOSS提供了网页检索和浏览包，当然也可以通过Bower的命令行参数来检索。

<h2 id="8">上传bower包</h2>
发布Bower包，需要配置.bowerrc文件，制定register的位置；需要指定包的git路径。
```json
{
    "registry" : {
        "search" : [
            "http://192.168.1.62:8081/nexus/repository/bower-all"
        ],
        "register" : "http://admin:admin123@192.168.1.62:8081/nexus/repository/bower-hosted"
   },
   "resolvers" : [ "bower-nexus3-resolver" ]
}
```
执行上传操作:
```shell
bower register example-package git://gitserver/project.git
```
测试安装:
```shell
bower install example-package
```

##　小结：
上次部分注意的是register的URL必须把授权参数填入，因为NexusOss侧需要权限验证，不写会一直报401的错误。