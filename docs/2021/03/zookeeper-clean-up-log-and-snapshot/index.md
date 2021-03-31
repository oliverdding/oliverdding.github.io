# Zookeeper清理事物日志和snapshot


zookeeper会生成大量的日志和快照文件，结果它竟然只管杀不管埋，我在部署大约60节点的druid集群时，运行不到3天就挂了(zookeeper盘满了)。

想要zookeeper自动清理这些文件，于是有了本文。

---

首先，日志和快照的目录由${ZOOKEEPER_HOME}/conf/zoo.cfg文件指定，在我的例子中分别是:

1. dataLogDir=/var/libzookeeper/log
2. dataDir=/var/lib/zookeeper/data

由于我的部署是helm+k8s的方式，无法使用cron、systemd这些工具设置定时清除的任务。于是有以下三种方式解决：

1. zookeeper自动清理
2. 自定义脚本
3. zookeeper自带zkCleanup.sh

## zookeeper自动清理

[ZooKeeper Administrator's Guide (apache.org)](https://zookeeper.apache.org/doc/r3.4.0/zookeeperAdmin.html)官方文档指出：

> Automatic purging of the snapshots and corresponding transaction logs was introduced in version 3.4.0 and can be enabled via the following configuration parameters **autopurge.snapRetainCount** and **autopurge.purgeInterval**. For more on this, see [Advanced Configuration](https://zookeeper.apache.org/doc/r3.4.0/zookeeperAdmin.html#sc_advancedConfiguration) below.

可以在${ZOOKEEPER_HOME}/conf/zoo.cfg文件中指定

* **autopurge.snapRetainCount**：需要保留的文件数量。默认是3。
* **autopurge.purgeInterval**：指自动清理频率，单位是小时。默认是0，表示不自动清理。

注意: 这个方法存在问题: [AutoPurge Not Working · Issue #30 · 31z4/zookeeper-docker (github.com)](https://github.com/31z4/zookeeper-docker/issues/30)。很不幸，我的集群中（zookeeper v3.4.10）也无法使用。

## 自定义脚本

互联网上存在大量的相关脚本，这里就不赘述了。值得一提的是，如何将脚本定期执行呢？

我的方案是，通过k8s的livenessProbeness功能实现:

```yaml
livenessProbe:
  exec:
    command: ["ls -t /var/lib/zookeeper/log/version-2/log.* | tail -n +21 | xargs rm -f && ls -t /var/lib/zookeeper/data/version-2/snapshot.* | tail -n +21 | xargs rm -f && zkOk.sh"]
```

非常简单就不解释了。值得一提的是，zkOk.sh是原本的command，用于检测这个zookeeper是否正常运作，把它放最后是因为在多个命令&&一起实行时，最后一个命令的返回码会作为总的返回码。

## zookeeper自带zkCleanup.sh

具体使用看这个：[StackOverflow](https://stackoverflow.com/a/48126338/13760009)

我没能成功使用=-=，一直参数错误，giao...
