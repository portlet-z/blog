```java
// 1. netty 中使用NioEventLoopGroup(简称 nio boss线程)来封装线程和selector
Selector selector = Selector.open();
// 2. 创建NioServerSocketChannel，同时会初始化它关联的handler, 以及为原生ssc 存储config
NioServerSocketChannel attachment = new NioServerSocketChannel();
// 3. 创建NioServerSocketChannel时，创建了java原生的ServerSocketChannel
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
// 4. 启动nio boss线程执行接下来的操作
// 5. 注册（仅关联selector和NioServerSocketChannel）,未关注事件
SelectionKey selectionKey = serverSocketChannel.register(selector, 0, attachment);
// 6. head -> 初始化器 -> ServerBootstrapAcceptor -> tail, 初始化器是一次性的，只为添加acceptor
// 7. 绑定端口
serverSocketChannel.bind(new InetSocketAddress(8080));
// 8. 触发channel active事件，在head中关注op_accept事件
selectionKey.interestOps(SelectionKey.OP_ACCEPT);
```

## 启动流程

`Selector selector = Selector.open();`

### init & register regFuture处理

1. init (main)

   - 创建NioServerSocketChannel (main)

   `ServerSocketChannel ssc = ServerSocketChannel.open();`

   - 添加NioServerSocketChannel,初始化handler (main)
     - 初始化handler等待调用(nio-thread 调用)
     - 向nio ssc加入了acceptor handler(在accept事件发生后建立连接)

2. register (切换线程)

   - 启动nio boss 线程 (main)
   - 原生ssc注册至selector未关注事件 (nio-thread)

   `SelectionKey selectionKey = ssc.register(selector, 0, nettySsc);`

   - 执行NioServerSocketChannel初始化handler (main)

### regFuture的回调doBind0 (nio-thread)

- 原生ServerSocketChannel绑定 (nio-thread)

`ssc.bind(new InetSocketAddress(8080, backlog));`

- 触发NioServerSocketChannel active事件 (nio-thread)

`selectionKey.interestOps(SelectionKey.OP_ACCEPT);`

## 源码

- 入口`io.netty.bootstrap.Serverstrap#bind`
- 关键代码`io.netty.bootstrap.AbstractBootstrap#doBind`

```java
    private ChannelFuture doBind(final SocketAddress localAddress) {
      // 1. 执行初始化和注册regFuture会由initAndRegister设置其是否完成，从而回调3.2处代码
        final ChannelFuture regFuture = initAndRegister();
        final Channel channel = regFuture.channel();
        if (regFuture.cause() != null) {
            return regFuture;
        }
			// 2. 因为是initAndRegister异步执行，需要分两种情况来看，调试时也需要通过suspend端点类型加以区分
      // 2.1 如果已完成
        if (regFuture.isDone()) {
            ChannelPromise promise = channel.newPromise();
          // 3.1 立刻调用 doBind0
            doBind0(regFuture, channel, localAddress, promise);
            return promise;
      // 2.2 还没有完成
        } else {
            final PendingRegistrationPromise promise = new PendingRegistrationPromise(channel);
          // 3.2 回调 doBind0
            regFuture.addListener(new ChannelFutureListener() {
                @Override
                public void operationComplete(ChannelFuture future) throws Exception {
                    Throwable cause = future.cause();
                    if (cause != null) {
                      // 处理异常
                        promise.setFailure(cause);
                    } else {
                        promise.registered();
											// 3. 由注册线程去执行doBind0
                        doBind0(regFuture, channel, localAddress, promise);
                    }
                }
            });
            return promise;
        }
    }
```

- 关键代码`io.netty.bootstrap.AbstractBootstrap#initAndRegister`

```java
    final ChannelFuture initAndRegister() {
        Channel channel = null;
        try {
            channel = channelFactory.newChannel();
          // 1.1 初始化 - 做的事情就是添加一个初始化器 ChannelInitializer
            init(channel);
        } catch (Throwable t) {
            if (channel != null) {
                channel.unsafe().closeForcibly();
              // 处理异常
                return new DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
            }
            return new DefaultChannelPromise(new FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);
        }
      // 1.2 注册 - 做的事情就是将原生channel注册到selector上
        ChannelFuture regFuture = config().group().register(channel);
        if (regFuture.cause() != null) {
            if (channel.isRegistered()) {
                channel.close();
            } else {
                channel.unsafe().closeForcibly();
            }
        }
        return regFuture;
    }
```

- 关键代码`io.netty.bootstrap.ServerBootstrap#init`

