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