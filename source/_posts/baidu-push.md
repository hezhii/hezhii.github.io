---
title: Node.js 实现百度主动推送
date: 2017-09-01 11:26:15
categories:
  - 也许这能帮到你
tags:
  - Node.js
  - Script
---

博客建立好之后，迟迟没有被百度收录。之前添加了站点地图和 NexT 主题提供的主动推送，看到百度站长平台还支持一个主动推送功能，说是能使提交的页面被更快的收录。于是，就用 Node.js 写了一个主动推送的小工具，该工具会提取指定网站 sitemap.xml 中的 url 并将所有提取到的 url 通过百度站长提供的主动推送接口推送给百度。

<!-- more -->

## 如何使用

我将脚本提交到了 GitHub 仓库中，该仓库中还包含了一些其他我用到的脚本([仓库地址戳这里](https://github.com/hezhii/scripts)）。直接将该仓库克隆到本地，然后进入到脚本文件所在目录，安装需要的依赖并执行脚本即可。

```bash
$ git clone git@github.com:hezhii/scripts.git
$ cd scripts/nodejs/pushurl
$ yarn install
$ URL=<your_site_url> TOKEN=<your_baidu_token> node index.js
```

**注意**：运行本脚本需要安装 Node.js 和 Yarn。

## 定时启动

在完成这个脚本后，我想每天能定时的推送一次博客中的文章。于是，就上网了解了一下 macOS 上计划任务相关的内容，网上有几种解决思路。最终我选择了使用 crontab 创建定时任务。

crontab 是一个定时任务的调度工具，我们可以通过编辑定时任务列表文件将任务添加到 crontab中。定时任务的格式如下：

```
* * * * *  <command to execute>
│ │ │ │ │
│ │ │ │ └─── day of week (0 - 6) (0 to 6 are Sunday to Saturday, or use names; 7 is Sunday, the same as 0)
│ │ │ └──────── month (1 - 12)
│ │ └───────────── day of month (1 - 31)
│ └────────────────── hour (0 - 23)
└─────────────────────── min (0 - 59)
```

下面我以设置每天上午 10 点执行主动推送脚本为例进行说明。

首先在终端输入 `crontab -e` 打开定时任务文件，然后按 `i` 编辑该文件，在文件的第一行插入下面内容：

```
0 10 * * * PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin;URL=your_site_url TOKEN=your_baidu_token ~/Development/projects/tools/nodejs/pushurl/index.js >> ~/Desktop/push_log.txt 2>&1
```

然后直接 `:wq` 保存并退出即可。我们可以在终端输入 `crontab -l` 查看已添加的定时任务。

根据上面的命令格式可以看出，内容的意思是说在每天的 10 点 执行`PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin;URL=your_site_url TOKEN=your_baidu_token ~/Development/projects/tools/nodejs/pushurl/index.js >> ~/Desktop/push_log.txt 2>&1` 命令。这行命令首先添加了 PATH，因为 crontab 在执行时的 PATH 是 `usr/bin`，可能会导致 node 命令不存在。然后设置了 `URL` 和 `TOKEN` 两个环境变量，用来指定站点和相应的接口 Token。最后，直接执行了百度推送脚本（需要先通过 `chmod` 添加脚本执行权限），并将结果输出到了桌面上的 `puth_log.txt` 文件中。

我们也可以不用上面的命令，直接使用 `<node_home>/node <script_home>/index.js` 执行脚本，其中 `node_path` 使用绝对路径。

## 如何实现

实现起来十分的简单，首先请求网站的站点地图文件，然后解析 xml 文件提取欲提交的 url。最后，直接调用百度站长上的提供的接口，发送一个 POST 请求即可。具体代码如下：

```javascript
#!/usr/bin/env node

console.log(`[${new Date().toLocaleString()}] -- 开始推送`);
const request = require('request');
const parseString = require('xml2js').parseString;

const URL = process.env.URL; // 提交到百度的网址
const TOKEN = process.env.TOKEN; // 百度站长主动推送token

fetchUrlDatas().then(urls => push(urls));

function push(urls) {
  if (!urls || !urls.length) {
    console.error('欲推送的地址数目不能为空');
    return;
  }

  const options = {
    url: `http://data.zz.baidu.com/urls?site=${URL}&token=${TOKEN}`,
    method: 'POST',
    headers: {
      'Content-Type': 'text/plain',
    },
    body: urls.join('\n')
  };

  request(options, (err, res, body) => {
    if (err) {
      console.error(`推送Url时遇到问题: ${err.message}`);
    } else {
      let result = JSON.parse(body);
      console.log(`[${new Date().toLocaleString()}] -- 推送完成。成功推送 ${result.success} 条url，今天剩余 ${result.remain} 条可推送url。`);
      console.log('------------------------------------');
    }
  });
}

function fetchUrlDatas() {
  return new Promise((resolve, reject) => {
    request(`http://${URL}/sitemap.xml`, (err, res, body) => {
      if (err) {
        console.error(`获取sitemap时遇到问题: ${err.message}`);
        reject(err);
      } else {
        extractUrls(body).then(
          urls => resolve(urls),
          err => reject(err)
        );
      }
    });
  });
}

function extractUrls(body) {
  return new Promise((resolve, reject) => {
    parseString(body, (err, result) => {
      if (err) {
        console.error(`解析sitemap时遇到问题：${err.message}`);
        reject(err);
      } else {
        let urls = result.urlset.url.map((url) => {
          return url.loc[0];
        });
        console.log(`从sitemap中提取${urls.length}个url：\n${urls.join('\n')}`);
        resolve(urls);
      }
    });
  })
}
```
