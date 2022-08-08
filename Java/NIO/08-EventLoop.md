## EventLoop 事件循环对象

EventLoop本质是一个单线程执行器（同时维护了一个selector），里面有run方法处理Channel上源源不断的IO事件

它的继承关系比较复杂

- 一条是 继承自j.u.c.ScheduledExecutorService因此包含了线程池中所有方法
- 另一条是继承自netty自己的OrderedEventExecutor
  - 提供了boolean inEventLoop(Thread thread)方法判断一个线程是否属于此EventLoop
  - 提供了parent方法来看看自己属于哪个EventLoopGroup

## EventLoopGroup 事件循环组

EventLoopGroup是一组EventLoop,Channel一般会调用EventLoopGroup的register方法来绑定其中一个EventLoop,后续这个Channel上的IO事件都由此EventLoop来处理（保证了IO事件处理时的线程安全）

- 继承自netty自己的EventExecutorGroup
  - 实现了Iterable接口提供遍历EventLoop的能力
  - 另有next方法获取集合中下一个EventLoop

```java
@Slf4j
public class TestEventLoop {
    public static void main(String[] args) {
        // 1. 创建事件循环组
        // NioEventLoopGroup可以处理IO事件，普通任务，定时任务
        EventLoopGroup group = new NioEventLoopGroup(2);
        // DefaultEventLoopGroup可以处理普通任务，定时任务，不能处理IO事件
        //DefaultEventLoopGroup group = new DefaultEventLoopGroup(2);
        // 2. 获取下一个事件循环对象
        log.debug("event: {}", group.next());
        log.debug("event: {}", group.next());
        log.debug("event: {}", group.next());
        log.debug("event: {}", group.next());
        // 3. 执行普通任务
        group.next().execute(() -> {
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            log.debug("ok");
        });
        log.debug("main");
        // 4. 执行定时任务
        group.next().scheduleAtFixedRate(() -> log.debug("ok"), 0, 1, TimeUnit.SECONDS);
        log.debug("main");
    }
}
```

## EventLoopGroup工作细分

```java
@Slf4j
public class EventLoopServer {
    public static void main(String[] args) {
        // 细分2：创建一个独立的EventLoopGroup
        EventLoopGroup group = new DefaultEventLoopGroup();
        new ServerBootstrap()
                // 细分1：boss只负责ServerSocketChannel上accept事件，worker只负责socketChannel上的读写
                .group(new NioEventLoopGroup(), new NioEventLoopGroup(2))
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    @Override
                    protected void initChannel(NioSocketChannel ch) throws Exception {
                        ch.pipeline().addLast("handler1" , new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf = (ByteBuf) msg;
                                log.debug(buf.toString(Charset.defaultCharset()));
                                // 让消息传递给下一个handler
                                ctx.fireChannelRead(msg);
                            }
                        });
                        ch.pipeline().addLast(group, "handler2", new ChannelInboundHandlerAdapter() {
                            @Override
                            public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                                ByteBuf buf = (ByteBuf) msg;
                                log.debug(buf.toString(Charset.defaultCharset()));
                            }
                        });
                    }
                }).bind(8080);
    }
}
```

### handler执行中如何换人？

关键代码`io.netty.channel.AbstractChannelHandlerContext`

如果两个handler绑定的是同一个线程，那么就直接调用。否则，把要调用的代码封装为一个任务对象，由下一个handler的线程来调用

```java
static void invokeChannelRead(final AbstractChannelHandlerContext next, Object msg) {
  final Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, "msg"), next);
  // 下一个handler的事件循环是否与当前的事件循环是同一个线程
  EventExecutor executor = next.executor(); // 返回下一个handler的eventLoop
  // 是，直接调用
  if (executor.inEventLoop()) { // 当前handler中的线程，是否和eventLoop是同一个线程
    next.invokeChannelRead(m);
  } else { // 否，将要执行的代码作为任务提交给下一个事件循环处理（换人）
    executor.execute(new Runnable() {
      public void run() {
        next.invokeChannelRead(m);
      }
    });
  }

}
```

