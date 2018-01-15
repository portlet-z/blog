## Hadoop集群安装

* ssh免密码登录

> 在每台机器上运行：ssh-keygen -t rsa
>
> 以hadoop000机器为主
>
> ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop000
>
> ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop001
>
> ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop002

* jdk安装
* Hadoop安装

> 在hadoop000机器上解压hadoop安装包，并设置HADOOP_HOME到系统环境变量

**hadoop-env.sh**

```shell
export JAVA_HOME=/opt/jdk
```

**core-site.xml**

```xml
<property>
  <name>fs.default.name</name>
  <value>hdfs://hadoop000:8020</value>
</property>
```

**hdfs-site.xml**

```xml
<property>
  <name>dfs.datanode.data.dir</name>
  <value>/home/hadoop/app/tmp/dfs/data</value>
</property>
```

**yarn-site.xml**

```xml
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>hadoop000</value>
</property>
```

**mapred-site.xml**

```xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```

**slaves**

```
hadoop000
hadoop001
hadoop002
```

* 分发安装包到hadoop001和hadoop002节点

```
scp -r ~/app hadoop@hadoop001:~/
scp -r ~/app hadoop@hadoop002:~/

scp ~/.bash_profile hadoop@hadoop001:~/
scp ~/.bash_profile hadoop@hadoop002:~/
```

> 在hadoop001和hadoop002机器上让.bash_profile生效

* 对NN做格式化：只要在hadoop上执行即可

```
bin/hdfs namenode -format
```

* 启动集群：只要在hadoop000上执行即可

```
sbin/start-all.sh
```

* 验证

```
jps

hadoop000:
SecondaryNameNode
DataNode
NodeManager
NameNode
ResourceManager

hadoop001:
NodeManager
DataNode

hadoop002:
NodeManger
DataNode


webui:
hadoop000:50070   hadoop000:8088
```

* 集群停止

```
stop-all.sh
```

