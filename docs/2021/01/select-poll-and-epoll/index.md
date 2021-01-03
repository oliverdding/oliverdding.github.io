# Linux中I/O多路复用(Select Poll and Epoll)


> 废话时间: 
> 
> 我第一次接触I/O多路复用是在自己人生中第一次外包项目, 给学校某个实验室的一个FPGA开发上位机. 使用串口的方式发送接受数据.
> 
> 一开始我用一个线程去监听串口, 处理数据并调用UI dispatcher更新界面, 结果处理过慢, 串口的FIFO缓冲区爆了.
>
> 接着我用一个线程去监听数据, 收到数据就调用其他线程去处理, 展示数据. (这就是I/O多路复用的雏形了, 当时并不知道)
> 
> 还有后续, 由于串口是一字节一字节发送的, 而我需要的数据都为7字节, 可能会存在半包导致. 于是我的办法是将数据读取到一块公共内存, 消费者线程要待数据足够才去消费.

## 概念

> I/O多路复用(multiplexing)的本质是通过一种机制(系统内核缓冲I/O数据), 让单个进程可以监视多个文件描述符,一旦某个描述符就绪(一般是读就绪或写就绪), 能够通知程序进行相应的读写操作.

用自己的话说, I/O多路复用就是让一个进程同步, 阻塞地监听多个文件描述符, 以此减少系统中地线程数量, 减少资源消耗, 提高负载能力.

而Linux提供了Select, Poll和Epoll三种I/O复用的方式.

### *nix四种IO模型

这里推荐这一篇[文章](https://zhuanlan.zhihu.com/p/115912936), 讲得很清晰明了.

1. blocking I/O
2. nonblocking I/O
3. IO multiplexing
4. signal driven I/O
5. asynchronous I/O

### *nix的文件描述符和句柄

每个进程都会有一个`打开的文件表`(fdtable), 表中每一项都是`struct file`类型, 包含打开的文件的一些属性, 比如偏移量, 读写访问模式等等, 这个表的每一个元素就是一个文件句柄. 而文件描述符就是这个句柄在这个fdtable中的下标.

## Select

select提供fd_set这一数据结构(一个long类型数组组成的structure), 程序员将其存储一系列文件句柄, 调用select()时, 由内核根据I/O状态从fd_set中删除无对应事件发生的句柄, 由此来通知执行select()的进程哪一个文件可读可写或发生异常.

因为是在注册好的fd_set中删掉没有相应事件的句柄, 因此可想而知select会有很多多余的操作.

### 函数声明

```c
int select(int maxfdp1,fd_set *readset,fd_set *writeset,fd_set *exceptset,const struct timeval *timeout);
```

操作fd_set集合的宏有:

```c
int FD_ZERO(int fd, fd_set *fdset);   
int FD_CLR(int fd, fd_set *fdset);   
int FD_SET(int fd, fd_set *fd_set);   
int FD_ISSET(int fd, fd_set *fdset);
```

### 缺点

1. 每次调用select()都需要把fd_set从用户态拷贝到内核态.
2. 每次调用select()都需要在内核遍历传递进来的fd_set, 返回时只返回数量, 同样需要遍历以找到对应的文件句柄.
3. 每次调用select()都需要重新注册句柄和关心的事件.
4. 为了减少数据拷贝的消耗, fd_set集合大小被限制, 在x86下是1024, x64下是2048.

## Poll

poll和select几乎一样, 除了以下两点:

1. poll提供的pollfd数据结构, 不再有数量上的限制.
2. poll提供的pollfd数据结构, 使用额外变量revents表示事件发生与否, 因此再次调用不必重新注册关心事件与句柄.

### 函数声明

```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

typedef struct pollfd {
        int fd;                         // 需要被检测或选择的文件描述符
        short events;                   // 对文件描述符fd上感兴趣的事件
        short revents;                  // 文件描述符fd上当前实际发生的事件
} pollfd_t;
```

### 缺点

1. 每次调用poll()都需要把pollfd数组从用户态拷贝到内核态.
2. 每次调用poll()都需要在内核遍历传递进来的pollfd数组, 返回时只返回数量, 同样需要遍历以找到对应的文件句柄.

## Epoll

epoll是Linux内核版本2.6提出的, 基于事件驱动的I/O复用方式. epoll使用一个epoll句柄管理多个句柄, 将用户关心的事件存放到内核的一个事件表中, 这样用户空间和内核空间的拷贝只会发生一次.

epoll是Linux内核为处理大批量文件描述符而作了改进的poll，是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率。原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。

### 函数声明

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
```

相关数据结构如下:

```c
struct epoll_event {
    __uint32_t events;  /* Epoll events */
    epoll_data_t data;  /* User data variable */
};

typedef union epoll_data {
    void *ptr;
    int fd;
    __uint32_t u32;
    __uint64_t u64;
} epoll_data_t;

```

### 触发方式

epoll除了像select/poll提供的水平触发(level triggered)外, 还提供了边缘触发(edge triggered)方式, 用户空间可能缓存I/O事件, 减少了epoll调用.

* **水平触发**(LT): 默认工作模式，即当epoll_wait检测到某描述符事件就绪并通知应用程序时，应用程序可以不立即处理该事件；下次调用epoll_wait时，会再次通知此事件
* **边缘触发**(ET): 当epoll_wait检测到某描述符事件就绪并通知应用程序时，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次通知此事件。（直到你做了某些操作导致该描述符变成未就绪状态了，也就是说边缘触发只在状态由未就绪变为就绪时只通知一次）

## 总结

|       | select                      | poll                    | epoll |
|:-----:|:---------------------------:|:-----------------------:|:-----:|
| 操作方式  | 遍历                          | 遍历                      | 回调    |
| 底层实现  | 数组                          | 链表                      | 哈希表   |
| I/O效率 | o(n)                        | o(n)                    | o(1)  |
| 最大连接数 | 1024(x86)/2048(x64)         | 无上限                     | 无上限   |
| 拷贝    | 每次调用需要重新注册fd_set并从用户态拷贝到内核态 | 每次调用需要把pollfd从用户态拷贝到内核态 | 拷贝一次  |


## 参考

* [搞懂Select，Poll，Epoll的区别](https://www.itqiankun.com/article/select-poll-epoll)
* [IO复用之select、poll、epoll模型](https://zhuanlan.zhihu.com/p/126278747)
