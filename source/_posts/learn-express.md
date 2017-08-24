---
title: Express 4.0
date: 2017-08-24 18:42:33
categories:
  - 学无止境
tags:
  - Node.js
  - Framework
---

## 概述

Express 是一个基于 Node.js 平台的极简、灵活的 web 应用开发框架，它提供一系列强大的特性，帮助你创建各种 Web 和移动设备应用。
Express 具有丰富的 HTTP 快捷方法和任意排列组合的 Connect 中间件，让你创建健壮、友好的 API 变得既快速又简单。
同时，Express 不对 Node.js 已有的特性进行二次抽象，我们只是在它之上扩展了 Web 应用所需的基本功能，因此不会出现由于过多的抽象和包装导致的性能损耗问题。

<!-- more -->

## 开始使用

Express 上手非常的简单，使用 Express 需要 Node.js，这里假定已经安装好了 Node.js，我们直接开始写一个 Hello World。

### Hello World

首先新建一个项目目录并进行一些初始化操作。

```bash
$ mkdir hello-world
$ cd hello-world
$ npm init
```

执行完上述操作后，hello-world 目录下会生成一个 package.json 文件，包含一些项目相关信息。

接下来直接通过 npm 安装 Express 就可以了。

```bash
$ npm install express
```

执行完上面的命令后，在项目的根目录下新建一个 app.js 文件，然后将下面的代码复制进去：

```js
var express = require('express');
var app = express();

app.get('/', function(req, res){
  res.send('<h1>Hello World!</h1>');
});

var server = app.listen(8080, function(){
  var host = server.address().address;
  var port = server.address().port;

  console.log('Example app listening at http://%s:%s', host, port);
});
```
上面的代码启动一个服务并监听从 8080 端口进入的所有连接请求，他将对所有 (/) URL 或 路由 返回 “Hello World!” 字符串。对于其他所有路径全部返回 **404 Not Found**。

我们可以通过下面的命令启动应用，然后通过浏览器访问`http://localhost:8080`，网页上会显示一个 Hello World。
```bash
$ node app.js
```

### 脚手架

我们可以通过脚手架工具快速创建一个 Express 应用。通过下面的命令全局安装脚手架工具。

```bash
$ npm install express-generator -g
```

安装完成后，我们可以通过下面命令创建一个名为 **my-first-app** 的应用。

```bash
$ express my-first-app -e
```

`-e` 参数表明使用的 ejs 作为模板引擎，默认使用的 jade。我们可以通过 `express -h` 查看支持的参数。

应用创建完成后，我们需要进入到该目录下安装所有的依赖。

```bash
$ cd my-first-app
$ npm install
```

然后就可以启动应用了，设置 `DEBUG` 环境变量为当前应用名称可以以调试模式启动应用。

```bash
$ DEBUG=my-first-app npm start
```

**注意**：上面的命令是 masOS 或者 Linux 平台，如果是 Windows 平台，需要使用下面的命令。

```bash
$ set DEBUG=my-first-app & npm start
```

应用启动后，在浏览器中打开 `http://localhost:3000/` 网址就可以看到这个应用了。

通过 Express 应用生成器创建的应用一般都有如下目录结构：

```
.
├── app.js
├── bin
│   └── www
├── package.json
├── public
│   ├── images
│   ├── javascripts
│   └── stylesheets
│       └── style.css
├── routes
│   ├── index.js
│   └── users.js
└── views
    ├── error.jade
    ├── index.jade
    └── layout.jade

7 directories, 9 files
```

## 内部原理

Express 框架的核心是对 Node.js 内置 http 模块的一个封装。原生 http 模块启动一个服务的代码如下：

```js
var http = require('http');
var server = http.createServer(function(request, response) {
  // Handle request.
});
server.listen(8080);
```

上面代码的关键部分是 http 模块的 `createServer` 方法，该方法接受一个回调函数，当有客户端的请求抵达服务器时，会触发该函数。该回调函数有两个参数，分别对应 `request` 和 `response` 对象。

Express 其实就是对 http 模块的一个封装，上面的代码使用 Express 改写如下：

```js
var http = require('http');
var express = require('express');

var app = express();
var server = http.createServer(app);

server.listen(8080);
```

通过上述代码不难看出，express 拦截了 http 请求，当请求到达时，交由 app 进行处理。那么，这个 “app” 是什么呢?`app` 其实就是一个 `Function`，并且接受 `request` 和 `response` 对象以及一个叫做 `next` 的参数。源码如下：

```js
function createApplication() {
  var app = function(req, res, next) {
    app.handle(req, res, next);
  };

  mixin(app, EventEmitter.prototype, false);
  mixin(app, proto, false);

  // expose the prototype that will get set on requests
  app.request = Object.create(req, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  // expose the prototype that will get set on responses
  app.response = Object.create(res, {
    app: { configurable: true, enumerable: true, writable: true, value: app }
  })

  app.init();
  return app;
}
```

