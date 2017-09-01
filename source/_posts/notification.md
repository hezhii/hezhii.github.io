---
title: HTML5 Notification
date: 2017-08-09 22:40:19
categories:
  - 也许这能帮到你
tags:
  - HTML5
  - Frontend
---
>The Notifications API allows web pages to control the display of system notifications to the end user. These are outside the top-level browsing context viewport, so therefore can be displayed even when the user has switched tabs or moved to a different app. The API is designed to be compatible with existing notification systems, across different platforms.

<!-- more -->

## 引子

今天打开优酷网站发现弹出了一个是否允许显示通知的对话框，如下图所示。

<div align="center"><img src="/images/youku_notification_permission.png" width="400" alt="优酷请求显示通知" title="优酷请求显示通知"></div>

当我选择允许后，过一会儿从电脑的右侧弹出了一个提示框，如下图所示。

<div align="center"><img src="/images/youku_notification.png" width="400" alt="优酷通知" title="优酷通知"></div>

当时觉得很有意思，便上网查询了一下相关的资料，了解到是HTML5的新API——Notification。于是，本着学习的想法，弄了一个小的demo。

## Notification简介

[Notification API](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)是HTML5中一个新的API，用于向用户配置和显示桌面通知，并且这一特性在Web Worker中可用。

### 构造方法

通过`let notification = new Notification(title, options);`可以初始化一个Notification实例。构造函数中包含两个参数，其中**title**是通知的标题，而**options**则是一些初始参数，具体的参数列表如下：
- dir : 文字的方向；它的值可以是 auto（自动）, ltr（从左到右）, or rtl（从右到左）
- lang: 指定通知中所使用的语言。这个字符串必须在 BCP 47 language tag 文档中是有效的。
- body: 通知中额外显示的字符串
- tag: 赋予通知一个ID，以便在必要的时候对通知进行刷新、替换或移除。
- icon: 一个图片的URL，将被用于显示通知的图标。

### 属性

Notification主要包含一个静态属性和若干个实例属性，实例属性全部都是只读的属性，并且就是初始化该实例时option中的内容。具体的属性如下所示。

1. Notification.permission：这是一个静态的只读属性，用于表明当前通知显示授权状态的字符串。可能的值包括：denied (用户拒绝了通知的显示), granted (用户允许了通知的显示), or default (因为不知道用户的选择，所以浏览器的行为与 denied 时相同)。
2. title：通知的标题，Readonly。
3. dir：通知的文本显示方向，Readonly。
4. lang：通知使用的语言，Readonly。
5. body：通知中的文本内容，Readonly。
6. tag：通知的ID，Readonly。
7. icon：通知中图片的url地址，Readonly。

### 主要方法

1. Notification.requestPermission：这是一个静态方法，作用就是请求用户当前来源的权限以显示通知，效果就是上文中打开优酷网页时弹出的一个对话框。
2. close：关闭通知。
3. onclick：处理click事件的处理。每当用户点击通知时被触发。
4. onshow：处理show事件的处理。当通知显示的时候被触发。
5. onerror：处理error事件的处理。每当通知遇到错误时被触发。
6. onclose：处理 close 事件的处理。当用户关闭通知时被触发。

## 实例演示

在熟悉了API之后，就尝试自己弄个实例来演示一下。

<iframe height='340' scrolling='no' title='Notification' src='//codepen.io/Sylvanass/embed/preview/Mvmbax/?height=340&theme-id=light&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/Sylvanass/pen/Mvmbax/'>Notification</a> by Sylvanass (<a href='https://codepen.io/Sylvanass'>@Sylvanass</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

点击DING!按钮，如果你是第一次，会出现下面的提示：

<div align="center"><img src="/images/notification_permission.png" width="400" alt="请求显示通知" title="请求显示通知"></div>

选择允许后，一会就会收到通知了，如下图所示。

<div align="center"><img src="/images/notification.png" width="400" alt="通知" title="通知"></div>

此时，刷新或者关闭浏览器，通知仍然会显示。点击通知，可以产生一些交互效果。
至此，关于HTML5 Notification的介绍和简单使用就完结了。



