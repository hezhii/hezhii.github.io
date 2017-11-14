---
title: ThinkJS 3.0
date: 2017-08-30 17:54:51
categories:
  - 技术
tags:
  - Node.js
  - ThinkJS
---

## 概述

ThinkJS 是一款面向未来开发的 Node.js 框架，底层基于 Koa 2.x 实现，内核小巧、性能优异，内置自动编译、自动更新机制，使用更优雅的 async/await 处理异步问题，兼容中间件并提供了扩展和适配器等插件方式。针对企业级应用开发，使开发变得更加简单、高效。

<!-- more -->

## 个人见解

这一节内容是我在学习完 ThinkJS 基本用法后加上的，主要谈谈关于 ThinkJS 的一些个人见解。

这里不得不引用官方文档中的架构图。

![ThinkJS 架构图](/images/thinkjs_framework.jpg)

从架构图我们可以清晰的看到 ThinkJS 底层是基于 Koa 2.x，主要分为中间件、扩展和适配器三个核心模块。其中中间件主要用来处理用户的请求，当用户向服务器发出请求时，顺序执行中间件，最终返回用户期望的结果。扩展和适配器往往搭配使用，提供一些诸如：Session、模板引擎、数据库模型等扩展功能。

在我看来，ThinkJS 其实就是对一个应用如何构建进行了划分，将一个应用分成了各个模块（中间件、扩展），各个模块各司其职，处理自己的逻辑。我们只需要将相应的逻辑写在相应的地方，然后进行相应的一些配置。在应用启动时，ThinkJS 根据配置文件装载相应的模块，当用户请求到达时，按照配置将处理权交给相应的模块，完成请求的响应。整个应用都是可配置的，大大增强了灵活性。

ThinkJS 的工作模式按照官方文档上的说法主要分为两个部分，一个是系统服务启动，另一个是用户请求处理。这一点我感觉没有什么特殊的地方，我觉得所有的后台应用都是这两个部分，都是需要启动服务监听端口，然后等待客户端的请求到来，处理请求并响应。只不过在这两个阶段进行的具体工作有些许差异罢了。ThinkJS 在系统服务启动阶段，除了启动服务监听端口外，最主要的工作就是装配。按照 ThinkJS 的架构，我们需要什么功能或者需要执行什么操作，只需要将相应的代码写在相应的地方，然后添加响应的配置。为什么我们这样做了就可以实现我们需要的功能？必然是 ThinkJS 在启动时进行了装配。

我们以在 ThinkJS 中添加模板页面功能为例，来梳理一下 ThinkJS 扩展和适配器机制以及系统服务启动。扩展功能在我理解就是在相应的环境（全局、上下文或者控制器）下添加了一个对象，我们可以直接使用它提供的一些方法，来实现相应的功能。按照 ThinkJS 的架构，我们需要先实现这个对象（扩展）或者直接使用别人的，这个对象至少有一个 `render` 方法，能将模板文件渲染成 html 文件。然后，在配置文件 `src/config/extend.js` 中我们需要引入这个对象并添加这个对象的配置。接着，我们还需要在适配器中添加一个具体的模板引擎配置。完成这几步后，我们就可以直接在控制器中通过 `this.render()` 渲染模板页面了。那么为什么经过这样的几个步骤后，就可以使用模板页面了呢？我们可以在控制器中直接使用 `this.render()`，肯定是 ThinkJS 在控制器对象上添加了这个方法，添加这个方法的依据应该就是我们在扩展配置文件中添加了 `view` 配置，而我们执行这个方法具体的实现则是根据适配器中的配置来的。扩展相当于定义了一个功能接口，而适配器中进行具体的实现。服务启动后，读取扩展配置，装配相应的对象（扩展功能）。在具体使用该功能时，根据适配器中的配置，不同的类型有不同的实现。

后台应用的核心主要还是处理用户请求，处理用户请求主要通过各种中间件来完成。通过中间件，将处理请求的流程截取成了多个部分，各个中间件完成各自的功能，整个处理过程类似于一个流水线生产。扩展也是为这一过程服务的，定义的扩展功能可以在请求处理的过程中调用，去完成更加具体的工作。我觉得扩展是作为中间件的一部分存在的，去实现中间件中的某一个具体功能。请求处理流程大致如下图所示。

