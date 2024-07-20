# 前言

- ChannelPipeline与ChannelHandler的关系是什么？他们之间是如何协同工作的？
- ChannelHandler的类型有哪些？有什么区别？
- Netty中IO事件是如何传播的？

# ChannelPipeline概述

- 原始的网络字节流经过Pipeline
- 被一步步加工包装，最后得到加工后的成品
- 他是Netty的核心处理链
- 用以实现网络事件的动态编排和有序传播

## ChannelPipeline内部结构

- ChannelPipeline作为Netty的核心编排组件，负责调度各种类型的ChannelHandler,实际数据的加工处理操作则是由ChannelHandler完成的