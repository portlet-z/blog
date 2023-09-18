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

#### ping(用来测试TCP/IP 配置是否正确)

```powershell
ping -n 10 -l 1000 -t -a 192.168.1.1
# -n 10 发送 10 个报文
# -l 1000 单个报文 1000字节
# -t 一直不停的 ping
# -a 返回 ip的主机名
```

#### tracert

```powershell
tracert 8.8.8.8
```

#### route用来操作网络的路由表

```powershell
# 0.0.0.0代表任意网络
# 打印路由表
route -4 print
# 添加路由条目
route add 112.53.42.53/32 192.168.33.1
# 删除路由条目
route delete 112.53.42.53
```

#### netstat

```powershell
#查看所有的 TCP链接，包括进程，以数字形式显示
netstat -anop tcp
# 查看路由表
netstat -r   #等同于route print
```



## Windows 用户管理

#### 用户账户

- 定义

  - 不同的用户身份拥有不同的权限
  - 每个用户包含了一个名称和密码
  - 每个用户账户具有唯一的安全标识符
  - 查看系统中的用户`net user`
  - 安全标识符(SID):
    - 查看当前用户的 SID`whoami /user`
    - 使用注册表进行查看打开注册表命令`regedit` `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`
    - 在 Windows系统中管理员的 SID是 500，普通用户的SID 是从 1000 开始

- 管理

  - 创建用户

    - 用户名：系统显示名
    - 全名：用户登录时的显示名
    - 密码：Windows 服务器默认需要符合复杂性要求
    - 账户已锁定：如果开启了账户锁定阈值，输错密码多次后将被锁定

    ```powershell
    # 创建用户不指定密码
    net user zhangsan /add
    # 创建用户指定明文密码
    net user list 123456 /add
    # 创建用户手动输入密码
    net user wangwu /add *
    # 删除用户
    net user zhangsan /del
    ```

  - 管理用户

  - 设置密码

  - 隐藏用户: 在用户后面加上了$符号，隐藏用户用 net user 命令是无法看到的

- Windows 的内置用户账户

  - 与使用者关联的
    - 管理员 Administrator: 在使用者中具有最高的权限，如果没有其他管理员的情况，不建议禁用
    - 普通用户：具有一般的读取权限，权限较低
    - 来宾用户 guest: 一般是提供给访客使用，在使用者中，权限最低，默认禁用
  - 与 Windows 组件关联的
    - System 本地系统，拥有最高权限
    - local service 本地服务，他的权限相对于普通用户组 users 会低一点
    - network service 网络服务，他的权限和普通用户组 users 一样

#### 用户组

- 概念：一组用户的集合，组中所有的用户具备所属组的权限
- 管理组

```powershell
# 创建组
net localgroup test /add
# 删除组
net localgroup test /del
# 创建 bing 将用户加入组
net localgroup test zhangsan /add
# 将用户从组中拿掉
net localgroup test zhangsan /del
```

- 内置组账户
  - 需要人为添加的
    - Administrators: 管理员组
    - Guests: 来宾用户组
    - power users: 向下兼容的组，现在一般没有使用
    - Users: 标准用户组，创建用户后默认处于此组中
  - 动态包含成员的组
    - interactive: 动态包含在本地登录的用户
    - authenticated users: 动态包含通过验证的用户
    - everyone: 所有人



## NTFS 权限

#### 文件系统

- Windows 早期使用 FAT16或 FAT32
- 目前 Windows操作系统基本使用的是 NTFS 文件系统
  - ACL(访问控制列表，设置权限)
  - EFS(加密文件系统，使用 BitLocker 进行磁盘加密)
  - 压缩以及磁盘配额
- ReFS 文件系统
- Linux: swap(交换文件系统，主要将磁盘的一部分空间划分给内存使用)，ext4
- 早期的 FAT 文件系统不支持单个大文件(超过 4GB)

```powershell
convert h:fs:ntfs  # h 表示的是需要转换的盘符
```

#### 文件权限

- 读取，写入，附加，删除，执行

#### 文件夹权限

- 列出，创建文件夹，创建文件，删除，删除子文件夹和文件

#### 权限分类

- 完全控制
- 修改
- 读取和执行
- 读取
- 写入
- 特殊权限

#### NTFS权限规则

- 权限的累加：用户分配的有效权限是分配欸用户所有权限的累加。假如一个用户设置读取权限，给用户所属组分配了修改权限，用户最终的权限就等于读取和修改
- 拒绝权限：拒绝的权限大于一切（在访问控制列表中，拒绝的权限优先级最高）；当出现权限冲突的时候，拒绝的权限优先级最高（举例：用户所属组读取权限，用户拒绝读取，最终用户没有读取权限）
- 继承权限：文件或文件夹的访问控制列表默认情况下会继承上级文件夹的权限
- 特殊权限
  - 读取权限（和读取文件或文件夹的内容没有任何关系）：读取文件或文件夹的访问控制列表；针对用户想要访问某个文件的内容，此权限必须勾选
  - 更改权限（和修改文件或文件夹的内容没有任何关系）：用户释放可以修改文件或文件夹的访问控制列表，由于此权限是可以为用户添加或删除权限，会造成很多不安全因素，此权限一般不会给；想要更改，前提必须能读取
- 取得所有权：能够修改文件或文件夹的所有者；前提必须取得读取和修改



## 本地安全策略

#### 基本内容

- 概念
  - 主要是对登录到计算机的账户进行一些安全设置
  - 主要影响是本地计算机安全设置
- 打开方式
  - 开始菜单 -> 管理工具 -> 本地安全策略
  - 使用命令`secpol.msc`
  - 从本地组策略进去 `gpedit.msc`

#### 账户策略

- 密码策略：
  - 密码必须符合复杂性要求(默认情况下，Windows server操作系统是开启)
  - 账户锁定策略(锁定时间，锁定阈值，重置账户锁定计数器时间，管理员是不受限制的)

#### 本地策略



## 组策略应用

#### 基本概念

- 组策略(Group Policy)是微软Windows NT家族操作系统的一个特性，他可以控制用户账户和计算机账户的工作环境。组策略提供了操作系统，应用程序和活动目录中用户设置的集中化管理和配置。组策略的其中一个版本名为本地组策略，这可以在独立且非域的计算机上管理组策略对象。组策略是一组策略的集合
- 打开方式`gpedit.msc`
- 刷新组策略`gpupdate /force`
- 模块
  - 计算机配置：针对本地计算机生效
  - 用户配置：针对用户生效
- 案例
  - 隐藏桌面的系统图片：用户配置 -> 管理模板
  - 保护任务栏和开始菜单： 用户配置->开始菜单和任务栏
  - 保护个人文档隐私：用户配置
  - 禁用在浏览器上新窗口中打开
  - 禁用控制面板 `console.exe`
  - 关闭自动播放功能
  - 配置自动更新



## 注册表

