### Elasticsearch单节点集群部署

配置文件

```yaml
cluster.name: my-elasticsearch
node.name: node-1001
node.master: true
node.data: true
network.host: localhost
http.port: 9201
transport.tcp.port: 9301

discovery.seed_host: ["localhost:9301", "localhost:9302", "localhost:9303"]
discovery.zen.fd.ping_timeout: 1m
discovery.zen.fd.ping_retries: 5

http.cors.enabled: true
http.cors.allow-origin: "*"

cluster.initial_master_nodes: node-1001
```



### Elasticsearch集群部署

- 创建用户

```shell
useradd es #新增es用户
passwd es #为es用户设置密码
userdel -r es #如果错了，删除
chown -R es:es /data/es-cluster #文件夹所有者
```

- 修改配置文件 vim config/elasticsearch.yml

```yaml
#集群名称
cluster.name: cluster-es
#节点名称，每个节点的名称不能重复
node.name: node-1
#ip地址，每个节点的ip地址不能重复
network.host: 192.128.0.1
#是不是主节点
node.master: true
node.data: true
http.port: 9200
# head插件需要这打开这两个配置
http.cors.allow-origin: "*"
http.cors.enabled: true
http.max_content_length: 200mb
#es7.x之后新增的配置，初始化一个新的集群时需要此配置来选举master
cluster.initial_master_nodes: ["node-1"]
#es7.x之后新增的配置，节点发现
discovery.seed_hosts: ["192.168.0.1:9300","192.168.0.2:9300","192.168.0.3:9300"]
gateway.recover_after_nodes: 2
network.tcp.keep_alive: true
network.tcp.no_delay: true
transport.tcp.compress: true
#集群内同时启动的数据任务个数，默认是2
cluster.routing.allocation.cluster_concurrent_rebalance: 16
#添加或删除节点及负载均衡时并发恢复的线程个数，默认是4个
cluster.routing.allocation.node_concurrent_recoveries: 16
#初始化数据恢复时，并发恢复线程的个数，默认4个
cluster.routing.allocation.node_initial_primaries_recoveries: 16
```

修改/etc/security/limits.conf

```
es soft nofile 65536
es hard nofile 65536
```

修改/etc/security/limits.d/20-nproc.conf

```
es soft nofile 65526
es hard nofile 65536
* hard nproc 4096
```

修改/etc/sysctl.conf

```
vm.max_map_count=655360
```

重新加载

```
sysctl -p
```

- 启动软件

```shell
bin/elasticsearch
#后台启动
bin/elasticsearch -d
```

- 测试集群

```
http://192.168.0.1:9200/_cat/nodes
```



### 路由计算&分片控制

hash(id) % 主分片的数量

用户可以访问任何一个节点数据，这个节点称之为协调节点



### 数据写流程

- 客户端请求集群节点(任意) - 协调节点

- 协调节点将请求转换到指定的节点

- 主分片需要将数据保存

- 主分片需要将数据发送到副本

- 副本保存后，进行反馈

- 主分片进行反馈

- 客户端获得反馈

  请求可选参数： consistency一致性。在默认设置下，即使仅仅是在试图执行一个写操作之前，主分片都会要求必须要有规定数量(quorum)的分片副本处于活跃可用状态，才会执行写操作。这是为了避免在发生网络故障的时候进行写操作进而导致的数据不一致

  1. one: 主分片状态ok就允许进行写操作
  2. all: 主分片和所有的副本分片的状态都没问题才允许进行写操作
  3. quorum: int((primary + number_of_replicas) / 2) + 1



### 数据读流程

- 客户端发送查询请求到协调节点
- 协调节点计算数据所在的分片以及全部的副本位置
- 为了能够负载均衡，可以轮询所有的节点
- 将请求转发给具体的节点
- 节点返回查询结果，将结果反馈给客户端



### 更新&批量操作请求

更新请求

