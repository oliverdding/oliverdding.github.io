# Docker入门


## 什么是Docker

> **Docker** 使用 `Google` 公司推出的 [Go 语言](https://golang.org/) 进行开发实现，基于 `Linux` 内核的 [cgroup](https://zh.wikipedia.org/wiki/Cgroups)，[namespace](https://en.wikipedia.org/wiki/Linux_namespaces)，以及 [OverlayFS](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) 类的 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 等技术，对进程进行封装隔离，属于 [操作系统层面的虚拟化技术](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。 -- [《Docker--从入门到实践》](https://yeasy.gitbook.io/docker_practice/introduction/what)

Docker是C/S架构的软件, 架构如下:

![Docker架构](https://docs.docker.com/engine/images/architecture.svg)

### 基本概念

#### 镜像(Image)

镜像是一层层的文件系统联合组成的可分发的应用. 利用Union FS的技术.

这里提到Docker中最重要的特性之一: **分层存储**.

Docker构建镜像时会一层层构建, 每一层构建好后将不再发生改变, 后续基于前一层的改变都只会变动自己这一层. 比如删除前一层文件, 实际上只是在本层将这个文件标记为删除, 最终运行后看不到这个文件, 但事实上它仍然存在.

---

Docker最佳实践:

1. 每一层操作完后一定要在构建完成前将本层镜像清除干净.
2. 尽量将变动不大的层放前面, 经常变动的层放后面.

---

#### 容器(Container)

容器就是部署运行的镜像, 就像是被加载到JVM中的类, 被实例化为一个对象. 容器的实质是实体机上的进程, 运行在独立的命名空间中, 拥有自己的root文件系统、网络配置、进程空间甚至用户ID空间.

容器以镜像为基础层, 在其上创建一个当前容器的存储层, 被称为**容器存储层**. 也就是说, 容器被删除后, 该层也将被删除, 任何运行产生的信息都将被删除.

---

Docker最佳实践:

不应往存储层写入任何数据, 容器存储层要保证无状态话. 所有的文件写入应当使用**数据卷**或者**绑定宿主目录**, 在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

---

#### 仓库(Repository)

Docker Registry就是用于存储、分发用户的镜像的服务.

一个Docker Registry可以包含多个仓库, 每个仓库可以包含多个标签(Tag), 每个标签对应一个镜像.

## 为什么要使用Docker

官网挂着的`Package Software into Standardized Units for Development, Shipment and Deployment`应该可以说明问题了.

**为了解决开发、部署的一致性问题.**

原本部署方式有两种:

1. 全部部署到实体机器上.

   缺点:

   1. 依赖交叉. 可能不同应用对某个相同库的不同版本有依赖.
   2. 资源问题. 某个应用完全有可能挤占其他应用的资源.

2. 实体机器上安装虚拟机后, 将应用部署到虚拟机上.

   缺点:

   1. 太重型. 每每部署应用都要整个系统环境, 占用大量存储与内存, 以及CPU.
   2. 性能低. 虚拟机的系统调用比不上实体机.

## 如何使用Docker

待续

## 底层介绍

待续
