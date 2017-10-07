---
title: Sequelize 基本用法
date: 2017-10-02 12:57:15
categories:
  - 学以致用
tags:
  - ORM
  - Sequelize
  - Node.js
---

最近在工作中开始写 Node.js 应用，数据库使用的 PostgreSQL，为了方便数据库操作打算使用一款 ORM 框架。在 GitHub 上找到了 [node-orm2](https://github.com/dresende/node-orm2) 和 [sequelize](https://github.com/sequelize/sequelize) 两款，最终因为 star 数和更加全面的文档选择了 sequelize。现通过这篇博客记录下 sequelize 的一些基本用法。

<!-- more -->

## 简单介绍

Seqeulize 是 Node.js 上的一个 ORM 框架。根据官网上的介绍，她支持多种数据库、支持原生 SQL、事务、关联关系、读复制等功能。

Sequelize 使用简单，有十分详细的官方文档，在 GitHub上也有一个 [express-example](https://github.com/sequelize/express-example) 说明在 Express 中的基本使用方法。她基于 Promise，因此可以很方便的进行异步处理，直接 catch 住异常。

## 基本用法

## 事务