```java
    @Override
// 这里channel实际上就是NioServerSocketChannel
    void init(Channel channel) throws Exception {
        final Map<ChannelOption<?>, Object> options = options0();
        synchronized (options) {
            setChannelOptions(channel, options, logger);
        }

        final Map<AttributeKey<?>, Object> attrs = attrs0();
        synchronized (attrs) {
            for (Entry<AttributeKey<?>, Object> e: attrs.entrySet()) {
                @SuppressWarnings("unchecked")
                AttributeKey<Object> key = (AttributeKey<Object>) e.getKey();
                channel.attr(key).set(e.getValue());
            }
        }

        ChannelPipeline p = channel.pipeline();

        final EventLoopGroup currentChildGroup = childGroup;
        final ChannelHandler currentChildHandler = childHandler;
        final Entry<ChannelOption<?>, Object>[] currentChildOptions;
        final Entry<AttributeKey<?>, Object>[] currentChildAttrs;
        synchronized (childOptions) {
            currentChildOptions = childOptions.entrySet().toArray(newOptionArray(0));
        }
        synchronized (childAttrs) {
            currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(0));
        }
			// 为NioServerSocketChannel添加初始化器
        p.addLast(new ChannelInitializer<Channel>() {
            @Override
            public void initChannel(final Channel ch) throws Exception {
                final ChannelPipeline pipeline = ch.pipeline();
                ChannelHandler handler = config.handler();
                if (handler != null) {
                    pipeline.addLast(handler);
                }
								// 初始化器的职责就是将ServerBootstrapAcceptor加入至NioServerSocketChannel上
                ch.eventLoop().execute(new Runnable() {
                    @Override
                    public void run() {
                        pipeline.addLast(new ServerBootstrapAcceptor(
                                ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
                    }
                });
            }
        });
    }
```

- 关键代码`io.netty.channel.AbstractChannel.AbstractUnsafe#register`

```java
        @Override
        public final void register(EventLoop eventLoop, final ChannelPromise promise) {
          // 一些检查
            if (eventLoop == null) {
                throw new NullPointerException("eventLoop");
            }
            if (isRegistered()) {
                promise.setFailure(new IllegalStateException("registered to an event loop already"));
                return;
            }
            if (!isCompatible(eventLoop)) {
                promise.setFailure(
                        new IllegalStateException("incompatible event loop type: " + eventLoop.getClass().getName()));
                return;
            }
            AbstractChannel.this.eventLoop = eventLoop;
            if (eventLoop.inEventLoop()) {
                register0(promise);
            } else {
                try {
                  /**
                   *  首次执行execute方法时，会启动nio线程，之后注册等操作在nio线程上执行
                   *  因为只有一个NioServerSocketChannel因此，也只有一个boss nio线程
                   *  这行代码完成的事情是 main -> nio boss 线程的切换
                   */
                    eventLoop.execute(new Runnable() {
                        @Override
                        public void run() {
                            register0(promise);
                        }
                    });
                } catch (Throwable t) {
                    logger.warn(
                            "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                            AbstractChannel.this, t);
                    closeForcibly();
                    closeFuture.setClosed();
                    safeSetFailure(promise, t);
                }
            }
        }
```

- 关键代码`io.netty.channel.AbstractChannel.AbstractUnsafe#register0`

```java
private void register0(ChannelPromise promise) {
  try {
    if (!promise.setUncancellable() || !ensureOpen(promise)) {
      return;
    }
    boolean firstRegistration = neverRegistered;
    // 1.2.1 原生的nio channel绑定到selector上，注意此时没有注册selector关注事件，附件为NioServerSocketChannel
    doRegister();
    neverRegistered = false;
    registered = true;
    // 1.2.2 执行NioServerSocketChannel初始化器的initChannel
    pipeline.invokeHandlerAddedIfNeeded();
    // 回调3.2 io.netty.bootstrap.AbstractBootstrap#doBind0
    safeSetSuccess(promise);
    pipeline.fireChannelRegistered();
    // 对应server socket channel还未绑定，isActive为false
    if (isActive()) {
      if (firstRegistration) {
        pipeline.fireChannelActive();
      } else if (config().isAutoRead()) {
        beginRead();
      }
    }
  } catch (Throwable t) {
    closeForcibly();
    closeFuture.setClosed();
    safeSetFailure(promise, t);
  }
}
```

- 关键代码`io.netty.channel.ChannelInitializer#initChannel`

