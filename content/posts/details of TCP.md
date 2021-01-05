---
title: "TCP细节: 为什么三次握手, 为什么四次挥手, 为什么有TIME_WAIT状态"
subtitle: "面试常问问题锦集"
date: 2021-01-05T11:27:26+08:00
lastmod: 2021-01-05T11:27:26+08:00
draft: true
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "你一定也在面试中遇到这几个问题吧? 不妨进来看看我是怎么想的, 对比下自己的思路."
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

## 为啥是三次握手

在本科课本中说的是防止失效的连接请求, 在连接完成后又发送到服务器。

但是我觉得这太**扯蛋**了，我的理解是:

握手的过程是双方约定一些东西的过程，A对B约定了自己的sequence number, 以及一些其他控制信息比如拥塞窗口, 拥塞缩放因子等等，B收到这些约定, 当然也要约定自己的信息, 并对A的约定做出回复; 第三次握手, 是A告诉B我收到的你的约定, 这样经过三次握手, 双方都告知对方自己的信息, 并都确认对方收到了自己的信息.

## 为啥是四次挥手

TCP是全双工协议，双方同时发送数据，因此断开连接时需要一步一步断开，一端没数据了就告知对方，并且收到回复。另一方仍然还有数据就继续发，没有了就也告知对方并收到回复。

这样就产生了四次挥手. 那么为什么握手是三次呢? 因为握手是不存在*一方还有数据*的情况的, 毕竟双方通信都没建立. 因此握手时B会把ACK和自己的SYN合并为一个包, 而挥手时不行, 因为自己还可能有数据.

## 为啥TIME_WAIT状态还要等2MSL

1. 确保ACK被接受. 自己最后发送的ACK有可能丢失, 等待一定时间留有机会重发ACK.
2. 为了防止下一个使用相同四元组的连接被超时抵达的FIN报文干扰. 也就是说,2MSL保证了上一组通信的报文在互联网上**完全消失**.