---
title: Sharding-jdbc初入门
categories:
  - Java
tags:
  - sharding-jdbc
  - ''
  - shardingsphere
  - Java
date: 2019-04-30 15:56:00
---

[Sharding-jdbc](https://shardingsphere.apache.org/document/current/cn/overview/)现在是Apache基金会顶级项目ShardingSphere下的一个产品，是以jdbc的形式支持读写分离、分库分表等类似数据库中间件功能，优点是入侵性小、配置方便，而且背后有个好爹。

Sharding-jdbc支持读写分离和分库分表两个核心功能，并且可以在读写分离基础上叠加分库分表。

### Maven坐标
Maven中央仓库中，有两个artifactId为sharding-jdbc的包，一个groupId是io.shardingjdbc，一个是org.apache.shardingsphere，注意不要选错了，apache的发布版本都是4.0.0之后的，之前的是老版本应该也不会继续更新了。
``` xml
<!-- https://mvnrepository.com/artifact/org.apache.shardingsphere/sharding-jdbc-core -->
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-core</artifactId>
    <version>4.0.0-RC1</version>
</dependency>
``` 