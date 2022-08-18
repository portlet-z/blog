- NioEventLoop线程不仅要处理IO事件，还要处理Task(包括普通任务和定时任务)
- NioEventLoop的重要组成：selector, 线程, 任务队列
- selector何时创建 - 在NioEventLoop构造方法调用时创建
- eventLoop为何有两个selector成员 - 为了在遍历selectedKeys提高性能
- eventLoop的nio线程在何时启动 - 当首次调用execute方法时，通过state状态位控制线程只会启动一次
- 提交普通任务会不会结束select阻塞
  - wakeup方法中的代码如何理解 - 只有其他线程提交任务时，才会调用selector的wakeup方法
  - wakenUp变量的作用是什么 - 如果有多个其他线程都来提交任务，为了避免wakeup被频繁调用
- 每次循环时，什么时候才会进入SelectStrategy.SELECT分支 - 当没有任务时，才会进入SelectStrategy.SELECT.当有任务时，会调用selectNow方法，顺便拿到io事件
- 何时会select阻塞，阻塞多久 - 没有定时任务的情况， selectDeadLineNanos: 截止时间 = 当前时间 + 1s  timeoutMillis: 超时时间 = 1s + 0.5ms
- nio空轮询Bug在哪里体现，如何解决 - 在jdk在linux的selector. 重新创建了一个selector, 替换了旧的selector
- ioRatio控制什么，设置为100有何作用？ - ioRatio控制处理IO事件所占用的时间比例， ioTime代表执行io事件处理耗费的时间

## 源码分析

- 提交任务代码`io.netty.util.concurrent.SingleThreadEventExecutor#execute`

```java
    public void execute(Runnable task) {
        if (task == null) {
            throw new NullPointerException("task");
        }
        boolean inEventLoop = inEventLoop();
      // 添加任务，其中队列使用了jctools提供的mpsc无锁队列
        addTask(task);
        if (!inEventLoop) {
          // inEventLoop如果为false表示由其他线程来调用execute, 即首次调用，这时需要向eventLoop提交首个任务，启动死循环，会执行到下面的doStartThread
            startThread();
            if (isShutdown()) {
                boolean reject = false;
                try {
                    if (removeTask(task)) {
                        reject = true;
                    }
                } catch (UnsupportedOperationException e) {
                }
                if (reject) {
                    reject();
                }
            }
        }
        if (!addTaskWakesUp && wakesUpForTask(task)) {
          // 如果线程由于IO select阻塞了，添加的任务的线程需要负责唤醒NioEventLoop线程
            wakeup(inEventLoop);
        }
    }
```

- 唤醒select线程`io.netty.channel.nio.NioEventLoop#wakeup`

```java
    @Override
    protected void wakeup(boolean inEventLoop) {
        if (!inEventLoop && wakenUp.compareAndSet(false, true)) {
            selector.wakeup();
        }
    }
```

- 启动EventLoop主循环`io.netty.util.concurrent.SingleThreadEventExecutor#doStartThread`

```java
    private void doStartThread() {
        assert thread == null;
        executor.execute(new Runnable() {
            @Override
            public void run() {
              // 将线程池的当前线程保存在成员变量中，以便后续使用
                thread = Thread.currentThread();
                if (interrupted) {
                    thread.interrupt();
                }
                boolean success = false;
                updateLastExecutionTime();
                try {
                  // 调用外部类SingleThreadEventExecutor的run方法，进入死循环，run方法
                    SingleThreadEventExecutor.this.run();
                    success = true;
                } catch (Throwable t) {
                    logger.warn("Unexpected exception from an event executor: ", t);
                } finally {
                  // ... 清理工作
                }
            }
        });
    }
```

- `io.netty.channel.nio.NioEventLoop#run`主要任务是执行死循环，不断看有没有新任务，有没有IO事件

