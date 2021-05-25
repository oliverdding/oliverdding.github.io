---
title: "如何平衡工作和生活"
subtitle: "记录一次与康哥的对话"
date: 2021-05-26T00:44:58+08:00
lastmod: 2021-05-26T00:44:58+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: ""
license: ""
images: []

tags: ["碎碎念", "生活", "工作"]
categories: ["人生信条"]
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

入职腾讯半年多来，前半段一直在做restsql。

现在总结起来就是一团糟，回顾下整个流程。

首先讲讲部门没做到的点：

1. 完全没有支持。把任务交给实习生后，只开过一次会，讲了下需求。当时还忙着复习准备考试。
2. 前端也交给我做。我是完全不会前端技术栈的，改写的特别痛苦也全是bug。
3. 没有仔细规划，没有需求导向，没有明确目的。

再讲讲自己的不足：

1. 贪功冒进。发现小问题就改动，最后大偏差。
2. 过于洁癖，喜欢重构。对旧代码充满怨念，觉得哪哪都设计的不好。自己设计也充满妥协，最终还可能不如原代码，至少能跑起来。
3. 没有及时沟通。整个流程一直自娱自乐，没有和导师做足沟通和探讨。
4. 偏见过大。对python、js充满偏见，导致自己完全写不好大一点点的工程。

原本的restsql勉强可以运行，但是出现了状况，pg的connection复用导致事务待回滚或提交的异常。而新的restsql又无法部署（事实上可以部署了，但是我心里完全没底，知道这是个无底洞。事实上本地都没怎么测试过，缺乏测试条件和环境）。

最终我快崩溃了，我决定和康哥（我的导师）坦白，讲明白前因后果。

康哥万岁！康哥说实习生不应该过得这么痛苦，他帮我推掉这个任务。但是我也应该要好好反思为什么会做得这么差劲。

---

好了进入正文:)

康哥觉得，工作和生活的平衡全靠自己把握，首先我所在的部门没有加班的风气，大家每天正常上下班，一年只有一两次，临时发版出现问题可能需要熬夜加班。

他将每日的时间分成10份，其中，只有1份用于编程（我也觉得，编程时间不能过多，除非是编码一线）；2、3份用于提升自身，看看论文、行业领先做法案例等等；5份用于沟通、对接、开会与解决日常琐事。

我比较赞同。

除此之外，对于职业规划上，康哥建议我走后台开发的方向，我也比较喜欢；至于深度广度上，我想走技术骨干路线。