```java
    private boolean initChannel(ChannelHandlerContext ctx) throws Exception {
        if (initMap.add(ctx)) { // Guard against re-entrance.
            try {
              // 1.2.2.1 执行初始化
                initChannel((C) ctx.channel());
            } catch (Throwable cause) {
                // Explicitly call exceptionCaught(...) as we removed the handler before calling initChannel(...).
                // We do so to prevent multiple calls to initChannel(...).
                exceptionCaught(ctx, cause);
            } finally {
              // 1.2.2.2 移除初始化器
                ChannelPipeline pipeline = ctx.pipeline();
                if (pipeline.context(this) != null) {
                    pipeline.remove(this);
                }
            }
            return true;
        }
        return false;
    }
```

- 关键代码`io.netty.channel.AbstractChannel#doBind0`

```java
// 3.1或3.2执行doBind0    
private static void doBind0(
            final ChannelFuture regFuture, final Channel channel,
            final SocketAddress localAddress, final ChannelPromise promise) {
        channel.eventLoop().execute(new Runnable() {
            @Override
            public void run() {
                if (regFuture.isSuccess()) {
                    channel.bind(localAddress, promise).addListener(ChannelFutureListener.CLOSE_ON_FAILURE);
                } else {
                    promise.setFailure(regFuture.cause());
                }
            }
        });
    }
```

- 关键代码`io.netty.channel.AbstractChannel.AbstractUnsafe#bind`

```java
        public final void bind(final SocketAddress localAddress, final ChannelPromise promise) {
            assertEventLoop();
            if (!promise.setUncancellable() || !ensureOpen(promise)) {
                return;
            }
            // See: https://github.com/netty/netty/issues/576
            if (Boolean.TRUE.equals(config().getOption(ChannelOption.SO_BROADCAST)) &&
                localAddress instanceof InetSocketAddress &&
                !((InetSocketAddress) localAddress).getAddress().isAnyLocalAddress() &&
                !PlatformDependent.isWindows() && !PlatformDependent.maybeSuperUser()) {
                logger.warn(
                        "A non-root user can't receive a broadcast packet if the socket " +
                        "is not bound to a wildcard address; binding to a non-wildcard " +
                        "address (" + localAddress + ") anyway as requested.");
            }
            boolean wasActive = isActive();
            try {
              // 3.3 执行端口绑定
                doBind(localAddress);
            } catch (Throwable t) {
                safeSetFailure(promise, t);
                closeIfClosed();
                return;
            }
            if (!wasActive && isActive()) {
                invokeLater(new Runnable() {
                    @Override
                    public void run() {
                      // 3.4 触发active事件
                        pipeline.fireChannelActive();
                    }
                });
            }
            safeSetSuccess(promise);
        }
```

- 3.3 关键代码`io.netty.channel.socket.nio.NioServerSocketChannel#doBind`

```java
    @Override
    protected void doBind(SocketAddress localAddress) throws Exception {
        if (PlatformDependent.javaVersion() >= 7) {
            javaChannel().bind(localAddress, config.getBacklog());
        } else {
            javaChannel().socket().bind(localAddress, config.getBacklog());
        }
    }
```

- 3.4 关键代码`io.netty.channel.DefaultChannelPipeline.HeadContext#channelActive`

```java
        @Override
        public void channelActive(ChannelHandlerContext ctx) {
            ctx.fireChannelActive();
						// 触发read (NioServerSocketChannel上的read 不是读取数据，只是为了触发channel的事件注册)
            readIfIsAutoRead();
        }
```

- 关键代码`io.netty.channel.nio.AbstractNioChannel#doBeginRead`

```java
    @Override
    protected void doBeginRead() throws Exception {
        // Channel.read() or ChannelHandlerContext.read() was called
        final SelectionKey selectionKey = this.selectionKey;
        if (!selectionKey.isValid()) {
            return;
        }
        readPending = true;
        final int interestOps = selectionKey.interestOps();
      // readInterestOps取值是16，在NioServerSocketChannel创建时初始化好，代表关注accept事件
        if ((interestOps & readInterestOp) == 0) {
            selectionKey.interestOps(interestOps | readInterestOp);
        }
    }
```

## accept流程

- selector.select()阻塞直到事件发生
- 遍历处理selectedKeys
- 拿到一个key, 判断事件类似是否为ACCEPT
- 创建SocketChannel,设置非阻塞并创建了NioSocketChannel
- 将SocketChannel注册至selector, sc.regsiter(selector, 0, NioSocketChannel),调用NioSocketChannel上的初始化器
- 关注selectionKey的read事件

### accept源码

- nio中代码如下，在netty中的流程

```java
// 1. 阻塞直到事件发生
selector.select();
Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
while(iterator.hasNext()) {
  // 2. 拿到一个事件
  SelectionKey key = iter.next();
  key.remove();
  // 3. 如果是accept事件
  if(key.isAcceptable()) {
    // 4. 执行accept
    SocketChannel channel = ssc.accept();
    channel.configureBlocking(false);
    // 5. 关注read事件
    SelectionKey scKey = sc.register(selector, 0, null);
    scKey.interestOps(SelectionKey.OP_READ);
  }
}
```

