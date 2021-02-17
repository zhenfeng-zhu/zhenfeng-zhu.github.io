---
date: "2018-08-22 19:39:12"
title: 谈谈聊天机器人框架的实现原理
---

在这篇文章不考虑人工智能，谈谈我对聊天机器人框架实现机制的理解。

## 聊天机器人

> **聊天机器人**（Chatterbot）是经由对话或文字进行交谈的计算机程序[[1\]](https://zh.wikipedia.org/wiki/%E8%81%8A%E5%A4%A9%E6%A9%9F%E5%99%A8%E4%BA%BA#cite_note-target-1)。能够模拟人类对话，通过[图灵测试](https://zh.wikipedia.org/wiki/%E5%9B%BE%E7%81%B5%E6%B5%8B%E8%AF%95)。

我们可以看到现有的IM工具上已经有了很多机器人，其实聊天机器人不只是单纯的和用户进行聊天，他其实还可以做很多事情，例如根据用户输入的一些话，可以帮用户订餐。另外在运维领域，也出现了chatops，通过和机器人聊天，进行运维操作。

## 机器人开发框架

作为聊天机器人开发者，面对如此多的IM工具和SDK，常会感到无所适从。Bot 开发框架就是对聊天机器人开发过程中的人工内容做抽象化处理。简单地解释，机器人开发框架就是用来制造机器人并定义其行为。

然而尽管很多机器人框架宣称「代码一旦写好可部署到任何地方」，但是还会是出现为每一个IM工具开发一个单独的聊天机器人。而一个良好的机器人框架主要包含开发SDK，连接器和模拟器等。

使用机器人框架其实并不适合初学者学习聊天机器人开发。它们尝试自动化太多工作，对初学者掩盖了基础机制。

## 实现方式

- webhook事件回调
- FSM状态机
- workflow工作流

最简单的机器人是没有上下文的语义理解的一问一答，仅仅是对用户的对话进行响应，这种就可以采用webhook的方式进行开发。不需要采用什么开发框架。

那么对于多轮对话的时候，就需要进行一定的对话管理。由此引入了FSM状态机。

可能有人不是很懂有限状态机，这里做一下简单说明。

> 有限状态机在现实生活中其实随处可见，伸缩式圆珠笔其实就是一个有限状态机（两种状态互相转换）。
>
> 有限状态机，缩写为FSM，又称为有限状态自动机，简称状态机。是表示有限个[状态](https://zh.wikipedia.org/wiki/%E7%8A%B6%E6%80%81)以及在这些状态之间的转移和动作等行为的[数学模型](https://zh.wikipedia.org/wiki/%E6%95%B0%E5%AD%A6%E6%A8%A1%E5%9E%8B)。 
>
> 可以总结为：f(state, action) => state’
>
> 也就是说，这个函数采用当前的状态和一次行动（即更改状态的方法），之后将该行动应用于这种状态并返回新的状态。
>
> 可以认为状态机是图灵完备的。

我们可以将对话看做是在有限状态内跳转的过程，每个状态都有对应的动作和回复，如果能从开始节点顺利的流转到终止节点，任务就完成了。

我们可以将对话的过程，分为一个个的状态，然后使用DSL来实现一个FSM，对于开发者来讲，我们只需要关注一个个状态函数即可。

特点是：

- 人为定义对话流程
- 完全有系统主导，系统问用户答
- 答非所问的情况直接忽略
- 建模简单，能清晰明了的把交互匹配到模型
- 难以扩展，很容易变的复杂
- 适用于简单的任务，难以处理复杂问题
- 缺少灵活性，表达能力有限，输入有限，对话结构和流转路径有限

示例：

```
const {startWith, when, goto, stay, stop} = botkit.DSL(fsm);
 
startWith(MyStates.IDLE, {counter: 0});

when(MyStates.IDLE)(async (sender, content, data) => {
           
});

when(MyStates.UI)((sender, content, data) => {
            
});

when(MyStates.STEP1)((sender, content, data) => {
            
});

when(MyStates.STEP2)((sender, content, data) => {
            
});

when(MyStates.DONE)((sender, content, data) => {
            
});

when(MyStates.EMPTY)((sender, content, data) => {
            
});

when(MyStates.LOOP)((sender, content, data) => {
           
});
```

从示例中可以发现，基于fsm的机器人框架需要使用类似DSL领域特定语言一样的描述语言，定义各种各样的状态，每一个状态都有触发点。当满足某个状态条件时，进入该状态，执行该状态的逻辑。这种基于状态机的机器人框架，对于简单的场景比较容易写，但是如果是遇到了复杂的场景，比如多轮对话中还附带上下文信息，就会写起来非常复杂。

于是引入了基于工作流的chatbot框架。其实工作流是对fsm的一种简化封装，本质上来讲，工作流能做到的，fsm状态机也能做到，而且fsm状态机或许能拆的更细，但是工作流的一个个function，或者是function的集合dialog，可以互相组合，开发起来更符合大部分人的直觉。

- routing dialog

  ```
  // hotels.js
  module.exports = [
      // Destination
      function (session) {
          session.send('Welcome to the Hotels finder!');
          builder.Prompts.text(session, 'Please enter your destination');
      },
      function (session, results, next) {
          session.dialogData.destination = results.response;
          session.send('Looking for hotels in %s', results.response); 
          next();
      },
      ...
  ];
  
  // app.js
  var bot = new builder.UniversalBot(connector, [
      function (session) {
          // ...
      },
      // ...
  ]);
  
  bot.dialog('hotels', require('./hotels'));
  bot.dialog('flights', require('./flights'));
  ```

  通过routing dialog，我们可以实现dialog的复用。

- waterfall dialog

  一个瀑布流的dialog，可以让我们在一个dialog中像流一样完成一系列的动作。就像fsm的多种状态的集合。

  ```
  [
      // Destination
      function (session) {
          session.send('Welcome to the Hotels finder!');
          builder.Prompts.text(session, 'Please enter your destination');
      },
      function (session, results, next) {
          session.dialogData.destination = results.response;
          session.send('Looking for hotels in %s', results.response); 
          next();
      },
      ...
      function (session) {
          var destination = session.dialogData.destination;
          var checkIn = new Date(session.dialogData.checkIn);
          var checkOut = checkIn.addDays(session.dialogData.nights);
  
          session.send(
              'Ok. Searching for Hotels in %s from %d/%d to %d/%d...',
              destination,
              checkIn.getMonth() + 1, checkIn.getDate(),
              checkOut.getMonth() + 1, checkOut.getDate());
  
          // Async search
          Store
              .searchHotels(destination, checkIn, checkOut)
              .then(function (hotels) {
                  // Results
                  session.send('I found in total %d hotels for your dates:', hotels.length);
  
                  var message = new builder.Message()
                      .attachmentLayout(builder.AttachmentLayout.carousel)
                      .attachments(hotels.map(hotelAsAttachment));
  
                  session.send(message);
  
                  // End
                  session.endDialog();
              });
      }
  ]
  ```

- state

  在一个dialog上下文中共享的数据，或者在多个dialog中共享的数据。对于微软的botbuilder来讲，他们提供了如下几个API：

  | Field                   | Use Cases                                                    |
  | ----------------------- | ------------------------------------------------------------ |
  | userData                | Stores information globally for the user across all conversations. |
  | conversationData        | Stores information globally for a single conversation. This data is visible to everyone within the conversation so care should be used to what’s stored there. It’s disabled by default and needs to be enabled using the bots [`persistConversationData`](https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata) setting. |
  | privateConversationData | Stores information globally for a single conversation but its private data for the current user. This data spans all dialogs so it’s useful for storing temporary state that you want cleaned up when the conversation ends. |
  | dialogData              | Persists information for a single dialog instance. This is essential for storing temporary information in between the steps of a waterfall. |

## Conversation UI

对话式 UI（Conversation UI，下文简称 CUI）。

CUI 到底是什么？很好理解，我们日常跟人聊天的微信、短信界面就是。由一条条消息组成，按时间先后展示出来，就可以看作 CUI。

chatbot在与用户交流时，不单单是只有文字，还会需要用户进行互动，这时候就是CUI的用武之地了。我们可以和移动端进行约定，对一些特定的消息格式进行渲染，这样就可以做出按钮，列表等。

## Bot Service

作为一个机器人框架，开发完成之后，还需要和telegram，Facebook messenger，slack等IM平台进行对接，如果要开发者一个个对接的话，将会特别麻烦。作为chatbot开发框架的一部分，bot service的工作就是对接IM平台。

## Bot Builder源码阅读

微软的botbuilder-js出到了V4版本，在新版本的机器人框架有着很大的变动，相比于V3目录结构变化了，而且机器人编写流程也有了一定的差异。

项目结构

```
├── botbuilder
├── botbuilder-ai
├── botbuilder-azure
├── botbuilder-core
├── botbuilder-dialogs
├── botframework-config
├── botframework-connector
├── botframework-schema
```

目录结构更加的组件化。

如果我们不使用微软的服务，那么botbuilder-ai和botbuilder-azure其实不重要。

### botbuilder

botbuilder是框架的入口，在这个package中做的事情比较简单：

```typescript
export * from './botFrameworkAdapter';
export * from './fileTranscriptStore';
export * from '../../botbuilder-core/lib';
```

导出botbuilder-core和继承了botAdapter的子类botFrameworkAdapter。

fileTranscriptStore是存储每个activity的transcript到文件中，Transcript是人和bot的对话动作的日志。

如果我们要定制自己的bot动作，其实就可以继承botAdapter，然后对接自己的IM等等。botAdapter也是botbuilder-core中的，所以botbuilder-core是核心，只要读懂了botbuilder-core，就可以说是理解了微软的机器人框架。

### botbuilder-core

看botbuilder-core，也从index.ts开始。

```typescript
export * from '../../botframework-schema/lib';
export * from './autoSaveStateMiddleware';
export * from './botAdapter';
export * from './botState';
export * from './botStatePropertyAccessor';
export * from './botStateSet';
export * from './browserStorage';
export * from './cardFactory';
export * from './conversationState';
export * from './memoryStorage';
export * from './memoryTranscriptStore';
export * from './messageFactory';
export * from './middlewareSet';
export * from './privateConversationState';
export * from './propertyManager';
export * from './recognizerResult';
export * from './showTypingMiddleware';
export * from './storage';
export * from './testAdapter';
export * from './transcriptLogger';
export * from './turnContext';
export * from './userState';
```

这里引入了一个botframework-schema，通过名字可以看出来，这就是一个类型定义的包，主要是机器人Activity的Schema。Activity是人和bot所做的会话的程序级别的表示，该schema中包含了文本协议、多媒体和非内容动作（如社交互动和打字指示符）的规定。



