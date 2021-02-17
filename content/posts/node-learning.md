---
date: "2018-04-22 15:41:21"
title: node学习笔记
---

写node也有一段时间了，整理一下学习笔记，共同进步

# 什么是node？

首先看一下什么是node.js

- Node 是一个服务器端 JavaScript 
- Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境
- Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效
- Node.js 的包管理器 npm，是全球最大的开源库生态系统

模块系统是node最基本也是最常用的。一般可以分为四类：

- 原生模块
- 文件模块
- 第三方模块
- 自定义模块

node社区崇尚DRY文化，即Don't repeate yourself。这种文化使得node的生态异常繁荣，同样也由于某些包的质量低下引来了一些诟病。

# 谈谈自定义模块

我们在写node程序的时候，一般都是在写自定义模块。

- 创建模块

  ```
  // b.js
  function FunA(){
      return "hello world";
  }

  // 暴露方法FunA
  module.exports = FunA;
  ```

- 加载模块

  ```
  // a.js
  const FunA=require('./b.js');

  // 运行FunA
  const name=FunA();
  console.log(name);
  ```

在做模块到处的时候有两种方式：

- module.exports

  module.exports 就 Node.js 用于对外暴露，或者说对外开放指定访问权限的一个对象。

  一个模块中有且仅有一个 module.exports，如果有多个那后面的则会覆盖前面的。

- exports

  exports 是 module 对象的一个属性，同时它也是一个对象。在很多时候一个 js 文件有多个需要暴露的方法或是对象，module.exports 又只能暴露一个，那这个时候就要用到 exports:

  ```
  function FunA(){
      return 'Tom';
  }

  function FunB(){
      return 'Sam';
  }

  exports.FunA = FunA;
  exports.FunB = FunB;
  ```

  ```
  //FunA = exports,exports 是一个对象
  var FunA = require('./b.js');
  var name1 = FunA.FunA();// 运行 FunA，name = 'Tom'
  var name2 = FunA.FunB();// 运行 FunB，name = 'Sam'
  console.log(name1);
  console.log(name2);
  ```

  当然在引入的时候也可以这样写：

  ```
  //FunA = exports,exports 是一个对象
  var {FunA, FunB} = require('./b.js');
  var name1 = FunA();// 运行 FunA，name = 'Tom'
  var name2 = FunB();// 运行 FunB，name = 'Sam'
  console.log(name1);
  console.log(name2);
  ```

# 常用的原生模块

常用的原生模块有如下四个：

- http
- url
- queryString
- fs

## http

所有后端的语言要想运行起来，都得有服务器。node通过原生的http模块来搭建服务器：

1. 加载 http 模块
2. 调用 http.createServer() 方法创建服务，方法接受一个回调函数，回调函数中有两个参数，第一个是请求体，第二个是响应体。
3. 在回调函数中一定要使用 response.end() 方法，用于结束当前请求，不然当前请求会一直处在等待的状态。
4. 调用 listen 监听一个端口。

```
//原生模块
var http = require('http');

http.createServer(function(reqeust, response){
    response.end('Hello Node');
}).listen(8080);
```

处理参数

- get

  当get请求的时候，服务器通过request.method来判断当前的请求方式并通过request.url来获取当前的请求参数：

  ```
  var http = require('http');
  var url = require('url');
   
  http.createServer(function(req, res){
      var params = url.parse(req.url, true).query;
      res.end(params);
   
  }).listen(3000);
  ```

- post

  post请求则不能通过url来获取，这时候就得对请求体进行事件监听。

  ```
  var http = require('http');
  var util = require('util');
  var querystring = require('querystring');
   
  http.createServer(function(req, res){
      // 定义了一个post变量，用于暂存请求体的信息
      var post = '';     
   
      // 通过req的data事件监听函数，每当接受到请求体的数据，就累加到post变量中
      req.on('data', function(chunk){    
          post += chunk;
      });
   
      // 在end事件触发后，通过querystring.parse将post解析为真正的POST请求格式，然后向客户端返回。
      req.on('end', function(){    
          post = querystring.parse(post);
          res.end(util.inspect(post));
      });
  }).listen(3000);
  ```

