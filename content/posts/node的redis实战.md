---
date: "2018-04-13 10:00:12"
title: node的redis实战
---

#  Node.js Redis客户端模块

为了追新，这里我使用的yarn，毕竟我是HDD（面向热点编程）编程实践者。

模块安装

```
yarn add redis
```

模块使用实例

```
const redis = require('redis')
const client = redis.createClient('6379', '127.0.0.1')

client.on("error", function (err) {
    console.log("Error " + err);
});

client.set("string key", "string val", redis.print);
client.hset("hash key", "hashtest 1", "some value", redis.print);
client.hset(["hash key", "hashtest 2", "some other value"], redis.print);
client.hkeys("hash key", function (err, replies) {
    console.log(replies.length + " replies:");
    replies.forEach(function (reply, i) {
        console.log("    " + i + ": " + reply);
    });
    client.quit();
});
```

输出的结果如下所示：

```
➜  node-example git:(master) ✗ node redis-demo.js
Reply: OK
Reply: 0
Reply: 0
2 replies:
    0: hashtest 1
    1: hashtest 2
```

Promises

如果是使用node 8或者之上的话，使用node的util.promisify来将请求变成promise的。

```
const {promisify}=require('util')
const redis = require('redis')
const client = redis.createClient('6379', '127.0.0.1')

const getAsync=promisify(client.get).bind(client)
function getFoo(){
    return getAsync('foo').then(res => {
        console.log(res)
    })
}

getFoo()
```

发送命令

每个redis命令都会通过client对象的一个函数暴露，所有这些函数都会有一个args数组选项和一个callback回调函数。

字符串操作

set key value

get key

哈希操作

hmset key field1 value1

hget key field1 value1

列表操作

lpush key value1 value2

lrange key 0 n

集合操作

sadd key member1 member2

smembers key

有序集合操作

zadd key index value

zrange key 0 n 





 

