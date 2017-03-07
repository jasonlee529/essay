---
title: '@Resource和@Autowired的异同'
date: 2017-03-07 18:06:29
categories:
  --spring
tags:
 --spring
 --java
---

## 相同点：
1. 两者都能做到注入一个Bean.
2. 两者都可应用在Field和Method上面。
3. 两者均为Runtime级别的Retention。

## 不同点：
1. 使用的场景有差异 @Resource可应用在类（TYPE)上面，@Autowired可以应用在构造方法(CONSTRUCTOR)和注解类型(ANNOTATION_TYPE)上面。
2. @Resource是在```javax.annotation```包下面，是JSR-250规范定义的注解，而@Autowired是由```org.springframework.beans.factory.annotation```包下面的，是由spring实现的。
3. @Autowired默认查找策略时byType,@Resource默认查找策略是byName。如果@Autowired想要按照byName加载bean，需要配合@Qualifier注解使用。
4. @Autowired 只有一个属性 required 表示这个依赖是否时必须的，即当依赖对象为null时也可以。如果注入的方法有多个参数，则每个参数的策略一致。
5. @Resource 用于类（TYPE）时，表示该类需要被容器初始化为一个bean.


### @Resource 查找策略
1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；
