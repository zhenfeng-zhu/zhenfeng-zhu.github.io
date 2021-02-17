---
date: "2018-06-02 16:00:52"
title: 译：vertx-kotlin-coroutine
---

> 尝试翻译vertx的文档。尊重原文，部分使用自己的理解。

Vert.x的kotlin协程提供了async/await或者和go类似的channel。这使得你能够以熟悉的顺序风格写垂直代码。

vertx-lang-kotlin-coroutines集成了kotlin协程，用于执行异步操作和处理事件。这样就能够以同步代码的模型编写代码，而且不会阻塞内核线程。

# 简介

vert.x与许多旧的应用平台相比的一个主要优势是它几乎完全是非阻塞的（内核线程）。这允许基于vert.x的程序使用极少数的内核线程处理大量的并发（例如：许多连接和消息），可以获得更好的伸缩性。

vert.x的非阻塞特性形成了非阻塞API。非阻塞API可以采用多种形式来实现，包括回调函数，promise，fibers或者响应式扩展。vert.x的核心API使用回调函数的风格，但是它也支持其他模型，如RxJava 1和2。

在某些情况下，使用异步的API编程可能比使用经典的顺序代码风格更具有挑战性，特别是需要按照顺序完成若干操作。另外，使用异步API时，错误的传播也更为复杂。

vertx-lang-kotlin-coroutines使用协程。协程是非常轻量级的线程，而且不与底层的内核线程对应。所以当协程需要“阻塞”时，它会暂停并释放当前的内核线程，使得另一个协程可以处理事件。

vertx-lang-kotlin-coroutines使用kotlinx.coroutines来实现协程。

>vertx-lang-kotlin-coroutines目前仅适用于kotlin，而且是kotlin1.1的一个实验特性。

# 从一个vertx.x的contex中启动协程

导入io.vertx.kotlin.coroutines.VertxCoroutine，launch（协程生成器）方法中允许运行一段代码作为可以暂停的协程：

```kotlin
val vertx = Vertx.vertx()
vertx.deployVerticle(ExampleVerticle())

launch(vertx.dispatcher()) {
  val timerId = awaitEvent<Long> { handler ->
    vertx.setTimer(1000, handler)
  }
  println("Event fired from timer with id ${timerId}")
}
```

vertx.dispatcher()返回一个使用vert.x的事件循环执行协程的disptacher。

awaitEvent函数暂停协程的执行直到定时器触发为止，并使用赋给handler的值恢复协程。

有关handlers，events和事件流的更多细节，将在下一节中给出。

# 继承CoroutineVerticle

你可以将代码部署为io.vertx.kotlin.coroutines.CoroutineVerticle的实例，这是kotlin协程的专用类型。你应该重载verticle的start()方法，stop()方法的重载是可选的：

```kotlin
class MyVerticle : CoroutineVerticle() {
  suspend override fun start() {
    // ...
  }
}
```

# 获得一次性的异步结果

vert.x的许多异步操作都采用Handler<AsyncResult<T>>作为最后一个参数。一个例子就是使用vert.x的mongo client执行对象检索，或者是发送一个事件总线消息之后等待回复。

这是通过awaitResult方法来实现，它返回一个值或者抛出一个异常。

协程会一直处于暂停的状态知道事件被处理，并且这时没有内核线程被阻塞。

The method is executed by specifying the asynchronous operation that needs to be executed in the form of a block that is passed to the handler at run-time.

这个方法是通过指定一个异步操作来执行，这个异步操作需要以块的形式执行，而这个异步操作块在运行时会被传给handler。

这里是一个例子：

```kotlin
suspend fun awaitResultExample() {
  val consumer = vertx.eventBus().localConsumer<String>("a.b.c")
  consumer.handler { message ->
    println("Consumer received: ${message.body()}")
    message.reply("pong")
  }

  // Send a message and wait for a reply
  val reply = awaitResult<Message<String>> { h ->
    vertx.eventBus().send("a.b.c", "ping", h)
  }
  println("Reply received: ${reply.body()}")
}
```

当此块产生失败时，调用者可以使用try/catch结构处理异常。

