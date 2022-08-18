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

