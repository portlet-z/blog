## 概述

### Netty是什么

```
Netty is an asynchronous event driven network application framework for rapid development of maintainable high performance protocol server & clients
```

Netty是一个异步的，基于事件驱动的网络应用程序框架，用于快速开发可维护，高性能的网络服务器和客户端

### Netty的地位

Netty在Java网络应用框架中的地位就好比：Spring框架在JavaEE开发中的地位

以下框架都使用了Netty，因为它们有网络通信需求

- Cassandra - nosql数据库
- Spark - 大数据分布式计算框架
- Hadoop - 大数据分布式存储框架
- RocketMQ - ali开源的消息队列
- ElasticSearch - 搜索引擎
- gRPC - rpc框架
- Dubbo - rpc框架
- Spring 5.x - flux api框架完全抛弃了Tomcat, 使用Netty作为服务器端
- Zookeeper - 分布式协调框架

### Netty优势

- Netty vs NIO, 工作量大，bug多
  - 需要自己构建协议
  - 解决TCP传输问题，如粘包，半包
  - epoll空轮询导致CPU 100%
  - 对API进行增强，使之更易用，如FastThreadLocal => ThreadLocal, ByteBuf => ByteBuffer
- Netty vs其它网络应用框架
  - Mina由Apache维护，将来3.x版本可能会有较大重构，破坏API不兼容性，Netty的开发迭代更迅速，API更简洁，文档更优秀
  - 久经考验
    - 2.x 2004
    - 3.x 2008
    - 4.x 2013
    - 5.x 已废弃（没有明显性能提升，维护成本高）

## Hello World

### 目标

开发一个简单的服务器端和客户端

- 客户端向服务器发送hell world
- 服务器仅接收，不返回

加入依赖

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.42.Final</version>
</dependency>
```