```kotlin
suspend fun awaitResultFailureExample() {
  val consumer = vertx.eventBus().localConsumer<String>("a.b.c")
  consumer.handler { message ->
    // The consumer will get a failure
    message.fail(0, "it failed!!!")
  }

  // Send a message and wait for a reply
  try {
    val reply: Message<String> = awaitResult<Message<String>> { h ->
         vertx.eventBus().send("a.b.c", "ping", h)
       }
  } catch(e: ReplyException) {
    // Handle specific reply exception here
    println("Reply failure: ${e.message}")
  }
}
```

# 获取一次性事件

使用awaitEvent函数处理一次性事件（而不是下一次出现的事件）：

```kotlin
suspend fun awaitEventExample() {
  val id = awaitEvent<Long> { h -> vertx.setTimer(2000L, h) }
  println("This should be fired in 2s by some time with id=$id")
}
```

# 获取一次性worker的结果

使用awaitBlocking函数处理阻塞计算：

```kotlin
suspend fun awaitBlockingExample() {
  val s = awaitBlocking<String> {
    Thread.sleep(1000)
    "some-string"
  }
}
```

# 事件流

在vert.x API的很多地方，事件流都是通过handler来处理。这些例子包括事件消息总线的使用者和http服务器的请求。

ReceiveChannelHandler类允许通过(suspendable)receive方法接收事件：

```kotlin
suspend fun streamExample() {
  val adapter = vertx.receiveChannelHandler<Message<Int>>()
  vertx.eventBus().localConsumer<Int>("a.b.c").handler(adapter)

  // Send 15 messages
  for (i in 0..15) vertx.eventBus().send("a.b.c", i)

  // Receive the first 10 messages
  for (i in 0..10) {
    val message = adapter.receive()
    println("Received: ${message.body()}")
  }
}
```

# 等待vert.x的future的完成

vert.x的future类实例的扩展方法await，可以暂停协程直到他们完成。在这种情况下，该方法返回相应的AsyncResult<T>对象。

```kotlin
suspend fun awaitingFuture() {
  val httpServerFuture = Future.future<HttpServer>()
  vertx.createHttpServer()
    .requestHandler { req -> req.response().end("Hello!") }
    .listen(8000, httpServerFuture)

  val httpServer = httpServerFuture.await()
  println("HTTP server port: ${httpServer.actualPort()}")

  val result = CompositeFuture.all(httpServerFuture, httpServerFuture).await()
  if (result.succeeded()) {
    println("The server is now running!")
  } else {
    result.cause().printStackTrace()
  }
}
```

# 通道

channel和java的BlockingQueue类似，只是Channel不会阻塞而是暂停协程。

- 将值发送到满了的Channel会暂停协程
- 从一个空Channel中接收值也会暂停协程

使用toChannel的扩展方法可以将vert.x的ReadStream 和 WriteStream适配成通道。

这些适配器负责管理背压和流终端：

- ReadStream<T> 适配为 ReceiveChannel<T>
- WriteStream<T> 适配为  SendChannel<T>

## 接收数据

当你需要处理一系列相关值的时候，channel可能非常有用：

```Kotlin
suspend fun handleTemperatureStream() {
  val stream = vertx.eventBus().consumer<Double>("temperature")
  val channel = stream.toChannel(vertx)

  var min = Double.MAX_VALUE
  var max = Double.MIN_VALUE

  // Iterate until the stream is closed
  // Non-blocking
  for (msg in channel) {
    val temperature = msg.body()
    min = Math.min(min, temperature)
    max = Math.max(max, temperature)
  }

  // The stream is now closed
}
```

他也可以用于解析协议。我们将构建一个非阻塞的http请求解析器来展示通道的功能。

我们将依靠RecordParser将以\r \n分割的buffer流进行分割。

这是解析器的初始版本，它只处理http的请求行。

```kotlin
val server = vertx.createNetServer().connectHandler { socket ->

  // The record parser provides a stream of buffers delimited by \r\n
  val stream = RecordParser.newDelimited("\r\n", socket)

  // Convert the stream to a Kotlin channel
  val channel = stream.toChannel(vertx)

  // Run the coroutine
  launch(vertx.dispatcher()) {

    // Receive the request-line
    // Non-blocking
    val line = channel.receive().toString().split(" ")
    val method = line[0]
    val uri = line[1]

    println("Received HTTP request ($method, $uri)")

    // Still need to parse headers and body...
  }
}
```

解析请求行就像在channel上调用receive一样简单。

下一步是通过接收块来解析http头，直到我们得到一个空块为止。