当服务器收到客户端的请求时，调用 `app` 方法，在 `app` 方法内部会执行 `app.handle` 那么 `app.handle` 中会做些什么呢？还是继续看源码：

```js
app.handle = function handle(req, res, callback) {
  var router = this._router;

  // final handler
  var done = callback || finalhandler(req, res, {
    env: this.get('env'),
    onerror: logerror.bind(this)
  });

  // no routes
  if (!router) {
    debug('no routes defined on app');
    done();
    return;
  }

  router.handle(req, res, done);
};
```

从上面的代码可以看出 `app.handle` 会将处理交由 `router` 进行处理，如果 `router` 不存在，则执行回调函数或者 `handler`。其中 `router` 就是路由容器，
负责分发请求，依次执行相应的处理函数。

看到这里就能大致推测出 Express 的内部工作机制了：Express 将 app 方法绑定为 http 请求的处理函数，当请求到达时触发该方法。
在该方法的内部，由路由容器对请求进行分发，交由相应的处理函数处理，而处理函数显然需要提前进行注册并且是被顺序调用的。这里的“处理函数“在 Express 中被成为中间件。

## 中间件

中间件说白了就像是流水线上一个工人，对产品（http 请求）进行某项特定的加工。它最大的特点就是顺序执行，Express 运行的过程其实就是顺序调用一系列的中间件。

每个中间件都会从 Express 实例对象中获取到三个参数：request 对象、response 对象和 next 对象，分别对应 http 请求、http 响应以及 负责调用下一个中间件的 next 方法。

Express 通过 **use** 方法注册中间件，该方法返回一个函数。下面是一个简单的示例：

```js
var express = require('express');

var app = express();

app.use(function(req, res, next) {
  console.log(req.method + ':' + request.url);
  next();
});

app.user(function(req, res, next) {
  res.end('<h1>Hello World!</h1');
});

app.listen(8080);
```

上面的代码使用了 `app.use` 方法注册了两个中间件，在收到 http 请求后会顺序调用这两个方法。首先在控制台输出一行信息，
然后调用 `next` 方法，执行下一个方法，响应 http 请求。

`use` 方法内部可以对访问路径进行判断，据此就能实现简单的路由，根据相应的请求路径，返回相应的请求结果。例如：

```js
var express = require('express');

var app = express();

app.user(function(req, res, next) {
  if (req.url === '/') {
    res.end('<h1>Hello World!</h1');
  } else {
    res.status(404).end();
  }
});

app.listen(8080);
```

上面的代码通过 `request.url` 属性，判断请求的网址，返回不同的内容。

## 路由

路由定义了匹配某个url的处理函数，由一个 url、http 请求（GET、POST等）和若干个处理函数组成，下面是一个基本的路由：

```js
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  res.send('hello world');
});
```

### 路由方法

路由方法来源于 http 请求，如：get、post、put等。其中 `app.all()` 是一个特殊的路由方法，没有任何 http 方法与其对应，
它的作用是对于一个路径上的所有请求加载中间件。下面是一些路由的实例：

```js
app.get('/', function (req, res) {
  res.send('GET request to the homepage');
});

app.post('/', function (req, res) {
  res.send('POST request to the homepage');
});

app.all('/secret', function (req, res, next) {
  console.log('Accessing the secret section ...');
  next(); // pass control to the next handler
});
```

### 路由路径

路由路径匹配 htpp 请求的路径，可以是字符串、字符串模式或者是正则表达式。下面是一些路由路径的示例：

```js
// 匹配根路径的请求
app.get('/', function (req, res) {
  res.send('root');
});

// 匹配 acd 和 abcd
app.get('/ab?cd', function(req, res) {
  res.send('ab?cd');
});

// 匹配 butterfly、dragonfly，不匹配 butterflyman、dragonfly man等
app.get(/.*fly$/, function(req, res) {
  res.send('/.*fly$/');
});
```

### 路由句柄

可以为一个请求提供多个处理函数，其行为类似中间件。路由句柄有多种形式，可以是一个函数、一个函数数组，或者是两者混合。如下所示：

```js
var fun1 = function (req, res, next) {
  console.log('fun1');
  next();
}

var fun2 = function (req, res, next) {
  console.log('fun2');
  next();
}

app.get('/example/d', [fun1, fun2], function (req, res, next) {
  console.log('response will be sent by the next function ...');
  next();
}, function (req, res) {
  res.send('Hello from D!');
});
```

使用 `app.route()` 可以创建链式的路由句柄。由于路径在一个地方指定，这样做有助于创建模块化的路由，而且减少了代码冗余和拼写错误。下面是一个简单的示例：

```js
app.route('/user')
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

### 路由中间件

Express.Router 是一个构造函数，用来创建一个模块化、可挂载的路由句柄，它好像小型的 Express 应用程序一样，有自己的 use、get、param 和 route 方法。
下面是一个路由中间件的基本例子。

在项目目录下新建一个 `login.js` 文件，文件的内容如下：

```js
var express = require('express');
var router = express.Router();