## url

url和http是配合使用的。一般情况下url都是字符串类型的，包含的信息也比较多，比如有：协议、主机名、端口、路径、参数、锚点等。如果是对字符串进行直接解析的话，相当麻烦，node提供的url模块便可轻松解决这一类的问题。

### 字符串转对象

- 格式：url.parse(urlstring, boolean)
- 参数
  - urlstring：字符串格式的 url
  - boolean：在 url 中有参数，默认参数为字符串，如果此参数为 true，则会自动将参数转转对象
- 常用属性
  - href： 解析前的完整原始 URL，协议名和主机名已转为小写
  - protocol： 请求协议，小写
  - host： url 主机名，包括端口信息，小写
  - hostname: 主机名，小写
  - port: 主机的端口号
  - pathname: URL中路径，下面例子的 /one
  - search: 查询对象，即：queryString，包括之前的问号“?”
  - path: pathname 和 search的合集
  - query: 查询字符串中的参数部分（问号后面部分字符串），或者使用 querystring.parse() 解析后返回的对象
  - hash: 锚点部分（即：“#”及其后的部分）

### 对象转字符串

- 格式：url.format(urlObj)
- 参数 urlObj 在格式化的时候会做如下处理
  - href: 会被忽略，不做处理
  - protocol：无论末尾是否有冒号都会处理，协议包括 http, https, ftp, gopher, file 后缀是 :// (冒号-斜杠-斜杠)
  - hostname：如果 host 属性没被定义，则会使用此属性
  - port：如果 host 属性没被定义，则会使用此属性
  - host：优先使用，将会替代 hostname 和port
  - pathname：将会同样处理无论结尾是否有/ (斜杠)
  - search：将会替代 query属性，无论前面是否有 ? (问号)，都会同样的处理
  - query：(object类型; 详细请看 querystring) 如果没有 search,将会使用此属性.
  - hash：无论前面是否有# (井号, 锚点)，都会同样处理

### 拼接

当有多个 url 需要拼接处理的时候，可以用到 url.resolve

```
var url = require('url');
url.resolve('http://dk-lan.com/', '/one')// 'http://dk-lan.com/one'
```

## querystring

url是对url字符串的处理，而querystring就是仅针对参数的处理。

### 字符串转对象

```
var str = 'firstname=dk&url=http%3A%2F%2Fdk-lan.com&lastname=tom&passowrd=123456';
var param = querystring.parse(param);
//结果
//{firstname:"dk", url:"http://dk-lan.com", lastname: 'tom', passowrd: 123456};
```

### 对象转字符串

```
var querystring = require('querystring');

var obj = {firstname:"dk", url:"http://dk-lan.com", lastname: 'tom', passowrd: 123456};
//将对象转换成字符串
var param = querystring.stringify(obj);
//结果
//firstname=dk&url=http%3A%2F%2Fdk-lan.com&lastname=tom&passowrd=123456
```

## fs

任何服务端语言都不能缺失文件的读写操作。

### 读取文本 -- 异步读取

```
var fs = require('fs');
// 异步读取
// 参数1：文件路径，
// 参数2：读取文件后的回调
fs.readFile('demoFile.txt', function (err, data) {
   if (err) {
       return console.error(err);
   }
   console.log("异步读取: " + data.toString());
});
```

### 读取文本 -- 同步读取

```
var fs = require('fs');
var data = fs.readFileSync('demoFile.txt');
console.log("同步读取: " + data.toString());
```

### 写入文本 -- 覆盖写入

```
var fs = require('fs');
//每次写入文本都会覆盖之前的文本内容
fs.writeFile('input.txt', '抵制一切不利于中国和世界和平的动机！',  function(err) {
   if (err) {
       return console.error(err);
   }
   console.log("数据写入成功！");
   console.log("--------我是分割线-------------")
   console.log("读取写入的数据！");
   fs.readFile('input.txt', function (err, data) {
      if (err) {
         return console.error(err);
      }
      console.log("异步读取文件数据: " + data.toString());
   });
});
```

### 写入文本 -- 追加写入

