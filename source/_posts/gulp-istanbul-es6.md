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

我使用 `gulp-mocha` 和 `gulp-istanbul` 插件执行单元测试并生成代码覆盖率报告，对应 `gulpfile.js` 中的内容如下：

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

上面的内容是 `gulp-istanbul` 插件文档中的示例用法，然而在执行时却出错了，主要的错误信息如下图所示。

![](/images/gulp_istanbul_error.png)

从上图中可以看出，出错的原因是出现了无法解析的字符，在查看了对应了代码后发现无法解析的是 `async/await`。

## 原因

## 解决办法