```java
		protected void run() {
        for (;;) {
            try {
                try {
                  // calculateStrategy的逻辑如下
                  // 有任务，会执行一次selectNow,清除上一次的wakeup结果，无论有没有IO事件，都会跳过switch
                  // 没有任务，会匹配SelectStrategy.SELECT，看是否应当阻塞
                    switch (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
                    case SelectStrategy.CONTINUE:
                        continue;
                    case SelectStrategy.BUSY_WAIT:
                    case SelectStrategy.SELECT:
                        // 因为IO线程和提交任务线程都有可能执行wakeup,而wakeup属于比较昂贵的操作，因此使用了一个原子布尔对象wakenup, 它取值为true时，表示由当前线程唤醒
                        // 进行select阻塞，并设置唤醒状态为false
                        select(wakenUp.getAndSet(false));
                        if (wakenUp.get()) {
                          // 如果在这个位置，非EventLoop线程抢先将wakenup设置为true,并wakeup
                          // 下面的select方法不会阻塞
                          // 等runAllTask处理完成后，到再循环进来这个阶段新增的任务会不会及时执行呢？
                          // 因为oldWakenUp为true, 因此下面的select方法就会阻塞，直到超时
                          // 才能执行，让select方法无畏阻塞
                            selector.wakeup();
                        }
                    default:
                    }
                } catch (IOException e) {
                    rebuildSelector0();
                    handleLoopException(e);
                    continue;
                }
                cancelledKeys = 0;
                needsToSelectAgain = false;
                final int ioRatio = this.ioRatio;
                if (ioRatio == 100) {
                    try {
                        processSelectedKeys();
                    } finally {
                      // ioRatio为100时，总是运行万所有非IO任务
                        runAllTasks();
                    }
                } else {
                    final long ioStartTime = System.nanoTime();
                    try {
                        processSelectedKeys();
                    } finally {
                      // 记录io事件处理耗时
                        final long ioTime = System.nanoTime() - ioStartTime;
                      // 运行非IO任务，一旦超时会退出runAllTasks
                        runAllTasks(ioTime * (100 - ioRatio) / ioRatio);
                    }
                }
            } catch (Throwable t) {
                handleLoopException(t);
            }
            try {
                if (isShuttingDown()) {
                    closeAll();
                    if (confirmShutdown()) {
                        return;
                    }
                }
            } catch (Throwable t) {
                handleLoopException(t);
            }
        }
    }
```

> 这里有个费解的地方就是wakeup, 它既可以由提交任务的线程来调用，也可以由EventLoop线程来调用
>
> - 由非EventLoop线程调用，会唤醒当前执行select阻塞的EventLoop线程
> - 由EventLoop线程自己调用，本次的wakeup会取消下一次的select操作

- `io.netty.channel.nio.NioEventLoop#select`

