---
title: "HTTP协议常见问题"
subtitle: "面试常见问题锦集"
date: 2021-01-05T11:49:30+08:00
lastmod: 2021-01-05T11:49:30+08:00
draft: true
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "HTTP相关问题整理"
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

## HTTP结构：

1. 请求报文: 请求行、首部行、实体主体
2. 应答报文: 状态行、首部行、实体主体

## HTTP请求报文类型: 

1. **GET：** 用于请求访问已经被URI（统一资源标识符）识别的资源，可以通过URL传参给服务器。
2. **POST：**用于传输信息给服务器，主要功能与GET方法类似，但一般推荐使用POST方式。
3. **PUT：** 传输文件，报文主体中包含文件内容，保存到对应URI位置。
4. **HEAD：** 获得报文首部，与GET方法类似，只是不返回报文主体，一般用于验证URI是否有效。
5. **DELETE：**删除文件，与PUT方法相反，删除对应URI位置的文件。
6. **OPTIONS：**查询相应URI支持的HTTP方法。

## GET和POST区别

先从看得到的讲: 

GET可以被浏览器收藏、能缓存、刷新后无害；POST不可收藏、不可缓存、刷新后会被重新提交。

---

再从报文上讲:

1. 请求行不同，一个是GET一个是POST
2. 参数方式不同，GET在url中，POST在body中

---

本质上讲，**没有区别**，都是基于TCP协议，添加了请求/状态行、首部行和实体主体。完全可以在GET中塞入request body传数据，也可以在POST中往URL塞数据。

## HTTPS

HTTP相比于HTTP多了一个S, 即使SSL/STL协议, 大致流程如下:

1. 完成TCP三次同步握手
2. 客户端验证服务器数字证书，通过，进入步骤3
3. DH算法协商对称加密算法的密钥、hash算法的密钥
4. SSL安全加密隧道协商完成
5. 网页以加密的方式传输，用协商的对称加密算法和密钥加密，保证数据机密性；用协商的hash算法进行数据完整性保护，保证数据不被篡改