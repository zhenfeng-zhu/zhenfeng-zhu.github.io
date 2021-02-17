---
date: "2018-04-08 16:01:48"
title: 谈谈web框架
---

这篇文章打的标签比较多，也基本涵盖了我所了解的一些知识，归纳总结一下自己对web框架的理解。自己了解的也不是很多，也请多多指教。

写程序免不了要做web相关的，现在由于前后端的分离，后端一般只提供rest接口，前端一般使用node来做渲染。在之前使用jsp那一套的时候，基本上都要写html+js的前端的一套，也要写后端java的CRUD。

我理解的web框架中，大致是分为这么两类：

- router框架
- mvc框架

## mvc类框架

mvc，初级程序员面试笔试的时候必考的一个知识点。model-view-controller，即模型-视图-控制器。

- m，模型主要用于封装与应用程序相关的数据以及对数据的处理方法。
- v，在 View 中一般没有程序上的逻辑。为了实现 View 上的刷新功能，View 需要访问它监视的数据模型（Model），因此应该事先在被它监视的数据那里注册。
- c，用于控制应用程序的流程。

我了解比较多的mvc框架是spring mvc。spring、spring mvc和spring boot等，他们并不是一个概念，也不是仅仅用于web开发。但是在这里我就不分那么细，统一用spring来代替。这里所说的spring都是指狭义上的web开发方面。

在做web开发的时候，项目目录一般是这样的：

```
 $ tree                                                              [16:23:43]
.
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               └── DemoApplication.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
            └── com
                └── example
                    └── demo
                        └── DemoApplicationTests.java

14 directories, 6 files
```

要渲染页面的时候，会把相关的类写在controller包下面，然后使用@Controlle注解表示这是一个controller。

如果是实体类，一般会放在entity包或者domain包中。

对数据库进行操作的类，一般会放在repository或者dao中。

controller一般不直接使用dao，而是会单独写一个service负责去做一些其他的事情。

每个包分工明确。

而url的路由拦截处理是在controller了中去实现的。

## router框架

我理解的router框架主要是以express为代表的框架。现在的轻量级的web框架会有路由这么一个重要的概念。

*路由*用于确定应用程序如何响应对特定端点的客户机请求，包含一个 URI（或路径）和一个特定的 HTTP 请求方法（GET、POST 等）。

每个路由可以具有一个或多个处理程序函数，这些函数在路由匹配时执行。

路由一般采用如下的结构：

```
router.METHOD(PATH, HANDLER)
```

其中：

- router是路由实例。
- METHOD是http的请求方法，如GET，POST等。
- PATH是URL请求路径。
- HANDLER是一个回调函数，在路由匹配成功时执行的。

可以发现，在router类框架中，handler是一个很常用的，这是一种编程的模式——行为参数化。

java的vertx和go的gin框架也是这样一种思路。

下面这个示例是gin的：

```
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	router:=gin.Default()

	router.GET("/hello", greeting)

    // 也可以写成匿名函数
	router.GET("/", func(context *gin.Context) {
		context.String(http.StatusOK, "I am Lucas")
	})

	router.Run()
}

// 可以单独写一个handler函数
func greeting(context *gin.Context)  {
	context.String(http.StatusOK, "hello world")
}

```

可以看到router类的框架特别轻量级，而且很适合写rest api接口。

### router的原理

一般情况下，router使用的数据结构是radix tree，压缩字典树。

字典树是一个比较常用的数据结构，下图是一个典型的字典树结构：

![](http://ww1.sinaimg.cn/large/b831e4c7gy1fq5cnri5emj212o0vagm1.jpg)

字典树一般用来进行字符串的检索。对于目标字符串，只要从根节点开始深度优先搜索，即可判断出该字符串是否曾经出现过，时间复杂度为 O(n)，n 可以认为是目标字符串的长度。为什么要这样做？字符串本身不像数值类型可以进行数值比较，两个字符串对比的时间复杂度取决于字符串长度。如果不用字典树来完成上述功能，要对历史字符串进行排序，再利用二分查找之类的算法去搜索，时间复杂度只高不低。可认为字典树是一种空间换时间的典型做法。

普通的字典树有一个比较明显的缺点，就是每个字母都需要建立一个孩子节点，这样会导致字典树的层树比较深，压缩字典树相对好地平衡了字典树的优点和缺点。下图是典型的压缩字典树结构：

![](http://ww1.sinaimg.cn/large/b831e4c7gy1fq5cpko9n2j212o0vaq3g.jpg)

每个节点上不只存储一个字母了，这也是压缩字典树中“压缩”的主要含义。使用压缩字典树可以减少树的层数，同时因为每个节点上数据存储也比通常的字典树要多，所以程序的局部性较好(一个节点的 path 加载到 cache 即可进行多个字符的对比)，从而对 CPU 缓存友好。

## 中间件

对于大多数的场景来讲，非业务的需求都是在 http 请求处理前做一些事情，或者/并且在响应完成之后做一些事情。我们有没有办法使用一些重构思路把这些公共的非业务功能代码剥离出去呢？

这个时候就是就引入了中间件的概念。

*中间件*函数能够访问[请求对象](http://expressjs.com/zh-cn/4x/api.html#req) (`req`)、[响应对象](http://expressjs.com/zh-cn/4x/api.html#res) (`res`) 以及应用程序的请求/响应循环中的下一个中间件函数。

中间件函数可以执行以下任务：

- 执行任何代码。
- 对请求和响应对象进行更改。
- 结束请求/响应循环。
- 调用堆栈中的下一个中间件。

这些中间件其实就是一些可插拔的函数组件，对请求和响应的对象进行封装处理。

### 哪些事情适合在中间件中做

以较流行的开源 go 框架 chi 为例：

```
compress.go
  => 对 http 的 response body 进行压缩处理
heartbeat.go
  => 设置一个特殊的路由，例如 /ping，/healthcheck，用来给 load balancer 一类的前置服务进行探活
logger.go
  => 打印 request 处理日志，例如请求处理时间，请求路由
profiler.go
  => 挂载 pprof 需要的路由，如 /pprof、/pprof/trace 到系统中
realip.go
  => 从请求头中读取 X-Forwarded-For 和 X-Real-IP，将 http.Request 中的 RemoteAddr 修改为得到的 RealIP 
requestid.go
  => 为本次请求生成单独的 requestid，可一路透传，用来生成分布式调用链路，也可用于在日志中串连单次请求的所有逻辑
timeout.go
  => 用 context.Timeout 设置超时时间，并将其通过 http.Request 一路透传下去
throttler.go
  => 通过定长大小的 channel 存储 token，并通过这些 token 对接口进行限流
```

我们可以发现，一些通用的非业务场景的都可以用中间件来包裹。

spring有AOP这个大杀器，它采用动态代理的方式也可以实现中间件的行为。