```
var fs = require('fs');
fs.appendFile('input.txt', '愿世界和平！', function (err) {
   if (err) {
       return console.error(err);
   }
   console.log("数据写入成功！");
   console.log("--------我是分割线-------------")
   console.log("读取写入的数据！");
   fs.readFile('input.txt', function (err, data) {
      if (err) {
         return console.error(err);
      }
      console.log("异步读取文件数据: " + data.toString());
   });
});
```

### 图片读取

图片读取不同于文本，因为文本读出来可以直接用 console.log() 打印，但图片则需要在浏览器中显示，所以需要先搭建 web 服务，然后把以字节方式读取的图片在浏览器中渲染。

1. 图片读取是以字节的方式
2. 图片在浏览器的渲染因为没有 img 标签，所以需要设置响应头为 image

```
var http = require('http');
var fs = require('fs');
var content =  fs.readFileSync('001.jpg', "binary");

http.createServer(function(request, response){
    response.writeHead(200, {'Content-Type': 'image/jpeg'});
    response.write(content, "binary");
    response.end();
}).listen(8888);

console.log('Server running at http://127.0.0.1:8888/');
```

## stream流处理

对http 服务器发起请求的request 对象就是一个 Stream，还有stdout（标准输出）。往往用于打开大型的文本文件，创建一个读取操作的数据流。所谓大型文本文件，指的是文本文件的体积很大，读取操作的缓存装不下，只能分成几次发送，每次发送会触发一个data事件，发送结束会触发end事件。

主要分为

- 读取流
- 写入流
- 管道流
- 链式流

这几种流都是fs的一部分。

## 路由

在BS架构中，路由的概念都是一样的，可以理解为根据客户端请求的url映射到不同的方法实现。一般web框架中都会有相应的路由模块。但是在原生node中去处理的话只能是解析url来进行映射，实现起来不够简洁。

# fetch

axios是一种对ajax的封装，fetch是一种浏览器原生实现的请求方式，跟ajax对等。

在现在发起http请求里，都是通过fetch来发送请求，和ajax类似。

```
const fetch=require('isomorphic-fetch');

const options={
    header:{},
    body:JSON.strify({}),
    method: ''
}

try{
    const res=await fetch('url', options);
}catch(err){
    
}
```

# Async

Node.js 是一个异步机制的服务端语言，在大量异步的场景下需要按顺序执行，那正常做法就是回调嵌套回调，回调嵌套太多的问题被称之回调地狱。

Node.js 为解决这一问题推出了异步控制流 ———— Async

Async/Await

Async/Await 就 ES7 的方案，结合 ES6 的 Promise 对象，使用前请确定 Node.js 的版本是 7.6 以上。

Async/await的主要益处是可以避免回调地狱（callback hell），且以最接近同步代码的方式编写异步代码。

基本规则

- async 表示这是一个async函数，await只能用在这个函数里面。
- await 表示在这里等待promise返回结果了，再继续执行。
- await 后面跟着的应该是一个promise对象

# express框架

使用node，都绕不开express。

## 简单使用

express的使用比较简单，由于我最早接触的是spring那套web框架，所以在使用到express的时候觉得node的web特别轻量简单。

加载模块

```
const express=require('express');
const app=express();
```

监听端口8080

```
app.listen(3000, ()=>consloe.log('running'));
```

## 路由

express对路由的处理特别简单，配合中间件body parser，很方便的提供rest接口：

```
app.get('/', (req, res)=>{
    res.send('hello world');
})
```

`response.send()` 可理解为 `response.end()`，其中一个不同点在于 `response.send()` 参数可为对象。

Node.js 默认是不能访问静态资源文件（*.html、*.js、*.css、*.jpg 等），如果要访问服务端的静态资源文件则要用到方法 `sendFile`

__dirname 为 Node.js 的系统变量，指向文件的绝对路径。

```
app.get('/index.html', function (req, res) {
   res.sendFile( __dirname + "/" + "index.html" );
});
```

Express -- GET 参数接收之路径方式

访问地址：`http://localhost:8080/getusers/admin/18`，可通过 `request.params` 来获取参数

