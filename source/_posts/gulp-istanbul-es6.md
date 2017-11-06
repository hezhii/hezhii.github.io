---
title: 关于 gulp-istanbul 无法解析 async/await 的问题
date: 2017-11-03 16:45:29
categories:
  - 技术
tags:
  - Gulp
  - Istanbul
---

最近，在使用 Gulp 执行 Mocha 单元测试并生成测试覆盖率报告时执行一直出错，出错的原因是因为代码解析时出现不能识别的代码。这篇博客记录了问题的原因及我最后选择的处理办法。

<!-- more -->

## 现象

我使用 **gulp-mocha** 和 **gulp-istanbul** 插件执行单元测试并生成代码覆盖率报告，对应 **gulpfile.js** 中的内容如下：

```js
const istanbul = require('gulp-istanbul');
const mocha = require('gulp-mocha');

gulp.task('pre-test', function () {
  return gulp.src(['src/**/*.js'])
    .pipe(istanbul())
    .pipe(istanbul.hookRequire());
});

gulp.task('test', ['pre-test'], function () {
  return gulp.src(['test/unit/**/*.test.js'])
    .pipe(mocha())
    .pipe(istanbul.writeReports())
    .pipe(istanbul.enforceThresholds({ thresholds: { global: 90 } }));
});
```

上面的内容是 gulp-istanbul 插件文档中的示例用法，然而在执行时却出错了，主要的错误信息如下图所示。

![](/images/gulp_istanbul_error.png)

从上图中可以看出，出错的原因是出现了无法解析的字符，在查看了对应了代码后发现无法解析的是 **async/await**。

## 原因

在发现 gulp-istanbul 无法解析 async/await 之后，我通过 Google 查找了相关资料，最后在 **istanbul** 的一个 issue 中找到了问题的原因所在：gulp-istanbul 依赖了 **istanbul**，而最新版本的 istanbul 无法解析 async/await，需要将 istanbul 升级到下一个版本，详情[戳这里](https://github.com/gotwarlost/istanbul/issues/733)。

我通过 `npm i -D istanbul@next` 命令将 istanbul 升级到下一个版本，然后单独的生成代码覆盖率报告，发现确实可以成功解析 **async/await**。然而，gulp-istanbul 插件依赖的是 istanbul 最新发布的稳定版本，该版本并不能解析 async/await。

于此同时，我在 **gulp-istanbul** 的 [issue](https://github.com/SBoudrias/gulp-istanbul/issues/116) 中也发现了有人提出了同样的问题。作者在 issue 中指出了稳定版的 istanbul 不支持 ES6，需要提供一个插装器。但是，我按照文档的示例添加插装器后，仍然无法解决该问题。该 issue 的提出者也表示了该方法并不能起作用，并希望 gulp-istanbul 的作者更新依赖的 istanbul 到下一个版本，然而作者表示没有时间去支持一个不稳定的版本，很乐意接受 PR。😂 😂 😂

到这里，我意识到目前想要直接通过 gulp-istanbul 插件生成代码覆盖率有点困难。

## 解决办法

我注意到 istanbul 已经升级到了 2.0 版本并且由一个新的团队负责开发，名称也改成了 **nyc**。它支持 await/nyc，但是很遗憾，我并没有找到相应的 gulp 插件，所以最终我打算在 task 中简单地执行 Node 脚本命令来执行 Mocha 并生成代码覆盖率报告。

根据 Gulp API 文档可知，`gulp.task(name[, deps], fn)` 方法就是定义了一个任务及任务对应的动作，执行某个任务就是执行对应的方法。同时，该方法支持异步操作并提供了多种方法：

1. 接受一个 callback 参数，当 task 结束时调用该方法即可，类似于 Mocha 中的 `done()`。
2. 返回一个 stream，stream 可以被 `pipe()` 到其他插件中。
3. 返回一个 promise。

因此，我只用提供一个方法，在该方法中运行 **mocha** 和 **nyc**，然后对运行的结果进行一个包装即可，[详情戳这里](http://www.whezh.com/2017/11/glup/#代码覆盖率)。
