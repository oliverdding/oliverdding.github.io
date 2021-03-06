---
title: "与FreeBSD一见钟情"
subtitle: "从没如此想要立刻深入了解过什么"
date: 2021-02-03T00:40:58+08:00
lastmod: 2021-03-17T00:40:58+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "FreeBSD系统, 作为UNIX的忠实传承者..."
license: ""
images: []

tags: ["操作系统", "折腾"]
categories: ["系统"]
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

我这个人对计算机有很重的洁癖, 若系统有一点不满足我的想法, 我会很难受很厌恶.

由于家里没有矿, 22年来经手的电脑共5台, 3台台式, 算是我的启蒙电脑, 都是家父为了让我跟上潮流学会电脑: "现在社会干什么都离不开电脑, 必须接触接触". 最终沦落为我的游戏机, 那无数日夜和我的植物们一起守护戴森的日子真是令我怀念. 而后独立生活(屁嘞还在用家里的💴), 拥有了两台笔记本. 开始了自己的折腾之路.

大笔记本是2018年的惠普的暗影精灵4顶配版, 8代i7+GTX1080, 自己加上了一条8G内存组成双通道16G, 将1T机械硬盘卸下换成500G固态, 它从大一开始陪着我, 我用它进入了了linux的世界, 从ubuntu到linux mint到archlinux最后回到windows. 而小电脑则是2019年大二入手的一台联想小新pro13amd版, 是AMD Ryzen 5 3550H with Radeon Vega 8.

## OEMN

首先讲讲自己的暗影精灵, 我叫它oemn. 高中就接触过linux, 不过是虚拟机, 由于学习c语言需要, windows下使用codeblocks编程一直不错, 但是老师说想学习c语言还是要用linux, 我信了他的鬼话, 直接格盘安装ubuntu. 不太记得什么情况了, 当时用得很不舒服, 于是听从了学校linux用户组老大的建议换成mint. 一直用了一两个月, 发现自己对linux, 对c语言其实没啥特别的进展, 而且gui还是不是卡一卡, 系统时不时出现问题, 特别是续航直接一泻千里...

一个暑假, 我在网上受网友怂恿, 直接安装上了archlinux, 开始了罪恶的半年. 整整半年, 不用gui, 仅仅是tty, 使用各种命令行工具. 我当时还有点骄傲...哈哈哈年轻真好...那半年对我来说其实没什么, 但其实又有什么, 唔...

好处:

1. 各种命令行工具熟练使用, 包括vim, tmux这些效率工具.
2. 兴趣来了.

坏处:

1. 伤眼睛...
2. 事实上深入的知识一点没学, 比如说文件系统, 比如说进程线程, 比如系统调用...

后来, 进入了嵌入式方向, 不得不回到win使用keil这一些嵌入式的工具, 就断绝了使用linux的想法.

## prothirteen

到了小新pro13的时代了. 我购买了小新pro13, 作为不想背千斤重的游戏本去上课的替代品. 说真的, 颜值, 续航都令我十分满意. 可是生命在于折腾啊, 我开始了折腾之旅...

试了无数的发行版, 由于硬件太新了, 始终存在sleep后无法唤醒的问题, 尝试了无数方案, 在archwiki论坛也提问了, 最终没能解决, 于是回到win.

到了2021年, 在腾讯实习的时候, 一直需要用helm, kubectl, docker等等工具, 我有尝试manjaro. 终于, 5.9内核的linux不存在无法唤醒的问题了, 当时嘛...

1. 自从知道了wayland和x, 就完完全全不想碰x, 只想用wayland, 甚至xwayland都不想要(洁癖开始了). 可是根本没几个可用的, 本身还有很多很多bugs和瑕疵, 特别是hidpi.
2. 续航尿崩, win下10小时, linux下2小时.

好嘛, 我回win了, hyper-v+window terminal我又是好汉了...不想折腾了, 就这样使用吧...

![image](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/arch 2021-03-17 135752.png)

## FreeBSD

公司给我发了一台iMac电脑, 这是我第一台unix系的电脑. 在搜索过程中我发现了FreeBSD. 进而了解了他们的故事.

目前我对FreeBSD完全没有了解, 只是简单安装在了Hyper-V上, 也就是简单配置了下. 但是系统的设计理念, 系统的哲学非常让我愉悦, 这才是K.I.S.S, 这才是正在的小而美.

终于到正题了喵喵喵? 好像跑题了

总之就是想深入bsd的源码...

目前安排是, 先从apue入手, 把unix的外层褪下, 再慢慢深入