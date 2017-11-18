---
title: Gulp 初体验
date: 2017-11-02 13:40:15
categories:
  - 技术
tags:
  - JavaScript
  - Gulp
---

> Gulp.js 是一个自动化构建工具，开发者可以使用它在项目开发过程中自动执行常见任务。Gulp.js 是基于 Node.js 构建的，利用 Node.js 流的威力，你可以快速构建项目并减少频繁的 IO 操作。Gulp.js 源文件和你用来定义任务的 Gulp 文件都是通过 JavaScript（或者 CoffeeScript ）源码来实现的。

<!-- more -->

## 安装

### 全局安装
```bash
$ npm i -g gulp
```

### 项目依赖
```bash
$ npm i -D gulp
```

## 开始使用

在项目的根目录下创建 `gulpfile.js` 文件，然后通过 API 以及各类插件定义所需的自动化任务。

Gulp 的 [API](https://github.com/gulpjs/gulp/blob/master/docs/API.md) 十分简单，只有四个，分别是：

- **gulp.src**: 载入资源
- **gulp.dest**: 输出数据
- **gulp.task**: 定义任务
- **gulp.watch**: 监视文件修改

在定义好了自动化构建的任务之后，执行 `gulp` 将会运行默认的名为 `default` 的任务。如果想要运行特定的任务，在命令后面添加相应任务的名称即可，`gulp <task> <othertask>`。

## 定义自动化任务

下面记录我在一个 Node.js 项目中初次使用 Gulp 的经历，通过 Gulp 我主要想完成代码审查、执行单元测试、测试覆盖率检查、打包这一自动化构建流程。

### ESlint

首先安装 ESlint 插件。

```bash
$ npm i -D gulp-eslint
```

然后将文档中介绍的用法直接复制到 `gulpfile.js` 中，此时 `gulpfile.js` 的内容如下：

```js
const gulp = require('gulp');
const eslint = require('gulp-eslint');

gulp.task('lint', () => {
  return gulp.src(['**/*.js', '!node_modules/**'])
    .pipe(eslint())
    .pipe(eslint.format())
    .pipe(eslint.failAfterError());
});

gulp.task('default', ['lint'], function () {
  // This will only run if the lint task is successful...
});
```

定义好了 `lint` 任务后，直接运行 gulp 命令，控制台输出如下：

![](/images/gulp_lint.png)

为了测试是否真的起作用，我故意添加一个审查错误的代码，出错时输出如下：

![](/images/gulp_lint_error.png)

### Mocha

首先安装 Mocha 插件。

```bash
$ npm i -D gulp-mocha
```

定义如下任务，并添加到默认任务中。

```js
gulp.task('test', function () {
  process.env.NODE_ENV = 'test';
  return gulp.src('test/unit/**/*.test.js')
    .pipe(mocha({
      reporter: 'nyan',
      timeout: 5000,
      globals: {
        should: require('should')
      }
    }))
    .once('error', () => {
      process.exit(1);
    })
    .once('end', () => {
      process.exit();
    });
});
```

Mocha 具体支持哪些参数见官网文档，这里需要注意的是在测试执行完成后手动的调用 `process.exit()` 结束进程，否则会处于阻塞状态。参考 gulp-mocha [文档](https://github.com/sindresorhus/gulp-mocha#test-suite-not-exiting)中的描述。

同时，在一开始的时候我的测试用例无法通过，最后发现是因为 `NODE_ENV` 没有指定导致，这里需要注意设置 `NODE_ENV`。

### 代码覆盖率

在这一部分遇到了一些问题，问题的主要原因是因为我在代码中使用了 `async/await`，关于这个问题的详细说明[请点这里](http://localhost:4000/2017/11/gulp-istanbul-es6/)。此处，只列出最终解决后 gulpfile 中的内容，如下：

```js
/**
 * 执行单元测试并生成代码覆盖率报告
 */
gulp.task('test', () => _exec('npm test'));

/**
 * 代码覆盖率
 */
gulp.task('coverage', ['test'], () => _exec('npm run coverage'));

function _exec (command, args = [], options = {shell: true, stdio: 'inherit'}) {
  return new Promise((resolve, reject) => spawn(normalize(command), args, options)
    .on('close', code => code ? reject(new Error(`${command}: ${code}`)) : resolve())
  );
}
```

从上面的代码可以看到，在执行测试以及代码覆盖率检查时我并没有使用 gulp 插件，而是执行了相应的脚本，`package.json` 中对应的脚本如下：

```json
"scripts": {
    "dev": "NODE_ENV=development DEBUG=host2.0:* node index.js",
    "lint": "eslint ./ || exit 0",
    "test": "NODE_ENV=test nyc --reporter=html --reporter=text mocha -R nyan --recursive ./test/unit",
    "coverage": "nyc check-coverage --lines 90 --functions 100 --branches 100"
  },
```

### 打包

代码审查及测试通过后需要打包便于发布，打包代码使用的是 `gulp-zip` 插件，首先安装插件：

```bash
$ npm i -D gulp-zip
```

然后再 gulpfile 中添加下面内容：

```js
/**
 * 打包
 */
gulp.task('pack', ['coverage'], () =>
  gulp.src(['src/**', 'index.js', 'package.json', 'package-lock.json', 'README.md', '.nvmrc'], {base: '.'})
    .pipe(zip('archive-' + getVersion() + '.zip'))
    .pipe(gulp.dest('./'))
);

/**
 * 获取版本号
 */
function getVersion () {
  return JSON.parse(fs.readFileSync('./package.json', 'utf8')).version;
}
```

这里有个需要注意的地方，一是获取版本号时直接读取的 json 文件而不是使用 require，这是因为 require 会缓存多次调用，这会导致版本号不会被更新掉；二是在使用 `gulp-zip` 打包时，如果希望打包后的文件在解压后依旧保留顶级目录，需要在调用 `gulp.src` 载入资源时，添加 `base` 参数，参看 [issue](https://github.com/sindresorhus/gulp-zip/issues/23)。

## 总结

Gulp 初次体验下来感觉很合我的胃口，目前只是简单地使用并没有太多深入的研究，主要是在用法上感觉很舒服。定义一个个的任务，然后通过任务的各种组合来完成不同的自动化流程，我感觉看着比 Grunt 要清晰，不过可能是因为我看到 Grunt 的配置功能更复杂的原因导致的。在今后，会尝试使用 Gulp 来完成更加复杂的构建工作，我相信它会给我带来更多的惊喜。
