# 概念

- tcpdump是一款强大的网络抓包工具，它使用libpcap库来抓取网络数据包

# 基本语法和使用方法

```bash
$ sudo tcpdump -i eth0 -nn -s0 -v port 80
```

- -i: 选择要捕获的接口，通常是以太网网卡或无线网卡，也可以是vlan或其他特殊接口。如果该系统只有一个网络接口，则无需指定
- -nn: 单个n表示不解析域名，直接显示ip（默认是用hostname显示）; 两个n表示不解析域名和端口。这样不仅方便查看IP和端口号，而且在抓取大量数据时非常高效，因为域名解析会降低抓取速度
- -s0: tcpdump默认只会截取前96字节的内容，要想截取所有的报文内容，可以使用 -s number, number就是你要截取的报文字节数，如果是0的话，表示截取报文全部内容
- -v -vv -vvv来显示更多的详细信息，通常会显示更多与特定协议相关的信息
- port 80: 这是一个常见的端口过滤器，表示仅抓取80端口上的流量，通常是HTTP

```bash
$ sudo tcpdump -n -e -c 5 not ip6
```

- -e: 显示链路层信息。默认情况下tcpdump不会显示数据链路层信息，使用-e选项可以显示源和目的MAC地址，以及VLAN tag信息
- -c: 指定要抓取包的数量

## 混杂模式

- -p: 不让网络接口进入混杂模式。默认情况下使用tcpdump抓包时，会让网络接口进入混杂模式。一般计算机网卡都工作在非混杂模式下，此时网络接口只接受来自网络端口的目的地址指向自己的数据。当网络工作在混杂模式下时，网卡将来自接口的所有数据都捕获并交给响应的驱动程序。如果设备接入的交换机开启了混杂模式，使用-p选项可以有效的过滤噪声

## 显示ASCII字符串

```bash
$ sudo tcpdump -A -s0 port 80
```

- -A表示使用ASCII字符串打印报文的全部数据，这样可以使读取更加简单，方便使用grep等工具解析输入内容。 
- -X 表示同时使用十六进制和ASCII字符串打印报文的全部数，这俩参数不能一起使用

## 抓取特定协议的数据

```bash
$ tcpdump -i eth0 udp
$ tcpdump -i eth0 proto 17
```

- 后面可以跟上协议名称来过滤特定协议的流量。以UDP为例，可以加上参数 udp或proto 17,这两个命令意思相同
- 同理tcp与protocol 6意思相同

## 抓取特定主机的数据

```bash
$ tcpdump -i eth0 host 10.10.1.1
```

- 使用过滤器host可以抓取特定目的地址和源IP地址的流量

```bash
$ tcpdump -i eth0 dst 10.10.1.20
$ tcpdump -i eth0 src 10.10.1.10
```

- 也可以使用src或dst只抓取源或目的IP

## 抓取数据写入文件

- 使用tcpdump截取数据报文的时候，默认会打印到屏幕的默认输出，-w选项可以把数据报文输出到文件

```bash
$ tcpdump -i eth0 -s0 -w test.pcap
```

## 行缓冲模式

- 如果想实时将抓取到的数据通过管道传递给其他工具来处理，需要使用-l选项来开启行缓冲模式。-l选项可以将输出通过立即发送给其他命令，其他命令会立即响应

```bash
$ tcpdump -i eth0 -s0 -l port 80 | grep 'Server:'
```

## 组合过滤器

```bash
and or &&
or or ||
not or !
```

# 过滤器

## Host过滤器

```bash
$ tcpdump host 1.2.3.4
```

- 抓取1.2.3.4主机上所有流入流出的流量

```bash
$ tcpdump src host 1.2.3.4
```

- 只抓取从1.2.3.4主机发出的流量

## Network过滤器

- Network过滤器用来过滤某个网段的数据，使用的是CIDR模式

```bash
$ tcpdump net 192.168.1
```

- 抓取192.168.1.0-192.168.1.255网段的所有流量

```bash
$ tcpdump net 10
```

- 抓取10.X.X.X网段的所有流量

```bash
$ tcpdump src/dst net 10
```

- 也可以配合src/dst使用

```bash
$ tcpdump src net 172.168.0.0/12
```

- 也可以使用CIDR格式

## Proto过滤器

- proto过滤器用来过滤某个协议的数据，关键字为proto,可省略。proto后面可以跟上协议号或者协议名称支持icmp, igmp, igrp, pim, ah, esp, carp, vrrp, ud, tcp。因为通常的协议名称是保留字段，所以在与proto指令一起使用时，必须根据shell类型使用一个或者两个反斜杠/来转义

```bash
$ tcpdump -n proto \\icmp
$ tcpdump -n icmp
```

## Port过滤器

```bash
$ tcpdump port 6379
```

- 过滤通过某个端口的数据报文

# 理解tcpdump的输出

