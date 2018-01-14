## YARN架构

#### MapReduce

Hadoop1.x时：

MapReduce: Master/Slave架构，1个JobTracker带多个TaskTracker

* JobTracker: 负责资源管理和作业调度
* TaskTracker: 定期向JT汇报本节点的健康状况、资源使用情况、作业执行情况；接受来自JT的命令：启动任务、杀死任务

#### YARN

> XXX on YRAN的好处：
>
> 与其他计算框架共享集群资源，按资源需要分配，进而提高集群资源的利用率
>
> XXX: Spark/MapReduce/Storm/Flink

* ResourceManager: RM

> 整个集群同一时间提供服务的RM只有一个，负责集群资源的统一管理和调度
>
> 处理客户端的请求：提交一个作业、杀死一个作业
>
> 监控我们的NM,一旦某个NM挂了，那么该NM上运行的任务需要告诉我们的AM来如何进行处理

* NodeManager: NM

> 整个几群众有多个负责自己本身节点资源管理和使用
>
> 定时向RM汇报本节点的资源使用情况
>
> 接收并处理来自RM的各种命令:启动Container
>
> 处理来自AM的命令
>
> 单个节点的资源管理

* ApplicationMaster: AM

> 每个应用程序对应一个:MR、Spark,负责应用程序的管理
>
> 为每个应用程序向RM申请资源(core,memory),分配给内部task



#### wordcount:统计文件中每个单词出现的次数

需求：求wc

* 文件内容小:shell
* 文件内容很大:TB GB mapreduce分而治之

(input) <k1,v1> -> map -> <k2,v2> -> combine -> <k2,v2> -> reduce -> <k3,v3> -> (output)

* 核心概念：

> Split: 交由MapReduce作业来处理的数据块，是MapReduce中最小的计算单元
>
> HDFS: blocksize是HDFS中最小的存储单元：128M
>
> 默认情况下：他们俩是一一对应的

* InputFormat:

> 将我们的输入数据进行分片(split): InputSplit[] getSplits(JobConf job, int numSplits) throw IOException;
>
> TextInputFormat: 处理文本格式的数据

* OutputFormat: 输出



#### MapReduce1.x的架构

* JobTracker: TT

作业的管理者    管理的

将作业分解成一堆的任务：Task(MapTask和ReduceTask)

将任务分派给TaskTracker运行

作业的监控、容错处理(task作业挂了，重启task的机制)

在一定的时间间隔内，JT没有收到TT的心跳信息，TT可能是挂了，TT上运行的任务会被分配到其他TT上去执行

* TaskTracker: TT

任务的执行者     干活的

在TT上执行我们的Task(MapTask和ReduceTask)

会与JT进行交互：执行、启动、停止作业、发送心跳信息给JT

* MapTask

自己开发的map任务交由该Task

解析每条记录的数据，交给自己的map方法处理

将map的输出结果写到本地磁盘（有些作业只有仅有map没有reduce ===> HDFS）

* ReduceTask

将Map Task输出的数据进行读取

按照数据进行分组传给我们自己编写的reduce方法处理

输出结果写到HDFS



#### Hadoop实战

**用户行为日志：**用户每次访问网站时所有的行为数据（访问、浏览、搜索、点击。。。）

用户行为轨迹、流量日志

**日志数据内容：**

1. 访问的系统属性：操作系统、浏览器等
2. 访问特征：点击的url、从哪个url跳转过来的(referer)、页面是三个的停留时间等
3. 访问信息：session_id、访问ip(访问城市)等

**数据处理流程：**

1. 数据采集： Flume: web日志写入到HDFS
2. 数据清洗： 脏数据  Spark,Hive,MapReduce或者是其他的一些分布式计算框架清洗完之后的数据可以存放在HDFS(Hive,Spark SQL)
3. 数据处理：安装我们的需要进行相应业务的统计和分析 Spark,Hive,MapReduce或者是其他的一些分布式计算框架
4. 处理结果入库： 接口可以存放到RDBMS,NoSql
5. 数据的可视化： 通过图形化展示的方式展现出来：饼图，柱状图，地图，折线图，ECharts,HUE,Zeppelin