- 先来看可接入事件处理(accpet)`io.netty.channe.nio.AbstractNioMessageChannel.NioMessageUnsafe.read`

```java
				public void read() {
            assert eventLoop().inEventLoop();
            final ChannelConfig config = config();
            final ChannelPipeline pipeline = pipeline();
            final RecvByteBufAllocator.Handle allocHandle = unsafe().recvBufAllocHandle();
            allocHandle.reset(config);
            boolean closed = false;
            Throwable exception = null;
            try {
                try {
                    do {
                      //doRadMessages中执行了accept并创建NioSocketChannel作为消息放入readBuf
                      //readBuf是一个ArrayList用来缓存消息
                        int localRead = doReadMessages(readBuf);
                        if (localRead == 0) {
                            break;
                        }
                        if (localRead < 0) {
                            closed = true;
                            break;
                        }
                      // localRead为1， 就一条消息，即接收一个客户端连接
                        allocHandle.incMessagesRead(localRead);
                    } while (allocHandle.continueReading());
                } catch (Throwable t) {
                    exception = t;
                }
                int size = readBuf.size();
                for (int i = 0; i < size; i ++) {
                    readPending = false;
                  // 触发read事件，让pipeline上的handler处理，这是时是处理io.netty.bootstrap.ServerBootstrap.ServerBootstrapAcceptor#channelRead
                    pipeline.fireChannelRead(readBuf.get(i));
                }
                readBuf.clear();
                allocHandle.readComplete();
                pipeline.fireChannelReadComplete();
                if (exception != null) {
                    closed = closeOnReadError(exception);
                    pipeline.fireExceptionCaught(exception);
                }
                if (closed) {
                    inputShutdown = true;
                    if (isOpen()) {
                        close(voidPromise());
                    }
                }
            } finally {
                if (!readPending && !config.isAutoRead()) {
                    removeReadOp();
                }
            }
        }
```

- 关键代码`io.netty.bootstrap.ServerBootstrap.ServerBootstrapAcceptor#channelRead`

```java
        public void channelRead(ChannelHandlerContext ctx, Object msg) {
          // 这时的msg是NioSocketChannel
            final Channel child = (Channel) msg;
          // NioSocketChannel添加childHandler即初始化器
            child.pipeline().addLast(childHandler);
          // 设置选项
            setChannelOptions(child, childOptions, logger);
            for (Entry<AttributeKey<?>, Object> e: childAttrs) {
                child.attr((AttributeKey<Object>) e.getKey()).set(e.getValue());
            }
            try {
              // 注册NioSocketChannel到nio worker线程，接下来的处理也移交至nio worker线程
                childGroup.register(child).addListener(new ChannelFutureListener() {
                    @Override
                    public void operationComplete(ChannelFuture future) throws Exception {
                        if (!future.isSuccess()) {
                            forceClose(child, future.cause());
                        }
                    }
                });
            } catch (Throwable t) {
                forceClose(child, t);
            }
        }
```

- 又回到了熟悉的`io.netty.channel.AbstractChannel.AbstractUnsafe#register`方法

```java
				public final void register(EventLoop eventLoop, final ChannelPromise promise) {
            if (eventLoop == null) {
                throw new NullPointerException("eventLoop");
            }
            if (isRegistered()) {
                promise.setFailure(new IllegalStateException("registered to an event loop already"));
                return;
            }
            if (!isCompatible(eventLoop)) {
                promise.setFailure(
                        new IllegalStateException("incompatible event loop type: " + eventLoop.getClass().getName()));
                return;
            }
            AbstractChannel.this.eventLoop = eventLoop;
            if (eventLoop.inEventLoop()) {
                register0(promise);
            } else {
                try {
                  // 这行代码完成的事实是nio boss -> nio worker线程的切换
                    eventLoop.execute(new Runnable() {
                        @Override
                        public void run() {
                            register0(promise);
                        }
                    });
                } catch (Throwable t) {
                    logger.warn(
                            "Force-closing a channel whose registration task was not accepted by an event loop: {}",
                            AbstractChannel.this, t);
                    closeForcibly();
                    closeFuture.setClosed();
                    safeSetFailure(promise, t);
                }
            }
        }
```

`io.netty.channel.AbstractChannel.AbstractUnsafe#register0`

