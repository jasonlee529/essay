---
title: Kafka-Java ProducerAPI的例子
date: 2019-08-30 15:11:30
categories:
- Java
- Kafka
tags:
- Kafka
- Java
- Producer
---
## Requirements
启动一个Kafka集群实例，启动方式参考[官网教程](http://kafka.apache.org/quickstart)。地址为：192.168.1.173:9092

## Project
```依赖
kafka客户端依赖，利用Java-ProducerAPI生成数据发送给kafka。
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.3.0</version>
</dependency>
```

## Main
``` java
Properties props = new Properties();
props.put("bootstrap.servers", "192.168.1.173:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
KafkaProducer producer = new KafkaProducer<String, String>(props);
ProducerRecord record = new ProducerRecord<String, String>(topic, null, null, JacksonUtils.nonDefaultMapper().toJson(student));
```
上面这段类似与shell启动一个Producer
```shell 
bin/kafka-console-producer.sh --broker-list 192.168.1.173:9092 --topic student
```
不过shell启动的方式只能通过控制台输入信息，Java部分通过代码生成数据发送。

## 运行：
发送端：
```shell 
发送数据: {"id":77064,"name":"bhdX","password":"xfZUDcVG","age":16}
发送数据: {"id":77065,"name":"HgkO","password":"JkJXpgOp","age":57}
发送数据: {"id":77066,"name":"fIMG","password":"VSivsZtI","age":26}
发送数据: {"id":77067,"name":"nzFk","password":"jupUFQDN","age":42}
发送数据: {"id":77068,"name":"czDo","password":"rhcvkLHO","age":52}
发送数据: {"id":77069,"name":"UzEo","password":"VhBYLAph","age":22}
发送数据: {"id":77070,"name":"Wult","password":"xhXxxwxg","age":22}
发送数据: {"id":77071,"name":"yvAv","password":"eoYLmEqD","age":15}
发送数据: {"id":77072,"name":"nqoF","password":"uvIDROzp","age":15}
发送数据: {"id":77073,"name":"LrXX","password":"JdvTFdEj","age":93}
发送数据: {"id":77074,"name":"HIZA","password":"xKKZBhbY","age":44}
发送数据: {"id":77075,"name":"UuNe","password":"XELywaGT","age":82}
```
另外启动一个kafka-consumer，查看输出：
``` shell  
$ bin/kafka-console-consumer.sh --bootstrap-server 192.168.1.173:9092 \
    --topic student \ 
    --from-beginning 
{"id":77064,"name":"bhdX","password":"xfZUDcVG","age":16}
{"id":77065,"name":"HgkO","password":"JkJXpgOp","age":57}
{"id":77066,"name":"fIMG","password":"VSivsZtI","age":26}
{"id":77067,"name":"nzFk","password":"jupUFQDN","age":42}
{"id":77068,"name":"czDo","password":"rhcvkLHO","age":52}
{"id":77069,"name":"UzEo","password":"VhBYLAph","age":22}
{"id":77070,"name":"Wult","password":"xhXxxwxg","age":22}
{"id":77071,"name":"yvAv","password":"eoYLmEqD","age":15}
{"id":77072,"name":"nqoF","password":"uvIDROzp","age":15}
{"id":77073,"name":"LrXX","password":"JdvTFdEj","age":93}
{"id":77074,"name":"HIZA","password":"xKKZBhbY","age":44}
{"id":77075,"name":"UuNe","password":"XELywaGT","age":82}

```

代码地址：[https://gitee.com/jasonlee0529/bigdata-all/tree/master/kafka/kafka-provider](https://gitee.com/jasonlee0529/bigdata-all/tree/master/kafka/kafka-provider)