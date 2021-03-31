---
title: "k8s对象runAsUser"
subtitle: ""
date: 2021-03-31T11:03:19+08:00
lastmod: 2021-03-31T11:03:19+08:00
draft: true
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: ""
license: ""
images: []

tags: ["经验", "kubernetes"]
categories: ["经验"]
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

通过官网文档我们知道了可以指定

```yaml
securityContext:
  runAsUser: 0
  runAsGroup: 0
  fsGroup: 1000
```

实现容器内进程以用户0、用户组0和额外用户组1000的权限运行。

可是这个应该放哪呢？

有两个位置：

1. `containers`列表的根下，也就是和`name`同级
2. `containers`外层的spec下，也就是和`containers`同级

第二个位置表示为所有的container设置，第二个位置会覆盖第一个位置。