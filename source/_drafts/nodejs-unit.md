---
title: Node.js 单元测试
date: 2017-10-16 22:10:19
categories:
  - 技术
tags:
  - Node.js
  - Mocha
  - Chai
  - Sinon
---

以前写 Java 程序的时候会使用 JUnit 写一些单元测试，一方便测试代码是否存在 bug，另一方面在日后修改代码后重新跑一边测试用例，如果能全部通过也可以改得放心。现在接手了一些 Node.js 的开发工作，单元测试同样也是必不可少的。

<!-- more -->

## 用到的库

目前在进行 Node.js 单元测试的工作中，主要用到了下面的一些技术：

- [Mocha](http://mochajs.org/):Mocha (我喜欢叫“抹茶”)是一个功能十分丰富的 JavaScript 测试框架，在浏览器和 Node.js 环境中都可以运行。
- [Chai](http://chaijs.com/):Chai 是一个断言库，它提供了大量的断言帮助我们更加轻松的测试代码。
- [Sinon](http://sinonjs.org/): Sinon.js 是一个测试工具，它提供了 spy，stub，mock 等功能帮助我们监测函数的运行情况或者轻松地创建一个函数替代品等。
- [Should](https://github.com/tj/should.js): BDD 风格的断言库。
- [Istanbul](https://istanbul.js.org/): Istanbul 是一个统计代码覆盖率的工具。

这里就不对以上框架或库进行详细的说明了，在它们的官网上有详细的介绍和文档。

## 基本使用

我们可以在 **Mocha** 官访文档上看到详细的使用说明，一个 Mocha 结合 Chai 编写的测试差不多长这样：

```JavaScript
describe('数据库操作测试', function() {
  describe('同步数据库表', function() {
    it('应该返回一个对象', function(done) {
      // 异步 callback 写法
      models.sequelize.sync().then(function(result) {
        // expect 风格断言，结果需要被 expect() 包裹起来
        expect(result).to.be.an('object');
        done();
      });
    });
  });

  describe('添加主机信息', function() {
    it('返回一个 model 实例', function() {
      // 异步 Promise 写法
      return models.Info.create({
        ID: 'unit_test',
        NAME: 'unit_test'
      }).then(function(model) {
        expect(model).to.be.an('object');
        expect(model.get('ID')).to.equal('unit_test');
      });
    });

    it('数据库表中应该有该条数据', function() {
      return models.Info.findOne({
        where: {
          ID: 'unit_test',
        }
      }).then(function(model) {
        expect(model.get('NAME')).to.equal('unit_test');
      });
    });
  });
});
```

其中 `describe` 定义了一类测试，相当于是一个分组，可以互相嵌套；`it` 则是一个测试单元，在里面编写具体的测试代码。测试代码就是执行正式代码并将执行后的结果与期望的结果进行比较，如果都相同那么测试通过，如果有一个不同则测试不通过。

断言库使用的 **Chai** ，它支持多种风格的断言，主要是 `expect` 和 `should`，我理解的区别主要有下面几点：

1. expect 风格只需要引入 expect 方法，而 should 需要引入 should()，方法需要执行。
2. 在写法上，should 比 expect 更加简单，`expect(result).to.equal(1)` 等同于 `result.should.equal(1)`。
3. expect 将结果进行了一个包装，可以在所有浏览器和 Node.js 上运行；should 扩展了原型对象，提供了一个 getter 作为断言的入口，不支持 IE。

### Sinon

目前主要用到了 Sinon 提供的 `spy` 和 `stub` 方法，这两个方法可以使我们更加轻松地达成测试条件。例如：我们要测试一个需要删除数据库中的一条数据的方法，而我们并不像真的删除数据库中的内容，即使是测试库。这时就轮到 `stub` 上场了。

`stub`可以提供为数据库操作提供一个替身，在测试的过程中原本数据库操作的方法并不会被调用，而是调用了替身方法并记录下来调用信息。我们甚至可以让替身方法返回不同的值，以满足不同的需求。`spy` 的功能与 `stub`类似，它也会记录方法的调用信息，但是原本的方法仍会被执行。Sinon 的用法如下：

```JavaScript
const should = require('should');
const sinon = require('sinon');

const cacheControl = require('../../../src/middleware/cache-control');

const sandbox = sinon.sandbox.create();

describe('测试中间件: cacheControl', function() {
  let res;

  beforeEach(function() {
    res = {
      set: sandbox.spy()
    };
  });

  afterEach(function() {
    sandbox.restore();
  });

  it('正确设置 HTTP header', function(done) {
    cacheControl('private')(null, res, function(args) {
      should.not.exist(args);
      res.set.calledOnce.should.be.true();
      res.set.calledWith({
        'Cache-Control': 'no-cache, private, no-store, must-revalidate, max-stale=0, post-check=0, pre-check=0'
      }).should.be.true();
      done();
    });
  });
});
```

**说明**：

1. 我这里使用了另一个断言库 `should`。
2. 无论是 `spy` 还是 `stub`，如果不执行 `restore` 方法，那么他们的统计结果将会不断累积。
3. 通过 `sandbox` 我们可以集中地进行 `restore`。同时，在 mocha 提供的钩子方法中进行更加爽歪歪。

### 引入代码覆盖率检测

## 在 WebStorm 中使用
