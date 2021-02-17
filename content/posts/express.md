---
date: "2018-03-24 17:41:01"
title: express
---

## Express 快速入门

### 安装

```
npm init

npm install --save express
```

### hello world

```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

执行命令运行应用程序

```
node app.js
```

然后，在浏览器中输入 <http://localhost:3000/> 以查看输出。

## express程序生成器

### 安装

```
npm install -g express-generator
```

### 示例

以下语句在当前工作目录中创建名为 *myapp* 的 Express 应用程序：

```
express --view=pug myapp
```

在 MacOS 或 Linux 上，采用以下命令运行此应用程序：

```
DEBUG=myapp:* npm start
```

然后在浏览器中输入 `http://localhost:3000/` 以访问此应用程序。

## 路由

### 基本路由

*路由*用于确定应用程序如何响应对特定端点的客户机请求，包含一个 URI（或路径）和一个特定的 HTTP 请求方法（GET、POST 等）。

每个路由可以具有一个或多个处理程序函数，这些函数在路由匹配时执行。

路由定义采用以下结构：

```
app.METHOD(PATH, HANDLER)
```

其中：

- `app` 是 `express` 的实例。
- `METHOD` 是 [HTTP 请求方法](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol)。
- `PATH` 是服务器上的路径。
- `HANDLER` 是在路由匹配时执行的函数。

比如简单的Hello world：

```
app.get('/', function (req, res) {
  res.send('Hello World!');
});
```

### 响应方法

下表中响应对象 (`res`) 的方法可以向客户机发送响应，并终止请求/响应循环。如果没有从路由处理程序调用其中任何方法，客户机请求将保持挂起状态。

| 方法                                                         | 描述                                             |
| ------------------------------------------------------------ | ------------------------------------------------ |
| [res.download()](http://expressjs.com/zh-cn/4x/api.html#res.download) | 提示将要下载文件。                               |
| [res.end()](http://expressjs.com/zh-cn/4x/api.html#res.end)  | 结束响应进程。                                   |
| [res.json()](http://expressjs.com/zh-cn/4x/api.html#res.json) | 发送 JSON 响应。                                 |
| [res.jsonp()](http://expressjs.com/zh-cn/4x/api.html#res.jsonp) | 在 JSONP 的支持下发送 JSON 响应。                |
| [res.redirect()](http://expressjs.com/zh-cn/4x/api.html#res.redirect) | 重定向请求。                                     |
| [res.render()](http://expressjs.com/zh-cn/4x/api.html#res.render) | 呈现视图模板。                                   |
| [res.send()](http://expressjs.com/zh-cn/4x/api.html#res.send) | 发送各种类型的响应。                             |
| [res.sendFile](http://expressjs.com/zh-cn/4x/api.html#res.sendFile) | 以八位元流形式发送文件。                         |
| [res.sendStatus()](http://expressjs.com/zh-cn/4x/api.html#res.sendStatus) | 设置响应状态码并以响应主体形式发送其字符串表示。 |

### app.route()

可以使用 `app.route()` 为路由路径创建可链接的路由处理程序。 因为在单一位置指定路径，所以可以减少冗余和输入错误。

```
app.route('/book')
  .get(function(req, res) {
    res.send('Get a random book');
  })
  .post(function(req, res) {
    res.send('Add a book');
  })
  .put(function(req, res) {
    res.send('Update the book');
  });
```

### express.Router

使用 `express.Router` 类来创建可安装的模块化路由处理程序。`Router` 实例是完整的中间件和路由系统；因此，常常将其称为“微型应用程序”。

以下示例将路由器创建为模块，在其中装入中间件，定义一些路由，然后安装在主应用程序的路径中。

在应用程序目录中创建名为 `birds.js` 的路由器文件，其中包含以下内容：

```
var express = require('express');
var router = express.Router();

// middleware that is specific to this router
router.use(function timeLog(req, res, next) {
  console.log('Time: ', Date.now());
  next();
});
// define the home page route
router.get('/', function(req, res) {
  res.send('Birds home page');
});
// define the about route
router.get('/about', function(req, res) {
  res.send('About birds');
});

module.exports = router;
```

接着，在应用程序中装入路由器模块：

```
var birds = require('./birds');
...
app.use('/birds', birds);
```

此应用程序现在可处理针对 `/birds` 和 `/birds/about` 的请求，调用特定于此路由的 `timeLog` 中间件函数。

## 中间件

*中间件*函数能够访问[请求对象](http://expressjs.com/zh-cn/4x/api.html#req) (`req`)、[响应对象](http://expressjs.com/zh-cn/4x/api.html#res) (`res`) 以及应用程序的请求/响应循环中的下一个中间件函数。下一个中间件函数通常由名为 `next` 的变量来表示。

> `next()` 函数不是 Node.js 或 Express API 的一部分，而是传递给中间件函数的第三自变量。`next()` 函数可以命名为任何名称，但是按约定，始终命名为“next”。

中间件函数可以执行以下任务：

- 执行任何代码。
- 对请求和响应对象进行更改。
- 结束请求/响应循环。
- 调用堆栈中的下一个中间件。

如果当前中间件函数没有结束请求/响应循环，那么它必须调用 `next()`，以将控制权传递给下一个中间件函数。否则，请求将保持挂起状态。

Express 应用程序可以使用以下类型的中间件：

- [应用层中间件](http://expressjs.com/zh-cn/guide/using-middleware.html#middleware.application)
- [路由器层中间件](http://expressjs.com/zh-cn/guide/using-middleware.html#middleware.router)
- [错误处理中间件](http://expressjs.com/zh-cn/guide/using-middleware.html#middleware.error-handling)
- [内置中间件](http://expressjs.com/zh-cn/guide/using-middleware.html#middleware.built-in)
- [第三方中间件](http://expressjs.com/zh-cn/guide/using-middleware.html#middleware.third-party)

## 模板引擎

在 Express 可以呈现模板文件之前，必须设置以下应用程序设置：

- `views`：模板文件所在目录。例如：`app.set('views', './views')`
- `view engine`：要使用的模板引擎。例如：`app.set('view engine', 'pug')`

然后安装对应的模板引擎 npm 包：

```
npm install pug --save
```

在 `views` 目录中创建名为 `index.pug` 的 Pug 模板文件，其中包含以下内容：

```
html
  head
    title!= title
  body
    h1!= message
```

随后创建路由以呈现 `index.pug` 文件。如果未设置 `view engine` 属性，必须指定 `view` 文件的扩展名。否则，可以将其忽略。

```
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!'});
});
```

向主页发出请求时，`index.pug` 文件将呈现为 HTML。