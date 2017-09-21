---
title: 解决博客搭建在GitHub上无法被百度收录的问题
tags:
  - SEO
  - GitHub
  - Coding
categories:
  - 航海日志
date: 2017-08-13 23:31:40
---
博客在搭建完成后进行了适当的SEO，然后我分别将站点提交了谷歌和百度进行收录，提交的是GitHub Pages的地址。然而，提交给谷歌后不久就被收录了，但是百度却迟迟没有收录。当时这个问题还一直找不到原因，直到我试图在百度站长平台检测自己网站Robots时，根据错误信息才发现百度爬虫无法抓取，因为GitHub禁掉了百度爬虫。现将解决的过程记录下来。

<!-- more -->

## 解决思路

这个问题的解决办法网上有几种说法，大致分为三种：

- 放弃将博客部署在GitHub上。
- 利用CDN。
- 同时部署到GitHub和Coding。

在知道这个问题的原因之后，第一时间想到的是就是搞一个云服务器，将博客部署到云服务器上。但是，考虑到云服务器只部署博客有点亏，而暂时也没有其他的东西需要用到云服务器，所以就放弃了，还是打算部署在GitHub上。
第二种利用CDN来代理GitHub Pages上的博客的方法并不能很好的解决这个问题，因为如果附近的节点没有缓存，爬虫仍然会去爬取GitHub Pages上的内容。
所以综合考虑，最后采取了第三种办法，同时部署到GitHub和Coding上，通过域名解析，将国内的请求解析到Coding上。

## 操作过程

### 部署到Coding

首先去Coding上注册了个账号并配置一下SSH Key，然后创建了一个`<username>.coding.me`的仓库，接着在hexo的配置文件中添加该仓库的地址，如下所示。

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo:
    - git@github.com:hezhii/hezhii.github.io.git
    - git@git.coding.net:hezhou/hezhou.coding.me.git
  branch: master
```

这样就能同时将博客内容推到GitHub和Coding上。在Coding上选择`项目-->代码-->Pages 服务`，然后选择相应的分支即可完成部署。这样，通过`<username>.github.io`和`<username>.coding.me`就都能访问到博客了。
这里有一个问题需要注意，如果使用了Travis CI进行自动部署并且是SSH方式登陆的，需要在ssh_config文件中添加上Coding相关配置，ssh_config中的内容如下所示。

```
Host github.com
User git
StrictHostKeyChecking no
IdentityFile ~/.ssh/id_rsa
IdentitiesOnly yes

Host git.coding.net
User git
StrictHostKeyChecking no
IdentityFile ~/.ssh/id_rsa
IdentitiesOnly yes
```

### 域名申请并解析

完成了博客的部署后，就需要去弄个域名并添加解析。我是在万网申请的域名，申请完成后添加4条CNAME记录，将国内和国外的请求分别解析到Coding和GitHub，如下所示。

<div align="center"><img src="/images/domain-resolution.png" alt="域名解析" title="域名解析"></div>

域名在申请后需要进行实名认证，否则将会停止解析，实名认证十分简单，上传个身份证正面照就可以了。审核所需时间说是3~5个工作日，但是我提交审核后过了6个工作日才完成。

### 域名绑定

域名解析成功后，并不能通过域名访问博客，需要分别在GitHub和Coding上绑定自己申请的域名。其中，在博客的source目录下新建一个CNAME文件，文件写上自己申请的域名（不需要`http://`，`www`可有可无）并提交，即可完成GitHub上的绑定；而Coding上的绑定则更加简单，在Pages服务页面，通过自定义域名更能进行绑定即可。绑定完成后，待域名的解析生效即可通过申请的域名访问到博客了。我通过开关VPN分别访问进行了测试，确实国内会被解析到Coding，国外会被解析到GitHub。

## 其他问题

1. 这样做之后，如果需要给其他的项目添加GitHub Pages服务，在国内就无法访问，我目前采取的方案是仍然同时部署到GitHub和Coding。
2. 在百度站长上进行网站验证时，通过HTML方式验证会返回一个302的错误，怀疑是因为Coding有一个跳转页导致，目前没想好解决办法，采用CNAME的方式进行验证。
3. 在百度站长上提交sitemap时，出现了一个主域校验失败的错误，检测sitemap.xml文件后，发现上面的地址仍是GitHub Pages的地址，需要将hexo配置文件中的url设置为申请的域名。