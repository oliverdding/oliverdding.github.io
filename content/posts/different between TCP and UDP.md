---
title: "TCP和UCP的区别整理"
subtitle: "面试常见问题锦集"
date: 2021-01-05T11:54:00+08:00
lastmod: 2021-01-05T11:54:00+08:00
draft: true
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "又到了面试锦集环节, 今天是TCP和UDP"
images: []

tags: ["计网", "基础", "面试"]
categories: ["计网"]
featuredImage: ""
featuredImagePreview: ""

hiddenFromHomePage: false
hiddenFromSearch: false
twemoji: false
lightgallery: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: true
rssFullText: false

toc:
  enable: true
  auto: true
code:
  copy: true
  # ...
math:
  enable: true
  # ...
mapbox:
  accessToken: ""
  # ...
share:
  enable: true
  # ...
comment:
  enable: true
  # ...
library:
  css:
    # someCSS = "some.css"
    # 位于 "assets/"
    # 或者
    # someCSS = "https://cdn.example.com/some.css"
  js:
    # someJS = "some.js"
    # 位于 "assets/"
    # 或者
    # someJS = "https://cdn.example.com/some.js"
seo:
  images: []
  # ...
---

|          | TCP    | UDP    |
| -------- | ------ | ------ |
| 基于     | 流     | 数据报 |
| 可靠服务 | 是     | 否     |
| 系统资源 | 高     | 低     |
| 区别     | 四元组 | 二元组 |
| 面向     | 连接   | 无连接 |
| 顺序     | 保证   | 不保证 |
| 拥塞控制 | 有     | 无     |
| 流量控制 | 有     | 无     |

这里提一嘴, TCP中拥塞控制和流量控制一定要分清楚, 许久不看就容易忘记弄混.

拥塞控制是双方之间在握手阶段协商好的拥塞控制窗口大小和拥塞控制缩放因子决定的, 控制发送速率, 防止发送过快导致接收方的缓冲区爆栈.

而流量控制是TCP协议用于充分利用网络环境的一种方式, 在网络环境不好时控制发送速率以减轻网络负担. 因此, 对于一些可以忍受少量丢包, 速率重于一切的应用场景来说, UDP会更好(比如语音, 视频).