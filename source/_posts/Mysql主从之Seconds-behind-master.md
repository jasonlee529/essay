---
title: Mysql主从之Seconds_behind_master
tags:
  - mysql
  - 主从
categories: 
	- mysql
date: 2019-04-03 18:24:00
---
Mysql配置主从的监控，主要的监控指标有三个：
* Slave_io_running  	负责与主机的io通信
* Slave_sql_running		负责自己的slave mysql进程
* Seconds_behind_master 

其中Seconds_behind_master表示从库与主库的同步时延，即主库的变更需要多久才能同步到从库上。计量单位是timestamp的差值，ms。

#### 什么时候sbm会是Null?
* SBM在slave关闭的时为null
* Slave_sql_running不是Yes的时候
* Slave_io_running是no的时候。


#### 解决sbm是null的问题？
首先检测问题
``` sql
show slave status ; 

	Slave_io_running  ： yes 
	Slave_sql_running :  no
	Seconds_behind_master : null
```
可以通过设置SQL_SLAVE_SKIP_COUNTER来重新开始sql同步。
``` sql
stop slave;
SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1; 
START SLAVE;  
show slave status ; 

	Slave_io_running  ： yes 
	Slave_sql_running :  yes
	Seconds_behind_master : 7928
```
解决了后发现sbm还是很大。

#### sbm不停增长？
未找到解决方案。