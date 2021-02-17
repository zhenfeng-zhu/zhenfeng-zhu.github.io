---
date: "2018-06-23 15:05:32"
title: java-reactive-web
---

Spring web mvc： 传统servlet web

spring web flux： Reactive web



- 编程模式： non-blocking  非阻塞
  - nio：同步？异步？
- 并行模型
  - sync 同步
  - async 异步

# Reactive 概念

Reactive programming： 响应式编程

In [computing](https://en.wikipedia.org/wiki/Computing), **reactive programming** is a declarative [programming paradigm](https://en.wikipedia.org/wiki/Programming_paradigm) concerned with [data streams](https://en.wikipedia.org/wiki/Dataflow_programming) and the propagation of change. With this paradigm it is possible to express static (e.g. arrays) or dynamic (e.g. event emitters) *data streams* with ease, and also communicate that an inferred dependency within the associated *execution model* exists, which facilitates the automatic propagation of the changed data flow.



## 实现框架

- RxJava

  ReactiveX is a library for composing asynchronous and event-based programs by using observable sequences.

  这种就是推的模式

  ```
  int[] a=[1, 2, 3]
  
  for(int i: a){
  
  }
  ```

  ```
  package com.reactive.demo.reactivedemo;
  
  import java.util.Observable;
  
  /**
   * todo
   *
   * @author zhuzhenfeng
   * @date 2018/6/23
   */
  public class ObserverPatternDemo {
      public static void main(String[] args) {
          MyObservable observable = new MyObservable();
  
          // 1 observable   n个observer
  
          observable.addObserver((o, value) -> {
              System.out.println("1 收到数据更新" + value);
          });
  
          observable.addObserver((o, value) -> {
              System.out.println("2 收到数据更新" + value);
          });
  
          observable.setChanged();
  
          observable.notifyObservers("hello world");//  push data 发布数据
  
  
      }
  
      public static class MyObservable extends Observable {
          @Override
          protected synchronized void setChanged() {
              super.setChanged();
          }
      }
  }
  ```

  ```
  /Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home/bin/java "-
  2 收到数据更新hello world
  1 收到数据更新hello world
  
  Process finished with exit code 0
  ```

  当时不阻塞，后续回调。非阻塞基本上采用callback的形式。

  对于java来讲，异步代表切换了线程。

  当前的实现： 同步+非阻塞

  如果是切换了线程，代表是异步 的非阻塞，一般是gui程序的。

- Reactor







## 特性

- 异步
- 非阻塞
- 事件驱动
- 可能有背压 backpressure
- 防止回调地狱
- 





# Reactive 使用场景

Long Live 模式： netty的io连接（rpc） timeout



short live模式：不太适合Reactive web，因为这是等待。只是会快速返回，但是并不会给你真正的结果。短频快的连接，不太有用武之地。

- http
- http超时时间



# Reactive 理解误区

web：快速响应

200 Q->200 T -> 50T

1-50

Tomcat connector thread pool(200)->reactive thread pool(50)

io连接从Tomcat->Reactive

连接

Reactive thread pool（50）

不太适合web请求。

webflux其实并不会提升性能。

![](http://ww1.sinaimg.cn/large/b831e4c7gy1fsl9ian4ezj20ma0g7tg4.jpg)

少量的线程，少量的内存来做更好的伸缩性，而并不是为了提升更好的性能。使用Reactive只会是使单位时间内接受请求的数量增加，单位时间内的处理请求的数量下降。