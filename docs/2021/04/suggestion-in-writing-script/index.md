# 写脚本的建议


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


