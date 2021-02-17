---
date: "2018-05-05 15:45:47"
title: node的cluster
---

我们知道js是运行单线程的，也就是说一个node进程只能运行在一个cpu上。那么如果用node来做web server的话，就无法享受到多核运算的好处。

一个问题就是：

```
如何榨干服务器资源，利用多核CPU的并发优势。
```

node官方提供的解决方案是cluster。

## 1 cluster是什么

简单来说：

- 在服务器上同时启动多个进程。
- 每个进程都跑的是同一份源码。
- 这些进程可以同时监听一个端口。

其中：

- 负责启动其他进程的叫做master进程，不做具体工作，只负责启动其他进程。
- 其他被启动的叫worker进程。他们接收请求，对外提供服务。
- worker进程的数量一般根据服务器的cpu核数来决定，这样就可以完美利用多核资源。

以下是官方文档的一个例子：

```
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
    console.log(`主进程 ${process.pid} 正在运行`);

    // 衍生工作进程。
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork();
    }

    cluster.on('exit', (worker, code, signal) => {
        console.log(`工作进程 ${worker.process.pid} 已退出`);
    });
} else {
    // 工作进程可以共享任何 TCP 连接。
    // 在本例子中，共享的是一个 HTTP 服务器。
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end('你好世界\n');
    }).listen(8000);

    console.log(`工作进程 ${process.pid} 已启动`);
}
```

```
% node cluster.js
主进程 16391 正在运行
工作进程 16394 已启动
工作进程 16393 已启动
工作进程 16395 已启动
工作进程 16392 已启动
```

## 2 多进程模型

官网的示例很简单，但是我们还要考虑很多东西。

- worker进程异常退出以后如何处理？
- 多个worker进程之间如何共享资源？
- 多个worker进程之间如何调度？
- 。。。

### 进程守护

健壮性是我们做大型应用要考虑的一个问题。一般来说，node的进程退出可以分为两类：

- 未捕获异常
- 内存溢出（OOM）或者系统异常

代码跑出了异常却没有没捕捉时，进程将会退出，此时node提供了process.on('uncaughtException', handler)来捕获。但是当一个worker进程遇到未 捕获的异常时，他已经处于一个不确定的状态，此时我们应该让这个进程优雅的退出。

优雅退出的流程是：

1. 关闭异常worker进程和所有的tcp server(将已有的连接快速断开，且不再接收新的连接)，断开和master的ipc通道，不再接收新的用户请求。
2. master立即fork一个新的worker进程，保证在线的“工人”总数不变。
3. 异常worker等待一段时间，处理完已经接受的请求后退出。

而当一个进程出现异常导致 crash 或者 OOM 被系统杀死时，不像未捕获异常发生时我们还有机会让进程继续执行，只能够让当前进程直接退出，Master 立刻 fork 一个新的 Worker。

### Agent机制

有些工作并不需要每个worker都去做，如果都做，一个是浪费资源，另一个更重要的是可能会导致多进程之间资源访问冲突。

比如：生产环境中一般会按照日期进行归档，在单进程模型的时候比较简单：

1. 每天凌晨，批量将日志文件重命名
2. 创建新的日志文件继续写入。

如果这个任务由4个进程同时做，就乱套了。所以对于这一类的后台运行逻辑，应该放到一个单独的进程去执行，这个进程就是agent。agent好比是给其他worker请的一个秘书，它不对外提供服务，只给worker打工，处理一些公共事务。

另外，关于 Agent Worker 还有几点需要注意的是：

1. 由于 Worker 依赖于 Agent，所以必须等 Agent 初始化完成后才能 fork Worker
2. Agent 虽然是 Worker 的『小秘』，但是业务相关的工作不应该放到 Agent 上去做。
3. 由于 Agent 的特殊定位，**我们应该保证它相对稳定**。当它发生未捕获异常，不应该像 Worker 一样让他退出重启，而是记录异常日志、报警等待人工处理。

## 3 进程间通讯（IPC）

虽然每个 Worker 进程是相对独立的，但是它们之间始终还是需要通讯的，叫进程间通讯（IPC）。

```
if (cluster.isMaster) {
  const worker = cluster.fork();
  worker.send('hi there');

} else if (cluster.isWorker) {
  process.on('message', (msg) => {
    process.send(msg);
  });
}
```

这个例子里面，工作进程将主进程发送的消息echo回去。

## 4 长连接

一些中间件客户端需要和服务器建立长连接，理论上一台服务器最好只建立一个长连接，但多进程模型会导致 n 倍（n = Worker 进程数）连接被创建。

为了尽可能的复用长连接（因为它们对于服务端来说是非常宝贵的资源），我们会把它放到 Agent 进程里维护，然后通过 messenger 将数据传递给各个 Worker。这种做法是可行的，但是往往需要写大量代码去封装接口和实现数据的传递，非常麻烦。

另外，通过 messenger 传递数据效率是比较低的，因为它会通过 Master 来做中转；万一 IPC 通道出现问题还可能将 Master 进程搞挂。

那么有没有更好的方法呢？答案是肯定的，我们提供一种新的模式来降低这类客户端封装的复杂度。通过建立 Agent 和 Worker 的 socket 直连跳过 Master 的中转。Agent 作为对外的门面维持多个 Worker 进程的共享连接。

### 核心思想

- 受到 [Leader/Follower](http://www.cs.wustl.edu/~schmidt/PDF/lf.pdf) 模式的启发
- 客户端会被区分为两种角色：
  - Leader: 负责和远程服务端维持连接，对于同一类的客户端只有一个 Leader
  - Follower: 会将具体的操作委托给 Leader，常见的是订阅模型（让 Leader 和远程服务端交互，并等待其返回）。
- 如何确定谁是 Leader，谁是 Follower 呢？有两种模式：
  - 自由竞争模式：客户端启动的时候通过本地端口的争夺来确定 Leader。例如：大家都尝试监听 7777 端口，最后只会有一个实例抢占到，那它就变成 Leader，其余的都是 Follower。
  - 强制指定模式：框架指定某一个 Leader，其余的就是 Follower
- 启动的时候 Master 会随机选择一个可用的端口作为 Cluster Client 监听的通讯端口，并将它通过参数传递给 Agent 和 App Worker
- Leader 和 Follower 之间通过 socket 直连（通过通讯端口），不再需要 Master 中转

---

## 5 参考资料

[多进程模型和进程间通讯](https://eggjs.org/zh-cn/core/cluster-and-ipc.html)

[Node.js v8.11.1 文档 cluster](http://nodejs.cn/api/cluster.html)