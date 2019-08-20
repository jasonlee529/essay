---
title: Postgresql安装
categories:
  - postgresql
tags:
  - postgresql
keywords: postgresql
date: 2019-08-20 10:05:00
---
#### yum安装
过程在官方文档都有：https://www.postgresql.org/download/linux/redhat/ 
{% asset_img installation.png 下载地址 %}

####  安装完成后修改登录密码：
``` shell
[root@localhost ~]# su - postgres
-bash-4.2$ psql -U postgres
postgres=# alter user postgres with password '123456';
ALTER ROLE
postgres=#\q
```
#### 开启远程访问
修改配置文件__/var/lib/pgsql/11/data/postgresql.conf__ 。将listen_addresses的值改为*。
``` shell
#listen_addresses = 'localhost'     # what IP address(es) to listen on;
                    # comma-separated list of addresses;
                    # defaults to 'localhost'; use '*' for all
                    # (change requires restart)
listen_addresses='*'   
```
#### 信任远程连接
``` shell
[root@localhost ~]# vim /var/lib/pgsql/11/data/pg_hba.conf
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident

改为：

# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
host    all             all             192.168.1.1/24          trust  
```
#### 致命错误: 没有用于主机 "xxx", 用户 "postgres", 数据库 "postgres", SSL 关闭 的 pg_hba.conf 记录
注意pa_hba.conf文件里面的主机地址(address)配置是否正确，链接地址与主机地址是否在同一子网中，注意子网掩码的配置。

