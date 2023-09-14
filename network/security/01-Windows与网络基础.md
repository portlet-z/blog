## Windows基础命令

#### dir

- 用于显示目录和文件列表
- 常用的用法直接使用dir

```powershell
dir
dir /a:h c:\   #查看C盘下的隐藏目录和文件
dir /o:-n c:\  #使用字母逆序方式查看
```

#### md或mkdir

- 创建目录（文件夹），也可以直接创建多级子目录

```powershell
md test
md test1\test2\test3
```

#### rd

- 用于删除目录

```powershell
rd test         #直接使用rd只能删除空目录
rd /s /q test   #如果要删除的目录中有子目录或文件，就必须使用/s选项，可以携带/q选项不提醒
```

#### move

- 用于移动，重命名

```powershell
move test.rar  d:\  #移动
move test.rar  test1.rar  #重命名
```

#### copy

- 用于复制文件

```powershell
copy test.rar  test1.rar   #直接复制文件到指定目录
copy 1.txt+2.txt  3.txt    #将两个文件中的内容直接融合到新的文件中
```

#### xcopy

- 用于复制目录

```powershell
xcopy /s test  c:\
```

#### del

- 用于删除文件

```powershell
del 1.txt
```



## 文本处理

#### type

- 用于显示文本文件内容

```powershell
D:\>type test.rar
Rar!
```

#### 重定向">"

```powershell
ipconfig > ip.txt
```

#### 管道符"|"

- 将当前命令执行的结果为后面命令的操作对象

#### findstr

- 查找文件内容(查找的某个字符串的内容)

```powershell
findstr 192 ip.txt
```

## 网络相关操作

#### 配置TCP/IP参数

- IP地址:标识了网站中的某一台主机
- 子网掩码：用于标识你的IP所处的网络范围，子网掩码越大，网络范围越小
- 默认网关：标识主机直连的路由器的IP地址
- DNS服务器：用于域名解析的

```powershell
#静态配置IP地址，子网掩码，默认网关
netsh interface ip set address "WLAN" static 192.168.21.231 255.255.255.0 192.168.31.1
#自动获取TCP/IP参数
netsh interface ip set address "WLAN" dhcp
#静态配置DNS服务器
netsh interface ip set dnsserver "WLAN" static 114.114.114.114
#添加备用DNS服务器
netsh interface ip add dnsserver "WLAN" 8.8.8.8 index=2  #index=2是索引，表示备用DNS
#自动获取DNS服务器
netsh interface ip set dnsserver "WLAN" dhcp
```

#### 查看TCP/IP参数，用ipconfig

```powershell
# 查看所有网卡的TCP/IP参数（IP地址，子网掩码，默认网关）
ipconfig
# 查看所有网卡的TCP/IP参数（IP地址，子网掩码，默认网关，mac地址，dhcp地址，dns地址，主机名）
ipconfig /all
# 释放TCP/IP参数
ipconfig /release
# 重新获取TCP/IP参数
ipconfig /renew
# 刷新dns缓存
ipconfig /flushdns
```

