---
date: "2018-07-30 14:51:26"
title: NATS streaming
---

市面上常见到的和Nats功能类似的消息通信系统有：

ActiveMQ（Java编写）、KafKa（Scala编写）、RabbitMq（Ruby编写）、Nats（之前是Ruby编写现已修改为Go）、Redis（C语言编写）、Kestrel（Scala编写不常用）、NSQ（Go语言编写），这些消息通信系统在Broker吞吐量方面的比较：

![](https://ws1.sinaimg.cn/large/b831e4c7gy1fts2j6kv9mj20mv0c8aa8.jpg)

可以看到NATS的吞吐量特别高， NATS原来是使用Ruby编写，可以实现每秒150k消息，后来使用Go语言重写，能够达到每秒8-11百万个消息，整个程序很小只有3M Docker image，它不支持持久化消息，如果你离线，你就不能获得消息。关于NATS的详细介绍，请参考上篇文章：[NATS简介](https://zhuanlan.zhihu.com/p/40871363)

## NATS Streaming

NATS Streaming是由NATS驱动的数据流系统，也是由go语言写成的，在保证吞吐量和时延的基础上，解决了Nats消息投递一致性的问题。nats streaming可以和核心nats平台无缝嵌入，扩展和互动。

![](https://ws1.sinaimg.cn/large/b831e4c7gy1fts2wctrfzj20tk0r8afy.jpg)

### 功能

除了nats平台的一些功能，nats streaming还支持以下的：

- 增强的消息协议
- 消息/事件持久化
- 至少一次投递
- 发布者速率限制
- 每个订阅者的速率匹配/限制
- 可重复消费
- 持久订阅

### 使用

首先安装nats-streaming-server服务，有多种方式，这里介绍两种：

- homebrew

  直接在命令行启动

  ```
  brew install nats-streaming-server
  ```

- go get

  这种方式可以让我们直接运行源码启动

  ```
  go get github.com/nats-io/nats-streaming-server
  ```

启动nats-streaming-server

有三种启动方式

- 直接启动

  ```
  nats-streaming-server
  ```

- 开启nats监控的启动

  ```
  nats-streaming-server -m 8222
  ```

- 源码方式启动

  ```
  cd $GOPATH/src/github.com/nats-io/nats-streaming-server
  
  go run nats-streaming-server.go
  ```

### 客户端

直接下载go的客户端

```
go get github.com/nats-io/go-nats-streaming
```

运行发布者

```
cd $GOPATH/src/github.com/nats-io/go-nats-streaming/examples/stan-pub

go run main.go foo "msg one"

go run main.go foo "msg two"

go run main.go foo "msg three"
```

如下图所示：

![](https://ws1.sinaimg.cn/large/b831e4c7gy1ftrx4gytrhj20q50g0juj.jpg)

运行订阅者

```
cd $GOPATH/src/github.com/nats-io/go-nats-streaming/examples/stan-sub

go run main.go --all -c test-cluster -id myID foo
```

![](https://ws1.sinaimg.cn/large/b831e4c7gy1ftrx8h85rpj20qq0fu41n.jpg)

### 实例

首先在本地启动nats-streaming-server，然后下面的代码展示了发布订阅的过程：

```
package main

import (
	"github.com/nats-io/go-nats-streaming"
	"github.com/nats-io/go-nats-streaming/pb"
	"log"
	"strconv"
	"time"
)

func main() {
	var clusterId string = "test-cluster"
	var clientId string = "test-client"

	sc, err := stan.Connect(clusterId, clientId, stan.NatsURL("nats://localhost:4222"))
	if err != nil {
		log.Fatal(err)
		return
	}

	// 开启一个协程，不停的生产数据
	go func() {
		m := 0
		for {
			m++
			sc.Publish("foo1", []byte("hello message "+strconv.Itoa(m)))
			time.Sleep(time.Second)
		}

	}()

	// 消费数据
	i := 0
	mcb := func(msg *stan.Msg) {
		i++
		log.Println(i, "---->", msg.Subject, msg)
	}
	startOpt := stan.StartAt(pb.StartPosition_LastReceived)
	//_, err = sc.QueueSubscribe("foo1", "", mcb, startOpt)   // 也可以用queue subscribe
	_, err = sc.Subscribe("foo1", mcb, startOpt)
	if err != nil {
		sc.Close()
		log.Fatal(err)
	}

	// 创建一个channel，阻塞着
	signalChan := make(chan int)
	<-signalChan
}
```

运行结果如下：

```
2018/07/30 18:04:01 2 ----> foo1 sequence:546 subject:"foo1" data:"hello message 1" timestamp:1532945041825538757 
2018/07/30 18:04:02 3 ----> foo1 sequence:547 subject:"foo1" data:"hello message 2" timestamp:1532945042828881383 
2018/07/30 18:04:03 4 ----> foo1 sequence:548 subject:"foo1" data:"hello message 3" timestamp:1532945043833360222 
2018/07/30 18:04:04 5 ----> foo1 sequence:549 subject:"foo1" data:"hello message 4" timestamp:1532945044833810697 
2018/07/30 18:04:05 6 ----> foo1 sequence:550 subject:"foo1" data:"hello message 5" timestamp:1532945045838056450 
2018/07/30 18:04:06 7 ----> foo1 sequence:551 subject:"foo1" data:"hello message 6" timestamp:1532945046838585417 
2018/07/30 18:04:07 8 ----> foo1 sequence:552 subject:"foo1" data:"hello message 7" timestamp:1532945047840775810 
```

源码在：[https://github.com/zhenfeng-zhu/nats-demo](https://github.com/zhenfeng-zhu/nats-demo)

## 总结

NATS Streaming的高级功能类似于 Apache Kafka 的功能，但当你考虑简单性而非复杂性时前者更优。由于 NATS Streaming 相对来说是一项新技术，与 Apache Kafka 相比，它在某些领域需要改进，尤其是为负载均衡场景提供更好的解决方案。