```bash
14:04:23.906317 IP (tos 0x0, ttl 38, id 60642, offset 0, flags [DF], proto TCP (6), length 52)
    12.26.33.82.2011 > 10.200.64.59.53099: Flags [F.], cksum 0xc30d (correct), seq 1564507100, ack 267937646, win 501, options [nop,nop,TS val 1623698080 ecr 1376735241], length 0
```

- 14:04:23.906317 时间点

- 12.26.33.82.2011 > 10.200.64.59.53099 ： 源IP: 12.26.33.82,源端口 2011， 目的IP10.200.64.59,目的端口53099。 > 符合代表数据的方向

- Flags [F.]: TCP报文的Flags字段

  ```
  [S]: SYN建立连接
  [F]: FIN管理链接
  [.]: ACK收到请求返回响应
  [P]: PSH数据传输，推送标志位PSH:接收方的TCP收到该标志位为1的报文段会尽快上交应用进程，而不必等到接收缓存都填满后再上交
  [R]: RST连接重置，当RST=1时，表明TCP连接出现了异常，必须释放连接，然后再重新建立连接。RST置1还用来拒绝一个非法的报文段或拒绝打开一个TCP连接
  
  # 组合使用
  [S.]: 收到并建立连接 三次握手SYN+ACK
  [F.]: 收到并关闭连接 
  [R.]: 收到并连接重置
  [P.]: 收到并传输数据
  ```

# 示例

## 提取HTTP用户代理

```bash
$ tcpdump -i eth0 -nn -s0 -l | grep 'User-Agent:'
```

- -l开启缓冲行
- grep -E 或 egrep正则过滤

```bash
$ tcpdump -i eth0 -nn -s0 -l | egrep 'User-Agent:Host:'
# 同时提取用户代理和主机名
```

## 只抓取GET和POST流量

```bash
$ tcpdump -s0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2) : 4] = 0x47455420'
# 抓取HTTP GET流量

$ tcpdump -s0 -A -vv 'tcp[((tcp[12:1] & 0xf0) >> 2) : 4] = 0x504f5354'
# 抓取HTTP POST流量
```

## 过滤HTTP请求的URL

```bash
$ tcpdump -i eth0 -s0 -n -l -v | egrep -i 'POST /|GET /|Host:'
```

## 提取HTTP POST请求中的密码

```bash
$ tcpdump -i eth0 -s0 -v -A -l | egrep -i "POST /|pwd=|paswd=|password=|Host:"
```

## 提取Cookies

- Set-Cookie(服务端Cookie), Cookie(客户端Cookie)

```bash
$ tcpdump -nn -A -s0 -l | egrep -i "Set-Cookie|Host:|Cookie:"
```

## 抓取ICMP数据包

```bash
$ tcpdump -n icmp
```

## 抓取非ECHO/REPLY类型的数据包

- 通过排除echo和reply类型的数据包使抓取到的数据包不包括ping包

```bash
$ tcpdump -i eth0 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'
```

## 抓取SMTP/POP3协议的邮件

```bash
$ tcpdump -nn -l port 25 | egrep -i 'MAIL FROM\|RCPT TO'
# 只提取电子邮件的收件人
```

## 抓取NTP服务的查询和响应

```bash
$ tcpdump dst port 123
```

## 抓取SNMP服务的查询可响应

```bash
$ tcpdump -n -s0 port 161 and udp
tcpdump: data link type PKTAP
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on pktap, link-type PKTAP (Apple DLT_PKTAP), snapshot length 524288 bytes
15:19:56.016250 IP 172.16.109.129.53457 > 10.200.64.59.161:  GetRequest(28)  .1.3.6.1.2.1.1.1.0
15:19:56.016261 IP 172.16.109.129.53457 > 10.200.64.59.161:  GetRequest(28)  .1.3.6.1.2.1.1.1.0
```

- 通过SNMP服务，渗透测试人员可以获取大量的设备和系统信息。在这些信息里，系统信息最为关键，如操作系统版本，内核版本。使用SNMP协议快速扫描程序onesixtyone,可以看到目标系统信息

## 切割pcap文件

- 当抓取大量数据并写入文件时，可以自动切割wei多个大小相同的文件

```bash
$ tcpdump -w /tmp/capture-%H.pcap -G 3600 -C 200
# -G 3600每3600秒创建一个新文件 capture-(hour).pcap
# -C 200每个文件大小不超过200 * 1000000字节
```

## 抓取IPv6流量

```bash
$ tcpdump -nn ip6 proto 6
# 过滤器ip6来抓取IPv6流量，同时指定协议TCP

$ tcpdump -nr ipv6-test.pcap ip6 proto 17
# 从之前的文件中读取ipv6 UDP的数据包
```

## 检测端口扫描

