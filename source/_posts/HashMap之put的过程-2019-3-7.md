---
title: HashMap之put的过程
date: 2019-03-07 14:27:41
categories:
- Java
- Map
tags:
- Java
- Map
- HashMap
---

HashMap是Java集合类中，使用比较多的一个集合类。类似request的参数集合、jdbcTemplate的数据库返回啊，这些不定长度的key-value型数据集合，大量的使用到了HashMap。那到底HashMap的api有啥神秘呢？put的过程到底有什么问题呢？

# PUT的实现