```java
    private void select(boolean oldWakenUp) throws IOException {
        Selector selector = this.selector;
        try {
            int selectCnt = 0;
            long currentTimeNanos = System.nanoTime();
          // 计算等待时机 没有scheduleTask超时时间为1s, 有scheduleTask,超时时间为下一个定时任务执行时间 - 当前时间
            long selectDeadLineNanos = currentTimeNanos + delayNanos(currentTimeNanos);
            for (;;) {
                long timeoutMillis = (selectDeadLineNanos - currentTimeNanos + 500000L) / 1000000L;
              // 超时退出循环
                if (timeoutMillis <= 0) {
                    if (selectCnt == 0) {
                        selector.selectNow();
                        selectCnt = 1;
                    }
                    break;
                }
              // 如果期间又有task退出循环，如果没有这个判断，那么任务就会等到下次select超时时才能被执行
              // wakenUp.compareAndSet(false, true) 是让非NioEventLoop不必再执行wakeup
                if (hasTasks() && wakenUp.compareAndSet(false, true)) {
                    selector.selectNow();
                    selectCnt = 1;
                    break;
                }
              // select有限时阻塞
              // 注意nio有bug, 当bug出现时，select方法即使没有事件发生，也不会阻塞，导致不断空轮询，cpu占用100%
                int selectedKeys = selector.select(timeoutMillis);
              // 技术加1
                selectCnt ++;
              // 醒来后，如果有IO事件，或是由非EventLoop线程唤醒，或者有任务，退出循环
                if (selectedKeys != 0 || oldWakenUp || wakenUp.get() || hasTasks() || hasScheduledTasks()) {
                    break;
                }
                if (Thread.interrupted()) {
                  // 线程被打断，退出循环，记录日志
                    if (logger.isDebugEnabled()) {
                        logger.debug("Selector.select() returned prematurely because " +
                                "Thread.currentThread().interrupt() was called. Use " +
                                "NioEventLoop.shutdownGracefully() to shutdown the NioEventLoop.");
                    }
                    selectCnt = 1;
                    break;
                }
                long time = System.nanoTime();
                if (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) >= currentTimeNanos) {
                  // 如果超时，计数重置为1， 下次循环就会break
                    selectCnt = 1;
                // 计数超过阈值，由io.netty.selectorAutoRebuildThreshold指定，默认512
                // 这是为了解决nio空轮询bug
                } else if (SELECTOR_AUTO_REBUILD_THRESHOLD > 0 &&
                        selectCnt >= SELECTOR_AUTO_REBUILD_THRESHOLD) {
                  // 重建selector
                    selector = selectRebuildSelector(selectCnt);
                    selectCnt = 1;
                    break;
                }

                currentTimeNanos = time;
            }
            if (selectCnt > MIN_PREMATURE_SELECTOR_RETURNS) {
                if (logger.isDebugEnabled()) {
                    logger.debug("Selector.select() returned prematurely {} times in a row for Selector {}.",
                            selectCnt - 1, selector);
                }
            }
        } catch (CancelledKeyException e) {
            if (logger.isDebugEnabled()) {
                logger.debug(CancelledKeyException.class.getSimpleName() + " raised by a Selector {} - JDK bug?",
                        selector, e);
            }
        }
    }
```

- 处理keys `io.netty.channel.nio.NioEventLoop#processSelectedKeys`

```java
    private void processSelectedKeys() {
        if (selectedKeys != null) {
          // 通过反射将Selector实现类中的就绪事件集合替换为SelectedSelectionKeySet
          // SelectedSelectionKeySet底层为数组实现，可以提高遍历性能（原本为HashSet）
            processSelectedKeysOptimized();
        } else {
            processSelectedKeysPlain(selector.selectedKeys());
        }
    }
```

- `io.netty.channel.nio.NioEventLoop#processSelectedKey`

```java
		private void processSelectedKey(SelectionKey k, AbstractNioChannel ch) {
        final AbstractNioChannel.NioUnsafe unsafe = ch.unsafe();
      // 当key取消或关闭时会导致这个key无效
        if (!k.isValid()) {
            final EventLoop eventLoop;
            try {
                eventLoop = ch.eventLoop();
            } catch (Throwable ignored) {
                return;
            }
            if (eventLoop != this || eventLoop == null) {
                return;
            }
            unsafe.close(unsafe.voidPromise());
            return;
        }

        try {
            int readyOps = k.readyOps();
          // 连接事件
            if ((readyOps & SelectionKey.OP_CONNECT) != 0) {
                int ops = k.interestOps();
                ops &= ~SelectionKey.OP_CONNECT;
                k.interestOps(ops);
                unsafe.finishConnect();
            }
          // 可写事件
            if ((readyOps & SelectionKey.OP_WRITE) != 0) {
                ch.unsafe().forceFlush();
            }
          // 可读或可接入事件
            if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
              // 如果是可接入 io.netty.channel.nio.AbstractNioMessageChannel.NioMessageUnsafe#read
              // 如果是可读 io.netty.channel.nio.AbstractNioByteChannel.NioByteUnsafe#read
                unsafe.read();
            }
        } catch (CancelledKeyException ignored) {
            unsafe.close(unsafe.voidPromise());
        }
    }
```

