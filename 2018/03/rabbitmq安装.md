## rabbitmq安装

* 安装erlang  

```
yum install gcc glibc-devel make ncurses-devel openssl-devel autoconf
wget http://erlang.org/download/otp_src_R16B01.tar.gz
tar -zxvf otp_src_R16B01.tar.gz
cd otp_src_R16B01
./configure && make && sudo make install
```

* 安装rabbitmq

```
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.3.5/rabbitmq-server-3.3.5-1.noarch.rpm
yum install rabbitmq-server-3.3.5-1.noarch.rpm
```

* 启动rabbitmq

```
systemctl start rabbitmq-server.service #centos 7
service rabbitmq-server start  #centos 6
```

* 启动插件

```
chmod 755 /etc/rabbitmq/enabled_plugins
rabbitmq-plugins enable rabbitmq_management
```

* 新增用户并赋予权限

```
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

