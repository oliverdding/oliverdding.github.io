# 浏览网站时会发生什么


> 由于这个问题非常泛, 几乎设计计算机发展以来所有的内容, 怎么发挥最好依照你面试的岗位.
> 
> 这篇回答是站在一个本科毕业生, 面试后端方向时的回答例子.

 分为6步.
 
 ## 1. 合成URL
 
浏览器判断是搜索内容还是URL.

若是搜索内容, 根据设置中定义的搜索引擎规则合成URL;

而例子是`www.google.com`, 浏览器会加上协议等等;

## 2. DNS域名解析

1. 查看本地`hosts`文件
2. 查看本地DNS缓存
3. 递归查询系统配置的本地DNS服务器(比如: `8.8.8.8`)
4. 本地DNS服务器将依次迭代查询根域名服务器、顶级域名服务器和权威域名服务器

获得服务器IP

>  在终端下可以使用`nslookup`跟踪DNS查询过程

## 3. 建立TCP连接

1. 第一次握手。客户端发送SYN=1报文，并约定自己的sequence number为x、窗口和窗口缩放因子，进入SYN_SEND状态。
2. 第二次握手。服务器收到SYN报文，返回SYN=1、ACK=1报文，设置acknowledgement number为x+1(RFC文档规定，SYN报文内容为空，但是仍然占1长度)和自己的sequence number为y，进入SYN_RECEV状态。(这里有拒绝服务攻击，客户端发送大量SYN报文后丢弃，让服务器的连接池爆掉)
3. 第三次握手。客户端收到SYN、ACK后，发送ACK报文，acknowledgement number为y+1回复服务器，进入ESTABLISHED状态，并且往往会附带数据。服务器收到后也进入ESTABLISHED状态，完成三次握手。

## 4. 发送HTTP请求，服务器响应

TCP建立连接后，浏览器利用HTTP、HTTPS协议发送请求。

这里需要注意，若目标服务器启用CDN，上一步本地DNS的负载均衡功能返回的IP则是离客户端最近的CDN的IP，因此请求会发送到DNS那。

CDN会定时询问源服务器`if-modified-since`，服务器会返回`304 NOT MODIFIED`或者200+数据。

## 5. 浏览器渲染

会根据header中`content-type`判断内容，浏览器会调用相关的处理方式。

比如html，会构建DOM树来渲染，浏览器还会发送GET请求获取favicon.ico图标。若是HTTP1.1会流水线获取favicon.ico网站图标。

若是header中有`RANGE`，表示支持分段传输，这里就有意思了，传大文件可以断点续传这些。

## 6. 关闭TCP连接

1. 第一次挥手。一方(叫它A吧)发送FIN报文，A进入FIN_WAIT_1状态，表示自己没消息了。
2. 第二次挥手。另一方(叫它B)收到FIN，发送ACK表示知道了。此时A进入FIN_WAIT_2，但是B还是能发消息。
3. 第三次挥手。B发送FIN给A，表示没消息了，进入LAST_ACK状态。
4. 第四次挥手。A收到FIN后，发送ACK，进入TIME_WAIT。B收到后就直接关闭socket，A还要等2MSL时间后关闭。

