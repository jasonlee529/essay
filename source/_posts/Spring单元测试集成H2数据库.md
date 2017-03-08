---
title: Spring单元测试集成H2数据库
date: 2017-03-08 15:11:13
categories:
 - Spring
tags:
 - Java
 - Spring
 - Junit
 - H2
---
>项目源代码在：[Spring-H2测试](https://github.com/jasonlee529/spring-examples/tree/master/test-envi)

## H2简介
 H2数据库是一种由Java编写的，极小，速度极快，可嵌入式的数据库。非常适合用在单元测试等数据不需要保存的场景下面。
 以下时其官网的介绍：
 {% blockquote h2 http://www.h2database.com/html/main.html h2 %}
 *Welcome to H2, the Java SQL database. The main features of H2 are:*
  *Very fast, open source, JDBC API*
  *Embedded and server modes; in-memory databases*
  *Browser based Console application*
  *Small footprint: around 1.5 MB jar file size*
 {% endblockquote %}


## spring-data直连测试

我们使用的maven工程来搭建测试环境，工程目录如下：

{% asset_img test-envi-001.png This is an example image %}

代码里面数据库映射使用了spring-data 和 Hibernate实现，方便实现，避免自己写sql。

数据连接配置信息
``` properties
jdbc.driver=net.sf.log4jdbc.DriverSpy
jdbc.url=jdbc:log4jdbc:h2:mem:test;MODE=MySql;DB_CLOSE_DELAY=-1
jdbc.username=sa
jdbc.password=
```
参数解析：
> 1. driver这里我用的时log4jdbc这个项目，可以方便的打印出带全部参数的sql
> 2. jdbc:h2:mem 意思使用h2的内存数据库版本，还可以有file等各种方式
> 3. test连接的数据库名称
> 4. MODE=MySql 以mysql的模式运行
> 5. DB_CLOSE_DELAY=-1 关闭参数，延时关闭为代码退出时间。

测试用例：
``` java
    @Test
    public void testFindAll() {
        List<User> list = (List<User>) userDao.findAll();
        for (User user : list) {
            System.out.println(user.toString());
        }
    }
```

执行结果：
``` shell
17:45:37.582 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executing SQL script from class path resource [db/sql/schema.sql]
17:45:37.587 [main] INFO  jdbc.sqltiming - DROP TABLE IF EXISTS sys_user
 {executed in 0 msec}
17:45:37.593 [main] INFO  jdbc.sqltiming - CREATE TABLE sys_user ( id BIGINT NOT NULL, name VARCHAR(255), PRIMARY KEY (id) )
 {executed in 5 msec}
17:45:37.594 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executed SQL script from class path resource [db/sql/schema.sql] in 10 ms.
17:45:37.594 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executing SQL script from class path resource [db/sql/import-data.sql]
17:45:37.596 [main] INFO  jdbc.sqltiming - INSERT INTO sys_user (id, name) VALUES (1111, '张三')
 {executed in 1 msec}
17:45:37.597 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executed SQL script from class path resource [db/sql/import-data.sql] in 3 ms.
17:45:37.598 [main] INFO  jdbc.connection - 2. Connection closed
17:45:37.756 [main] INFO  jdbc.connection - 3. Connection opened
17:45:37.758 [main] INFO  o.s.t.c.t.TransactionContext - Began transaction (1) for test context [DefaultTestContext@af9a89f testClass = UserDaoTest, testInstance = cn.lee.jason.repository.UserDaoTest@6482eef, testMethod = testFindAll@UserDaoTest, testException = [null], mergedContextConfiguration = [MergedContextConfiguration@2ab2710 testClass = UserDaoTest, locations = '{classpath:/applicationContext.xml}', classes = '{}', contextInitializerClasses = '[]', activeProfiles = '{test}', propertySourceLocations = '{}', propertySourceProperties = '{}', contextLoader = 'org.springframework.test.context.support.DelegatingSmartContextLoader', parent = [null]]]; transaction manager [org.springframework.orm.jpa.JpaTransactionManager@1fcf9739]; rollback [true]
17:45:37.879 [main] INFO  jdbc.sqltiming - select user0_.id as id1_0_, user0_.name as name2_0_ from sys_user user0_
 {executed in 0 msec}
User{id=1111, name='张三'}
17:45:37.906 [main] INFO  jdbc.connection - 3. Connection closed
```

测试通过，H2搭建测试环境成功。


## 添加schema
2.0版本添加数据库跨库查询；
``` java
@Entity
@Table(name = "sys_user",schema = "sys")
public class User implements Serializable {
    private Long id;
    private String name;
    //省略getter和setter方法
}
```
sql同时也要修改为：
``` sql
CREATE SCHEMA IF NOT EXISTS sys;

DROP TABLE IF EXISTS sys.sys_user;

CREATE TABLE sys.sys_user (
  id   BIGINT NOT NULL,
  name VARCHAR(255),
  PRIMARY KEY (id)
)

INSERT INTO sys.sys_user (id, name) VALUES (1111, '张三');
```
jdbc连接的url不变。
执行测试用例，结果完全一致：
``` shell
17:56:58.160 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executing SQL script from class path resource [db/sql/schema.sql]
17:56:58.165 [main] INFO  jdbc.sqltiming - CREATE SCHEMA IF NOT EXISTS sys
 {executed in 0 msec}
17:56:58.165 [main] INFO  jdbc.sqltiming - DROP TABLE IF EXISTS sys.sys_user
 {executed in 0 msec}
17:56:58.170 [main] INFO  jdbc.sqltiming - CREATE TABLE sys.sys_user ( id BIGINT NOT NULL, name VARCHAR(255), PRIMARY KEY (id) )
 {executed in 4 msec}
17:56:58.170 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executed SQL script from class path resource [db/sql/schema.sql] in 10 ms.
17:56:58.170 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executing SQL script from class path resource [db/sql/import-data.sql]
17:56:58.172 [main] INFO  jdbc.sqltiming - INSERT INTO sys.sys_user (id, name) VALUES (1111, '张三')
 {executed in 1 msec}
17:56:58.172 [main] INFO  o.s.jdbc.datasource.init.ScriptUtils - Executed SQL script from class path resource [db/sql/import-data.sql] in 2 ms.
17:56:58.318 [main] INFO  o.s.t.c.t.TransactionContext - Began transaction (1) for test context [DefaultTestContext@af9a89f testClass = UserDaoTest, testInstance = cn.lee.jason.repository.UserDaoTest@6482eef, testMethod = testFindAll@UserDaoTest, testException = [null], mergedContextConfiguration = [MergedContextConfiguration@2ab2710 testClass = UserDaoTest, locations = '{classpath:/applicationContext.xml}', classes = '{}', contextInitializerClasses = '[]', activeProfiles = '{test}', propertySourceLocations = '{}', propertySourceProperties = '{}', contextLoader = 'org.springframework.test.context.support.DelegatingSmartContextLoader', parent = [null]]]; transaction manager [org.springframework.orm.jpa.JpaTransactionManager@1fcf9739]; rollback [true]
17:56:58.431 [main] INFO  jdbc.sqltiming - select user0_.id as id1_0_, user0_.name as name2_0_ from sys.sys_user user0_
 {executed in 1 msec}
User{id=1111, name='张三'}
17:56:58.451 [main] INFO  jdbc.connection - 3. Connection closed
```
schema添加后，只需要将对应的创建表和语句对应修改即可。
