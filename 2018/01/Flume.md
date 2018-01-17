## Flume

* 如何解决我们的数据从其他的server上移动到Hadoop之上？

> webserver ==> flume ==> hdfs

#### 概述

* [官网](http://flume.apache.org)
* Flume是由Cloudera提供的一个分布式、高可靠、高可用的服务，用于分布式的海量日志的高效收集、聚合、移动系统
* 设计目标：
  - 可靠性
  - 扩展性
  - 管理性
* 同类比较
  - Flume: Cloudera/Apache  Java
  - Scribe: Facebook C/C++  不再维护
  - Chukwa: Yahoo/Apache  Java 不再维护
  - Fluentd: Ruby
  - Logstash: ELK 

#### 架构及核心组件

* Source 收集
* Channel 聚集
* Sink 输出

#### 环境及部署

* 前置条件
  - Java
  - Memory Sufficient
  - Disk Sufficient
  - Directory Permissions
* 下载Flume配置环境变量
* flume-env.sh配置: export JAVA_HOME=/jdk