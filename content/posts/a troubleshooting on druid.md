---
title: "一次druid异常排查"
subtitle: "大数据工程师成长之路"
date: 2021-03-30T16:22:12+08:00
lastmod: 2021-03-30T16:22:12+08:00
draft: false
author: "Charmer"
authorLink: "https://oliverdding.github.io"
description: "我根据组内需求探索部署了druid，目前收集到的数据有百亿量级，但是经常遇到各种奇怪的问题。比如今天遇到了task一直失败的问题，记录排查解决过程。"
license: ""
images: []

tags: ["大数据", "apache", "经验"]
categories: ["大数据"]
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

早上一来公司，熟练的port-forward一下router，打开web端查看下昨晚的ingestion任务执行的怎么样，嗯非常完美。

于是增加一个supervisor，紧接着任务大量失败；

那就先暂停这个supervisor，结果仍然失败，包括之前一切正常的任务。

## 现象

1. 任务大量失败，部分任务甚至未分配到ip（也就是说coordinator调度了任务，但是没给他分配资源）。

   ![没有资源的task](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/task-without-resource.png)

2. 出错的任务有两种，一种是从头到五没有资源的任务，它没任何日志，包括middlemanager和hdfs上（我们使用hdfs作为druid的deep-storage）；第二种是正常执行，但是出错，日志显示java.lang.InterruptedException: null、org.apache.kafka.common.errors.InterruptException: java.lang.InterruptedException；

   ```
   org.apache.kafka.common.errors.InterruptException: java.lang.InterruptedException
   	at org.apache.kafka.clients.consumer.internals.ConsumerNetworkClient.maybeThrowInterruptException(ConsumerNetworkClient.java:520) ~[?:?]
   	at org.apache.kafka.clients.consumer.internals.ConsumerNetworkClient.poll(ConsumerNetworkClient.java:281) ~[?:?]
   	at org.apache.kafka.clients.consumer.internals.ConsumerNetworkClient.poll(ConsumerNetworkClient.java:236) ~[?:?]
   	at org.apache.kafka.clients.consumer.KafkaConsumer.pollForFetches(KafkaConsumer.java:1301) ~[?:?]
   	at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1237) ~[?:?]
   	at org.apache.kafka.clients.consumer.KafkaConsumer.poll(KafkaConsumer.java:1210) ~[?:?]
   	at org.apache.druid.indexing.kafka.KafkaRecordSupplier.poll(KafkaRecordSupplier.java:124) ~[?:?]
   	at org.apache.druid.indexing.kafka.IncrementalPublishingKafkaIndexTaskRunner.getRecords(IncrementalPublishingKafkaIndexTaskRunner.java:98) ~[?:?]
   	at org.apache.druid.indexing.seekablestream.SeekableStreamIndexTaskRunner.runInternal(SeekableStreamIndexTaskRunner.java:603) [druid-indexing-service-0.20.0.jar:0.20.0]
   	at org.apache.druid.indexing.seekablestream.SeekableStreamIndexTaskRunner.run(SeekableStreamIndexTaskRunner.java:267) [druid-indexing-service-0.20.0.jar:0.20.0]
   	at org.apache.druid.indexing.seekablestream.SeekableStreamIndexTask.run(SeekableStreamIndexTask.java:145) [druid-indexing-service-0.20.0.jar:0.20.0]
   	at org.apache.druid.indexing.overlord.SingleTaskBackgroundRunner$SingleTaskBackgroundRunnerCallable.call(SingleTaskBackgroundRunner.java:451) [druid-indexing-service-0.20.0.jar:0.20.0]
   	at org.apache.druid.indexing.overlord.SingleTaskBackgroundRunner$SingleTaskBackgroundRunnerCallable.call(SingleTaskBackgroundRunner.java:423) [druid-indexing-service-0.20.0.jar:0.20.0]
   	at java.util.concurrent.FutureTask.run(FutureTask.java:266) [?:1.8.0_265]
   	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [?:1.8.0_265]
   	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [?:1.8.0_265]
   	at java.lang.Thread.run(Thread.java:748) [?:1.8.0_265]
   Caused by: java.lang.InterruptedException
   	... 17 more
   ```

3. 在druid web的service界面，发现一个特殊的historical节点，它只使用了40G的cache（我给每个historical node分配了2030G，其中2000G作为cache），但是它有着上千个segment在排队等待写入。

   ![奇怪的historical node](https://raw.githubusercontent.com/oliverdding/imgur/main/blog/historical-node-status.png)

## 排查过程

首先搜寻日志，web上的日志，hdfs上的日志。

查看到所有正常执行最后failed的task都产生过这段日志：

```
2021-03-30T07:08:07,133 INFO [coordinator_handoff_scheduled_0] org.apache.druid.segment.realtime.plumber.CoordinatorBasedSegmentHandoffNotifier - Still waiting for Handoff for Segments : [[SegmentDescriptor{interval=2021-03-30T06:00:00.000Z/2021-03-30T07:00:00.000Z, version='2021-03-30T06:00:00.050Z', partitionNumber=170}]]
```

不难判断出，task是完成segment的shard构建的，但是想发送出去失败了，一直waiting最终超时被interrupt。

直觉告诉我，这个和那个奇怪的historical node相关，于是上去查看一番，日志显示：

```
java.io.IOException: No space left on device
ERROR [ZKCoordinator--0] org.apache.druid.server.coordination.SegmentLoadDropHandler - Failed to load segment for dataSource
Caused by: org.apache.druid.segment.loading.SegmentLoadingException: Failed to load segment tencent-json_kv_3_2021-03-29T15:00:00.000Z_2021-03-29T16:00:00.000Z_2021-03-29T15:00:00.055Z_578 in all locations.
```

我只是觉得不可能不科学这不应该发生...好吧，**排除了一切可能的原因，剩下的五论多不可能都只能是原因**，盘不够了！

最终康哥给了我提示，去pvc看一眼，发现这个node的pvc绑定的pv只有40G...

## 结果

helm对于边界逻辑的处理出现问题，historical node共10个重复配置的pod，都需要2030G的pvc，但是0号pod只有40G，软件中却写着2030G，于是所有druid节点都把这个实际只有40G的pod当成2030G使用。特别是后续的任务，集群看到这个node竟然只用了40G，其他node都6、700G了，于是把所有生成的segment都往上调度，结果就被卡住，最终超时被杀。

于是，解决办法就是删除pv、pvc，重新分配正确的pv、pvc，然后杀掉这个pod，等statfulset重新部署这个0号pod。