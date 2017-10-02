---
title: 如何在WebStorm中使用Git Submodule
date: 2017-08-20 17:30:47
categories:
  - 学以致用
tags:
  - Git
  - WebStorm
---
有时候我们需要在一个项目中使用另一个项目，即当前的父项目下包含一个子项目。我们可能会对子项目中的内容进行修改，并且希望父项目和子项目中的修改可以分别提交。Git通过子模块来实现这个需求，子模块允许将一个Git仓库作为另一个Git仓库的子目录。它能让我们将另一个仓库克隆到自己的项目中，同时还保持提交的独立。接下来，介绍下如何在WebStorm中使用子模块。

<!-- more -->

## 开始使用

1. 首先可以通过终端命令添加子模块到指定目录下，不能提前创建好该目录，否则会报错。
```bash
$ git submodule add <repo> <directory>
```
2. 添加完子模块之后，在当前目录下会生成一个.gitmodules的文件，该置文件保存了项目URL与已经拉取的本地目录之间的映射，这个文件也应该提交到Git中。此时如果执行`git status`命令或者在WebStorm的`Version Control > Local Changes`标签页下可以看到有一个未提交的修改，对应的是Clone的子模块文件夹。当不在那个目录中时，Git并不会跟踪它的内容，而是将它看作该仓库中的一个特殊提交。
```
[submodule "public/home-assistant-polymer"]
	path = public/home-assistant-polymer
	url = git@github.com:hezhii/home-assistant-polymer.git
```
3. 上面说到，Git并不会跟踪子仓库中的内容，而是视为一个子模块。但是，通过WebStorm我们可以看到子模块中的具体修改内容，就像是一个文件夹一样。在WebStorm的设置中，选择`Version Control`，我们可以看到克隆的子模块，只需要选中它并点击下方的`+`就可以注册该模块，这样子模块中的内容变化也会显示在`Local Changes`中。
4. 在后续的开发中，无论是父模块还是子模块中的内容发生了修改，都可以直接提交，WebStorm会帮我们将修改提交到对应的仓库中。

## 克隆含有子模块的项目

当我们在克隆一个包含子模块的项目是，默认会包含该子模块目录，但其中还没有任何文件。我们必须执行下面两个命令：

- `git submodule init`用来初始化本地配置文件。
- `git submodule update`从子模块中获取所有数据并检出父项目中列出的提交。

或者可以在执行git clone命令时指定`--recursive`参数，这样就会自动初始化并更新仓库中的每一个子模块。

## 删除子模块

1. 编辑`.gitmodules`和`.git/config`文件，删除子模块相关配置。
2. 执行`git rm --cached path_to_submodule`命令，清除子模块缓存。
3. 删除子模块文件夹及相关文件。
4. 提交修改。