```
app.get('/getUsers/:username/:age', function(request, response){
    var params = {
        username: request.params.username,
        age: request.params.age
    }
    response.send(params);
})
```

Express -- POST

- post 参数接收，可依赖第三方模块 body-parser 进行转换会更方便、更简单，该模块用于处理 JSON, Raw, Text 和 URL 编码的数据。
- 安装 body-parser `npm install body-parser`
- 参数接受和 GET 基本一样，不同的在于 GET 是 `request.query` 而 POST 的是 `request.body`

```
var bodyParser = require('body-parser');
// 创建 application/x-www-form-urlencoded 编码解析
var urlencodedParser = bodyParser.urlencoded({ extended: false })
app.post('/getUsers', urlencodedParser, function (request, response) {
    var params = {
        username: request.body.username,
        age: request.body.age
    }
   response.send(params);
});
```

Express -- 跨域支持(放在最前面)

```
app.all('*', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Content-Type,Content-Length, Authorization, Accept,X-Requested-With");
    res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By",' 3.2.1')
    if(req.method=="OPTIONS") {
      res.send(200);/*让options请求快速返回*/
    } else{
      next();
    }
});
```

## 中间件

express的中间件编写——过滤器

简单使用

```
const express = require('express')
const app = express();

let filter = (req, res, next) => {
    if(req.params.name == 'admin' && req.params.pwd == 'admin'){
        next()
    } else {
        next('用户名密码不正确')
    }
    
}

app.get('/:name/:pwd', filter, (req, res) => {
    res.send('ok')
}).listen(88)
```

这里写了一个filter方法，有一个next参数。在路由的时候，把filter作为一个参数，则就可以先执行filter函数，然后执行路由的逻辑。

如果想要全局使用的话，就直接使用use方法即可。

```
app.use(filter);
```

## 文件上传

前面说到的body-parser不支持文件上传，那么使用multer则可以实现。

# 操作数据库

node一般会使用mongo和mysql，使用下面这个例子即可：

## 操作 MongoDB

官方 api `http://mongodb.github.io/node-mongodb-native/`

```
var mongodb = require('mongodb');
var MongoClient = mongodb.MongoClient;
var db;

MongoClient.connect("mongodb://localhost:27017/test1705candel", function(err, database) {
  if(err) throw err;

  db = database;
});

module.exports = {
    insert: function(_collection, _data, _callback){
        var i = db.collection(_collection).insert(_data).then(function(result){
            _callback(result);
        });
    },
    select: function(_collection, _condition, _callback){
        var i = db.collection(_collection).find(_condition || {}).toArray(function(error, dataset){
            _callback({status: true, data: dataset});
        })
    }
}
```

## 操作 MySql

```
var mysql = require('mysql');

//创建连接池
var pool  = mysql.createPool({
  host     : 'localhost',
  user     : 'root',
  password : 'root',
  port: 3306,
  database: '1000phone',
  multipleStatements: true
});


module.exports = {
    select: function(tsql, callback){
        pool.query(tsql, function(error, rows){
      if(rows.length > 1){
        callback({rowsCount: rows[1][0]['rowsCount'], data: rows[0]});
      } else {
        callback(rows);
      }
        })
    }
}
```

# session

Session 是一种记录客户状态的机制，不同的是 Cookie 保存在客户端浏览器中，而 Session 保存在服务器上的进程中。

客户端浏览器访问服务器的时候，服务器把客户端信息以某种形式记录在服务器上，这就是 Session。客户端浏览器再次访问时只需要从该 Session 中查找该客户的状态就可以了。

如果说 Cookie 机制是通过检查客户身上的“通行证”来确定客户身份的话，那么 Session 机制就是通过检查服务器上的“客户明细表”来确认客户身份。

Session 相当于程序在服务器上建立的一份客户档案，客户来访的时候只需要查询客户档案表就可以了。

Session 不能跨域。

node操作session和cookie也很简单，也是通过中间件的形式。