![ThinkJS 请求处理流程](/images/thinkjs_workflow.png)

各种中间件分别对 `request` 进行加工，然后由路由分配给指定的控制器去处理，在控制器处理之前 ThinkJS 中会先经过一个叫做 **Logic** 的中间件，对数据进行验证、过滤。控制器在处理请求时，可以随时调用扩展功能去实现某个具体的操作，接口都是统一的，由适配器去完成各种不同的实现。最后，将响应结果返回给客户端。

那么，使用 ThinkJS 能带来什么好处呢？首先，应用如何启动我们不需要花费太多的精力，通过配置文件以及 worker 即可完成。其次，对于请求处理，通过中间件、扩展和适配器以及相应的配置，我们可以很快速的添加相应的处理逻辑。同时，还可以保证一个良好的代码结构。最后，ThinkJS 提供了很多通用的机制，去解决一些通用问题，例如：日志、数据模型、Cookie、错误处理等。

总之，就目前的了解来看，ThinkJS 最优秀的地方在于它很简单，可以快速的上手。只要按照它的架构，就能很轻松的构建出整个应用。框架本身并不算复杂，但通过一些内部机制又可以增添很多丰富的功能。内置的一些功能模块也解决的了很多通用的问题。更具体的优缺点还得待具体使用后才知道。

## 开始使用

ThinkJS提供了脚手架工具，可以帮助我们快速创建一个项目，要求 Node.js 的版本大于 `6.x`。

```bash
$ npm install -g think-cli
$ thinkjs new <project-name>
```

执行完上述命令后，就可以创建出一个项目骨架。进入该项目，安装依赖后可直接启动。

```bash
$ cd <project-name>
$ npm install
$ npm start
```

启动后，访问 `http://127.0.0.1:8360` 可以看到初始化页面。

通过脚手架构建的项目目录结构如下：

```
.
|--- development.js   //开发环境下的入口文件
|--- nginx.conf  //nginx 配置文件
|--- package.json
|--- pm2.json //pm2 配置文件
|--- production.js //生产环境下的入口文件
|--- README.md
|--- src
| |--- bootstrap  //启动自动执行目录
| | |--- master.js //Master 进程下自动执行
| | |--- worker.js //Worker 进程下自动执行
| |--- config  //配置文件目录
| | |--- adapter.js  // adapter 配置文件
| | |--- config.js  // 默认配置文件
| | |--- config.production.js  //生产环境下的默认配置文件，和 config.js 合并
| | |--- extend.js  //extend 配置文件
| | |--- middleware.js //middleware 配置文件
| | |--- router.js //自定义路由配置文件
| |--- controller  //控制器目录
| | |--- base.js
| | |--- index.js
| |--- logic //logic 目录
| | |--- index.js
| |--- model //模型目录
| | |--- index.js
|--- view  //模板目录
| |--- index_index.html
|--- www
| |--- static  //静态资源目录
| | |--- css
| | |--- img
| | |--- js
```

## 基本用法

关于ThinkJS的基本用法主要是参考了官方文档的内容，根据文档说明依次尝试了相应的模块并记录下核心的内容。

### 配置

所有的配置文件都放在 `src/config/` 目录下，根据不同的功能划分为不同的配置文件。其中，通用配置和适配器配置支持在不同的环境下配置不同的值，只需要将配置文件命令为 `[name].[env].js` 这种形式即可。例如：`config.production.js` 表明这是生产环境中的配置，该文件中的 key 会覆盖 `config.js` 中同名的 key。运行环境在实例化 Application 对象时通过 `env` 参数指定，如下所示。

```js
const instance = new Application({
  ROOT_PATH: __dirname,
  proxy: true, // use proxy
  env: 'production'
});
```

在程序运行时可以获取和动态设置配置，获取配置可以通过上下文、controller 和全局的 think 对象，动态设置只能通过 think。下面是获取和设置配置的例子。

