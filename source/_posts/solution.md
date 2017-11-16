---
title: 问题汇总
date: 2017-09-20 13:26:45
categories:
  - 收藏
tags:
  - 前端
---
学习、工作当中常常会遇到一些问题，有的问题第一次遇到不知道如何解决就会去百度、谷歌。但是，有的一些问题我发现当我再次遇到的时候，仍然无法第一时间解决它，因为我忘记了具体操作或者是一些命令。所以，我决定把我通过百度、谷歌才解决的一些问题记录在博客当中，当我再次遇到而又记不清具体怎么解决的时候，就可以直接翻看这篇博客而不用再在网上花时间找解决办法。

<!-- more -->

## NPM

### Error with code: EINTEGRITY

通过 `npm install` 安装依赖时出现上述错误代码，百度说通过 `npm cache clean --force` 清除缓存，但是这对我来说没有作用。最后，Google 到 npm 的 issues，删除 `package-lock.json` 解决了该问题。

## WebStorm

### 在 Webpack 中配置别名后，WebStorm 出现文件找不到的提示

**WebStorm 2017.2 EAP, 172.2827** 版本已经支持，它会自动解析项目根目录下的 Webpack 配置文件，在 `Preferences/Languages & Frameworks/JavaScript/Webpack` 中可以指定配置文件。

参考：https://blog.jetbrains.com/webstorm/2017/06/webstorm-2017-2-eap-172-2827/

## React

### 热模块替换失效

原因：我在导入 `containers/App/App.js` 时导入了全部的 `containers`。

解决办法：在导入时只导入 App 组件即可。如下面代码所示：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import {AppContainer} from 'react-hot-loader';

// import {App} from 'containers';
import App from 'containers/App/App';

const render = (RootElement) => {
  ReactDOM.render(
    <AppContainer>
      <RootElement/>
    </AppContainer>
    , document.getElementById('root'));
};

if (module.hot) {
  module.hot.accept('containers/App/App', () => {
    const App = require('containers/App/App').default;
    render(App);
  });
}

render(App);
```