```
const express = require('express')
const path = require('path')
const app = express();

const bodyParser = require('body-parser');

const cp = require('cookie-parser');
const session = require('express-session');

app.use(cp());
app.use(session({
    secret: '12345',//用来对session数据进行加密的字符串.这个属性值为必须指定的属性
    name: 'testapp',   //这里的name值得是cookie的name，默认cookie的name是：connect.sid
    cookie: {maxAge: 5000 },  //设置maxAge是5000ms，即5s后session和相应的cookie失效过期
    resave: false,
    saveUninitialized: true,    
}))
app.use(bodyParser.urlencoded({extended: false}));
app.use(express.static(path.join(__dirname, '/')));

app.get('/setsession', (request, response) => {
    request.session.user = {username: 'admin'};
    response.send('set session success');
})

app.get('/getsession', (request, response) => {
    response.send(request.session.user);
})

app.get('/delsession', (request, response) => {
    delete reqeust.session.user;
    response.send(request.session.user);
})

app.listen(88)
```



# Token

Token的特点

- 随机性
- 不可预测性
- 时效性
- 无状态、可扩展
- 跨域

基于Token的身份验证场景

1. 客户端使用用户名和密码请求登录
2. 服务端收到请求，验证登录是否成功
3. 验证成功后，服务端会返回一个 Token 给客户端，反之，返回身份验证失败的信息
4. 客户端收到 Token 后把 Token 用一种方式(cookie/localstorage/sessionstorage/其他)存储起来
5. 客户端每次发起请求时都选哦将 Token 发给服务端
6. 服务端收到请求后，验证Token的合法性，合法就返回客户端所需数据，反之，返回验证失败的信息

Token 身份验证实现 —— jsonwebtoken

先安装第三方模块 jsonwebtoken `npm install jsonwebtoken`

```
const express = require('express')
const path = require('path')
const app = express();
const bodyParser = require('body-parser');
const jwt = require('jsonwebtoken');

app.use(bodyParser.urlencoded({extended: false}));
app.use(express.static(path.join(__dirname, '/')));

app.all('*', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Content-Type,Content-Length, Auth, Accept,X-Requested-With");
    res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By",' 3.2.1')
    if(req.method=="OPTIONS") {
          res.sendStatus(200);/*让options请求快速返回*/
    } else{
          next();
    }
});


app.get('/createtoken', (request, response) => {
    //要生成 token 的主题信息
    let user = {
        username: 'admin',
    }
    //这是加密的 key（密钥）
    let secret = 'dktoken';
    //生成 Token
    let token = jwt.sign(user, secret, {
        'expiresIn': 60*60*24 // 设置过期时间, 24 小时
    })      
    response.send({status: true, token});
})

app.post('/verifytoken', (request, response) => {
    //这是加密的 key（密钥），和生成 token 时的必须一样
    let secret = 'dktoken';
    let token = request.headers['auth'];
    if(!token){
        response.send({status: false, message: 'token不能为空'});
    }
    jwt.verify(token, secret, (error, result) => {
        if(error){
            response.send({status: false});
        } else {
            response.send({status: true, data: result});
        }
    })
})

app.listen(88)
```

# web socket

HTTP 协议可以总结几个特点：

- 一次性的、无状态的短连接：客户端发起请求、服务端响应、结束。
- 被动性响应：只有当客户端请求时才被执行，给予响应，不能主动向客户端发起响应。
- 信息安全性：得在服务器添加 SSL 证书，访问时用 HTTPS。
- 跨域：服务器默认不支持跨域，可在服务端设置支持跨域的代码或对应的配置。

TCP 协议可以总结几个特点：

- 有状态的长连接：客户端发起连接请求，服务端响应并建立连接，连接会一直保持直到一方主动断开。
- 主动性：建立起与客户端的连接后，服务端可主动向客户端发起调用。
- 信息安全性：同样可以使用 SSL 证书进行信息加密，访问时用 WSS 。
- 跨域：默认支持跨域。



安装第三方模块 ws：`npm install ws`

开启一个 WebSocket 的服务器，端口为 8080

```
var socketServer = require('ws').Server;
var wss = new socketServer({
    port: 8080
});
```

也可以利用 Express 来开启 WebSocket 的服务器

```
var app = require('express')();
var server = require('http').Server(app);

var socketServer = require('ws').Server;
var wss = new socketServer({server: server, port: 8080});
```