```js
// HTTP 服务启动前执行
think.beforeStartServer(async () => {
  console.log(`全局环境下读取端口号配置: ${think.config('port')}`);
  console.log('动态配置端口号为: 8088');
  think.config('port', '8088'); // 动态配置端口号。
});
```

通过 `DEBUG=think-loader-config-* npm start` 命令来启动项目可以在控制台查看配置文件的详细加载情况。

### 上下文

> Context 是 Koa 中处理用户请求中的一个对象，贯穿整个请求生命周期。一般在 middleware、controller、logic 中使用，简称为 ctx。

Context 中包含了许多信息以及一些便捷操作，下面列举部分，详情见 [ThinkJS 文档](https://thinkjs.org/zh-cn/doc/3.0/context.html)

- 可以通过上下文获取 request 和 response 对象。
- 可以过ctx.state在中间件中传递信息，而不是直接添加到 ctx 上。
- ctx.throw 抛出包含 `.status` 属性的异常，默认状态码为 500。
- ctx.json 输出 JSON 格式数据。
- ctx.success 和 ctx.fail 输出成功和失败数据。
- ctx.confg 读取配置。

### 中间件

ThinkJS 中通过统一的配置文件来管理中间件，配置文件为 `src/config/middleware.js`。中间件分为框架内置中间件和项目自定义中间件两种。内置的中间件可以通过字符串的方式直接引用，内置中间件有下面这些：

- meta 显示一些 meta 信息，如：发送 ThinkJS 的版本号，接口的处理时间等等。
- resource 处理静态资源，生产环境建议关闭，直接用 webserver 处理即可。
- trace 处理报错，开发环境将详细的报错信息显示处理，也可以自定义显示错误页面。
- payload 处理表单提交和文件上传，类似于 koa-bodyparser 等 middleware。
- router 路由解析，包含自定义路由解析。
- logic logic 调用，数据校验。
- controller controller 和 action 调用。

项目自定义的中间件定义在 `src/middleware` 目录下，然后也可以通过字符串的方式引用。下面是一个计算当前请求执行时间的中间件的例子。

首先在 `src/middleware` 目录下新建一个 `compute-request-time.js` 的文件，文件中写下如下代码：

```js
const defaultOptions = {
  consoleExecTime: false // 是否打印执行时间的配置
}
module.exports = (options = {}) => {
  // 合并传递进来的配置
  options = Object.assign({}, defaultOptions, options);
  return (ctx, next) => {
    if (!options.consoleExecTime) {
      return next(); // 如果不需要打印执行时间，直接调用后续执行逻辑
    }
    const startTime = Date.now();
    let err = null;
    // 调用 next 统计后续执行逻辑的所有时间
    return next().catch(e => {
      err = e; // 这里先将错误保存在一个错误对象上，方便统计出错情况下的执行时间
    }).then(() => {
      const endTime = Date.now();
      console.log(`request exec time: ${endTime - startTime}ms`);
      if (err) return Promise.reject(err); // 如果后续执行逻辑有错误，则将错误返回
    })
  }
}
```

接着在配置文件 `src/config/middleware.js` 中增加如下配置：

```js
module.exports = [{
  handle: 'compute-request-time',
  enable: think.env === 'development',  // 这个中间件只在开发环境下生效
  options: {
    consoleExecTime: true
  },
  match: '/'  // 请求的 URL 是 / 打头时这个中间件才启用
}];
```

参数及含义如下：

- handle 中间件的处理函数。
- enable 是否启用中间件。
- options 传递给中间件的参数，是一个对象，可以通过函数返回。
- match 匹配特定的规则后才执行该中间件，支持二种方式，一种是路径匹配，一种是自定义函数匹配。

**注意**：

1. 如果中间件没有定义在 `src/middleware/` 目录下，在配置文件中配置时需要引入。
2. 中间件的执行顺序是配置文件中的顺序。

### Logic

Logic 层在 Controller 之前执行，主要是完成一些过滤、数据校验、权限判断等一些逻辑性不是特别强但又和业务逻辑相关的工作，降低 Action 中代码的复杂程度。Logic 里的 Action 和控制器里的 Action 一一对应，系统在调用控制器里的 Action 之前会自动调用 Logic 里的 Action。

Logic 代码类似如下：

```js
module.exports = class extends think.Logic {
 __before() {
    // todo
 }
 indexAction() {
    // todo
 }
 __after() {
    // todo
 }
}
```

**要点**：

1. Logic 和 Controller 的文件名应该相同。
2. Logic 中进行数据校验时，会根据请求类型自动获取，也可以手动获取设置到数据的 `value` 属性上。

### 控制器

控制器负责处理用户请求的逻辑，每一个操作对应一个 Action。Controller 代码类似如下：

```js
const Base = require('./base.js');

module.exports = class extends Base {
  __before() {
    console.log('Before user action.');
  }

  indexAction() {
    this.body = 'hello world!';
  }

  profileAction() {
    this.body = 'User profile!';
  }

  __call() {
    //如果相应的Action不存在则调用改方法
    this.body = 'Not Found';
  }

  __after() {
    console.log('After user action.');
  }
};
```

访问 `/user` 或 `/user/index` 会执行 `indexAction`；访问 `/user/profile` 会执行 `profileAction`。

在 controller 目录下创建子目录并添加 controller 可以进行多级匹配。例如：在 controller 目录下创建一个 user 目录并添加一个名为 login.js 的控制器，当访问 `/user/login` 时，会执行 login.js 中的 `indexAction`。

Controller 里的处理顺序依次为 `__before`、`xxxAction`、`__after`，`return false` 可以提前结束请求。

### View

ThinkJS 3.0 没有内置 View 功能，需要通过 Extend 和 Adapter 中的配置来实现，具体如何配置参考[官网文档](https://thinkjs.org/zh-cn/doc/3.0/view.html#toc-9ad)。

配置了 Extend 和 Adapter 后，就可以在 Controller 里使用了：

```js
module.exports = class extends think.Controller {
  indexAction(){
    this.assign('title', 'thinkjs'); //给模板赋值

    /*
     * 渲染模板，实际上调用了 this.reader() 方法获取渲染后的内容，然后将内容赋值到 ctx.body上。
     */
    return this.display();
  }
}
```

在模板中可以使用 `assign` 方法添加的变量。同时，系统在渲染模板的时候，自动注入 controller、config、ctx 变量，以便于在模板里直接使用。

### 路由

上面说到当访问 `/user/profile` 时，会执行 user 控制器中的 `profileAction`，这一过程由路由来完成。ThinkJS 使用 `think-router` 中间件。

当 Controller 有子目录时，会优先匹配子 Controller，然后才匹配 action。

支持在 `src/config/router.js` 中配置自定义路由规则，路由规则为一个二维数组，如下所示：

```js
module.exports = [
  [/libs\/(.*)/i, '/libs/:1', 'get'],
  [/fonts\/(.*)/i, '/fonts/:1', 'get,post'],
];
```

每一条路由规则也为一个数组，数组里面的项分别对应为：match、pathname、method、options：

- match {String | RegExp} pathname 匹配规则，可以是字符串或者正则。如果是字符串，那么会通过 path-to-regexp 模块转为正则。
- pathname {String} 匹配后映射后的 pathname，后续会根据这个映射的 pathname 解析出对应的 controller、action。
- method {String} 该条路由规则支持的请求类型，默认为所有。多个请求类型中间用逗号隔开，如：get,post。
- options {Object} 额外的选项，如：跳转时指定 statusCode。

### 适配器

Adapter 是一类功能的不同实现，这些实现提供一套相同的接口，例如：多种数据库、模板引擎。Adapter 一般是不能独立使用的，而是配合对应的扩展一起使用。

Adapter 可以根据不同的运行环境设置不同的配置，例如：创建 `adapter.production.js` 文件指定生产环境下的适配器配置。

除了引入外部的 Adapter 外，项目内也可以创建 Adapter 来使用。Adapter 文件放在 `src/adapter/` 目录下（多模块项目放在 src/common/adapter/），如：`src/adapter/cache/xcache.js`，表示加了一个名为 xcache 的 cache Adapter 类型，然后该文件实现 cache 类型一样的接口即可。

### 扩展

扩展在 `src/config/extend.js` 文件中配置，格式为数组，如下所示：

```js
const view = require('think-view');

module.exports = [
  view //make application support view
];
```

支持自定义扩展，文件放在 `src/extend/` 目录下，下面是给 ctx 添加判断当前请求是不是手机访问的扩展的例子。

首先在 `src/extend` 目录下新建一个 `context.js` 的文件，文件中写下如下代码：

```js
module.exports = {
  get isMobile() {
    const userAgent = this.userAgent.toLowerCase();
    const mList = ['iphone', 'android'];
    return mList.some(item => userAgent.indexOf(item) > -1);
  }
}
```

这样就可以通过 `ctx.isMobile` 来判断是否是手机访问了。

### 异步/错误处理

这一部分主要利用了 ES2017 标准中引入的 async 函数。基本的用法与 async 函数的用法相同，没有太多的封装。

由于 Node.js 原生的异步方法都是 callback 方式，为了方便的将 callback 接口封装为 Promise 接口，ThinkJS 提供了 `think.promisify` 用来快速的封装，如：

```js
const fs = require('fs');
const readFile = think.promisify(fs.readFile, fs);

const parseFile = async (filepath) => {
  const content = await readFile(filepath, 'utf8'); // readFile 返回 Promise
  doSomethingWithContent();
}
```

但是，这个方法只能封装 `callback(err, data)` 形式的回调函数，否则只能手动封装。

错误处理除了 `try/catch` 和 `then/catch` 之外，还提供了一个 **trace** 中间件来统一处理运行时的异常。

## 模型/数据库

ThinkJS 默认没有提供模型功能，而是通过扩展的方式实现，对应的模块为 **think-model**。在 `src/config/extend.js`中添加如下配置即可：

```js
const model = require('think-model');

module.exports = [
  model(think.app) // 让框架支持模型的功能
]
```

添加了模型扩展后，就可以通过 `think.model`、`ctx.model`、`controller.model`等进行使用。

模型可以支持多种数据库，在 `scr/config/adapter.js` 中配置即可。如下所示：

```js
const mysql = require('think-model-mysql');
exports.model = {
  type: 'mysql',  // 默认使用的类型，调用时可以指定参数切换
  common: { // 通用配置
    logConnect: isDev,  // 是否打印数据库连接信息
    logSql: isDev,  // 是否打印 SQL 语句
    logger: msg => think.logger.info(msg) // 打印信息的 logger
  },
  mysql: {  // mysql 配置
    handle: mysql,
    database: '',
    prefix: 'think_',
    encoding: 'utf8',
    host: '127.0.0.1',
    port: '',
    user: 'root',
    password: 'root',
    dateStrings: true
  },
  sqlite: {  // sqlite 配置

  },
};
```

在适配器中添加相关配置后，在使用模型时指定相应的类型即可。例如使用 sqlite 的配置：`const user = think.model('user', 'sqlite');`

模型文件创建在 `src/model` 目录下，继承模型基类 `think.Model`，model 代码类似如下：

```js
// src/model/user.js
module.exports = class extends think.Model {
  getList() {
    return this.field('name').select();
  }
}
```

定义了模型后，通过 `const usreModel = think.model('user', 'sqlite')` 获取模型实例，然后就可以调用模型提供的方法 `usreModel.getList()`。

think.Model 基类提供了丰富的方法进行 CRUD 操作，我们可以对这些方法进行封装，例如上面的例子。具体有哪些方法详见[官网文档](https://thinkjs.org/zh-cn/doc/3.0/relation_model.html#api)。

**think-model** 扩展主要用来支持关系型数据库，还提供了 **think-mongo** 扩展来支持 MongoDB，添加如下扩展即可。

```js
const mongo = require('think-mongo');

module.exports = [
  mongo(think.app) // 让框架支持模型的功能
]
```

添加扩展后，可以通过 `think.mongo`、`ctx.mongo` 等方法使用。

MondoDB 数据库适配器的配置复用了关系型数据的配置。模型的定义和关系型数据库类似，只是继承的是 `think.mongo`。[官方API](https://thinkjs.org/zh-cn/doc/3.0/mongo.html#api)