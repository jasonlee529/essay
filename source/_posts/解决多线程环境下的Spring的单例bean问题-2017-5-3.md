---
title: 解决多线程环境下的Spring的单例bean问题
date: 2017-05-03 14:37:54
categories:
  - Java
tags:
  - Java
  - Spring
---

# 还原场景
原来有一个多线程代码，目的是同步mysql的数据到elasticsearch，原方案是启动一个线程把mysql数据读取出来，组织成目标数据结构，效率不够理想。新方案想改成生产者消费者模式，一个线程负责读取mysql数据，一个线程负责向elasticsearch发送数据。

改的过程中发现一个问题，具体报错信息是：
```Java
org.springframework.beans.factory.BeanCreationNotAllowedException: Error creating bean with name 'caseProvider': Singleton bean creation not allowed while the singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:216)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
	at org.springframework.context.support.AbstractApplicationContext.getBean(AbstractApplicationContext.java:1054)
	at cn.infisa.base.holder.SpringContextHolder.getBean(SpringContextHolder.java:33)
	at cn.infisa.extracting.service.record.es.runner.Producer.getProvider(Producer.java:82)
	at cn.infisa.extracting.service.record.es.runner.Producer.run(Producer.java:52)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

```
<!-- more -->

# 定位问题

经推测这应该由于单元测试中执行异步方法导致的问题，原因在于JUnit是System.exit()方法推出，Spring的容器环境随之消失，导致无法获取bean。

# 尝试方法
## 使用CountLatch
很多人推荐使用CountLatch来阻止线程退出，这个办法不适合我这个步骤，线程启动不在junit测试类中生命，而在service的方法中，如果注入一个CountLatch实例，接口的依赖太多了。

未完待续
