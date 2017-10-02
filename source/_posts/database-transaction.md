---
title: 数据库事务及锁机制
date: 2017-09-21 23:41:26
categories:
  - 学无止境
tags:
  - Database
  - Concurrency
  - Transaction
---

昨天，在实际的开发过程中，有一个从数据库中取一条数据的操作，需要保证每条数据只会被取用一次。最终，通过数据库事务解决了该问题。在解决的过程中，可以说是重新的学习了一遍数据库锁机制和事务，补了个钙。

<!-- more -->

## 前言

在实现该功能时，首先在数据库表中添加了一个 `is_used` 字段，用来是标记该条数据是否被使用过，其中为 `1` 时表示已使用，为 `0` 时表示尚未被使用。每次获取一条可用数据的同时，需要将该条数据的 `is_used` 字段置为 `1`，表示该条数据已经被使用了。在这种情况下，在多个获取数据的请求并发执行的过程中，可能前一个请求处理过程中还未来得及将 `is_used` 字段置为 `1`，下一个请求就到来，导致两个请求获取到同一条数据。

最初，计划是通过在内存中维护一个队列，队列中缓存了部分可用的数据。当一个请求到来时，从队列中取出一个可用数据返回给客户端，并将该条数据对应的 `is_used` 字段置为 `1`。通过这样的方式，将并发的取数操作变为了同步的队列操作，可以保证不会取到同一条数据。

后来，在开始实现时考虑到是否可以从数据库层面实现这个问题？能不能确保获取一条数据并设置 `is_used` 字段为 `1` 的过程中，其他的请求不会取到该条数据？最后，突然想到了数据库的事务，因为事务具有原子性和隔离性，正好可以应对这种并发的情况。

## 数据库锁机制

关于数据库锁经常会听到"共享锁"、"排他锁"、"乐观锁"、"行级锁"等名词，之前并不是很熟悉，借着这次机会我重新的了解了一下数据库锁机制。数据库锁主要分为以下几类：

1. 按锁的粒度划分，可分为**行级锁**、**表级锁**、**页级锁**。
2. 按锁级别划分，可分为**共享锁**、**排他锁**。
3. 按使用方式划分，可分为**乐观锁**、**悲观锁**
4. 按操作划分，可分为 **DML 锁**和** DDL 锁**。
5. 按加锁方式划分，可分为**自动锁**、**显示锁**

### 行级锁、表级锁和页级锁

行级锁是锁定粒度最细的一种锁，表示只针对当前操作的行进行加锁。行级锁能大大减少数据库操作的冲突。其加锁粒度最小，但加锁的开销也最大。行级锁分为**共享锁**和**排他锁**。

表级锁是锁定粒度最大的一种锁，表示对当前操作的整张表加锁，它实现简单，资源消耗较少。表级锁定分为**表共享读锁**和**表独占写锁**。

页级锁是锁定粒度介于行级锁和表级锁中间的一种锁。表级锁速度快，但冲突多，行级冲突少，但速度慢。页级锁选择了一个折中的方案，一次锁定相邻的一组记录。

### 共享锁和排他锁

#### 共享锁

共享锁又称 **S 锁**、**读锁**，是读取操作创建的锁。加上共享锁后，其他用户可以并发读取数据（继续加共享锁），但不能对数据进行修改（无法加排他锁），直到所有共享锁被释放。

#### 排它锁

排他锁又称 **X 锁**、**写锁**，如果想到修改数据，必须先获得排他锁，获得排他锁的事务可以读、写数据。在给数据加上排它锁之后，其他事务不能再添加其他任何类型的锁。

### 乐观锁和悲观锁

乐观锁和悲观锁不是数据库中的概念，而是人们定义出来的概念，用来描述数据库并发控制的方案。

#### 乐观锁

乐观锁假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。在进行数据库处理时，乐观锁通常不使用数据库提供的锁机制，而是给数据添加一个版本标识，在读取数据时连同版本标识一同读出，然后再更新数据时检查当前标识与第一次读取出来的标识进行对比，如果相同则进行更新，否则认为数据过期。

#### 悲观锁

悲观锁指的是对数据是否可能被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守（悲观）态度，认为在自己的处理过程中，数据往往都会被外界修改，因此，在整个数据处理过程中，将数据处于锁定状态。

### DML 锁和 DDL 锁

DML(Data Manipulation Language)指数据操纵语言，在对数据进行操作（DML 操作）是会自动加上 DML 锁，如 insert、delete、update、select 等，

DDL(Data Definition Language)指数据定义语言，在进行数据库定义操作（DDL 操作）时会自动加上 DDL 锁如 create、alert、drop 等。

## 事务

数据库事务(Database Transaction) ，是指作为单个逻辑工作单元执行的一系列操作，要么完全地执行，要么完全地不执行。一个逻辑工作单元要成为事务，必须满足所谓的 **ACID**（原子性、一致性、隔离性和持久性）属性。

### 原子性

原子性是指事务包含的所有操作要么全部成功，要么全部失败回滚，使多个数据库操作看起来像一个操作一样。

### 一致性

一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。

### 隔离性

隔离性是当多个用户并发访问数据库时，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。事务的隔离有 4 中级别，由低到高分别为 **Read uncommitted** 、**Read committed** 、**Repeatable read** 、**Serializable**。

- Read uncommitted：顾名思义指一个事务会读取到另一个事务中未提交的读取，会产生**脏读**问题。
- Read committed：顾名思义指一个事务只会读取到另一个事务提交后的数据，可以解决**脏读**问题，但是会产生**不可重复读问题**。
- Repeatable read：意思是说**可重复读**，为了解决**不可重复读**的问题而产生。这里重复读指的是在一个事务执行过程中，多次对一条数据进行读取操作。其中不可重复读指多次读取操作中会读取到不同的数据，而可重复读指多次读取操作中读取的是相同的数据。这里针对的时同一条数据，如果是插入一条新的数据，那么无法进行控制，因此会产生**幻读**。
- Serializable：最高的事务隔离级别，在该级别下，多个事务顺序执行，可以解决**幻读**问题。

### 持久性

持久性是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。

## 实际应用

在补充了数据库锁机制及事务相关的知识后，再来看之前的问题就变得简单许多。在这个问题中，我们需要保证获取一个 `is_used` 字段为 `0` 的数据并将该数据的该字段设置为 `1` 是一个原子操作，并且不会被其他并发操作影响。因此，我们只用把查询数据和更新操作放在一个事务种并设置合适的隔离级别即可。那么，事务的隔离级别应该设置为哪个呢？

其实，这里的关键点在于，一个事务在读取一条可用数据并将其设为不可用状态之前，如果后来的事务也读取到了相同的数据，那么该事务的更新操作不能完成。也就是说，前面的事务执行过程中，不会读取到后一个事务提交的数据，对应的隔离级别就是 **Repeatable read**。如果用锁机制来解释，就是事务开始时，给读取到的数据加上**排他锁**，直到完成数据的更新操作后释放。这样，其他事务只能读取到该事务执行前或者执行完成后的数据。

关于事务的具体使用见我的另一篇博客：[Sequelize 基本用法](http://www.whezh.com/2017/10/sequelize-usage/#事务)

## 参考链接

1. http://blog.csdn.net/lexang1/article/details/52248686
2. http://www.cnblogs.com/fjdingsd/p/5273008.html