```
172.16.109.129.60799 > 10.200.64.59.995: Flags [S], cksum 0xa90f (correct), seq 13210710, win 1024, options [mss 1460], length 0
18:48:01.643261 IP (tos 0x0, ttl 53, id 36636, offset 0, flags [none], proto TCP (6), length 44)
    172.16.109.129.60799 > 10.200.64.59.445: Flags [S], cksum 0xab35 (correct), seq 13210710, win 1024, options [mss 1460], length 0
18:48:01.643261 IP (tos 0x0, ttl 53, id 36636, offset 0, flags [none], proto TCP (6), length 44)
    172.16.109.129.60799 > 10.200.64.59.445: Flags [S], cksum 0xab35 (correct), seq 13210710, win 1024, options [mss 1460], length 0
18:48:01.643273 IP (tos 0x0, ttl 54, id 11315, offset 0, flags [none], proto TCP (6), length 44)
    172.16.109.129.60799 > 10.200.64.59.554: Flags [S], cksum 0xaac8 (correct), seq 13210710, win 1024, options [mss 1460], length 0
18:48:01.643274 IP (tos 0x0, ttl 54, id 11315, offset 0, flags [none], proto TCP (6), length 44)
    172.16.109.129.60799 > 10.200.64.59.554: Flags [S], cksum 0xaac8 (correct), seq 13210710, win 1024, options [mss 1460], length 0
18:48:01.644657 IP (tos 0x0, ttl 128, id 2781, offset 0, flags [none], proto TCP (6), length 40)
10.200.64.59.21 > 172.16.109.129.60799: Flags [R.], cksum 0xed0b (correct), seq 4090883251, ack 13210711, win 64240, length 0
18:48:01.644663 IP (tos 0x0, ttl 128, id 2781, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.21 > 172.16.109.129.60799: Flags [R.], cksum 0xed0b (correct), seq 0, ack 1, win 64240, length 0
18:48:01.644830 IP (tos 0x0, ttl 128, id 2782, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.135 > 172.16.109.129.60799: Flags [R.], cksum 0xf713 (correct), seq 3158776264, ack 13210711, win 64240, length 0
18:48:01.644832 IP (tos 0x0, ttl 128, id 2782, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.135 > 172.16.109.129.60799: Flags [R.], cksum 0xf713 (correct), seq 0, ack 1, win 64240, length 0
18:48:01.644834 IP (tos 0x0, ttl 128, id 2783, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.110 > 172.16.109.129.60799: Flags [R.], cksum 0x6af3 (correct), seq 2827139526, ack 13210711, win 64240, length 0
18:48:01.644835 IP (tos 0x0, ttl 128, id 2783, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.110 > 172.16.109.129.60799: Flags [R.], cksum 0x6af3 (correct), seq 0, ack 1, win 64240, length 0
18:48:01.645018 IP (tos 0x0, ttl 128, id 2784, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.587 > 172.16.109.129.60799: Flags [R.], cksum 0x050c (correct), seq 3009483506, ack 13210711, win 64240, length 0
18:48:01.645019 IP (tos 0x0, ttl 128, id 2784, offset 0, flags [none], proto TCP (6), length 40)
    10.200.64.59.587 > 172.16.109.129.60799: Flags [R.], cksum 0x050c (correct), seq 0, ack 1, win 64240, length 0
```

- 目标主机执行抓包命令

```bash 
$ tcpdump tcp and host 172.16.109.129 -nn -v
```

- 源主机执行端口扫描命令

```bash 
$ nmap 10.200.64.59
```

- 上面的抓包打印信息源和目标IP一直没变，标志位[S],当发生SYN后，如果目标主机的端口没有打开，就会返回一个RST, 这是nmap等端口扫描工具的标准做法

## 抓取DNS

```bash
$ tcpdump -i eth0 -s0 port 53
```

## 抓取HTTP有效数据包

- 抓取80端口的HTTP有效数据包，排除TCP连接过程的数据包(SYN/FIN/ACK)

```bash
$ tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

## 输出内容重定向到Wireshark

```bash
$ ssh root@remotesystem 'tcpdump -s0 -c 1000 -nn -w - not port 22' | /Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i -
$ ssh root@remotesystem 'tcpdump -s0 -c 1000 -nn -w - port 53' | /Applications/Wireshark.app/Contents/MacOS/Wireshark -k -i -
```

## 找出一段时间内发包最多的IP

```bash
$ tcpdump -nnn -t -c 200 | cut -f 1,2,3,4 -d '.' | sort | uniq -c | sort -nr | head -n 20
```

## 抓取用户名密码

```bash
$ tcpdump port http or port ftp or port smtp or port imap or port pop3 or port telnet -l -A | egrep -i -B5 'pass=|pwd=|log=|login=|user=|username=|pw=|passw=|passwd=|password=|pass:|user:|username:|password:|login:|pass |user '
```

## 抓取DHCP报文

```bash
$ tcpdump -v -n port 67 or 68
```