- 用 on 来进行事件监听
- connection：连接监听，当客户端连接到服务端时触发该事件
- close：连接断开监听，当客户端断开与服务器的连接时触发
- message：消息接受监听，当客户端向服务端发送信息时触发该事件
- send: 向客户端推送信息

soket.io 可以理解为对 WebSocket 的一种封装。好比前端的 jQuery 对原生 javascript 的封装。
soket.io 依靠事件驱动的模式，灵活的使用了自定义事件和调用事件来完成更多的场景，不必依赖过多的原生事件。

- 安装第三方模块 `npm install express socket.io`
- 开户 Socket 服务器，端口为 88

```
var express = require('express');
var app = express();
var http = require('http').Server(app);
var io = require('socket.io')(http);
http.listen(88);
```

- 用 on 来进行事件监听和定义事件
- connection：监听客户端连接,回调函数会传递本次连接的socket
- emit：触发用客户端的事件

```
io.on('connection', function(client){
    //把当前登录的用户保存到对象 onlinePersons，并向所有在线的用户发起上线提示
    //serverLogin 为自定义事件，供客户端调用
    client.on('serverLogin', function(_person){
        var _personObj = JSON.parse(_person);
        onlinePersons[_personObj.id] = _personObj;
        //向所有在线的用户发起上线提示
        //触发客户端的 clientTips 事件
        //clientTips 为客户端的自定义事件
        io.emit('clientTips', JSON.stringify(onlinePersons));
    })

    //当监听到客户端有用户在移动，就向所有在线用户发起移动信息，触发客户端 clientMove 事件
    //serverMove 为自定义事件，供客户端调用
    client.on('serverMove', function(_person){
        var _personObj = JSON.parse(_person);
        onlinePersons[_personObj.id] = _personObj;
        console.log('serverLogin', onlinePersons);
        //clientTips 为客户端的自定义事件
        io.emit('clientMove', _person);
    });
})
```
# kafka-node

node也可以去读写kafka，而且很简单。只需要引入kafka的库即可。

```
yarn add kafka-node
```

具体api可以看文档：https://github.com/SOHU-Co/kafka-node

生产者

```
var kafka = require('..');
var Producer = kafka.Producer;
var KeyedMessage = kafka.KeyedMessage;
var Client = kafka.Client;
var client = new Client('localhost:2181');
var argv = require('optimist').argv;
var topic = argv.topic || 'topic1';
var p = argv.p || 0;
var a = argv.a || 0;
var producer = new Producer(client, { requireAcks: 1 });

producer.on('ready', function () {
  var message = 'a message';
  var keyedMessage = new KeyedMessage('keyed', 'a keyed message');

  producer.send([
    { topic: topic, partition: p, messages: [message, keyedMessage], attributes: a }
  ], function (err, result) {
    console.log(err || result);
    process.exit();
  });
});

producer.on('error', function (err) {
  console.log('error', err);
});
```

消费者

```
'use strict';

var kafka = require('..');
var Consumer = kafka.Consumer;
var Offset = kafka.Offset;
var Client = kafka.Client;
var argv = require('optimist').argv;
var topic = argv.topic || 'topic1';

var client = new Client('localhost:2181');
var topics = [{ topic: topic, partition: 1 }, { topic: topic, partition: 0 }];
var options = { autoCommit: false, fetchMaxWaitMs: 1000, fetchMaxBytes: 1024 * 1024 };

var consumer = new Consumer(client, topics, options);
var offset = new Offset(client);

consumer.on('message', function (message) {
  console.log(message);
});

consumer.on('error', function (err) {
  console.log('error', err);
});

/*
* If consumer get `offsetOutOfRange` event, fetch data from the smallest(oldest) offset
*/
consumer.on('offsetOutOfRange', function (topic) {
  topic.maxNum = 2;
  offset.fetch([topic], function (err, offsets) {
    if (err) {
      return console.error(err);
    }
    var min = Math.min.apply(null, offsets[topic.topic][topic.partition]);
    consumer.setOffset(topic.topic, topic.partition, min);
  });
});
```



# Node单元测试

