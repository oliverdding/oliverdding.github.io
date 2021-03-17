# unix文件系统


> 引子: 还记得大二操作系统课上, 在讲到文件系统时, 我问了老师问什么剪切文件(夹)后粘贴会如此迅速访问没有经过拷贝, 而复制啊, 跨盘啊都会经过读盘. 老师莞尔一笑说这个问题我交给你自己探索.

本文将从较为底层的方式, 将我理解的APUE 4.14小节部分道出.

## 总体架构

首先, 文件系统中最大的单位是`disk`, 也就是一块存储, 机械硬盘、SSD、flash甚至SD之类的. `disk`会依照我们的需要分为多份`file system`(这里指文件系统格式, 比如ext4、fat32之类), 而每个`file system`头两个`block`分别是`boot block`和`super block`. 之后的空间被分为一个个`cylinder group`, 编号从0-n. 而对于每个cylinder group, 依次会存在`super block copy`, `cg info`, `inode map`, `inodes`和`data blocks`.

![structure](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/disk%2021-03-17%153411.png)

我们今天就聚焦在`inodes`和`data blocks`.

## 元素

uinux中一切皆文件, 我们人为的将unix文件划分为7类:

1. `Regular file`.
2. `Diectory file`.
3. `Block special file`.
4. `Character special file`.
5. `FIFO`.
6. `Socket`.
7. `Symbolic link`.

每个inode代表一个文件, 存储了文件的各种相关信息: 文件类型, 文件访问权限位, 文件大小, 指向`data blocks`的指针们等等. 大部分`stat`系统调用的信息来源于inode. 但有两个例外: 文件名和inode号(没错你妹看错, inode不知道自己的name和id).

### Regular file

![structure](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/file%2021-03-17%153543.png)

指向`data blocks`的指针们便是文件的真正内容, 注意, 指针指向的是一块块数据块, 也就是说无论用不用的完, 这一块就是分配给你了. 这里就导致`du`命令和`ls`命令查到的大小不同, 因为一个是磁盘上真正占用的空间, 一个是文件的大小.

> 这里我又有个疑问, 要是文件巨大无比, inode的指针预留空间无法存放完, 这该怎么办? 若是我我会考虑二级指针. 欸...留个坑, 后续填...

### Directory file

![structure](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/directory%2021-03-17%153559.png)

对于目录文件, 指向的`data blocks`存储着一对对`inode number`:`file name`. 并且, 对于每个目录文件, unix系统默认插入两对数据: `inode number x`:`.`与`inode number y`:`..`. 没错, 这就是大名鼎鼎的当前目录和父目录. 注意对于`/`节点来说, `.`和`..`都是指向当前节点.

这里顺带提出`hard link`的概念. 所谓的`hard link`也就是在一个目录文件中插入了源文件`inode number`的record. 也就是说, `hard link`之间的`inode number`都是相同的, 可以用命令`ls -i`查看验证. 而在inode数据结构中有一个属性是`st_nlink`, 代表着文件引用计数. 每每多一个`hard link`都会导致这个变量+1.

> 啊这里我又又有疑问了, 我试着unlink这两个特殊的record, 结果失败了, 我想知道有没有方法可以干掉它们?

### Block special file

> 留个坑

### Character special file

> 留个坑

### FIFO

> 留个坑

### Socket

> 留个坑

### Symbolic link

> 待验证

软链接文件非常特殊, 它有自己的inode, inode的指向的`data block`存储着调用`symlink`系统调用时传入的源路径(无论是相对还是绝对都会原封不动填入).

## 引子解答

对于同一文件系统的移动, 仅仅是删除原来目录文件下的record并在目标目录文件下创建新record, inode和真正的`data blocks`并无任何变动, 这样在我们看来就是一瞬间(windows的文件系统也有相似的概念).


