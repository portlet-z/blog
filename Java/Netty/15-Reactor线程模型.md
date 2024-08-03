# 前言

- EventLoop是Reactor线程模型的核心处理引擎
- EventLoop到底是如何实现的？
- 又是如何保证高性能和线程安全性的呢？

# Reactor线程执行的主流程

- 轮询IO事件

```java
switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
    case SelectStrategy.CONTINUE:
        continue;
    case SelectStrategy.BUSY_WAIT:
    case SelectStrategy.SELECT:
        select(wakenUp.getAndSet(false));
    default:
}
```

- 处理IO事件
  - Netty通过ioRatio参数控制IO事件处理和任务处理的时间比例
  - 默认ioRatio=50
  - 如果ioRatio=100，表示每次都处理完IO事件后，会执行所有的task
  - 如果ioRatio<100, 也会优先处理IO事件，再处理异步任务队列
- processSelectedKeysPlain的主流程
  - 判断needsToSelectAgain决定是否需要重新轮询
  - 如果needsToSelectAgain == true, 会调用selectAgain()方法进行重新轮询
  - 该方法会将needsToSelectAgain再次置为false, 然后调用selecttoNow()后立即返回
- 处理IO事件时有两种选择
  - 处理Netty优化过的selectedKeys
  - 正常的处理逻辑
- NioEventLoop内部
  - execute()普通任务队列
  - schedule()定时任务队列
- 处理异步任务队列的两种实现
  - runAllTasks()处理所有任务
  - runAllTasks(long timeoutNanos)带有超时时间来处理任务

# 总结

- Reactor线程模型时Netty最核心的内容
- 轮询IO事件、处理IO事件、处理异步任务队列
- Netty的NioEventLoop是如何实现的
- 它为什么能够保证Channel的操作是线程安全的
- Netty是如何解决JDK epoll空轮询Bug
- NioEventLoop是如何实现无锁化的