以function为最小单位，验证特定情况下的input和output是否正确。

- 防止改A坏B，避免不能跑的代码比能跑的还多。
- 明确指出问题所在、告知正确的行为是什么，减少debug的时间。

对于node来说，单元测试也很容易做。

测试主要分为两种，TDD和BDD。

## TDD VS. BDD

比较TDD 与BDD 的差异。

|      | TDD                                                          | BDD                                                          |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 全名 | 测试驱动开发Test-Driven Development                          | 行为驱动开发Behavior Driven Development                      |
| 定义 | 在开发前先撰写测试程式，以确保程式码品质与符合验收规格。     | TDD的进化版。除了实作前先写测试外，还要写一份「可以执行的规格」。 |
| 特性 | 从测试去思考程式如何实作。强调小步前进、快速且持续回馈、拥抱变化、重视沟通、满足需求。 | 从用户的需求出发，强调系统行为。使用自然语言描述测试案例，以减少使用者和工程师的沟通成本。测试后的输出结果可以直接做为文件阅读。 |

从代码层面来看：

TDD

```
suite('Array', ()=>{
    setup(()={
        
    });
    
    test('equal -1 when index beyond array length', ()=>{
       assert.equal(-1, [1,2,3].indexOf(4)); 
    });
})
```

BDD

```
describe('Array', function() {
  before(function() {
  });

  it('should return -1 when no such index', function() {
    [1,2,3].indexOf(4).should.equal(-1);
  });
});
```

对比了这两种类型的语法之后，我选择了BDD。

## 测试框架实践

在node社区，比较成熟的是mocha。mocha本身是不提供断言库的，一般来说断言库比较常用的是chai。mocha和chai，合起来就被戏称为抹茶。

mocha一般需要全局安装，chai安装到项目目录下即可。

```
yarn global add mocha

yarn add chai
```

### mocha

#### 语法说明

- `describe()`：描述场景或圈出特定区块，例如：标明测试的功能或function。
- `it()`：撰写测试案例（Test Case）。
- `before()`：在所有测试开始前会执行的代码。
- `after()`：在所有测试结束后会执行的代码。
- `beforeEach()`：在每个Test Case 开始前执行的代码。
- `afterEach()`：在每个Test Case 结束后执行的代码。

#### 代码示例

```
describe('hooks', function() { 
  before(function() {
    
  });

  after(function() {
    
  });

  beforeEach(function() {
    
  });

  afterEach(function() {
    
  });

  
  it('should ...', function() {
    
  });
});

```

### chai

#### assert

assert(expression, message)：测试这个项目的expression是否为真，若为假则显示错误消息message。

#### Expect / Should

预期3 等于（===）2。这是使用可串连的操作符 来完成断言。这些可串联的有to、is、have 等。它很像英文，用很口语的方式做判断。

## 覆盖率

既然是给功能代码写单元测试，那就应该有个指标去衡量单元测试覆盖了哪些功能代码，这就是接下来要介绍的测试覆盖率。

在 Node.js 中，我们使用 istanbul 作为覆盖率统计的工具，istanbul 可以帮助我们统计到代码的语句覆盖率、分支覆盖率、函数覆盖率以及行覆盖率。

全局安装：

```
yarn global add istanbul
```

只需要使用istanbul cover就可以得到覆盖率。

```
istanbul cover simple.js
```

可以和mocha配合使用：

```
isbuntal cover _mocha test/simple-test.js
```

mocha 和 _mocha 是两个不同的命令，前者会新建一个进程执行测试，而后者是在当前进程（即 istanbul 所在的进程）执行测试，只有这样， istanbul 才会捕捉到覆盖率数据。其他测试框架也是如此，必须在同一个进程执行测试。

# 引入typescript

typescript其实就是加了类型的js。

所谓类型，就是约定变量的内存布局。js作为一个动态弱类型的语言，在开发大型项目的时候，不免可能出现问题，所以有类型的语言可以在编译期就能检测到错误，减少debug的时间。

## 安装

```
yarn global add typescript
```

## 新项目引入ts

现在新建文件`server.ts`：

