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
以下引自：[理解inode](https://www.cnblogs.com/xiexj/p/7214502.html)
>> inode是什么？

>> 理解inode，要从文件储存说起。

>> 文件储存在硬盘上，硬盘的最小存储单位叫做"扇区"（Sector）。每个扇区储存512字节（相当于0.5KB）。

>> 操作系统读取硬盘的时候，不会一个个扇区地读取，这样效率太低，而是一次性连续读取多个扇区，即一次性读取一个"块"（block）。这种由多个扇区组成的"块"，是文件存取的最小单位。"块"的大小，最常见的是4KB，即连续八个 sector组成一个 block。

>> 文件数据都储存在"块"中，那么很显然，我们还必须找到一个地方储存文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等等。这种储存文件元信息的区域就叫做inode，中文译名为"索引节点"。

>> 每一个文件都有对应的inode，里面包含了与该文件有关的一些信息。

inode也需要占用一些存储空间，因为inode是预分配的，所以格式化的时候就已经确定inode的数量了。

每个inode节点的大小，一般是128字节或256字节。inode节点的总数，在格式化时就给定，一般是每1KB或每2KB就设置一个inode。假定在一块1GB的硬盘中，每个inode节点的大小为128字节，每1KB就设置一个inode，那么inode table的大小就会达到128MB，占整块硬盘的12.8%。

格式化时，每bytes-per-inode(或inode_ratio)字节大小的空间就创建一个inode，在分区大小固定前提下，该值越大，inode个数越少，data block就越多，该值越小，inode个数越多，data block就越少；以默认值16384(即16KiB)为例，如果文件系统上所有文件大小均为16KiB(或平均值)，则data block耗尽的同时inode也将耗尽，二者占文件系统比例处于最理想状态，对于大量小文件的业务，通常将该值调小以增加inode数量；

``` /etc/mke2fs.conf
[defaults]
        base_features = sparse_super,filetype,resize_inode,dir_index,ext_attr
        default_mntopts = acl,user_xattr
        enable_periodic_fsck = 0
        blocksize = 4096
        inode_size = 256
        inode_ratio = 16384

[fs_types]
        ext3 = {
                features = has_journal
        }
        ext4 = {
                features = has_journal,extent,huge_file,flex_bg,uninit_bg,dir_nlink,extra_isize,64bit
                inode_size = 256
        }
        big = {
                inode_ratio = 32768
        }
        huge = {
                inode_ratio = 65536
        }
        news = {
                inode_ratio = 4096
        }
        largefile = {
                inode_ratio = 1048576
                blocksize = -1
        }
        largefile4 = {
                inode_ratio = 4194304
                blocksize = -1
        }
```

通过更改inode_ratio的数值来控制inode的数量，用来控制inode的数量及空间，这部分影响格式化的速度。

	如果是和上述相反的大文件存储业务，可以将inode_ratio值调大，以增加data block数量，如使用“-T largefile”选项对应的inode_ratio值为1MiB，在1.8TiB大小的分区创建ext4文件系统时，可增加20~30GiB左右的数据空间。

	对于不能准备评估或需求特殊(如海量小文件)的存储业务，可考虑使用ReiserFS、XFS、JFS等，以避免inode耗尽的风险