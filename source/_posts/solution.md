---
title: 问题汇总
date: 2017-09-20 13:26:45
categories:
  - 也许这能帮到你
tags:
  -
---

## NPM

### Error with code: EINTEGRITY

通过 `npm install` 安装依赖时出现上述错误代码，百度说通过 `npm cache clean --force` 清除缓存，但是这对我来说没有作用。最后，Google 到 npm 的 issues，删除 `package-lock.json` 解决了该问题。