```kotlin
// Receive HTTP headers
val headers = HashMap<String, String>()
while (true) {

  // Non-blocking
  val header = channel.receive().toString()

  // Done with parsing headers
  if (header.isEmpty()) {
    break
  }

  val pos = header.indexOf(':')
  headers[header.substring(0, pos).toLowerCase()] = header.substring(pos + 1).trim()
}

println("Received HTTP request ($method, $uri) with headers ${headers.keys}")

```

最后，我们通过处理可选的请求体来终止解析器。

```kotlin
// Receive the request body
val transferEncoding = headers["transfer-encoding"]
val contentLength = headers["content-length"]

val body : Buffer?
if (transferEncoding == "chunked") {

  // Handle chunked encoding, e.g
  // 5\r\n
  // HELLO\r\n
  // 0\r\n
  // \r\n

  body = Buffer.buffer()
  while (true) {

    // Parse length chunk
    // Non-blocking
    val len = channel.receive().toString().toInt(16)
    if (len == 0) {
      break
    }

    // The stream is flipped to parse a chunk of the exact size
    stream.fixedSizeMode(len + 2)

    // Receive the chunk and append it
    // Non-blocking
    val chunk = channel.receive()
    body.appendBuffer(chunk, 0, chunk.length() - 2)

    // The stream is flipped back to the \r\n delimiter to parse the next chunk
    stream.delimitedMode("\r\n")
  }
} else if (contentLength != null) {

  // The stream is flipped to parse a body of the exact size
  stream.fixedSizeMode(contentLength.toInt())

  // Non-blocking
  body = channel.receive()
} else {
  body = null
}

println("Received HTTP request ($method, $uri) with headers ${headers.keys} and body with size ${body?.length() ?: 0}")

```

## 发送数据

使用channel发送数据也非常直接：

```Kotlin
suspend fun sendChannel() {
  val stream = vertx.eventBus().publisher<Double>("temperature")
  val channel = stream.toChannel(vertx)

  while (true) {
    val temperature = readTemperatureSensor()

    // Broadcast the temperature
    // Non-blocking but could be suspended
    channel.send(temperature)

    // Wait for one second
    awaitEvent<Long> { vertx.setTimer(1000, it)  }
  }
}

```

SendChannel#send 和 WriteStream#write都是非阻塞操作。不像当channel满的时候SendChannel#send可以停止执行，而等效WriteStream#writ的无channel操作可能像这样：

```kotlin
// Check we can write in the stream
if (stream.writeQueueFull()) {

  // We can't write so we set a drain handler to be called when we can write again
  stream.drainHandler { broadcastTemperature() }
} else {

  // Read temperature
  val temperature = readTemperatureSensor()

  // Write it to the stream
  stream.write(temperature)

  // Wait for one second
  vertx.setTimer(1000) {
    broadcastTemperature()
  }
}
```

# 延迟，取消和超时

借助于vert.x的定时器，vert.x的调度器完全支持协程的delay函数：

```kotlin
launch(vertx.dispatcher()) {
  // Set a one second Vertx timer
  delay(1000)
}
```

定时器也支持取消：

```kotlin
val job = launch(vertx.dispatcher()) {
  // Set a one second Vertx timer
  while (true) {
    delay(1000)
    // Do something periodically
  }
}

// Sometimes later
job.cancel()
```

取消是合作的。

你也可以使用withTimeout函数安排超时。

```Kotlin
launch(vertx.dispatcher()) {
  try {
    val id = withTimeout<String>(1000) {
      return awaitEvent<String> { anAsyncMethod(it) }
    }
  } catch (e: TimeoutCancellationException) {
    // Cancelled
  }
}
```

Vert.x支持所有的协程构建器：launch，async和runBlocking。runBlocking构建器不能再vert.x的时间循环线程中使用。

# 协程的互操作性

vert.x集成协程被设计为完全可以和kotlin协程互操作。

kotlinx.coroutines.experimental.sync.Mutex被执行在使用vert.x调度器的事件循环线程。

# RxJava的互操作性

虽然vertx-lang-kotlin-coroutines模块没有与RxJava特定集成，但是kotlin协程提供了RxJava的集成。RxJava可以和vertx-lang-kotlin-coroutines很好的协同工作。

你可以阅读响应流和协程的指南。