- 客户端向node1发送更新请求
- 它将请求转发到主分片所在的node3
- node3从主分片检索文档，修改_source字段中的json,并且尝试重新索引主分片的文档。如果文档已经被另一个进程修改，他会重试此步骤，超过retry_on_conflict次后放弃
- 如果node3成功的更新文档，它将新版本的文档进行转发到nod1和node2上的副本分片，重新建立索引。一旦所有副本分片都返回成功，node3向协调节点也返回成功，协调节点向客户端返回成功

多文档操作流程

- mget和bulkAPI的模式类似于单文档模式。区别在于协调节点知道每个文档存在于哪个分片中。它将整个多文档请求分解成每个分片的多文档请求，并且将这些请求并行转发到每个参与节点
- 协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端



### 分片原理

分片是es最小的工作单元。传统的数据库每个字段存储单个值，但这对全文检索并不够。文本字段的每个单词需要被搜索，对数据库意味这需要单个字段有索引多值的能力。最好的支持是一个字段多个值需求的数据结构是倒排索引

- 倒排索引

  分词器： keyword:关键词不能分词  text:文本字段可以分词

  ik_max_work:最细粒度的拆分  ik_smart:粗粒度的拆分

  词条：索引中最小的存储和查询单元

  词典：字典，词条的集合 B+, HashMap

  倒排表：关键词出现在什么位置，以及频率。在倒排表中每条记录称为倒排项

  倒排索引搜索过程

  1. 查询单词是否在词典中，如果不在就结束
  2. 如果在的话，看一下单词在倒排表中的指针
  3. 通过倒排列表去获取单词所对应的文档id的列表
  4. 根据文档id去查询对应的数据



### 文档分析

- 将一块文本分成适合于倒排索引的独立的词条
- 将这些词条统一化为标准格式以提高他们的可搜索行，或者recall

分析器执行上面的工作。分析器实际上是将三个功能封装到了一个包里

1. 字符过滤器：首先，字符串按照顺序通过每个字符过滤器。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML或者将&转化成and
2. 分词器：其次，字符串被分词器分成单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条
3. Token过滤器：最后，词条按顺序通过每个token过滤器。这个过程可能会改变词条(例如，小写化Quick),删除词条(a,and,the等无用词)，或者增加词条(例如，像jump和leap这种同义词)



### ik分词插件安装

- 解压文件到 plugins/elasticsearch-analysis-ik-7.12.0
- 删除data目录

- 重启es服务

扩展词汇

- cd plugins/elasticsearch-analysis-ik-7.12.0/config文件夹

- 创建custom.dic文件，将词汇写入

- 打开IKAnalyzer.cfg.xml文件，将新建的custom.dic配置其中，重启ES服务器

  

### kibana

vim config/kibana.yml

```yaml
#默认端口
server.port: 5601
#ES服务器地址
elasticsearch.hosts: ["http://localhost:9200"]
#索引名
kibana.index: ".kibana"
#支持中文
i18n.locale: "zh-CN"
```



### Elasticsearch优化

1. 硬件选择

   - 使用SSD
   - 使用RAID 0.条带化RAID会提高磁盘I/O,代价显然就是当一块硬盘故障的时整个就故障了，不要使用镜像或者奇偶校验RAID因为副本已经提供了这个功能
   - 另外，使用多块磁盘，并允许Elasticsearch通过多个path.data目录配置把数据条带化分配到他们上面
   - 不要使用远程挂载的存储，比如NFS或者SMB/CIFS，这个引入的延迟对性能来说完全是背道而驰的

2. 分片策略

   - 一个分片的底层即为一个Lucene索引，会消耗一定文件句柄、内存、以及CPU运转
   - 每一个搜索请求都需要命中索引中的每一个分片，如果每一个分片都处于不同的节点还好，但如果多个分片都需要在同一节点上竞争使用相同的资源就有些糟糕了
   - 用于计算相关度的词项统计信息是基于分片的。如果有许多分片，每一个都只有很少的数据会导致很低的相关度

