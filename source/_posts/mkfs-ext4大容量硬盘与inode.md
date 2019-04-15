---
title: mkfs.ext4大容量硬盘与inode
tags:
  - inode
  - mkfs.ext4
  - ext4
  - linux
categories:
  - linux
date: 2019-04-16 00:13:00
---
有一台机器，已经建立了一个vg，容量是20T，现在准备新建一个lv，作为数据盘根目录，创建此LV过程中出现了各种的问题。

环境：
``` shell 
centos 6.5
e2fsprogs 1.41
磁盘分区：gpt
```

### 问题1:New size too large to be expressed in 32 bits

ext4分区理论可支持最大1EB，单文件体积16TB。但格式化的过程出现了：__New size too large to be expressed in 32 bits__ 这个异常，晚上检索查找到，老版本的e2fsprogs命令存在16T的限制，此BUG修复要在[E2fsprogs 1.42.12 (August 29, 2014)](http://e2fsprogs.sourceforge.net/e2fsprogs-release.html#1.44.5)版本。
centos6.5默认是1.41，刚好错过一个版本，因为是甲方远程主机，更新出问题还要出差去修复，没敢做。

解决方案就是分成了两个10T的lv使用。

#### 问题2：mkfs.ext4格式化极慢

