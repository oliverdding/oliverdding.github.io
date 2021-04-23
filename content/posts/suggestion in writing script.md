---
title: "写脚本的建议"
subtitle: ""
date: 2021-04-23T16:46:58+08:00
lastmod: 2021-04-23T16:46:58+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "今日协助排查银行问题，自己的脚本出现问题，在线调试=="
license: ""
images: []

tags: ["经验", "工作"]
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

第一个问题是自己基础不够，不经常写bash。

当要传递命令，特别是包含管道的命令时，使用sh/bash的-c参数：

```sh
sh -c "${command here}"
```

这个问题是用kubectl exec传入一次性命令时排查问题遇到的。

第二个问题比较弱智而且很危险了。

需求是将一个目录下的所有文件truncate。

我给出的脚本是：

```sh
ls -1 ${PATH_TO_TRUNC} | xargs -s 0
```

问题在哪？

kubectl exec进去后，所在目录是`/`，也就是说，xargs收到的参数是`/${FILE_TO_TRUNC}`，找不到...所以没有任何效果

这里给出一个最佳实践：

> 写脚本请一定使用绝对路径，对相对路径敏感些

方案就是先移动到目标目录:

```sh
cd ${PATH_TO_TRUNC}; ls -1 . | xargs -s 0
```

