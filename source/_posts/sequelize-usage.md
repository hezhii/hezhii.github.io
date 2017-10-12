---
title: Sequelize 基本用法
date: 2017-10-02 12:57:15
categories:
  - 技术
tags:
  - ORM
  - Sequelize
  - Node.js
  - 数据库
---

最近在工作中开始写 Node.js 应用，数据库使用的 PostgreSQL，为了方便数据库操作打算使用一款 ORM 框架。在 GitHub 上找到了 [node-orm2](https://github.com/dresende/node-orm2) 和 [sequelize](https://github.com/sequelize/sequelize) 两款，最终因为 star 数和更加全面的文档选择了 sequelize。现通过这篇博客记录下 sequelize 的一些基本用法。

<!-- more -->

## 简单介绍

> Sequelize is a promise-based ORM for Node.js v4 and up. It supports the dialects PostgreSQL, MySQL, SQLite and MSSQL and features solid transaction support, relations, read replication and more.

Seqeulize 是 Node.js 上的一个 ORM 框架。根据官网上的介绍，她支持多种数据库、支持写原生 SQL、事务、关联关系、读复制等功能。

Sequelize 使用简单，有十分详细的官方文档，在 GitHub上也有一个 [express-example](https://github.com/sequelize/express-example) 说明在 Express 中的基本使用方法。她基于 Promise，因此可以很方便的进行异步处理。

## 基本用法

Sequelize 的用法在官网上有很详细的介绍，这里只记录下比较常用的几点基本用法。

### 安装

直接通过 NPM 或者 Yarn 安装即可，需要同时安装相应数据库连接的驱动。

```bash
   // Using NPM
   $ npm install --save sequelize

   # And one of the following:
   $ npm install --save pg pg-hstore
   $ npm install --save mysql2
   $ npm install --save sqlite3
   $ npm install --save tedious // MSSQL

   // Using Yarn
   $ yarn add sequelize

   # And one of the following:
   $ yarn add pg pg-hstore
   $ yarn add mysql2
   $ yarn add sqlite3
   $ yarn add tedious // MSSQL
```

### 建立连接

建立连接就是初始化一个 **Sequelize** 对象。在初始化 Sequelize 实例时会创建一个数据库连接池，所以在单线程模式下最好一个数据库下新建一个实例；如果是多线程模式则需要在创建多个实例，并且设置每一个连接池的最大连接数为总的最大连接数除以线程个数。

Sequelize 支持多个初始化参数，同时支持多种写法：

```javascript
// without password and options
const sequelize = new Sequelize('database', 'username')

// without options
const sequelize = new Sequelize('database', 'username', 'password')

// without password / with blank password
const sequelize = new Sequelize('database', 'username', null, {})

// with password and options
const sequelize = new Sequelize('my_database', 'john', 'doe', {})

// with database, username, and password in the options object
const sequelize = new Sequelize({ database, username, password });

// with uri
const sequelize = new Sequelize('mysql://localhost:3306/database', {})
```

我习惯通过一个对象传入所有的参数，这样直接从配置文件中读取并传入即可，不需要过多的操作。例如：

```javascript
const sequelize = new Sequelize({
  host: 'localhost',
  database: 'test',
  port: 3432,
  dialect: 'postgres',  // 连接的数据库类型，mysql, postgres, sqlite and mssql
  username: 'test',
  password: 'test',
  pool: { // 连接池配置
    max: 5,
    min: 0,
    idle: 10000
  }
});
```

### 定义模型

在初始化 Sequelize 对象的实例后，通过调用实例对象的 *define* 方法可以定义模型，例如：

```javascript
sequelize.define('modelName', {
    columnA: {
        type: Sequelize.BOOLEAN,
        validate: {
          is: ["[a-z]",'i'],        // will only allow letters
          max: 23,                  // only allow values <= 23
          isIn: {
            args: [['en', 'zh']],
            msg: "Must be English or Chinese"
          }
        },
        field: 'column_a'
        // Other attributes here
    },
    columnB: Sequelize.STRING,
    columnC: 'MY VERY OWN COLUMN TYPE'
})

sequelize.models.modelName; // The model will now be available in models under the name given to define
```

*define* 方法支持三个参数，分别是：模型名称、属性、可选参数，具体可以查阅[官方文档](http://docs.sequelizejs.com/class/lib/sequelize.js~Sequelize.html#instance-method-define)。同时，Sequelize 还提供了一个 *import* 方法，支持从一个文件中导入模型，具体的写法可以参考官网提供的 [Demo](https://github.com/sequelize/express-example)。

每一个模型对应一张数据库表，初始化模型时的每一个属性对应一个表中的一个字段。其中，字段是一个对象，可以指定字段类型、是否为主键、是否唯一、校验等。

定义完模型后，即可通过 Sequelize 对象实例的 *sync* 方法同步模型到数据库表中。默认情况下当模型对应的数据表不存在时，会创建对应的数据库表，可以通过 **force** 参数控制。这里需要注意的是：在创建数据库表时，Sequelize 会使用 `'modelName' + 's'` 作为表名称，如果想使用 `modelName` 作为表名，在定义模型时设置 `freezeTableName` 参数为 `true` 即可。

### CURD

在定义模型并同步模型到数据库表后，通过模型的静态方法和实例方法，可以很方便的进行数据库操作。下面主要介绍最为常见的增删改查方法。

#### 添加数据

添加数据主要有两种方法，一种是先创建一个模型实例，然后调用实例的 *save* 方法；另一种则是直接通过模型的静态 *create* 方法。例如：

```javascript

// 方法1：先创建实例，然后调用实例的 save 方法
const user = User.build({
  'emp_id': '1',
  'nick': '小红',
  'department': '技术部'
});

user.save().then(function() {
  // success
}).catch(function(err) {
  // error
});

// 方法2：通过静态 create 方法
const user = User.create({
  'emp_id': '2',
  'nick': '小明',
  'department': '技术部'
}).then(function() {
  // success
}).catch(function(err) {
  // error
});
```

#### 删除数据

直接调用模型的静态方法 *destroy* 即可，通过 `options.where` 可以指定查询参数。例如：

```javascript
User.destroy({
  where: {
    emp_id: '1',
  }
}).then(function(count) {
  // success
}).catch(function(err) {
  // error
});
```

#### 修改数据

修改数据可以直接调用静态的 *update* 方法，也可以通过查询方法获取到数据对应的实例对象，然后直接修改对象的属性，最后调用 *save* 方法写入到数据库中。例如：

```javascript
// 方法1：操作对象属性（不会操作db），调用save后操作db
user.nick = '小白';
user = yield user.save();

// 方法2：直接update操作db
user = yield user.update({
    'nick': '小白白'
});
```

#### 查询数据

查询数据主要通过模型的静态方法，模型提供了多个 API 以满足查询需求。例如：

```javascript
User.findAndCount({
  where: {
    sex: '女'
  },
  order: [
    ['emp_id', 'age']
  ],
  limit: 10,
  offset: 0
}).then(function(result) {
  // success
  const data = [];
  const total = result.count;
  result.rows.forEach(function(item) {
    data.push(item.toJSON());
  });

  logger.debug('查询完成，耗时 %d ms，查询到 %d 条结果。结果为： %s', Date.now() - startTime, total, JSON.stringify(data));
}).catch(function(err) {
  // error
});
```

### 事务

在 **Seqeulize** 中使用事务主要通过对象实例的 *transaction* 方法。该方法会创建一个 **Transaction** 对象实例，接着在执行数据库操作时指定 transaction 参数为该实例，这样就可以在事务下进行数据库操作。当事务完成后，通过 *commit* 方法提交事务，通过 *rollback* 方法回滚。例如：

```javascript
sequelize.transaction({
  isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.READ_COMMITTED
}, transaction => {
  return User.findOne({
    where: {
      name: '小明'
    },
    transaction: t
  }).then(function(model) {
    if (model) {
      model.age = model.age + 1;
      return model.save({
        transaction: t
      });
    }
  });
}).then(result => {
  // transaction has been committed. Do something after the commit if required.
}).catch(err => {
  // do something with the err.
});
```

创建事务时可以通过 isolationLevel 参数指定事务的隔离级别，默认级别是 **REPEATABLE_READ**。

给 *sequelize.transaction* 方法指定**回调函数**可以将事务托管给 Sequelize，它会自动提交和回滚事务（例如上面的例子）。如果不使用回调函数的写法，而是使用 `promise.then()`，则需要手动调用 `t.commit()` 提交事务，`t.callback()` 回滚事务。

## 总结

目前，我也只是使用了 **Sequelize** 的一些基本功能，并不涉及到复杂的操作，不太适合过多的进行评价，给我的感觉还是相当不错的，简单、好用。
