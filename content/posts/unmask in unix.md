---
title: "探索linux - umask"
subtitle: "apue学习"
date: 2021-05-10T19:07:43+08:00
lastmod: 2021-05-10T19:07:43+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: ""
license: ""
images: []

tags: ["linux", "基础"]
categories: ["Linux"]
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

在[CS631课程](https://youtu.be/bRAR2bv2HSM)中，详细讲了umask的作用。但是我仍然充满疑惑，这篇博客记录了探索umask的实验过程。

## 定义

某个进程创建文件时linux会指定默认权限，umask用于控制这个默认权限。这是进程的属性。

顾名思义，umask意味遮蔽，umask中设为1的位会屏蔽。

## 命令行实验

### 实验一：新建文件

```bash
01:13:35 charmer@arch unmask → touch test.`umask`
01:13:43 charmer@arch unmask → ls -l
total 0
-rw-r--r-- 1 charmer charmer 0 May 11 13:13 test.0022
```

可以看到，umask=0022时，对于group、other的写权限被限制；

> 为什么执行权限也没有呢？
>
> umask对于文件的限制是从666开始缩减的(文件默认不存在执行权限)；而文件夹需要执行权限去访问内部的条目，所以对于文件夹而言是从777开始缩减；

### 实验二：新建文件夹

```bash
01:19:22 charmer@arch unmask → mkdir test2.`umask`
mkdir: created directory 'test2.0022'
01:19:33 charmer@arch unmask → ls -l
total 4
-rw-r--r-- 1 charmer charmer    0 May 11 13:13 test.0022
drwxr-xr-x 2 charmer charmer 4096 May 11 13:19 test2.0022
```

同样的，对于文件夹而言，group、other的写权限被限制；

### 实验三：修改umask

```bash
01:22:03 charmer@arch unmask → umask 0077
01:22:10 charmer@arch unmask → touch test3.`umask`
01:22:22 charmer@arch unmask → mkdir test4.`umask`
mkdir: created directory 'test4.0077'
01:22:31 charmer@arch unmask → ls -l
total 8
-rw-r--r-- 1 charmer charmer    0 May 11 13:13 test.0022
drwxr-xr-x 2 charmer charmer 4096 May 11 13:19 test2.0022
-rw------- 1 charmer charmer    0 May 11 13:22 test3.0077
drwx------ 2 charmer charmer 4096 May 11 13:22 test4.0077
```

无需多言，符合预期。

## 疑问一

umask到底是如何实现限制呢？是touch、mkdir这些命令在创建文件时读取umask的值，对文件进行限制吗？还是他们创建文件后内核自动调整呢？让我们接着做点实验。

## 代码实验

代码：

```c
#include <sys/types.h>
#include <sys/stat.h>

#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int
main(void){
    int fd;

    // --- 0022
    umask(0022);
    if ((fd=open("test5.0022", O_CREAT | O_EXCL | O_WRONLY,
                              S_IRUSR | S_IWUSR | S_IXUSR |
                              S_IRGRP | S_IWGRP | S_IXGRP |
                              S_IROTH | S_IWOTH | S_IXOTH)) == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    (void)close(fd);
    mkdir("test6.0022", 0777);

    // --- 0077
    umask(0077);
    if ((fd=open("test7.0077", O_CREAT | O_EXCL | O_WRONLY,
                              S_IRUSR | S_IWUSR | S_IXUSR |
                              S_IRGRP | S_IWGRP | S_IXGRP |
                              S_IROTH | S_IWOTH | S_IXOTH)) == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }
    (void)close(fd);
    mkdir("test8.0077", 0777);
    exit(EXIT_SUCCESS);
}
```

运行结果：

```bash
01:36:26 charmer@arch unmask → cc umask_test1.c -o umask_test1.out
01:36:33 charmer@arch unmask → ./umask_test1.out
01:36:35 charmer@arch unmask → ls -l
...
-rwxr-xr-x 1 charmer charmer     0 May 11 13:36 test5.0022
drwxr-xr-x 2 charmer charmer  4096 May 11 13:36 test6.0022
-rwx------ 1 charmer charmer     0 May 11 13:36 test7.0077
drwx------ 2 charmer charmer  4096 May 11 13:36 test8.0077
...
```

不难发现，我们要求创建777的文件、目录，都被系统自动umask了。

也就是说，**通过系统调用创建文件、文件夹时，内核会自动将用户需要的权限umask一下**。

## 疑问二

那么umask存储在什么地方呢？我第一时间想的是PCB，毕竟这是个进程的属性。

在[sched.h - include/linux/sched.h - Linux source code (v5.12.2) - Bootlin](https://elixir.bootlin.com/linux/latest/source/include/linux/sched.h#L649)中查找，未能发现umask相关的属性。那么会存储在哪里呢？

这个疑问还得等我再探索探索。