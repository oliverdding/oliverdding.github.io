---
title: "如何解决问题"
subtitle: "一步步走"
date: 2021-05-26T00:59:48+08:00
lastmod: 2021-05-26T00:59:48+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: ""
license: ""
images: []

tags: ["经验", "生活", "工作"]
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

我在做SRE相关的事情时，发现自己真的很容易拖拉、避开问题。

先是报表没有数据，这个问题仔细寻找原因按理说最多一天，但是我拖了一周。这期间时间花在各种奇怪的地方，比如多个方向都去试，碰壁了就放下试另一条路。

现在报表数据异常，我一直忽视它，觉得有数据可以了。

终于，rowan和我沟通，我意识到自己的缺点，我需要重视这个现象。再想到康哥谈话时谈及的解决问题的路径，我添加人生信条：

遇到问题后，依次

1. 分析表面现象，分析原因，按照可能性从大到小排序/从最常见原因开始分析
2. 针对每个原因去证明不是这个原因，若不能证明则找到原因
3. 找到原因后，解决问题/若是没找到，继续寻找不太可能的原因

不要想着问题多多，不要满足现状。

对于问题的发现，可以考虑“与预期结果不符”，即为异常。

希望自己能牢记这一点：“遇到问题，解决问题”。