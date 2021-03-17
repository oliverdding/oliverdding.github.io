---
title: "linux boot process(UEFI)"
subtitle: "基于Arch探索linux的启动过程"
date: 2021-03-17T13:26:45+08:00
lastmod: 2021-03-17T13:26:45+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "一直很好奇操作系统的启动过程，而且一直以来对BIOS、UEFI只知道UEFI更新，现在深入了解UEFI的启动过程"
license: ""
images: []

tags: ["linux", "操作系统"]
categories: ["Linux", "系统"]
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

> 注意: 不涉及Network booting、secure booting, 略微提到BIOS

> 注意: 本文基于ArchLinux, 使用UEFI+System Boot, 而不是grub

archlinux启动过程如下:

```mermaid
graph LR
	A[UEFI] --> B[Boot Loader]
	B --> C[Kernel]
	C --> D[initramfs]
	D --> E[init]
	E --> F[getty]
```

## UEFI

固件(firmware)是指提供对具体硬件底层直接控制的软件, 一般比较小. 而目前的固件类型有BIOS(Basic input output system)和UEFI(Unified Extensible Firmware Interface)两种, 他们的区别的联系可以看:

[UEFI, BIOS, GPT, MBR - What's the Difference? - Fossbytes](https://fossbytes.com/uefi-bios-gpt-mbr-whats-difference/)

当开机键被按下:

1. 首先启动UEFI, 它会进行硬件启动自检(Power-On-Self-Test, 简称POST).

2. UEFI加载启动过程需要的硬件, 比如硬盘、键盘驱动(较新的UEFI甚至会有鼠标)等等.

3. UEFI读取[NVRAM](https://en.wikipedia.org/wiki/Non-volatile_random-access_memory)中的第一条启动项. 启动项顺序可以在开机时连按ESC后进入修改启动选项.

   启动项往往是UEFI扫描硬盘的ESP(EFI System Partition)后写入NVRAM的. 扫描的时机不清楚, 但是每每插入系统安装U盘, UEFI中可以看看到启动选项, 应该是每次开机都会扫描?

   对于我的系统来说, 就是这个文件:

   ![My Boot Entry](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/boot 2021-03-17 142514.png)

4. UEFI执行对应的EFI应用程序. 此时把控制权交到了EFI应用程序手里.

   EFI应用程序有两种:

   1. boot loader. 例如本文使用的System Boot, 或者现在大一统的Grub等等.
   2. linux内核自己(使用EFISTUB)

## Boot Loader

Boot Loader种类繁多, 具体可以看archwiki页面:

[Arch boot process - ArchWiki (archlinux.org)](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)

拿System Boot举例, 安装好system boot后, 需要手动添加一条entry:

![My Boot Entry](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/entry 2021-03-17 143254.png)

这个文件会在system boot的启动页面添加一条entry, 它加载linux kernel、微码和initramfs文件.

## Kernel

kernel是操作系统的核心。它在底层(内核空间)运行，在机器的硬件和使用该硬件运行的程序之间进行交互。

继续上图, `vmlinuz-linux`就是内核. 为什么又vm呢? 这个表示Virtual Memory. 为什么是linuz呢? 这个表示压缩过的.

## Initramfs

kernel启动后, 就会解压initramfs(initial RAM filesystem)压缩包到空的rootfs(initial root filesystem, specifically a ramfs or tmpfs)中.

首先解压的initramfs是内核构建期间嵌入内核二进制文件的initramfs，之后解压外部initramfs文件。因此，外部initramfs中的文件会覆盖嵌入initramfs中同名的文件。

再之后, kernel会在rootfs中执行`/init`, 这个将作为第一个进程. 此时, 早期用户空间启动(early userspace).

> 对于archlinux而言, 内部的initramfs是空的, 外部的文件是通过mkinitcpio、dracut或者booster生产的.

![My Boot Entry](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/modules 2021-03-17 145317.png)

载入kernel的目的显而易见, 那么为啥要又initramfs这一步呢? 目的是为了引导系统(bootstrap, 拔鞋带, 这里作引申意)到可以访问root文件系统. 这意味着任何需要访问设备(比如IDE、SCSI、SATA、USB/FW)的模块必须是从initramfs中可载入的(如果内核没提供). 一旦正确的模块被载入, 启动进程将会继续. 因此, 写在`/etc/mkinitcpio.conf`文件中的modules都是访问根文件系统所必须的模块, 而不是所有模块. 其他的模块会在接下来的`init`进程中由`udev`加载.

## init process

这是早期用户空间的最后一步, 载入正在的root文件系统(real root), 并代替先前的`initial root`文件系统. 调用`/sbin/init`进程, 替代`/init`进程. 现在的linux都使用`systemd`作为默认的init进程.

> 这一点也不K.I.S.S

## getty

init会在每个虚拟终端(6个左右)都调用getty, 这会初始化tty并弹出输入账户密码的提示, 之后会将用户输入的账户密码和`/etc/passwd`和`/etc/shadow`进行对比, 成功后会调用`login`, 算是正式进入系统.

## Reference

1. [Arch Boot Process](https://wiki.archlinux.org/index.php/Arch_boot_process)
2. [Unified Extensible Firmware Interface](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface)