3. 分片原则

   - 控制每个分片占用的硬盘容量不超过ES的最大JVM的堆空间设置(一般设置不超过32G),因此，如果索引的总容量在500G左右，那分片大小在16个左右即可，当然最好同时参考下面一个规则
   - 考虑一下node数量，一般一个节点有时候就是一台物理机，如果分片数过多，大大超过了节点数，很可能导致一个节点上存在多个分片，一旦该节点故障，即使保持了1个以上的副本，同样有可能会导致数据丢失，集群无法恢复，所以，一般都设置分片数不超过节点数的3倍
   - 主分片，副本和节点最大数之间数量，我们分配的时候可以参考以下关系
     - 节点数 <= 主分片数 *（副本数 + 1）

4. 推迟分片分配 

   对于节点瞬时中断的问题，默认情况，集群会等待一分钟来查看节点是否会重洗加入，如果这个节点在此期间重新加入，重新加入的节点会保持其现有的分片数据，不会触发新的分片分配。这样就可以减少ES在自动再平衡可用分片时所带来的极大开销

   ```
   PUT /_all/_settings
   {
   	"settings": {
   		"index.unassigned.node_left.delayed_timeout": "5m"
   	}
   }
   ```

5. ES堆内存设置

   - 不要超过物理内存的50%
   - 堆内存大小最好不要超过32G
   - 假设机器128G,可以创建两个节点，每个节点内存分配不要超过32G



### 面试题

- 为什么要使用Elasticsearch

  系统中的数据，随着业务的发展，时间的推移，将会非常多，而业务中往往采用模糊查询进行数据的搜索，而模糊搜索查询会导致查询引擎放弃索引，导致系统查询数据时都是全表扫描，在百万级别的数据库中，查询效率是非常低下的，而我们使用ES做一个全文索引，将经常查询的系统功能的某些字段，比如说电商系统的商品表中商品名，描述，价格还有id这些字段我们放入ES索引库里，可以提高查询速度

- Elasticsearch的master选举流程

  - ES的选主是ZenDiscovery模块负责的，主要包含Ping(节点之间通过这个RPC来发现彼此)和Unicast(单播模块包含一个主机列表以控制哪些节点需要ping通)这两部分
  - 对所有可以成为master的节点(node.master: true)根据nodeId字典排序，每次选举每个节点都把自己所知道节点排一次序，然后选出第一个节点，暂且认为它是master节点
  - 如果对某个节点的投票数达到一定的值并且该节点自己也选举自己，那这个节点就是master,否则重新选举直到满足上述条件
  - master节点的职责主要包括集群、节点和索引的管理，不负责文档级别的管理；data节点可以关闭http功能

- es集群脑裂问题

  脑裂问题可能的成因

  1. 网络问题：集群间的网络延迟导致一些节点访问不到master,认为master挂掉了从而选举出新的master,并对master上的分片和副本标红，分配新的主分片
  2. 节点负载：主节点的角色既为master又为data,访问量较大时可能会导致ES停止响应造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点
  3. 内存回收：data节点上的ES进程占用的内存较大，引发JVM的大规模内存回收，造成ES进程失去响应

  脑裂问题解决方案

  1. 减少误判：discovery.zen.ping_timeout节点状态的响应时间，默认为3s,可适当调大，如果master在该响应时间的范围内没有做出响应应答，判断该节点已经挂掉了。调大参数(6s),可适当减少误判
  2. 选举触发：discover.zen.minimum_master_nodes: 1该参数是用于控制选举行为发生的最小集群主节点数量。当备选主节点的个数大于等于该参数的值，且备选主节点中有该参数个节点认为主节点挂了，进行选举。官方建议为(n/2)+1,n为主节点个数(即有资格成为主节点的节点个数)
  3. 角色分离：即master节点与data节点分离，限制角色
     - 主节点配置为: node.master: true node.data: false
     - 从节点配置为: node.master: false node.data: true

- es部署时，对Linux设置有哪些建议

  - 64GB内存的机器是非常理想的
  - 在更快的CPU和更多的核心之间选择核心更多的
  - SSD
  - 避免跨集群
  - 通过设置gateway.recover_after_nodes\gateway.expected_nodes\gateway.recover_after_time可以在集群重启的时候避免过多的分片交换
  - 单播替换组播
  - 不要随意修改垃圾回收器和各个线程池的大小
  - 文件描述符设置一个很大的值