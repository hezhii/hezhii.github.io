---
title: 博客搭建过程
date: 2017-08-07 23:02:01
categories:
- 我走过的路
tags:
- GitHub
- Hexo
- Travis CI
---
博客基于GitHub Pages和Hexo搭建，使用Travis CI自动部署，具体如何操作网上有全面的教程，我这里也就没有必要一一讲述了，也不是很复杂的一个东西。这篇博客，也权当作是我自己的一个笔记，记录下我自己的搭建过程以及我稍微花了点时间去处理的一些问题。

<!-- more -->

## 环境准备

- Git
- Node.js
- Ruby

我自己电脑上本来也就有这些环境，所以这一步骤就直接跳过了。各种环境如何安装，网上一查也就知道了。

## 安装Hexo

安装以及使用Hexo是直接参考[Hexo官网](https://hexo.io/)上的文档完成的。官网上的文档还是比较全面的，基本读一遍就能耍起来了。
不过在安装的过程中却是在这个地方遇到一点问题，当时直接就`npm install hexo-cli -g`进行安装，在执行的过程中却卡住了相当长的一段时间，并且也没有任何的错误提示。起初我还以为是需要翻墙，等了一会以后就直接结束安装，然后打开vpn重新安装，然而结果还是一样。但是，这一次我没有退出，而是先去干了点别的事情，过了一会再来看的时候，给了个没有权限的错误提示，然后加上`sudo`就安装成功了。

### 初始化及配置

这一步参考官网上的文档进行操作即可，基本就是`hexo init <folder>`初始化一个项目，接着进入到该目录`npm install`安装依赖，然后修改下配置文件的过程。有一点需要注意的是，就算没有自己的一个站点，也不要删除配置文件中的`url`参数。

### 基本使用

配置完成后，执行`hexo server`即可在本地启动一个服务，通过浏览器访问`http://localhost:4000/`就可以看到博客页面。在使用时通常执行`hexo n <name>`初始化一篇博文，然后对博文的内容进行编辑，如果启动了本地服务，刷新页面可以看到博客内容的修改。博文完成后，通过`hexo deploy`即可部署到配置文件中的指定站点，在部署到GitHub上时，需要安装`hexo-deployer-git`插件。

## GitHub Pages

在GitHub上新建一个`yourname.github.io`的仓库，仓库master分支中的静态文件即可通过`https://yourname.github.io`直接进行访问。如果希望其他仓库中的内容也能通过GitHub Pages进行访问，那么就在该仓库中创建一个**gh-pages**的分支，该分支下的内容可以通过`yourname.github.io/repoitoryName`进行访问。
为了方便管理，我将博客部署到了`hezhii.github.io`中的master分支上，博客的一些源文件则提交到了该仓库的hexo分支上，利用Travis CI自动将hexo分支中的更新部署到master分支上。

## Travis CI

Travis CI结合hexo具体如何使用，网上也有相当多的教程，主要涉及到Travis CI命令行工具的安装和使用、访问GitHub的一个配置以及Travis CI的配置。其中命令行工具的安装和使用十分简单，找个教程照着做就可以了。
关于GitHub访问的一个配置，我使用的是SSH的方式，还可以使用GitHub Access Token，这里需要注意的是不要忘记hexo配置文件中GitHub的仓库地址也应该使用SSH。
Travis CI中的配置可以参考[我的配置](https://github.com/hezhii/hezhii.github.io/blob/hexo/.travis.yml)，其中如果没有设置时区的话，GitHub上的提交时间就不是北京时间。

## 主题

主题我使用的是相当火的[NexT](http://theme-next.iissnan.com/)主题，主题的安装和使用在官方文档上也有详细的说明，我在使用的过程中参考了[这篇博文](http://blog.ynxiu.com/2016/hexo-next-theme-optimize.html)中的一些配置。