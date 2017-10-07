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

### 定义模型

### CURD

### 事务
