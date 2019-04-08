---
title: 极简Mysql主从配置
tags:
  - 主从
  - Mysql
categories:
  - Mysql
date: 2019-04-08 22:54:00
---
Mysql搭建主从其实配置特别简单，核心配置项目就那么几个就够了。
#### 数据库配置
Master的配置项目
``` cnf
[mysqld]
server-id=1
log-bin=log
binlog-ignore-db=mysql,performance_schema,information_schema
```
其中只有server-id和log-bin这两项是最重要的，当然一般情况下会显示的指定binlog的格式，binlog日志文件的大小等配置。

Slave的配置：
``` cnf
[mysqld]
server-id=2
replicate-ignore-db=mysql,performance_schema,information_schema
```
其中binlog-ignore-db和replicate-ignore-db分别在master和slave上面配置的不进行同步的数据库，建议两边完全对等，不要出现差异。

#### Slave启动同步
##### prepare 同步

开启同步前要么保证主从状态一致，要么保证master的binlog足够slave同步到master一致的状态，不然Slave_sql_ruuning这个标识位会在某些时段变为NO，即便通过__set global sql_slave_skip_counter=1__设置将状态同步到一致，但数据极大可能仍然不是一致的，因为这个操作是跳过了master的部分操作。所以启动同步前一定要保证binlog足够，不然就手工同步一遍master到slave,再开启操作。

##### 获取master状态

``` shell
mysql> show master status\G;
        File: log.000004
     Position: 154
   Binlog_Do_DB: 
 Binlog_Ignore_DB: mysql,performance_schema,information_schema
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```
从中提前两个结果 File和Position作为slave启动的脚本。
##### 启动slave
将master的file和position填入slave的启动语句中。
``` shell
stop slave;
change master to 
master_host="******",
MASTER_USER = 'root',
 MASTER_PASSWORD = '*****',
 MASTER_LOG_FILE = 'log.000004'  ,
  MASTER_LOG_POS = 154  
;
start slave;

show slave status\G;
               Slave_IO_State: Waiting for master to send event
                  Master_Host: ******
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: log.000004
          Read_Master_Log_Pos: 154
               Relay_Log_File: 8d5dd16836bd-relay-bin.000005
                Relay_Log_Pos: 355
        Relay_Master_Log_File: log.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Exec_Master_Log_Pos: 154
               Seconds_Behind_Master: 0

1 row in set (0.00 sec)

```
最后一个show slave status是为了确认是不是启动了额。监测指标核心是三个：Relay_Master_Log_File、Exec_Master_Log_Pos、Seconds_Behind_Master。前两个是核心指标，即slave读取到了master的binlog文件和位置，如果这两个参数的结果和master的binlog文件名称和postion一致，即可认为master和slave状态一致。SBM一定意义上表示了slave和master的时延是多少，这个时延是以slave的时间减去master的binlog时间戳，有作弊的可能，因此主要查看前两个参数。