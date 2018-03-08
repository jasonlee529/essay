---
title: SVN迁移到gitlab的步骤和问题
date: 2018-03-08 17:12:32
categories:
 - 工具
tags:
 - SVN
 - git
 - 工具
---
工具使用svn2git ，gitlab官方推荐的工具，安装方法详细见[官网](https://docs.gitlab.com/ce/user/project/import/svn.html)。

迁移步骤分为４步：
1. 确定svn路径及目录分布（branch,tags,trunk)目录。
2. 执行迁出代码和转换为git工程（svn2git)
3. 创建gitlab工程。
4. 提交到git。

#### 详细步骤
##### 确定svn路径及目录分布
首先要确认待转换的SVN目录，比如svn://svn.com/web/product/sample

这个目录下面有branch,tags和trunk目录。

>svn的标准目录是branches,tags，trunk三个目录，分别存放分支，版本和主干代码。

##### 执行迁出代码和转换为git工程

svn2git主要api解释
```shell
Usage: svn2git SVN_URL [options]
Specific options:
        --trunk TRUNK_PATH           重新指定trunk目录路径 (default: trunk)
        --branches BRANCHES_PATH     重新指定braches目录，在本文中需指向branch目录
        --tags TAGS_PATH             重新指定tags目录，在本文中需指向tags目录

        --authors AUTHORS_FILE       帐号映射文件
        --exclude REGEX              排除目录
    -v, --verbose                  　过程日志限制
    -h, --help                       Show this message
```
authors.txt是映射svn用户和gitlab目录的情况。比如
```txt
abc=abc<abc@gitlab.com>
```
如果现有的目录结构不一致，需要在目录里面填写对应的目录映射。执行以下命令，根据svn工程的大小决定执行时间。
```shell
svn2git svn://svn.com/web/product/sample --branches branch --tags tags --authors authors.txt -v
```
###### 可能出现的问题
1. no associate commit message
  这个是由于branch或者tag的路径映射问题，修改.git/config文件中的branches和tags的映射可解决。
  ``` txt
  branches = product/checkup/branch/*:refs/remotes/svn/*
	tags = product/checkup/tags/*:refs/remotes/svn/tags/*
  ==>
  branches = product/checkup/branch/*:refs/remotes/svn/branches/*
	tags = product/checkup/tags/*:refs/remotes/svn/tags/*
  ```
2.


##### 创建gitlab工程
在gitlab 中创建对应的工程，获得指定的工程路径，推荐使用ssh通道提交。
使用ssh通道提交，需要提前配置ssh公钥，具体方法参考官方[文档](https://docs.gitlab.com/ce/ssh/README.html)

##### 提交到git
```shell
git push --all
git push --tags
```
然后gitlab页面查看时，能找到数量大于svn中的branch和tags即可。
{% asset_img gitlab-branches-tags.png  %}


#### 完整shell流程
``` shell
svn2git svn://svn.com/web/product/sample --branches branch --tags tags --authors authors.txt -v
git remote add origin git@gitlab.com:sample/sample.git
git push --all
git push --tags
```