// 该路由使用的中间件
router.use(function timeLog(req, res, next) {
  console.log('Time: ', Date.now());
  next();
});

router.post('/', function(req, res) {
  // Login...
});

module.exports = router;
```

然后在 `app.js` 中加载路由模块：

```js
var login = require('./login');

app.user('/login', login);
```

这样应用就可以处理发到 `/login` 上的请求，并且调用为该路由指定的 `timeLog` 中间件。

## 模板引擎

在 Express 中使用模板引擎，需要在应用中进行如下设置才能让 Express 渲染模板引擎：

```js
var express = require('express');
var path = require('path');

var app = express();

// 设置模板文件目录
app.set('views', path.join(__dirname, 'views'));

// 设置模板引擎
app.set('view engine', 'jade');
```

模板引擎设置完成后，在 views 目录下创建一个名为 index.ejs 的文件，内容如下：

```jade
html
  head
    title!= title
  body
    h1!= message
```

然后创建一个路由渲染 index.jade 文件。如果没有设置 view engine，您需要指明视图文件的后缀，否则就会遗漏它。

```js
app.get('/', function (req, res) {
  res.render('index', { title: 'Hey', message: 'Hello there!'});
});
```

完成上述操作后，访问主页就可以看到 index.jade 被渲染为 HTML。

## 常用对象及其属性和方法

### Express

#### 静态方法

Express.static(root[, options])

&emsp;&emsp;托管 Express 应用内的静态资源。root 参数指的是静态资源文件所在的根目录；options 可选的参数对象，
包含一些缓存相关的字段：etag、lastModified、maxAge，以及一个设置 HTTP 相应头的函数等。

#### 实例方法

app.listen(port[, hostname][, backlog][, callback])

&emsp;&emsp;监听指定的主机和端口号

app.use([path, ]function[, function...])

&emsp;&emsp;将中间件挂载到指定的路径，默认是根路径

app.all(path, callback[, callback...])

&emsp;&emsp;匹配所有的请求方法

app.get(path, callback[, callback...])

&emsp;&emsp;匹配浏览器**get**请求

app.post(path, callback[, callback...])

&emsp;&emsp;匹配浏览器**post**请求

app.render(view[, locals][, callback])

&emsp;&emsp;通过回调函数渲染模板页面

app.route(path)

&emsp;&emsp;创建链式的路由句柄

### Request

#### 实例属性

req.app

&emsp;&emsp;Express app 实例

req.baseUrl

&emsp;&emsp;路由的挂载的路径

req.body

&emsp;&emsp;请求体

req.cookies

&emsp;&emsp;cookies

req.query

&emsp;&emsp;get请求参数对象

#### 实例方法

req.get(field)

&emsp;&emsp;获取 Http 请求头参数

req.param(name[, defualtValue])

&emsp;&emsp;读取请求的参数，会从url和body中查找

### Response

#### 实例属性

res.app

&emsp;&emsp;Express app 实例

res.locals

&emsp;&emsp;本次请求中的本地变量，不同于app.locals。

#### 实例方法

res.send([body])

&emsp;&emsp;返回 Http 响应

res.sendStatus(statusCode)

&emsp;&emsp;返回状态吗

res.json([body])

&emsp;&emsp;返回JSON数据

res.append(field[, value])

&emsp;&emsp;向 Http 响应头中添加变量

res.render(view [, locals] [, callback])

&emsp;&emsp;渲染并返回模板页面

res.cookie(name, value[, options])

&emsp;&emsp;设置cookie

res.end([data][, encoding])

&emsp;&emsp;结束相应

res.redirect([status, ]path)

&emsp;&emsp;重定向

## 总结

这几天对 Express 的学习和运用，给我最大的感觉就是 Express 的极简和灵活以及那种流水线的工作模式，它完全就是一个由路由和中间件构成的 Web 框架。
基于 Express 可以很快速的构建项目，各种功能按需添加，即插即用。结构十分清晰，各个模块之间解耦、职责分明。不过，也正是因为 Express 的灵活，如果
在使用的过程中没有清晰的思路和好的规划，仍然会导致代码杂乱无章，这里插一点东西，那里插一点东西。

对于 Express 我感觉用来构建一个循序渐进的项目是最适合不过了，开始的时候只需要少量的工作就可以让项目跑起来。接着，在后续的开发过程中，可以通过
路由和中间件不断的添加新的功能，逐步的完成整个项目的定制。而且，还可以对 Express 进行一些基本的封装，把一些较为通用的功能预先集成进去，然后在此
基础上进行各类项目的定制。

总而言之，Express 可以很快速的构建小型项目，也可以完成大型项目的深度定制，可谓是文武双全，适应性及其广泛。只是，这样的灵活性也意味着功能较为简单，
仍然有一些较为通用的处理需要去做，对于企业级开发，也许抽象层次更高，功能更加丰富的框架更为适合。

