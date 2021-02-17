---
date: "2018-06-21 17:49:46"
title: kafka
---

启动zookeeper

```
bin/zookeeper-server-start.sh config/zookeeper.properties
```

启动kafka

```
bin/kafka-server-start.sh config/server.properties
```

创建一个主题

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

```
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

生产者

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

消费者

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```





kafka connect



```
echo -e "zhisheng\ntian" > test.txt
```

ls

```
zhuzhenfengdeMacBook-Pro➜  kafka_2.12-1.1.0  ᐅ  echo -e "zhisheng\ntian" > test.txt

zhuzhenfengdeMacBook-Pro➜  kafka_2.12-1.1.0  ᐅ
zhuzhenfengdeMacBook-Pro➜  kafka_2.12-1.1.0  ᐅ  ls
LICENSE   NOTICE    bin       config    libs      logs      site-docs test.txt
zhuzhenfengdeMacBook-Pro➜  kafka_2.12-1.1.0  ᐅ
```



启动连接器

```
bin/connect-standalone.sh  config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

然后发现多了一个文件

```
/Users/zhuzhenfeng/Documents/software/kafka_2.12-1.1.0
[kafka_2.12-1.1.0] ls                                                                                                 18:25:38
LICENSE       NOTICE        bin           config        libs          logs          site-docs     test.sink.txt test.txt
[kafka_2.12-1.1.0]
```



然后消费

```
>> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning

{"schema":{"type":"string","optional":false},"payload":"zhisheng"}
{"schema":{"type":"string","optional":false},"payload":"tian"}
{"schema":{"type":"string","optional":false},"payload":"sd"}
{"schema":{"type":"string","optional":false},"payload":"sdafasdfasdfdsa"}
```

