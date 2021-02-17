---
date: "2018-04-04 13:48:57"
title: Go语言体会
---

最近公司要统一技术栈，在kotlin和go之间选。我心里是比较倾向go的，主要有如下几点体会。

- 语言简单，上手快。
- gorotuine
- 易发布
- 垃圾回收
- 约定大于配置

我最早听说协程，是在大三找实习的时候，那个时候面试会问线程和进程的关系，问的深一些就是协程和线程的区别。游戏公司基本都用lua，看了lua的资料后，对协程有了一些自己的了解，随后就是在做Unity相关的开发，在unity中使用了很多的协程，但是在unity中使用的协程好像跟主流的不太一样，在看了go之后，豁然开朗。

goroutine使用的内存比线程更少，go在运行的时候会自动在配置的一组逻辑处理器上调度执行。比如：

```
func log(msg string){
    ...
}

go log("")
```

使用关键字go，即可让log函数在一个goroutine里执行了。

并发最难的部分是要确保其他并发运行的进程、线程或者goroutine不会以外的修改数据。go使用了Channel的方式来解决这个问题。对于通道模式，保证同一时刻只会有一个goroutine修改数据。

说起go的语言简单，其实主要是他的类型比较简单。go使用的是组合模式，只需要将一个类型嵌入到另外一个类型就可以复用所有的功能。而且go还具有独特的接口实现机制，允许用户对行为进行建模，在go中不需要声明某个类型实现了某个接口，编译器会自动判断一个实例是使用什么接口。

对于java来说，所有的设计都是围绕着接口展开，于是在设计模式中，就是面向接口编程：

```
interface User{
    void login();
    void logout();
}
```

在java中，继承的类必须显式声明继承了此接口。而在go中接口只是描述一个动作，如果说是实现这个接口，只需要让某个实例实现了这个接口中的所有方法就行了。

```
type Reader interface{
    Read(p []byte))(n int, err error)
}
```

这其实和传统的oop语言的接口有着本质的区别，go中的接口一般只定义一个单一的动作，实际使用的过程中，这更有利于使用组合来复用代码。

约定大于配置这点，go在这方面上做的感觉有点儿吹毛求疵了，但是这样也使得程序可读性更强，没有很多垃圾代码。比如go的文件结构必须是src pkg 和bin 三个包，而且go也不允许你声明一个变量却不使用，导入了一个包却不使用，而且程序的代码也有约定，init方法比main方法更早执行。

## go的并发

说到并发，就会想到另外一个概念，并行。可以简单这样的理解：

```
并发是同时管理多个事情，而并行是同时做很多事情。也就是并发是manage，并行是run。
```

对于单核处理器来讲，同一时刻只能有一个任务在执行，那么并发就是同时管理多个任务，让他们交替执行。并行是针对于多核处理器的，同一时刻可以把多个任务放在不同的处理器上执行，这样就可以同时执行。

在go里面主要是采用协程来实现并发的，也就是goroutine。与其他语言不同的是，go是在语法层面做到的，即go func();

### 语法

```
go f(x, y)
```

go是关键字，后面跟函数。

### 例子

```
package main

import (
	"log"
	"time"
)

func doSomething(id int) {
	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	doSomething(1)
	doSomething(2)
	doSomething(3)
}
```

这个例子的输出是：

```
2018/04/15 17:06:05 before do job:(1) 
2018/04/15 17:06:08 after do job:(1) 
2018/04/15 17:06:08 before do job:(2) 
2018/04/15 17:06:11 after do job:(2) 
2018/04/15 17:06:11 before do job:(3) 
2018/04/15 17:06:14 after do job:(3) 
```

可以看到是用了9秒的时间才完成，如果是采用goroutine的话，就很快。很简单，就是在执行doSomething之前，加上go关键字。

```
import (
	"log"
	"time"
)

func doSomething(id int) {
	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	go doSomething(1)
	go doSomething(2)
	go doSomething(3)
}
```

但是这样的话，什么结果也没有，是因为main函数本身也是一个goroutine，main执行完之后，其他的还没开始，所以什么也看不到。最简单的办法就是让main函数等待一段时间再结束，但是这样不够优雅。

我们应该采用sync.WaitGroup来等待所有的goroutine结束。

```
package main

import (
	"log"
	"sync"
	"time"
)

func doSomething(id int, wg *sync.WaitGroup) {
	defer wg.Done()

	log.Printf("before do job:(%d) \n", id)
	time.Sleep(3 * time.Second)
	log.Printf("after do job:(%d) \n", id)
}

func main() {
	var wg sync.WaitGroup
	wg.Add(3)

    // 因为我们要修改wg的状态，所以要传指针过去
	go doSomething(1, &wg)
	go doSomething(2, &wg)
	go doSomething(3, &wg)

	wg.Wait()
	log.Printf("finish all jobs\n")
}
```

执行结果是：

```
2018/04/15 17:13:14 before do job:(3) 
2018/04/15 17:13:14 before do job:(2) 
2018/04/15 17:13:14 before do job:(1) 
2018/04/15 17:13:17 after do job:(2) 
2018/04/15 17:13:17 after do job:(1) 
2018/04/15 17:13:17 after do job:(3) 
```

可以看到这次只用了3秒左右就执行完了，而且他们的执行顺序也不确定，竞争执行。

### channel

每个协程之间要进行通信，那么在通信的时候采用的是Channel的形式，即一个goroutine将数据传递给Channel，另一个goroutine从Channel中读取数据。

创建Channel有两种方式：

使用内建函数 make 可以创建 channel，举例如下：

```
ch := make(chan int)  // 注意： channel 必须定义其传递的数据类型
```

也可以用 var 声明 channel, 如下：

```
var ch chan int
```

以上声明的 channel 都是双向的，意味着可以该 channel 可以发送数据，也可以接收数据。





