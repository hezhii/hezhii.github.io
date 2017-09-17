---
title: 跨域 POST 请求之 CROS
date: 2017-09-16 20:00:57
categories:
  - 我走过的路
tags:
  - CROS
---

跨域资源共享（Cross-origin resource sharing，简称 "CROS"）是一个新的 W3C 标准，它允许浏览器跨域请求服务器上的资源，就像请求同源的服务器资源一样。对于跨域问题常见的解决方法就是使用 JSONP，但是 JSONP 是不支持 POST 请求的，在 jQeury 中即使设置为 POST 也依旧会以 GET 请求提交。当不需要兼容低版本浏览器时，相较于 iframe、proxy 等，CROS 不失为一个简单、优雅的解决方案。

<!-- more -->

## 前言

最近在项目中需要跨域提交 POST 请求，使用的是 CROS 作为解决方案。之前对于 CROS 有过一些了解，但没有具体的尝试使用，只是觉得 CROS 机制并不复杂：无外乎就是提供了一种跨域请求资源时，如果服务器同意就允许此次请求的机制。但是，这次在具体使用时却遇到了麻烦。

## 基本使用

CROS 使用起来十分简单，前端不需要做任何处理，只需要服务器在响应请求时，在响应头中设置几个特殊的字段即可，具体是哪些字段根据请求类的型不同而不同。

利用 CROS 跨域请求被分为两种：一种是不需要进行预检的简单请求，另一种是需要进行预检的非简单请求。简单请求和非简单请求主要通过请求类型和请求头进行决定，如果请求满足下面的条件就是简单请求，否则就是非简单请求。

1. 请求方法是 HEAD、GET 或者 POST 方法。
2. 请求头中只包含这些头：Accept、Accept-Language、Content-Language、Last-Event-ID 和 Content-Type，其中 Content-Type 的值只能是 application/x-www-form-urlencoded、multipart/form-data 或者 text/plain。

### 简单请求和非简单请求的区别

两种跨域请求的区别主要就是非简单请求在正式请求之前浏览器会主动发出一个方法为 OPTIONS 的预检请求，预检请求的目的就是询问服务器是否允许此 **METHOD** 或者 **headers** 访问服务器。