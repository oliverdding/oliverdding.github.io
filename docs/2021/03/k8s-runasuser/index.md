# k8s对象runAsUser


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
