---
title: Spring实践之AOP
date: 2017-05-05 22:55:21
categories:
 - Spring
tags:
 - Spring
 - Java
---

如何用一句话通俗的解释AOP？

答：在不改变原有代码逻辑的基础上，实现特定的业务逻辑。

# 概念
## 什么是切面编程？
AOP,中文译作“面向切面编程”，即通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。
## 术语
* 通知(Advice)：Spring的Aop支持4种类型的Advice:

> 1. before advice 在方法执行前执行。
> 2. after returning  advice 在方法执行后返回一个结果后执行。
> 3. after throwing advice 在方法执行过程中抛出异常的时候执行。
> 4. Around advice 在方法执行前后和抛出异常时执行，相当于综合了以上三种通知。

* 连接点(JoinPoint): 连接点是在应用程序中能够插入切面的一个点。
* 切点(PontCut): 业务逻辑的切入点，某些连接点上面织入逻辑。
* 切面(Aspect):
* 引入(Introduction):
* 织入(Weaving):

# Spring对AOP的实现
## Spring是如何实现AOP的？
Spring是运用动态代理的方法实现AOP的，基于动态代理的版本__只能支持方法连接点__。这个与其他AOP框架相比略有不足，无法满足字段和构造器的接入点。如果需要方法之外的连接点，那么就引入AspectJ来搞定吧。
## Spring的实现了具体形式？

### 基于注解的

### 基于XML配置的

# 一个例子
##

# 总结
