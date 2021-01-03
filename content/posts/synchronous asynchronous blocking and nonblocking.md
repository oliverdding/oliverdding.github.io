---
title: "Synchronous, Asynchronous, Blocking and Non-blocking"
subtitle: "同步, 异步, 阻塞与非阻塞的辨析"
date: 2021-01-02T10:55:50+08:00
lastmod: 2021-01-02T10:55:50+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "记录下辨析这四个容易混乱的概念"
license: ""
images: []

tags: []
categories: []
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

在大三学习中, 经常会遇到这四个名词交替出现, 概念比较模糊. 于是记录下自己的理解, 留作备份.

> 说真的, 对于这类概念的东西, 需要真正用在实处才能真正掌握, 如果没有遇到就去"学", 那只能说记住了...

## 同步与异步

同步(Synchronous)和异步(Asynchronous)描述方法调用结果返回的过程.

* 同步: 调用一旦开始，调用者必须等到被调用者返回结果后, 才能继续后续的行为
* 异步: 调用方法后就会立即返回, 调用者就可以继续后续的操作, 而调用者在后台真正地运行

> 重点在于:
> 1. **有效的**异步不是随便实现的, 必须是任务之间没有关联, 不存在先后关系的. 比如我烧水时可以去读报纸, 但是不能泡咖啡(要等水烧开)
> 2. 异步需要提供相应的消息机制, 得以通知调用者获取结果

## 阻塞与非阻塞

阻塞和非阻塞关注的是调用者在等待调用结果时的状态, 专指调用者的.

* 阻塞: 是指调用结果返回之前, 当前线程会被挂起. 调用者所在线程只有在得到结果之后才会继续往下执行
* 非阻塞: 是指在不能立刻得到结果之前, 调用者所在线程不会被阻塞, 而可以继续往下执行

## 例子

假设你刚刚起床, 从要做的一堆事情里面拿出三件事连续的事情:

1. 烧水
2. 泡咖啡
3. 看报纸

|    | 阻塞                            | 非阻塞                               |
|:---:|:-----------------------------:|:---------------------------------:|
| 同步 | 你先烧水, 并且盯着水烧开后泡咖啡, 然后看报纸      | 你先烧水, 发呆并时不时看一眼水烧开没, 然后泡咖啡, 最后看报纸 |
| 异步 | 你先烧水, 然后发呆, 直到听到水响了泡咖啡, 最后看报纸 | 你先烧水, 然后看报纸, 听到水响了泡咖啡, 继续看报纸      |

在例子中, 同步异步重点在于水烧开这个结果你是如何获取的. 同步时你一定要看到水烧开这个结果返回了, 你才继续干别的事情; 异步时你不管水怎么样了, 直接做别的事情, 等到水烧开了发出响声你再来处理.

阻塞非阻塞则关注你烧水后干啥了, 阻塞时你只能等水烧开才能继续后续事情; 非阻塞时你能继续做别的事情.