```typescript
import * as http from 'http';

const server = http.createServer(function (req, res) {
  res.end('Hello, world');
});

server.listen(3000, function () {
  console.log('server is listening');
});
```

为了能执行此文件，需要通过 **tsc** 命令来编译该 TypeScript 源码：

```
tsc server.ts
```

如果没有什么意外的话，此时控制台会打印出以下的出错信息：

```
server.ts(1,23): error TS2307: Cannot find module 'http'.
```

这表示没有找到`http`这个模块定义（TyprScript 编译时是通过查找模块的 typings 声明文件来判断模块是否存在的，而不是根据真实的 js 文件，下文会详细解释），但是我们当前目录下还是生成了一个新的文件`server.js`，我们可以试着执行它：

```
node server.js
```

如果一切顺利，那么控制台将会打印出 **server is listening** 这样的信息，并且我们在浏览器中访问 [http://127.0.0.1:3000](http://127.0.0.1:3000/)时也能看到正确的结果：**Hello, world**

现在再回过头来看看刚才的编译错误信息。由于这是一个 Node.js 项目，typescript 语言中并没有定义`http`这个模块，所以我们需要安装 Node.js 运行环境的声明文件：

```
yarn global add @types/node 
```

安装完毕之后，再重复上文的编译过程，此时 **tsc** 不再报错了。

大多数时候，为了方便我们可以直接使用 **ts-node** 命令执行 TypeScript 源文件而不需要预先编译。首先执行以下命令安装 **ts-node**：

```
yarn global add -g ts-node
```

然后使用 **ts-node** 命令执行即可：

```
ts-node --no-cache server.ts
```

说明：使用 **ts-node** 执行 TypeScript 程序时，为了提高编译速度，默认会缓存未修改过的 **.ts** 文件，但有时候会导致一些 Bug，所以建议启动时加上 `--no-cache` 参数。

### tsconfig.json 配置文件

每个 TypeScript 项目都需要一个 **tsconfig.json** 文件来指定相关的配置，比如告诉 TypeScript 编译器要将代码转换成 ES5 还是 ES6 代码等。

可以使用tsc命令生成。

```
tsc --init
```

### 使用第三方模块

一般情况下在 TypeScript 中是不能”*直接*“使用 npm 上的模块的，比如我们要使用 express 模块，先执行以下命令安装：

```
yarn add express
```

然后新建文件 `server.ts` :

```typescript
import * as express from 'express';

const app = express();
app.get('/', function (req, res) {
  res.end('hello, world');
})

app.listen(3000, function () {
  console.log('server is listening');
});
```

然后使用以下命令执行：

```
ts-node server.ts
```

如果不出意外，我们将会看到这样的报错信息：

```
src/server.ts(1,26): error TS7016: Could not find a declaration file for module 'express'.
```

报错的信息表明没有找到`express`模块的声明文件。由于 TypeScript 项目最终会编译成 JavaScript 代码执行，当我们在 TypeScript 源码中引入这些被编译成 JavaScript 的模块时，它需要相应的声明文件（**.d.ts**文件）来知道该模块类型信息，这些声明文件可以通过设置`tsconfig.json`中的`declaration: true`来自动生成。而那些不是使用 TypeScript 编写的模块，也可以通过手动编写声明文件来兼容 TypeScript。

当遇到缺少模块声明文件的情况，开发者可以尝试通过 yarn addl @types/xxx 来安装模块声明文件即可。

现在我们尝试执行以下命令安装 **express** 模块的声明文件：

```
yarn add @types/express
```

没有意外，果然能成功安装。现在再通过 **ts-node** 来执行的时候，发现已经没有报错了。

### 单元测试

直接使用mocha和chai，进行ts的测试。

## 旧项目迁移

通常来说这个过程包括了以下步骤：

- 添加 `tsconfig.json`
- 将你的源代码文件扩展名从 `.js` 改成 `.ts`。使用 `any` 来开始*抑止*错误。
- 使用 TypeScript 来编写新的代码并且尽可能少地使用 `any`。
- 返回到旧代码里并且开始加入类型标注和解决发现的 bugs。
- 为第三方 JavaScript 代码使用环境定义。