```java
				private void register0(ChannelPromise promise) {
            try {
                if (!promise.setUncancellable() || !ensureOpen(promise)) {
                    return;
                }
                boolean firstRegistration = neverRegistered;
                doRegister();
                neverRegistered = false;
                registered = true;
              // 执行初始化器，执行前pipeline中只有head -> 初始化器 -> tail
                pipeline.invokeHandlerAddedIfNeeded();
							// 执行后就是head -> logging handler -> my handler -> tail
                safeSetSuccess(promise);
                pipeline.fireChannelRegistered();
                if (isActive()) {
                    if (firstRegistration) {
                        pipeline.fireChannelActive();
                    } else if (config().isAutoRead()) {
                        beginRead();
                    }
                }
            } catch (Throwable t) {
                // Close the channel directly to avoid FD leak.
                closeForcibly();
                closeFuture.setClosed();
                safeSetFailure(promise, t);
            }
        }
```

- 回到了熟悉的代码`io.netty.channel.DefaultChannelPipeline.HeadContext#channelActive`

```java
public void channelActive(ChannelHandlerContext ctx) {
    ctx.fireChannelActive();
	// 触发 read (NioSocketChannel 这里 read，只是为了触发 channel 的事件注册，还未涉及数据读取)
    readIfIsAutoRead();
}
```

- `io.netty.channel.nio.AbstractNioChannel#doBeginRead`

```java
protected void doBeginRead() throws Exception {
    // Channel.read() or ChannelHandlerContext.read() was called
    final SelectionKey selectionKey = this.selectionKey;
    if (!selectionKey.isValid()) {
        return;
    }

    readPending = true;
	// 这时候 interestOps 是 0
    final int interestOps = selectionKey.interestOps();
    if ((interestOps & readInterestOp) == 0) {
        // 关注 read 事件
        selectionKey.interestOps(interestOps | readInterestOp);
    }
}
```

## read流程

- selector.select()阻塞直到事件发生
- 遍历处理selectedKeys
- 拿到一个key, 判断事件类型是否为read
- 读取操作

### read源码

再来看可读事件`io.netty.channel.nio.AbstractNioByteChannel.NioByteUnsafe#read`,注意发送的数据未必能够一次读完，因此会触发多次nio read事件，一件事件内会触发多次pipeline read, 一次事件会触发一次pipeline read complete

```java
        public final void read() {
            final ChannelConfig config = config();
            if (shouldBreakReadReady(config)) {
                clearReadPending();
                return;
            }
            final ChannelPipeline pipeline = pipeline();
          // io.netty.allocator.type决定allocator实现
            final ByteBufAllocator allocator = config.getAllocator();
          // 用来分配byteBuf,确定单次读取大小
            final RecvByteBufAllocator.Handle allocHandle = recvBufAllocHandle();
            allocHandle.reset(config);
            ByteBuf byteBuf = null;
            boolean close = false;
            try {
                do {
                    byteBuf = allocHandle.allocate(allocator);
                    allocHandle.lastBytesRead(doReadBytes(byteBuf));
                    if (allocHandle.lastBytesRead() <= 0) {
                        byteBuf.release();
                        byteBuf = null;
                        close = allocHandle.lastBytesRead() < 0;
                        if (close) {
                            readPending = false;
                        }
                        break;
                    }
                    allocHandle.incMessagesRead(1);
                    readPending = false;
                  // 触发read事件，让pipeline上的handler处理，这时是处理NioSocketChannel上的handler
                    pipeline.fireChannelRead(byteBuf);
                    byteBuf = null;
                  // 是否要继续循环
                } while (allocHandle.continueReading());
                allocHandle.readComplete();
              // 触发read complete事件
                pipeline.fireChannelReadComplete();
                if (close) {
                    closeOnRead(pipeline);
                }
            } catch (Throwable t) {
                handleReadException(pipeline, byteBuf, t, close, allocHandle);
            } finally {
                if (!readPending && !config.isAutoRead()) {
                    removeReadOp();
                }
            }
        }
```

- `io.netty.channel.DefaultMaxMessagesRecvByteBufAllocator.MaxMessageHandler#continueReading`

```java
        public boolean continueReading(UncheckedBooleanSupplier maybeMoreDataSupplier) {
            return config.isAutoRead() && // 一般为true
             			// respectMaybeMoreData默认为true
                  // maybeMoreDataSupplier的逻辑是如果预期读取字节与实际读取字节相等，返回true
                   (!respectMaybeMoreData || maybeMoreDataSupplier.get()) &&
                  // 小于最大次数,maxMessagePerRead默认16
                   totalMessages < maxMessagePerRead &&
                  // 实际读到了数据
                   totalBytesRead > 0;
        }
```

