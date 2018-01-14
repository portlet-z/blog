## HDFS架构

* Master(NameNode/NN) 带N个Slaves(DataNode/DN)

> HDFS/YARN/HBase

**1个文件会被拆分成多个Block**

**blocksize:128M**

**130M ==> 2个Block: 128M和2M**

* NN：

1. 负责客户端请求的响应
2. 负责元数据(文件的名称、副本系数、Block存放的DN)的管理

* DN:

1. 存储用户的文件对应的数据块(Block)
2. 要定期向NN发送心跳信息，汇报本身及其所有的block信息，健康状况

Name + N个DataNode

**建议：NN和DN是部署在不同的节点上**

replication factor：副本系数、副本因子

All blocks in a file except the last block are the same size



