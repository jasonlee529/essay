---
title: Connection is read-only. Queries leading to data modification are not allowed
tags:
  - mysql
  - mycat
  - jdbc
categories:
  - mysql
date: 2019-04-04 17:24:00
---
### 环境
* Java 8
* mysql 5.7 1主1从
* mycat 1.6
* druid 1.4



### 错误情况
 启动应用，使用jmeter做压力测试，先将所有url都执行一遍，所有的select和insert类型的请求通过，包含update或者delete的请求失败，查看日志，报错信息：Connection is read-only. Queries leading to data modification are not allowed
 
| select | insert | update | delete |
| ------ | ------ | ------ | ------ |
| o      | o      | x      | x      |


### 更改连接池配置
添加spring-boot的数据库链接池为非只读
``` properties
spring.datasource.druid.default-read-only=false
```

| select | insert | update | delete |
| ------ | ------ | ------ | ------ |
| o      | o      | x      | x      |

### 添加事务，网上给出最多的方法
给所有涉及update或者delete的service类添加事务声明
``` java
import org.springframework.transaction.annotation.Transactional;
@Transactional(readOnly = false)
```
| select | insert | update | delete |
| ------ | ------ | ------ | ------ |
| o      | o      | x      | x      |

### 切换为直连数据库，意外收获。
| select | insert | update | delete |
| ------ | ------ | ------ | ------ |
| o      | o      | o      | o     |

没有写入的问题，可以证明是mycat的连接的问题。

### 排查mycat的配置信息
```  schema.xml
 <schema name="testdb" checkSQLschema="false" sqlMaxLimit="100">
 	<table name="t1" datanode = "dn1" rule=""/>
    ....
 </schema>
```
``` server.xml
  <user name="root" defaultAccount="true">
    <property name="password">123456</property>
    <property name="schemas">testdb</property>
    <property name="readOnly">false</property>
  </user>
```
这部分生命信息看上去没有任何问题。

### spring 事务的源代码
org.springframework.orm.jpa.vendor.HibernateJpaDialect.java beginTransaction方法
``` java
 @Override
	public Object beginTransaction(EntityManager entityManager, TransactionDefinition definition)throws PersistenceException, SQLException, TransactionException {
		Session session = getSession(entityManager);
		if (definition.getTimeout() != TransactionDefinition.TIMEOUT_DEFAULT) {
			session.getTransaction().setTimeout(definition.getTimeout());
		}
		boolean isolationLevelNeeded = (definition.getIsolationLevel() !=TransactionDefinition.ISOLATION_DEFAULT);
		Integer previousIsolationLevel = null;
		Connection preparedCon = null;
		if (isolationLevelNeeded || definition.isReadOnly()) {
			if (this.prepareConnection) {
				preparedCon = HibernateConnectionHandle.doGetConnection(session);
                //下面一行的调用起到了关键作用
				previousIsolationLevel = DataSourceUtils.prepareConnectionForTransaction(preparedCon, definition);
			}else if (isolationLevelNeeded) {
				throw new InvalidIsolationLevelException(getClass().getSimpleName() +
						" does not support custom isolation levels since the 'prepareConnection' flag is off.");
			}
		}		
		entityManager.getTransaction().begin();
		FlushMode previousFlushMode = prepareFlushMode(session, definition.isReadOnly());
		return new SessionTransactionData(session, previousFlushMode, preparedCon, previousIsolationLevel);
	}
```

org.springframework.jdbc.datasource.DataSourceUtils.java Line:91 

```java

	@Nullable
	public static Integer prepareConnectionForTransaction(Connection con, @Nullable TransactionDefinition definition) throws SQLException {
        Assert.notNull(con, "No Connection specified");
        if (definition != null && definition.isReadOnly()) {
            try {
                if (logger.isDebugEnabled()) {
                    logger.debug("Setting JDBC Connection [" + con + "] read-only");
                }
				//直接写死为readOnly=true。这个原因需要调查。
                con.setReadOnly(true);
            } catch (RuntimeException | SQLException var4) {
                for(Object exToCheck = var4; exToCheck != null; exToCheck = ((Throwable)exToCheck).getCause()) {
                    if (exToCheck.getClass().getSimpleName().contains("Timeout")) {
                        throw var4;
                    }
                }
                logger.debug("Could not set JDBC Connection read-only", var4);
            }
        }
        Integer previousIsolationLevel = null;
        if (definition != null && definition.getIsolationLevel() != -1) {
            if (logger.isDebugEnabled()) {
                logger.debug("Changing isolation level of JDBC Connection [" + con + "] to" + definition.getIsolationLevel());
            }
            int currentIsolation = con.getTransactionIsolation();
            if (currentIsolation != definition.getIsolationLevel()) {
                previousIsolationLevel = currentIsolation;
                con.setTransactionIsolation(definition.getIsolationLevel());
            }
        }
        return previousIsolationLevel;
    }
```
这个方法的解释是针对select方法,hibernate会强制设为只读请求，但为啥update方法会来这里，没有找到原因。

### 添加参数useLocalSessionState=true
``` properties
spring.datasource.url=jdbc:log4jdbc:mysql://ip:8066/testdb?useUnicode=true&characterEncoding=utf-8&useLocalSessionState=true
```

解决了这个问题，关于useLocalSessionState的解释是：__
默认情况下，我们的连接串信息没有包含useLocalSessionState参数的设置，这个值默认为false。
这个值的作用是驱动程序是否使用autocommit，read_only和transaction isolation的内部值(jdbc端的本地值)。
如果设置为false，则需要这个判断这三个参数的场景，都需要发语句到远端请求，比如更新语句前，
需要发语句select @@session.tx_read_only确认会话是否只读。
如果设置为true，则只需要取本地值即可。__


似乎是本地和远程服务器的配置有关，但远程服务器是mycat，莫非mycat的链接不是只读的，但没有找到mycat是只读的配置选项。

参数可以解决readOnly的问题，但只读链接的产生过程和原因不明。