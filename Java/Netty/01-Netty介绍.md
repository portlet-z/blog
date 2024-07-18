# Netty知识点大纲

![](./images/Netty核心原理.png)

# 为什么选择Netty

## IO请求

- IO调用阶段：用户进程向内核发起系统调用
- IO执行阶段：内核等待IO请求处理完成返回

![](./images/IO请求.png)

## IO模型

